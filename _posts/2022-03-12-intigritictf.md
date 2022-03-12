---
layout: post
current: post
cover: 'assets/images/intigritictf/cover.gif'
navigation: True
title: "Intigriti 1337UP CTF 2022"
date: 2022-03-12 00:00:00
tags: [intigriti, ctf]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/intigritictf/cover.gif)

### Overview

These are a few write ups from the 1337UP CTF hosted by [Intigriti]](https://www.intigriti.com). It was a 24 hour CTF so didn't get as much time on it as I would have liked however it was still a lot of fun.

# Mirage - Misc

The link for the 'Mirage' challenge was https://mirage.ctf.intigriti.io/. There were no additional files with this challenge. Looking at the website was just a static page with no working links.

![mirage](/assets/images/intigritictf/mirage.png)

I looked at /robots.txt and found a whole list of entries.

![miragerobots](/assets/images/intigritictf/miragerobots.png)

There are a few entires that lead no where and a some are trolls, the important ones are: 

```highlight
Disallow: /wordlists.txt
Disallow: /ok.txt
```

I used ```wget https://mirage.ctf.intigriti.io/wordlists.txt``` to download the wordlist to my machine. On /ok.txt is some text which includes '/uncclzrny.wct' with a hint of using ROT.

![mirageok](/assets/images/intigritictf/mirageok.png)

I went over to [CyberChef](https://gchq.github.io/CyberChef/) and used rot13 on the text. This provided the url '/happymeal.jpg'. 

![cyberchefrot](/assets/images/intigritictf/cyberchefrot.png) 

This led to another page 'HelpMeOut.txt'.

![miragehelp](/assets/images/intigritictf/miragehelp.png) 

This page provides a link to a download. Again using wget I download the zip file.

![miragedownload](/assets/images/intigritictf/miragedownload.png) 

The zip is password protected.

```highlight
└──╼ $unzip flag.zip 
Archive:  flag.zip
[flag.zip] flag.txt password: 
   skipping: flag.txt                incorrect password
```

I used zip2john to create a hash.

```highlight
└──╼ $zip2john flag.zip > zip.john                                                                          
ver 1.0 efh 5455 efh 7875 flag.zip/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=68, decmplen=56, crc=CC303849
```

Then use john to crack the hash with the word list downloaded earlier.

```highlight
└──╼ $john --wordlist=wordlists.txt zip.john 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Soeasypeasy214   (flag.zip/flag.txt)
1g 0:00:00:00 DONE (2022-03-11 20:32) 100.0g/s 15500p/s 15500c/s 15500C/s ##this will help you later..violent
Use the "--show" option to display all of the cracked passwords reliably
Session completed
┌─[daz@parrotos]─[~/Documents/1337UPCTF/Mirage]
└──╼ $unzip flag.zip 
Archive:  flag.zip
[flag.zip] flag.txt password: 
 extracting: flag.txt                
┌─[daz@parrotos]─[~/Documents/1337UPCTF/Mirage]
└──╼ $cat flag.txt 
1337UP{Wh4tAM3ss.txt.jpg.whyareyouputtingmethroughthis}
```

# Traveler - Web

The link for the 'Traveler' challenge was https://traveller.ctf.intigriti.io/. There were no additional files with this challenge. Looking at the link its a travel agent based website. 

![traveler](/assets/images/intigritictf/traveler.png)

Poking around the website I found a section to check availability. 

![travelerinjection](/assets/images/intigritictf/travelerinjection.png)

I sent the request to Burp and started fuzzing the pack name field.

![travelerapi](/assets/images/intigritictf/travelerapi.png)

When submitting ```Single'```  an error was generated showing 

> An error occurred whilst executing: bash check.sh Couple'

![travelererror](/assets/images/intigritictf/travelererror.png)

It doesn't appear to be verifying user input so I should be able to just append bash commands to the syntax so I tried ```Single&ls``` because of the special character I URL encoded it.

![travelerls](/assets/images/intigritictf/travelerls.png)

It worked, so getting the flag was simple using the payload ```pack=Single%26%63%61%74%20%2e%2e%2f%2e%2e%2f%2e%2e%2f%66%6c%61%67%2e%74%78%74&submit=Submit``` which is ```&cat ../../../flag.txt``` URL encoded.

![travelerflag](/assets/images/intigritictf/travelerflag.png)

# Lovely Kitten Pictures 1 - Lovely Kitten Pictures

The link for the 'Lovely Kitten Pictures 1' challenge was https://lovelykittenpictures.ctf.intigriti.io/. No files were included as part of this challenge. Navigating to the website just showed a picture of a cat with an option to switch.

![cats](/assets/images/intigritictf/cats.png)

I sent the requests through Burp and saw a request to '/pictures.php?path=assets/1.jpg'. That looks it could be vulnerable to an LFI (Local File Inclusion).

![catsassets](/assets/images/intigritictf/catsassets.png)

I played around for a while trying the basics such as ../../../../flag.txt, ../../../../etc/passwd & etc but just got 404's so decided to try '/pictures.php?path=pictures.php' which if it worked would include the source code of the php file and I would be able to see how the request worked and it worked!

![catsflag](/assets/images/intigritictf/catsflag.png)

Reading the code it looks like any file other than .jpg would return the contents of flag1.txt I look at the headers and the flag was included1.

Thanks for reading!

==========================================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

