# Recon
```
$ sudo nmap -p- --min-rate 10000 10.10.11.139                       
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-26 03:03 EEST
Warning: 10.10.11.139 giving up on port because retransmission cap hit (10).
Stats: 0:00:57 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 99.99% done; ETC: 03:04 (0:00:00 remaining)
Nmap scan report for 10.10.11.139
Host is up (1.1s latency).
Not shown: 55207 closed tcp ports (reset), 10326 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 59.03 seconds
```
- we try to open the webpage with port 5000 and try tcp scan with version scan
```
└─$ sudo nmap -p22,5000 -sCV 10.10.11.139
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-26 03:21 EEST
Nmap scan report for 10.10.11.139
Host is up (0.100s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea8421a3224a7df9b525517983a4f5f2 (RSA)
|   256 b8399ef488beaa01732d10fb447f8461 (ECDSA)
|_  256 2221e9f485908745161f733641ee3b32 (ED25519)
5000/tcp open  http    Node.js (Express middleware)
|_http-title: Blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.80 seconds

```
# Shell
- The “Login” button leads to /login, which is a login form:
- we launch burp and start observing the requests
- nmap identified the site is running NodeJS with Express. The response headers confirm that
- run feroxbuster against the site and no new
- Some basic SQL injections didn’t do anything, nor did a quick sqlmap run against the login form
-  we want Node to handle the input as a JSON object
-  in response header we has Content-Type: application/x-www-form-urlencoded we change it to application/json
-  we launch burp and start modifying the request
-  since its node its likely to run nosql so we try payloadsALlTheTHings repo for nosql
-  we send this payload **({"user": "admin", "password": {"$ne": "wrongpassword"}})** send it to the repeater and yes we got a cookie so we are logged in
-  we try to login by intercept in wrong pass request again by burp and put payload and change format to json forward the request and we are logged in
-  there is and upload button when i try to upload somethingits says **Invalid XML Example: Example DescriptionExample Markdown**
-  
-  
