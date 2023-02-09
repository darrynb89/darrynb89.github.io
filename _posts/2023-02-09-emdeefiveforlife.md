---
layout: post
current: post
cover: 'assets/images/emdeefiveforlife/cover.png'
navigation: True
title: "Emdee Five for Life Walk through"
date: 2023-02-09 00:00:00
tags: [hackthebox, ctf, easy]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/emdeefiveforlife/cover.png)

### Overview
[Emdee Five for Life](https://app.hackthebox.com/challenges/emdee-five-for-life) is a easy challenge from [Hack The Box](https://hackthebox.com). The challenge description is:

> Can you encrypt fast enough?

You can also find my video walk through [here](https://www.youtube.com/watch?v=i5JCGgSWnGA&t).

### Challenge  

Once you start the challenge and navigate to the IP & port you get the following page:

![md54life](/assets/images/emdeefiveforlife/md54life.png)

I copied the string and went to [CyberChef](https://gchq.github.io/CyberChef/). You can do this a few ways but I like CyberChef, its quick and easy. I pasted the string in to the input field and used the md5 operator to get a hash.

![cyberchef](/assets/images/emdeefiveforlife/cyberchef.png)

With the MD5 I paste that in to the web application and click submit but get the response "Too slow!". 

No matter how many times I try and do it, even with both browsers up, im not quick enough so this needs to be scripted.

### Final script

I created the following script which will go a GET request to the application to get the string. Then it will use hashlib to generate a md5 hash then finally POST the hash to the applicaton to get the flag.

{% highlight python %}

#!/usr/bin/python3

import requests
import re
import hashlib

url = "http://UPDATEME"     # Add challenge IP:PORT

session = requests.Session()

# step 1 - Get string

r = session.get(url)
html = r.text
match = re.search("[a-zA-Z0-9]{20}",html)

# Step 2 - Encrypt string with md5

string = match.group()
hash = hashlib.md5(string.encode("utf")).hexdigest()

# Step 3 - Post hash to web

p = session.post(url, data={"hash":hash})

print(p.text)

{% endhighlight %}

Thats the challenge, very simple but good for people new to scripting.

Thanks for reading!

==========================================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
