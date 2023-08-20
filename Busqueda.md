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
- port 80 says apache 
- we go to port 80 on broswer but no response we need to add seracher.htb to etc/hosts
- we run curl on searcher.htb
```
──(kali㉿kali)-[~/Desktop/HTB/Busqueda]
└─$ curl -v -s searcher.htb 1>/dev/null
*   Trying 10.10.11.208:80...
* Connected to searcher.htb (10.10.11.208) port 80 (#0)
> GET / HTTP/1.1
> Host: searcher.htb
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Sun, 20 Aug 2023 15:45:48 GMT
< Server: Werkzeug/2.1.2 Python/3.10.6
< Content-Type: text/html; charset=utf-8
< Content-Length: 13519
< Vary: Accept-Encoding
< 
{ [2472 bytes data]
* Connection #0 to host searcher.htb left intact
```
- we see a differnet server which is python                                                 
```
echo "10.10.11.208 searcher.htb" | sudo tee -a /etc/hosts
```
- appears to be a search engine aggregator that allows users to search for information on various search engines.
- Users can select a search engine, type a query, and get redirected automatically or get the URL of the search results
- After pressing the "Search" button, the website provides the URL for the specified search engine and the entered query
- now fire up burp and intercept the search button request , send to reapeter and save to file
- open file and put FUZZ after the value on query
- now luch ffuf
```
─$ ffuf -request search.req -request-proto http -w /usr/share/seclists/Fuzzing/special-chars.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://searcher.htb/search
 :: Wordlist         : FUZZ: /usr/share/seclists/Fuzzing/special-chars.txt
 :: Header           : Host: searcher.htb
 :: Header           : Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
 :: Header           : Accept-Encoding: gzip, deflate
 :: Header           : Referer: http://searcher.htb/
 :: Header           : Upgrade-Insecure-Requests: 1
 :: Header           : User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
 :: Header           : Accept-Language: en-US,en;q=0.5
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Header           : Origin: http://searcher.htb
 :: Header           : Connection: close
 :: Data             : engine=Google&query=islamFUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 37, Words: 1, Lines: 1, Duration: 1360ms]
    * FUZZ: &

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 1783ms]
    * FUZZ: ^

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2037ms]
    * FUZZ: #

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2313ms]
    * FUZZ: !

[Status: 200, Size: 38, Words: 1, Lines: 1, Duration: 2313ms]
    * FUZZ: -

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2322ms]
    * FUZZ: "

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2513ms]
    * FUZZ: *

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2513ms]
    * FUZZ: @

[Status: 200, Size: 38, Words: 1, Lines: 1, Duration: 2513ms]
    * FUZZ: ~

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2549ms]
    * FUZZ: %

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2481ms]
    * FUZZ: :

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2595ms]
    * FUZZ: $

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2527ms]
    * FUZZ: >

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2608ms]
    * FUZZ: /

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2609ms]
    * FUZZ: +

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 2676ms]
    * FUZZ: '

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2666ms]
    * FUZZ: <

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2689ms]
    * FUZZ: (

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2758ms]
    * FUZZ: )

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2711ms]
    * FUZZ: |

[Status: 200, Size: 38, Words: 1, Lines: 1, Duration: 2729ms]
    * FUZZ: _

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 2721ms]
    * FUZZ: \

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2721ms]
    * FUZZ: ?

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2729ms]
    * FUZZ: ]

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2740ms]
    * FUZZ: ,

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2740ms]
    * FUZZ: [

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2758ms]
    * FUZZ: ;

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2758ms]
    * FUZZ: }

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2762ms]
    * FUZZ: =

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2761ms]
    * FUZZ: {

[Status: 200, Size: 38, Words: 1, Lines: 1, Duration: 2762ms]
    * FUZZ: .

[Status: 200, Size: 40, Words: 1, Lines: 1, Duration: 2808ms]
    * FUZZ: `

:: Progress: [32/32] :: Job [1/1] :: 22 req/sec :: Duration: [0:00:02] :: Errors: 0 ::
```
- we see some intersting values which are 0 in size
- we match size of zero in ffuf
```
─$ ffuf -request search.req -request-proto http -w /usr/share/seclists/Fuzzing/special-chars.txt -ms 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://searcher.htb/search
 :: Wordlist         : FUZZ: /usr/share/seclists/Fuzzing/special-chars.txt
 :: Header           : Accept-Encoding: gzip, deflate
 :: Header           : Connection: close
 :: Header           : Accept-Language: en-US,en;q=0.5
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Header           : Origin: http://searcher.htb
 :: Header           : Referer: http://searcher.htb/
 :: Header           : Upgrade-Insecure-Requests: 1
 :: Header           : Host: searcher.htb
 :: Header           : User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
 :: Header           : Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
 :: Data             : engine=Google&query=islamFUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response size: 0
________________________________________________

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 2959ms]
    * FUZZ: \

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 3090ms]
    * FUZZ: '

:: Progress: [32/32] :: Job [1/1] :: 28 req/sec :: Duration: [0:00:03] :: Errors: 0 ::

```
- we see 2 values
- since its python we do ssti check we go to cobalt ssti and find how to check with this
```
${{<%[%'"}}%\.
```
- we add it to the query on repeater and send and we got nothing back we start deleting chachters and send until we see response after we delete the single quate we see a response
- this confirms that (') gives 0 size in response we try SQLI
- we add # and space after and before and send we still got no response
- since there is a python server we could try something
- in python we can assume the server uses print statment so it would be like this
```
print('input')
```
- so we can add ) and see what happens we send it and we got response
```
engine=Google&query=islam')+%23+
```
