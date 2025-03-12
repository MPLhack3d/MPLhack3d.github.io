---
title: "W1seGuy"
date: 2025-03-11
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF W1seGuy
categories: [TryHackMe, Easy]
tags: [linux, xor]
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
    <td>W1seGuy</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>A w1se guy 0nce said, the answer is usually as plain as day.</td>
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
    <td><a href="https://tryhackme.com/room/w1seguy">https://tryhackme.com/room/w1seguy</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge ###.  

## Source Analysis 

The challenge provides the source code of an application which will provide the flags. I began by analyzing the source code. First of all, it is a Python application that runs a TCP socket server on port 1337. Before starting, it generates a key consisting of random letters of the length of 5:
```python
res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
key = str(res)
```

This key is used to encrypt the flag using a XOR encryption:
```python
for i in range(0,len(flag)):
    xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))
```

In the code section above lies the most important information for decrypting the flag later on. The XOR operation is performed using the character index `i` modulo the key lengt. This means the flag should be a multiple of the key lengt, which is 5. To verfiy this, I interacted with the server and reveived this example of an enrypted flag, which is indeed 80 characters long:
```python
>>> len("1e063a323f7b2f1b273b0f3603083b3e7a14222c0b20057a2e26020e211a383a0e793a3836383b32")
	80
```

## Decryption

Before diving deeper into the decryption, I reviewed the challenge description, which stated that the flag has a syntax like `THM{}`. This was very helpful in the decryption process. The flag is represented in hexadecimal, meaning that every two chars correspond to one character in the flag. Let's summarize what I know so far: the flag should look something like this:
```text
encrypted: 1e 06 3a 32 3f 7b 2f 1b 27 3b 0f 36 03 08 3b 3e 7a 14 22 2c 0b 20 05 7a 2e 26 02 0e 21 1a 38 3a 0e 79 3a 38 36 38 3b 32
decrypted: T  H  M  {  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  *  }
```

From this perspective, I can only can work with the first four and the last character. To begin, I converted the hex back to its ASCII representation:
```python
input   = "1e063a323f7b2f1b273b0f3603083b3e7a14222c0b20057a2e26020e211a383a0e793a3836383b32"
unhexed = ""

for a in range(0,len(input),2):
    unhexed += bytearray.fromhex(input[a:a+2]).decode()
```

Now I was able to use this to generate the key. First, I needed another variable containing the flag syntax as the input, and I performed an XOR operation with the encrypted value. This provided the key that was used for the initial encryption:
```python
for b in range(0,len(flag_format)):
    if b != 4:
        key += chr(ord(flag_format[b]) ^ ord(unhexed[b%key_length]))
    else:
        key += chr(ord(flag_format[b]) ^ ord(unhexed[-1]))
```

The key generation process is shown below:
```text
1e ^ T = J
06 ^ H = N
3a ^ M = W
32 ^ { = I
32 ^ } = O
```

The complete decryption script is shown below:
```python
input = "<< insert encrypted key >>"

key = ""
key_length = 5

flag_format = "THM{}"
flag_1 = ""

unhexed = ""

for a in range(0,len(input),2):
    unhexed += bytearray.fromhex(input[a:a+2]).decode()

for b in range(0,len(flag_format)):
    if b != 4:
        key += chr(ord(flag_format[b]) ^ ord(unhexed[b%key_length]))
    else:
        key += chr(ord(flag_format[b]) ^ ord(unhexed[-1]))


for c in range(0,len(unhexed)):
    flag_1 += chr(ord(unhexed[c]) ^ ord(key[c%key_length]))

print("   key:", key)
print("flag 1:", flag_1)
```

## Verify

Great! The challenge can be solved by interacting with the server using netcat. The server returns the encrypted flag, which needs to be pasted into the decryption script. Running the script will print both the key and flag 1. With the key, the flag 2 can be retrieved from the server:

![flag one](/assets/img/tryhackme/Wiseguy/thm_wiseguy_1.jpg)

![flag two](/assets/img/tryhackme/Wiseguy/thm_wiseguy_2.jpg)

solved! :)
