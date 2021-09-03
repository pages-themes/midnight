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

# --add horizontall image here

Now nothing really stuck out to me when I actually accessed the website. There was an input box but the submission button did nothing. I tried enumerating some directories with gobuster and those didn't give me any significant findings. So now I'm going to try to search for any subdomains using gobuster. I ran
`gobuster vhost -w /<path to wordlist> -u horizontall.htb` and I got the result
