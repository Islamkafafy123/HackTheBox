# recon
- **nmap -p- --min-rate 10000 -oA scans/nmap-alltcp  10.10.11.133**
- found seven open TCP ports, SSH (22), Kubernetes (2379, 2380, and 8443), and InfluxDB (10249, 10250, and 10256)
- **nmap -p 22,2379,2380,8443,10249,10250,10256 -sCV -oA scans/nmap-tcpscripts 10.10.11.133**
- Based on this [Doc](https://kubernetes.io/docs/reference/networking/ports-and-protocols/) not only are 2379 and 2380 a part of Kubernetes, but the ports that nmap labeled as InfluxDB are as well.
- Based on some of the names, this seems like an instance of minikube
- **Kubernetes manages large deployments of containers.**
- TCP 8443 is the main API server for the cluster
- A kubelet is an agent running on each node that controls it, and typically listens on TCP 10250
 ![kub](https://github.com/Islamkafafy123/HackTheBox/blob/main/pictures/kub.jpeg)
