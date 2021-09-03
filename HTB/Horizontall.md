---
title: Hack The Box&#58; Horizontall
permalink: /HTB/Horizontall
layout: default
---

Horizontall is an easy level box on HackTheBox and was quite the difficult box for me to solve. It felt way more difficult than all the other easy boxes I have done until this
point. I had to use techniques I haven't done yet, such as port forwarding, virtual host enumeration, and chaining CVE's.
# Walkthrough
I started off with a quick nmap scan on all ports `nmap -p- <ip>` just to see what is running and I got these results
```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-03 19:46 EDT
Nmap scan report for 10.10.11.105
Host is up (0.083s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.71 seconds
```
