---
layout: post
current: post
cover: 'assets/images/githappens/cover.png'
navigation: True
title: Git-Happens Write Up
date: 2020-09-12 00:00:00
tags: [tryhackme, ctf,]
class: post-template
subclass: 'post'
author: darryn
---
![GitHappens](/assets/images/githappens/cover.png)

### Overview

Git-Happens is an easy room from [TryHackMe](https://tryhackme.com) with the description:

> Boss wanted me to create a prototype, so here it is! We even used something called "version control" that made deploying this really easy!

We only have 1 task, find the super secret password.

### Nmap

TryHackMe will assign a dynamic IP as part of the deployment, the IP I will be targeting is '10.10.39.144'. I started by doing an nmap scan to check what ports are open.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]
└──╼ $sudo nmap -sC -sV -oA nmap/initial 10.10.39.144
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-12 13:07 BST
Nmap scan report for 10.10.39.144
Host is up (0.025s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
| http-git: 
|   10.10.39.144:80/.git/
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Super Awesome Site!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.83 seconds
```

The only port is HTTP, I browsed to the machine and found what appears to be a login page but nothing happened when I tried to log in.

![WebPage](/assets/images/githappens/WebPage.png)

I continued with some basic web enumeration by running Gobuster but didnt find anything.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]
└──╼ $gobuster dir -u http://10.10.39.144 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.39.144
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/09/12 13:15:14 Starting gobuster
===============================================================
/css (Status: 301)
===============================================================
2020/09/12 13:17:12 Finished
===============================================================
```

However nmap did find /.git/ so I looked at that next. 

![GitPage](/assets/images/githappens/GitPage.png)

I tried to run git clone to download the files from the server but had no joy.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]
└──╼ $git clone http://10.10.39.144/
Cloning into '10.10.39.144'...
fatal: repository 'http://10.10.39.144/' not found
┌─[✗]─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]
└──╼ $git clone http://10.10.39.144/.git
Cloning into '10.10.39.144'...
fatal: repository 'http://10.10.39.144/.git/' not found
┌─[✗]─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]
└──╼ $git clone http://10.10.39.144/.git/
Cloning into '10.10.39.144'...
fatal: repository 'http://10.10.39.144/.git/' not found
```

I did find some tools to dump the git files. However, I used wget using the -r flag to download the files.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]                                                                                                                             
└──╼ $wget -r http://10.10.39.144/.git/                                                                                                                                       
--2020-09-12 13:23:22--  http://10.10.39.144/.git/                                                                                                                            
Connecting to 10.10.39.144:80... connected.                                                                                                                                   
HTTP request sent, awaiting response... 200 OK                                                                                                                                
Length: unspecified [text/html]                                                                                                                                               
Saving to: ‘10.10.39.144/.git/index.html’                                                                                                                                     
                                                                                                                                                                              
10.10.39.144/.git/index.html                     [ <=>                                                                                           ]   1.36K  --.-KB/s    in 0s 

2020-09-12 13:23:22 (146 MB/s) - ‘10.10.39.144/.git/index.html’ saved [1391]

Loading robots.txt; please ignore errors.
--2020-09-12 13:23:22--  http://10.10.39.144/robots.txt
Reusing existing connection to 10.10.39.144:80.
HTTP request sent, awaiting response... 404 Not Found
2020-09-12 13:23:22 ERROR 404: Not Found.

--2020-09-12 13:23:22--  http://10.10.39.144/ 
Reusing existing connection to 10.10.39.144:80.
HTTP request sent, awaiting response... 200 OK
Length: 6890 (6.7K) [text/html]
Saving to: ‘10.10.39.144/index.html’

10.10.39.144/index.html                      100%[==============================================================================================>]   6.73K  --.-KB/s    in 0s 

2020-09-12 13:23:22 (16.1 MB/s) - ‘10.10.39.144/index.html’ saved [6890/6890]

--2020-09-12 13:23:22--  http://10.10.39.144/.git/branches/
Reusing existing connection to 10.10.39.144:80.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘10.10.39.144/.git/branches/index.html’

<<SNIP>>
```

Now I had all the files, I took a look through but nothing jumped out as a super secret password so did some research on the git commands to see how I could use the files.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]
└──╼ $ls
10.10.39.144  nmap
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens]
└──╼ $cd 10.10.39.144/
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens/10.10.39.144]
└──╼ $ls -lah
total 8.0K
drwxr-xr-x 1 daz daz   34 Sep 12 13:23 .
drwxr-xr-x 1 daz daz   32 Sep 12 13:23 ..
drwxr-xr-x 1 daz daz   18 Sep 12 13:23 css
drwxr-xr-x 1 daz daz  158 Sep 12 13:23 .git
-rw-r--r-- 1 daz daz 6.8K Jul 23 23:39 index.html
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens/10.10.39.144]
└──╼ $ls -lah .git
total 24K
drwxr-xr-x 1 daz daz  158 Sep 12 13:23 .
drwxr-xr-x 1 daz daz   34 Sep 12 13:23 ..
drwxr-xr-x 1 daz daz   20 Sep 12 13:23 branches
-rw-r--r-- 1 daz daz  110 Jul 24 07:25 config
-rw-r--r-- 1 daz daz   73 Jul 23 23:39 description
-rw-r--r-- 1 daz daz   23 Jul 23 23:39 HEAD
drwxr-xr-x 1 daz daz  434 Sep 12 13:23 hooks
-rw-r--r-- 1 daz daz  645 Jul 23 23:39 index
-rw-r--r-- 1 daz daz 1.4K Sep 12 13:23 index.html
drwxr-xr-x 1 daz daz   34 Sep 12 13:23 info
drwxr-xr-x 1 daz daz   36 Sep 12 13:23 logs
drwxr-xr-x 1 daz daz  164 Sep 12 13:23 objects
-rw-r--r-- 1 daz daz  102 Jul 24 07:25 packed-refs
drwxr-xr-x 1 daz daz   52 Sep 12 13:23 refs
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens/10.10.39.144]
└──╼ $
```

Running git without any flags shows the help output and some of available commands. 

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens/10.10.39.144]                     
└──╼ $git                                                                          
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]                    
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]         
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]     
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]            
           <command> [<args>]                                                      
                                                                                   
These are common Git commands used in various situations:                          
                                                                                   
start a working area (see also: git help tutorial)
   clone             Clone a repository into a new directory
   init              Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add               Add file contents to the index
   mv                Move or rename a file, a directory, or a symlink
   restore           Restore working tree files
   rm                Remove files from the working tree and from the index
   sparse-checkout   Initialize and modify the sparse-checkout

examine the history and state (see also: git help revisions)
   bisect            Use binary search to find the commit that introduced a bug
   diff              Show changes between commits, commit and working tree, etc
   grep              Print lines matching a pattern
   log               Show commit logs
   show              Show various types of objects
   status            Show the working tree status

grow, mark and tweak your common history
   branch            List, create, or delete branches
   commit            Record changes to the repository
   merge             Join two or more development histories together
   rebase            Reapply commits on top of another base tip
   reset             Reset current HEAD to the specified state
   switch            Switch branches
   tag               Create, list, delete or verify a tag object signed with GPG

collaborate (see also: git help workflows)
   fetch             Download objects and refs from another repository
   pull              Fetch from and integrate with another repository or a local branch
   push              Update remote refs along with associated objects

'git help -a' and 'git help -g' list available subcommands and some
concept guides. See 'git help <command>' or 'git help <concept>'
to read about a specific subcommand or concept.
See 'git help git' for an overview of the system.
```

First I ran the 'log' command to check the commits.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens/10.10.39.144]             
└──╼ $git log                                                              
commit d0b3578a628889f38c0affb1b75457146a4678e5 (HEAD -> master, tag: v1.0)
Author: Adam Bertrand <hydragyrum@gmail.com>                               
Date:   Thu Jul 23 22:22:16 2020 +0000                                     
                                                                           
    Update .gitlab-ci.yml                                                  
                                                                           
commit 77aab78e2624ec9400f9ed3f43a6f0c942eeb82d                            
Author: Hydragyrum <hydragyrum@gmail.com>                                  
Date:   Fri Jul 24 00:21:25 2020 +0200                                     
                                                                           
    add gitlab-ci config to build docker file.                             
                                                                           
commit 2eb93ac3534155069a8ef59cb25b9c1971d5d199                            
Author: Hydragyrum <hydragyrum@gmail.com>                                  
Date:   Fri Jul 24 00:08:38 2020 +0200                                     
                                                                           
    setup dockerfile and setup defaults.                                   
                                                                           
commit d6df4000639981d032f628af2b4d03b8eff31213                            
Author: Hydragyrum <hydragyrum@gmail.com>                                  
Date:   Thu Jul 23 23:42:30 2020 +0200

    Make sure the css is standard-ish!

commit d954a99b96ff11c37a558a5d93ce52d0f3702a7d
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:41:12 2020 +0200

    re-obfuscating the code to be really secure!

commit bc8054d9d95854d278359a432b6d97c27e24061d
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:37:32 2020 +0200

    Security says obfuscation isn't enough.
    
    They want me to use something called 'SHA-512'

commit e56eaa8e29b589976f33d76bc58a0c4dfb9315b1
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:25:52 2020 +0200

    Obfuscated the source code.
    
    Hopefully security will be happy!

commit 395e087334d613d5e423cdf8f7be27196a360459
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:17:43 2020 +0200

    Made the login page, boss!

commit 2f423697bf81fe5956684f66fb6fc6596a1903cc
Author: Adam Bertrand <hydragyrum@gmail.com>
Date:   Mon Jul 20 20:46:28 2020 +0000

    Initial commit
```

Great, we can now see all the commits. Commits 2 and 3 look interesting, clearly the security team were not happy with the configuration, so lets take a look at the source code. I will use git diff to compare. We could also use git checkout.

```highlight
┌─[daz@parrot]─[~/Documents/TryHackMe/GitHappens/10.10.39.144]
└──╼ $git diff 395e087334d613d5e423cdf8f7be27196a360459
<<SNIP>>
     <script>
-      function login() {
-        let form = document.getElementById("login-form");
-        console.log(form.elements);
-        let username = form.elements["username"].value;
-        let password = form.elements["password"].value;
-        if (
-          username === "admin" &&
-          password === "{{REDACTED}}"
-        ) {
-          document.cookie = "login=1";
-          window.location.href = "/dashboard.html";
-        } else {
-          document.getElementById("error").innerHTML =
-            "INVALID USERNAME OR PASSWORD!"; 
-        }
-      }

<<SNIP>>
```
Found it! Ive redacted it from the output as part of the TryHackMe write up rules however the password is part of the login function.

Thanks for reading!

============================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

