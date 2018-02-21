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


# Exploitation


# Setup Damn Vulnerable Web Application

# Resources

1.
2.

