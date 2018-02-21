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

Example command for assessing googl
`theharvester -d telenordigital.com -l 500 -b all > telenor-digital.txt`

## nmap


# Finding vulnerabilities

# Exploitation

# Resources

## Vulnerable applications
1. Damn Vulnerable Web Application: http://www.dvwa.co.uk/
3. WebGoat: https://github.com/WebGoat/WebGoat
2. Mutillidae 2: https://www.owasp.org/index.php/OWASP_Mutillidae_2_Project
