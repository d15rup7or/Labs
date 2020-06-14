# Synopsis
Inclusiveness is a boot2root CTF you will find on Vulnhub. It was intended for beginning/intermediate practitioners of hacking skills. The machine utilizes some interesting services that will lead us from www-data to user to root. Enjoy going down the rabbit hole and chasing the vulnerabilities with support of the following walkthrough!

## 1. Enumeration
`nmap -sV -A -T4 -p- -v 10.0.2.0/24`

Nmap reveals to us three services running on this machine:
* FTP (21/TCP)
* SSH (22/TCP)
* HTTP (80/TCP)

Having the SSH version, there is not much more information to gather. 

## 2. 

## Summary
LFI
Simple python socket reverse connection payload
