# Synopsis
In the following walkthrough, we're gonna have a closer look at a chosen machine from HackTheBox. For those of you, who are not familiar with the name yet, HTB is an advanced online platform for grinding your penetration testing skills and sharpening infosec capabilities. Getting user and root on the box will require us to perform various SMB & Windows enumeration. After doing it right, it becomes relatively easy to find the flag, which is the ultimate goal of this challenge.

## Initial Scan
Let us kick things off with a preliminary scan
```
$ sudo nmap -sV -Pn -A -T4 -v -p- 10.10.10.178
```
> an unprivileged Nmap scan will use the TCP Connect scan (-sT), whereas a privileged one will use TCP Syn Stealth (-sS)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/Nmap-Nest.png)
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/Nmap-scan-result.png)

It reveals to us two interesting services running on this box:
* SMB (445/TCP)
* HQK Reporting Service V1.2 (4386/TCP)

Having figured out what are we dealing with in this case, we can start enumerating the SMB shares. Now it's just a matter of trying null session and digging deeper to find any reliable credentials.

## SMB Shares Enumeration
Port 445 is opened and literally waiting for our initial actions. Utilizing `smbclient` at this moment seems like a good idea. 
```
$ smbclient -L \\\\10.10.10.178
```

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-shares.png)
Both `Data` & `Users` shares look like they are accessible via null session (anonymously).

Before we make any moves, let's list all the available users

```
$ smbclient \\\\10.10.10.178\\Users
```
Enabling directory recursion \
`smb: \> recurse` \
And listing assets \
`smb: \> ls`
The result is an error NT_STATUS_ACCESS_DENIED
Next step will be giving a try to a different Share named `Users`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-users.png)

OK, above information might be helpful later. Now let's move one step further and see what does the `Data` directory hides.

```
$ smbclient \\\\10.10.10.178\\Data
``` 
`smb: \> recurse` \
`smb: \> ls`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-data.png)

Basically, what should immediately strike our attention here are two files. Their names are `Maintenance Alerts.txt` & `Welcome Email.txt` respectively. Let's retrieve them with `smbclient`. \
Firstly, we exit the smbclient `smb: \> exit` \
Then, we create a directory labeled Data as a destination for our files
```
$ mkdir Data
```
```
$ cd Data
```
```
$ smbclient \\\\10.10.10.178\\Data
```
`smb: \> recurse` \
`smb: \> prompt` \
We're gonna download all of the assets at once with the following command
`smb: \> mget *` \
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/mget-data-share.png)
After reading the content of .txt files, we obtain credentials for `TempUser`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/sublimetext-data.png)
Time to make use of our discovery
```
$ smbclient \\\\10.10.10.178\\Data -U TempUser
```
`smb: \> recurse` \
`smb: \> prompt` \
`smb: \> mget *`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-data-tempuser.png)

Careful analysis of the content results in gathering valuable information from .xml files \
Path to the 1st interesting file **Data/IT/Configs/RU Scanner/RU_Config.xml**
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/ru-config-xml.png)
The password obviously looks like as if it were encrypted. Unfortunately using it to authenticate to SMB shares doesn't work. Seems like the only way to use it, is to decrypt it in advance. 
Path to the 2nd interesting file **Data/IT/Configs/NotepadPlusPlus/config.xml**
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/IT-Carl-Temptxt.png)
Secure share looks tempting. Wonder whether we can access anything inside it?
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-secure.png)

We weren't allowed to list `IT`, though there was no problem with listing `IT\Carl\` directory. The quick decision is to recursively download its content. According to what we saw, in `WIP` directory there is some source code written in Visual Basic.

