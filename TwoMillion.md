# recon
- First i use **nmap -p- --min-rate=10000** 10.10.11.221 to check for open ports
- we got 2 ports open 22 and 80 we run nmap -sC -sV -p22,80 10.10.11.221
- after nmap we found port 80 has cookie  PHPSESSID: httponly flag not set
- we go check the site an no response only redirct to 2million.htb so we modify etc/hosts file with the ip to the site
- its a hackthebox site we run feroxbuster with command **feroxbuster -u http://2million.htb**
- we go check the login page i tryed admin@hackthebox.eu and password is password and we got usernotfound after several trys we also get user not found
- i try reflected xss from the url and its not working
- ![xss]()
