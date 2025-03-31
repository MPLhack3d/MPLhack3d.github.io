---
title: "Breaking RSA"
date: 2025-03-31
image: /assets/img/tryhackme/BreakingRSA/BreakingRSA_image.jpg
description: Writeup of the TryHackMe-CTF Breaking RSA
categories: [TryHackMe, Medium]
tags: [linux, rsa, python]
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
    <td>Breaking RSA</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Hop in and break poorly implemented RSA using Fermat's factorization algorithm.</td>
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
    <td><a href="https://tryhackme.com/room/breakrsa">https://tryhackme.com/room/breakrsa</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Breaking RSA.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/breakrsa$ nmap -p- 10.10.118.50           
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-31 05:15 EDT
  Nmap scan report for 10.10.118.50
  Host is up (0.051s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 36.07 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/breakrsa$ nmap -A -p 22,80 10.10.118.50
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-31 05:16 EDT
  Nmap scan report for 10.10.118.50
  Host is up (0.044s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 ff:8c:c9:bb:9c:6f:6e:12:92:c0:96:0f:b5:58:c6:f8 (RSA)
  |   256 67:ff:d4:09:ee:2c:8d:eb:94:b3:af:17:8e:dc:94:ae (ECDSA)
  |_  256 81:0e:b2:0e:f6:64:76:3c:c3:39:72:c1:29:59:c3:3c (ED25519)
  80/tcp open  http    nginx 1.18.0 (Ubuntu)
  |_http-title: Jack Of All Trades
  |_http-server-header: nginx/1.18.0 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.01 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The website displayed only the sentence `"A jack of all trades is a master of none but oftentimes better than a master of one"`. I proceeded with a Gobuster directory enumeration:
```bash
kali@kali:~/ctf/breakrsa$ gobuster dir --url http://10.10.118.50/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,php 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.118.50/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              txt,html,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /index.html           (Status: 200) [Size: 384]
  /development          (Status: 301) [Size: 178] [--> http://10.10.118.50/development/]
```

Gobuster identified a directory called `/development`, which contains the RSA prerequisites:

![Hidden Directory](/assets/img/tryhackme/BreakingRSA/thm_breakingrsa_1.jpg)

The `log.txt` contained a hint that this is a weakly implemented RSA algorithm and SSH root login was enabled:
```bash
kali@kali:~/ctf/breakrsa$ cat log.txt   
  The library we are using to generate SSH keys implements RSA poorly. The two
  randomly selected prime numbers (p and q) are very close to one another. Such
  bad keys can easily be broken with Fermat's factorization method.

  Also, SSH root login is enabled.

  <https://github.com/murtaza-u/zet/tree/main/20220808171808>

  ---
```

## RSA Public Key Analysis

I continued analyzing the RSA public key with `ssh-keygen`, which returned the length of the key:
```bash
kali@kali:~/ctf/breakrsa$ ssh-keygen -l -f id_rsa.pub    
  [removed] SHA256:DIqTDIhboydTh2QU6i58JP+5aDRnLBPT8GwVun1n0Co no comment (RSA)
```

To proceed, I needed the modulus N extracted from the public key. While researching how this can be done using Python, I found this <a href="https://stackoverflow.com/questions/42504079/how-do-you-extract-n-and-e-from-a-rsa-public-key-in-python">Stackoverflow</a> entry.
```python
from Crypto.PublicKey import RSA

# read the public key
public_key = RSA.importKey(open('id_rsa.pub', 'r').read())

# get N and E of the public key
n = public_key.n
e = public_key.e
```

Since I had the N value of the public key, I was able to use the factorization function provided in the challenge description:
```python
# function to factorize N (source TryHackMe):
def factorize(n):
    # since even nos. are always divisible by 2, one of the factors will
    # always be 2
    if (n & 1) == 0:
        return (n/2, 2)

    # isqrt returns the integer square root of n
    a = isqrt(n)

    # if n is a perfect square the factors will be ( sqrt(n), sqrt(n) )
    if a * a == n:
        return a, a

    while True:
        a = a + 1
        bsq = a * a - n
        b = isqrt(bsq)
        if b * b == bsq:
            break

    return a + b, a - b
```

Great! Using the function, I was able to factorize N and obtain p and q. With the values, I calculated Phi_n, which I needed for the decryption exponent d:
```python
# calculate phi n
phi_n = (p - 1) * (q - 1)

# calculate d
d = int(gmpy2.invert(e, phi_n))

print("phi_n:", phi_n)
print("d:", d)
```

To create the private key, I read the documentation of the `RSA.construct` module, which stated:
```text
Construct an RSA key from a tuple of valid RSA components.
rsa_components : tuple
A tuple of integers, with at least 2 and no more than 6 items. The items come in the following order:

        1. RSA modulus *n*.
        2. Public exponent *e*.
        3. Private exponent *d*.
           Only required if the key is private.
        4. First factor of *n* (*p*).
           Optional, but the other factor *q* must also be present.
        5. Second factor of *n* (*q*). Optional.
        6. CRT coefficient *q*, that 
```

After reading the documentation, I was able to generate the private key file:
```python
RSA_params = (n, e, d)
priv_key = RSA.construct(RSA_params)
f = open("id_rsa", "wb")
f.write(priv_key.export_key('PEM'))
f.close()
```

I changed the permission of the key file and successfully logged in as root via SSH. There, I retrieved the root flag:
```bash
kali@kali:~/ctf/breakrsa$ chmod 600 id_rsa
kali@kali:~/ctf/breakrsa$ ssh -i id_rsa root@10.10.247.92
root@thm:~# id
  uid=0(root) gid=0(root) groups=0(root)
root@thm:~# cat flag 
  breakingRSAissuperfun20220809134031
```

## Output of The Script
This is the output based on the private key provided within this room:

![Script Output](/assets/img/tryhackme/BreakingRSA/thm_breakingrsa_2.jpg)

## Complete Python Script
This is the complete Python script I used for this challenge:
```python
from Crypto.PublicKey import RSA
import gmpy2 
from gmpy2 import isqrt
from math import lcm

print("\033[91m+---------- PUBLIC KEY ANALYSIS ----------+\033[0m")
public_key = "id_rsa.pub"

# read the public key
public_key = RSA.importKey(open(public_key, 'r').read())

# get N and E of the public key
n = public_key.n
e = public_key.e

# output
print("\033[93mn:\033[0m", n)
print("\033[93me:\033[0m", e)

# function to factorize N (source TryHackMe):
def factorize(n):
    # since even nos. are always divisible by 2, one of the factors will
    # always be 2
    if (n & 1) == 0:
        return (n/2, 2)

    # isqrt returns the integer square root of n
    a = isqrt(n)

    # if n is a perfect square the factors will be ( sqrt(n), sqrt(n) )
    if a * a == n:
        return a, a

    while True:
        a = a + 1
        bsq = a * a - n
        b = isqrt(bsq)
        if b * b == bsq:
            break

    return a + b, a - b

# get the p and q values 
p, q = factorize(n)

# output
print("\033[93mp:\033[0m", p)
print("\033[93mq:\033[0m", q)

# difference of p and q
print("\033[93mdifference of p and q:\033[0m", p-q)

# calcualte the private key d
print("\033[91m+---------- PRIVATE KEY GENERATION ----------+\033[0m")

# calculate phi n
phi_n = (p - 1) * (q - 1)

# calculate d
d = int(gmpy2.invert(e, phi_n))

print("\033[93mphi_n:\033[0m", phi_n)
print("\033[93md:\033[0m", d)

RSA_params = (n, e, d)
priv_key = RSA.construct(RSA_params)

print("\033[92mkey generation completed...Private key will saved to current directory...\033[0m")

f = open("id_rsa", "wb")
f.write(priv_key.export_key('PEM'))
f.close()

print("\033[92m\ndone!\033[0m")
```

solved! :)