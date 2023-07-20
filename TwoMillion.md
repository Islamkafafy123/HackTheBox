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
  
