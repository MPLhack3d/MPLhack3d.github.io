---
title: "Git Happens"
date: 2025-02-02
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Git Happens
categories: [TryHackMe, Easy]
tags: [linux, git]
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
    <td>Git Happens</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Boss wanted me to create a prototype, so here it is! We even used something called "version control" that made deploying this really easy!</td>
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
    <td><a href="https://tryhackme.com/r/room/githappens">https://tryhackme.com/r/room/githappens</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Git Happens.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.89.255                             
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-02 12:21 EST
  Nmap scan report for 10.10.89.255
  Host is up (0.038s latency).
  Not shown: 65534 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 24.64 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 80 10.10.89.255
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-02 12:22 EST
  Nmap scan report for 10.10.89.255
  Host is up (0.037s latency).

  PORT   STATE SERVICE VERSION
  80/tcp open  http    nginx 1.14.0 (Ubuntu)
  | http-git: 
  |   10.10.89.255:80/.git/
  |     Git repository found!
  |_    Repository description: Unnamed repository; edit this file 'description' to name the...
  |_http-title: Super Awesome Site!
  |_http-server-header: nginx/1.14.0 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 7.31 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

After discovering a hidden git directory in the nmap output, I used <a href="https://github.com/internetwache/GitTools/blob/master/Dumper/gitdumper.sh">GitDumpster</a> to extract the repository :

```bash
$ ./gitdumper.sh http://10.10.89.255/.git/ dump
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########

[*] Destination folder does not exist
[+] Creating dump/.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[-] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[+] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[+] Downloaded: objects/d0/b3578a628889f38c0affb1b75457146a4678e5
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/b8/6ab47bacf3550a5450b0eb324e36ce46ba73f1
[+] Downloaded: objects/77/aab78e2624ec9400f9ed3f43a6f0c942eeb82d
[+] Downloaded: objects/f1/4bcee8053e39eeb414053db4ec7b985f65edc8
[+] Downloaded: objects/9d/74a92581071ae7c4a470ff035e0de4598877e5
[+] Downloaded: objects/20/9515b2f7cbdfb731d275c4b089e41ba35c3bc8
[+] Downloaded: objects/5a/35c9b7c787c22f689d0364cf57b013a11561a2
[+] Downloaded: objects/08/906612dfe6821cebc21794eb85601fc4f54de9
[+] Downloaded: objects/4a/2aab268541cbcc434e0565b4f4f2deca29ee5f
[+] Downloaded: objects/7c/578d86a8713b67af2cb1b1d7c524c23cefe7aa
[+] Downloaded: objects/4e/7178fa5b68fec15e54f2b79ace6f9ce0169e01
[+] Downloaded: objects/2e/b93ac3534155069a8ef59cb25b9c1971d5d199
[+] Downloaded: objects/4c/f757268c6824041664d132a29908aa9c362a26
[+] Downloaded: objects/3a/39b02d3b9d12222bac4737ee67e31403d62f13
[+] Downloaded: objects/ae/f68b1e25df81a8c96ee4d57b20cc9f7a1ebee5
[+] Downloaded: objects/d6/df4000639981d032f628af2b4d03b8eff31213
[+] Downloaded: objects/56/820adbbd5ac0f66f61916122c94ea52937e9b2
[+] Downloaded: objects/d9/54a99b96ff11c37a558a5d93ce52d0f3702a7d
[+] Downloaded: objects/06/012255f074d7bc4acc6fadbcff004380b5f83b
[+] Downloaded: objects/bc/8054d9d95854d278359a432b6d97c27e24061d
[+] Downloaded: objects/dd/13038df878d41b774ce4fd4552091d46873c25
[+] Downloaded: objects/8c/94b154aef92380e29a3f16f1a889b56127cf13
[+] Downloaded: objects/e5/6eaa8e29b589976f33d76bc58a0c4dfb9315b1
[+] Downloaded: objects/48/926fdeb371c8ba174b1669d102e8c873afabf1
[+] Downloaded: objects/ce/b8d530ebcf79806dffc981905ec8c2e0d7a65b
[+] Downloaded: objects/87/bcbcb476578c6cc90ed39f9404292539fe1c9c
[+] Downloaded: objects/39/5e087334d613d5e423cdf8f7be27196a360459
[-] Downloaded: objects/40/04c23a71fd6ba9b03ec9cb7eed08471197d843
[-] Downloaded: objects/19/a865c5442a9d6a7c7cbea070f3cb6aa5106ef8
[-] Downloaded: objects/0f/679a88dbbaf89ff64cb351a151a5f29819a3c0
[+] Downloaded: objects/0e/abcfcd62467d64fb30b889e8de5886e028c3ed
[+] Downloaded: objects/ba/5e4a76e3f7b6c49850c41716f8f1091fbdc84e
[+] Downloaded: objects/2f/423697bf81fe5956684f66fb6fc6596a1903cc
[+] Downloaded: objects/e3/8d9df9b13e6499b749e36e064ec30f2fa45657
[+] Downloaded: objects/0e/0de07611ada4690fc0ea5b5c04721ba6f3fd0d
[+] Downloaded: objects/66/64f4e548df7591da3728d7662b6376debfce8d
```

Once the script completed, I used `git log` to inspect the commits.
```bash
$ git log

[...]

commit 395e087334d613d5e423cdf8f7be27196a360459
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:17:43 2020 +0200

    Made the login page, boss!

```

Because the description sounded interesting, I took a closer look with `git show`:

```bash
$ git show 395e087334d613d5e423cdf8f7be27196a36045
```

![Source Code of the Commit](/assets/img/tryhackme/GitHappens/thm_githappens_1.jpg)

solved! :)
