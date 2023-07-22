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
