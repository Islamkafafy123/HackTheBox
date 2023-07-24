# Recon
```
└─$ nmap -p- --min-rate 10000  10.10.10.161 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-24 20:39 EEST
Nmap scan report for 10.10.10.161
Host is up (0.15s latency).
Not shown: 65508 filtered tcp ports (no-response)
PORT     STATE  SERVICE
21/tcp   closed ftp
22/tcp   closed ssh
23/tcp   closed telnet
25/tcp   closed smtp
53/tcp   open   domain
80/tcp   closed http
110/tcp  closed pop3
111/tcp  closed rpcbind
113/tcp  closed ident
135/tcp  open   msrpc
139/tcp  open   netbios-ssn
143/tcp  closed imap
256/tcp  closed fw1-secureremote
443/tcp  closed https
445/tcp  open   microsoft-ds
554/tcp  closed rtsp
587/tcp  closed submission
993/tcp  closed imaps
995/tcp  closed pop3s
1025/tcp closed NFS-or-IIS
1720/tcp closed h323q931
1723/tcp closed pptp
3306/tcp closed mysql
3389/tcp closed ms-wbt-server
5900/tcp closed vnc
8080/tcp closed http-proxy
8888/tcp closed sun-answerbook

Nmap done: 1 IP address (1 host up) scanned in 20.23 seconds
```
- Alot of Ports like windows machine wiht no firwall
```
└─$ nmap -p 53,135,139,445 -sCV 10.10.10.161
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-24 20:36 EEST
Stats: 0:00:07 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 20:36 (0:00:06 remaining)
Nmap scan report for 10.10.10.161
Host is up (0.12s latency).

PORT    STATE SERVICE      VERSION
53/tcp  open  domain       Simple DNS Plus
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2023-07-24T10:43:40-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2023-07-24T17:43:41
|_  start_date: 2023-07-23T23:49:38
|_clock-skew: mean: 2h26m50s, deviation: 4h02m31s, median: 6m48s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.66 seconds
```
