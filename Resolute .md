# Recon
```
─$ nmap -sC -sV 10.10.10.169
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-08 16:25 EEST
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 15.28% done; ETC: 16:25 (0:00:22 remaining)
Stats: 0:00:34 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 88.08% done; ETC: 16:26 (0:00:05 remaining)
Nmap scan report for 10.10.10.169
Host is up (0.30s latency).
Not shown: 964 closed tcp ports (conn-refused)
PORT      STATE    SERVICE       VERSION
3/tcp     filtered compressnet
53/tcp    open     domain        Simple DNS Plus
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2023-08-08 12:33:33Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
211/tcp   filtered 914c-g
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds  Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open     tcpwrapped
800/tcp   filtered mdbs_daemon
1043/tcp  filtered boinc
1334/tcp  filtered writesrv
1503/tcp  filtered imtc-mcs
1935/tcp  filtered rtmp
2717/tcp  filtered pn-requester
2967/tcp  filtered symantec-av
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
3324/tcp  filtered active-net
3851/tcp  filtered spectraport
3920/tcp  filtered exasoftport1
5120/tcp  filtered barracuda-bbs
5414/tcp  filtered statusd
5678/tcp  filtered rrac
6346/tcp  filtered gnutella
8200/tcp  filtered trivnet1
9001/tcp  filtered tor-orport
9090/tcp  filtered zeus-admin
9593/tcp  filtered cba8
9917/tcp  filtered unknown
10024/tcp filtered unknown
49165/tcp filtered unknown
49175/tcp filtered unknown
50300/tcp filtered unknown
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
|_clock-skew: mean: 1h27m06s, deviation: 4h02m30s, median: -52m54s
| smb2-time: 
|   date: 2023-08-08T12:33:42
|_  start_date: 2023-08-08T12:23:27
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Resolute
|   NetBIOS computer name: RESOLUTE\x00
|   Domain name: megabank.local
|   Forest name: megabank.local
|   FQDN: Resolute.megabank.local
|_  System time: 2023-08-08T05:33:44-07:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 79.30 seconds
```
- since the machine is still booting up i may run the nmap cammand later
- we found smb 445 so smbclinet and smbmap no luck with them
- we try the rpcclient but no luck but i try with null authenticayion and it works
```
rpcclient -U "" -N 10.10.10.169
```
- try to enumurate users by (enumdomusers and queryusers)
- copy all user and put itin text file and filter using
```
cat users.txt | awk -F\[ '{print $2}'
```
- get less information about all users with querydispinfo
