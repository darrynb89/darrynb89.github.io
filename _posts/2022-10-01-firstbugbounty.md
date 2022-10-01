---
layout: post
current: post
cover: 'assets/images/firstbugbounty/cover.png'
navigation: True
title: "My first accepted bug bounty"
date: 2022-10-01 00:00:00
tags: [bugbounty, synack]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/firstbugbounty/cover.png)

### Overview
In this short blog post, I am going to explain how I found my first accepted bug, what the vulnerability was and my advice to people starting their journey in to the world of bug hunting.

### Background
To be clear I am not an experienced bug hunter. For the past few years I have been studying and learning as much as I can about ethical hacking to obtain a role as a full time penetration tester which I'm pleased to say I have achieved starting October 2022. I completed my OSCP in 2020, CRTP in 2021 and will hopefully take the Burp Suite exam and TCM's PNPT exam this year (2022). 

My experience in bug bounty is very limited, I successfully became a member of the Synack Red Team in May 2022 and have slowly been spending more and more time on the platform to find vulnerabilities. I have to say [Synack](https://www.synack.com) have been amazing, its a great platform to work on with a fantastic community of people. 

### The bug!
The bug I found was a actually very simple, a JWT authentication bypass via unverified signature. The first thing I do when looking at a target is just navigate around the application and click all the buttons, links etc with my traffic being proxies through Burp. Then once I have an understanding about the target and the functionally I review my Burp history and see if I can spot any interesting requests or parameters. 

For this bug I noticed after I logged in a JWT token is sent via a POST request to the target. I decoded the JWT and replaced my username with another username. I then re-encoded the token and forwarded the request. To my surprise I was now logged in as the replacement username essentially allowing me to login in as anyone with a registered username on the platform.

At this point the adrenaline kicked in, I was so excited, my first legitimate vulnerability! It was about 11:30pm, I had work at 9am the next day but I wanted to be sure of my finding. I must have tested the bypass about 10 times with 4 different user accounts to be sure. I reported my findings around 1am and went to bed. 

After what felt like a lifetime, I finally I got confirmation from Synack my vulnerability was accepted! 

A great resource for learning about JWT attacks is on [Port Swigger Academy](https://portswigger.net/web-security/jwt).

### Recommendations
I am just starting out on this bug bounty journey but I wanted to share the recommendations and resources I have found useful. Before hunting for your first bugs, study! Learn the common vulnerabilities, understand the [OWASP top 10](https://owasp.org/www-project-top-ten/). Complete CTF challenges to get the hands on experience of using tools like Burp and practice finding and exploiting vulnerabilities.

There is no quick fix or miracle methodology. You simple need to put in the hard work, the time in front of the computer and practice and preserver. I have seen lots of people ask, "how do I find my first bug" or say "I cant find any bugs". I have been there, for months I couldn't find anything but don't give up, keep trying and eventually you will find one.... YOU GOT THIS!!

> YOU GOT THIS!

My list of recommended learning material is:

- [Port Swigger Academy](https://portswigger.net/web-security)
- [Nahamsec Bug Bounty Course](https://www.udemy.com/course/intro-to-bug-bounty-by-nahamsec/)
- [Jason Haddix The Bug Hunter's Methodology (TBHM)](https://github.com/jhaddix/tbhm)
- [PhD Security](https://www.youtube.com/channel/UCAndnmvdiphDqLLDrGnBuhA/about)
- [John Hammond](https://www.youtube.com/c/JohnHammond010)
- [Bug Bounty Bootcamp](https://www.amazon.co.uk/Bug-Bounty-Bootcamp-Reporting-Vulnerabilities/dp/1718501544)
- [Tryhackme](https://tryhackme.com)
- [Hackthebox](https://www.hackthebox.com)
- [TCM Security Academy](https://academy.tcm-sec.com)

### Next steps...
Find more bugs..... :) I have now been bitten by the bug (pun intended) and want to find more. There is still a lot more to learn and bugs to find and looking forward to see what the journey holds.

I hope to release more blog posts soon with further bug report write ups and tips and tricks I learn along the way.

Thanks for reading!

==========================================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
