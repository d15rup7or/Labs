# Synopsis
In the following walkthrough, we're gonna have a closer look at a certain machine from HackTheBox. For those of you, who are not familiar with the name, HTB is an advanced, online platform for grinding your penetration testing skills and sharpening infosec capabilities. Getting user and root on the box will require us to perform various SMB & Windows enumeration. After doing it right, it becomes relatively easy to find the flag which is the ultimate goal of this challenge.

## Initial Scan
Let us kick things of with a preliminary scan \
`sudo nmap -sV -Pn -A -T4 -v -p- 10.10.10.178`
> an unprivileged Nmap scan will use the TCP Connect scan (-sT), whereas a privileged one will use TCP Syn Stealth (-sS)

It reveals to us two interesting services running on this box:
* SMB (445/TCP)
* Unknown (4386/TCP)

Having figured out what are we dealing with in this case, we can start enumerating the SMB shares. Now it's just a matter of trying null session and digging deeper to find any reliable credentials.

## SMB Shares Enumeration
Port no. 445 is opened and literally waiting for our initial actions. Utilizing `smbclient` seems like a good idea. \
`smbclient -L \\\\10.10.10.178`
The `Data` share looks like it is accessible via null session (anonymously).
