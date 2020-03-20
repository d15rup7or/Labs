# Mr. Robot
Long story short, the following is the output of playing with the machine entitled Mr. Robot

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mr%20Robot/img/MrRobot-login.png)

## 1. Initial Scan

Let's begin with utilizing the network mapper to audit and discover open ports<br>
`-T4` switch will provide us with faster execution and `-A` will perform OS and service detection

`nmap -sV -sT -p- -v -A -T4 192.168.56.1/24`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mr%20Robot/img/nmap-output.png)

I'm dying to see what awaits us there

`firefox 192.168.56.109`

We strike on a web server with an uncanny webpage, thematically based on the famous TV series

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mr%20Robot/img/192.168.56.109.png)

As always, firstly we check the `robots.txt` to see the instructions for web robots (to put it simply it tells the web crawlers i.e. Googlebot, which website resources can be visited and which can be indexed)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mr%20Robot/img/robots.txt.png)

Actually, both of the files look very useful! Judging by the name, one of them may possibly contain what we are looking for - the first flag. And the second one is a dictionary file.

Apparently, we were right
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mr%20Robot/img/key-1-of-3.txt.png)

How about further enumeration? ;-) <br>
`nikto -h 192.168.56.109`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Mr%20Robot/img/nikto-host-scan.png)

Running nikto scan reveals that the website hosted at this IP is powered by Wordpress. Additionally there is an entry referring to OSVDB-3092. Googling it leads us to:
>Vulnerability allows attackers to write anywhere in the file system.

From now on, we're gonna stick to the previously detected Wordpress content, as it always is a good idea not to neglect it :-)

`wpscan --url 192.168.56.109 --enumerate vp`
