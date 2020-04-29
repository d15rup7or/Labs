# Synopsis
Today we'll be hacking Mongo from HackTheBox. This medium-difficulty Linux machine with Apache is a perfect match for grinding enumeration and scripting skills. The main difficulty lies in exfiltration of user credentials and utilizing them for further privilege escalation.

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mango/img/Mango-HackTheBox.png)

# 1. Initial scan
`nmap -sV -sC 10.10.10.162`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mango/img/initial-scan.png)

Nmap reveals some information about the target allowing us to see open services and ports on the given host.
* OpenSSH 7.6p1 running on 22/tcp
* Apache https 2.4.29 running on 80/tcp
* Apache httpd 2.4.29 running on 443/tcp

# 2. Enumeration
## Apache
Checking the port 80 returns a 403 Error but if we browse to 443 (HTTPS) then we see a search engine.
