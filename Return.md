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
- we dont find much we go to the website
- Navigating to Settings reveals a username and domain name
