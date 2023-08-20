# Recom
```
─$ nmap -sC -sV 10.10.11.208     
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-20 18:41 EEST
Nmap scan report for 10.10.11.208
Host is up (0.22s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4fe3a667a227f9118dc30ed773a02c28 (ECDSA)
|_  256 816e78766b8aea7d1babd436b7f8ecc4 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://searcher.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.38 seconds

```
- 2 Ports Are open
- we go to port 80 on broswer but no response we need to add seracher.htb to etc/hosts
```
echo "10.10.11.208 searcher.htb" | sudo tee -a /etc/hosts
```
- appears to be a search engine aggregator that allows users to search for information on various search engines.
- Users can select a search engine, type a query, and get redirected automatically or get the URL of the search results
- After pressing the "Search" button, the website provides the URL for the specified search engine and the entered query
- now fire up burp 
