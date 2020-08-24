---
layout : post
title: PoseidonCTF
description: Forensic writeups
date: 2020-08-10
image: /images/poseidon/logo.png
tags:
   - ctf
   - forensics
author: s0ur_bl00d
categories: Forensics
---
![logo](/images/poseidon/logo.png)
# DB64:

## Description;
![db64](/images/poseidon/forensics/db.png)


## Writeup;

As there in description, I have started checking firefoxhistory and chromehistory.
Checking chromehistory I got these;

![Screenshot from 2020-08-12 03-55-16](/images/poseidon/forensics/90007553-cbec2780-dc4f-11ea-816b-7294bcf8ba00.png)


So I've gone to that link and downloaded the zip which contained a .kbdx file.
The thing is .kbdx files are supported/opened by keepass [a password manager] which requires master key for sure.
Now I've started searching for a password in **envars,cmdscan,cmdline,consoles,files...**
Intially using **clipboard** plugin Ive got something like this;

![Screenshot from 2020-08-12 03-59-23](/images/poseidon/forensics/90007834-49179c80-dc50-11ea-83cd-4024951a7284.png)

The thing which is there in clipboard didn't worked as password so let's keep that a side and try to get password.
After sometime I went through description and there they mentioned about passwords.
Intially I have used **hashdump** gave me hashes but the hashes mentioned there were uncrackable.
So my teammate used **mimikatz** which will display passwords in ascii form.
The output is as follows;

![WhatsApp Image 2020-08-12 at 04 19 03](/images/poseidon/forensics/90009519-0c00d980-dc53-11ea-8f51-04e6f252edd5.jpeg)

Now this password also didn't work. Coming back to clipboard we suspected that we mighe have to use pastebin.
But in pastebin which message to open? And this password seemed to be part of the link using that we get a list of words like this.

![Screenshot from 2020-08-12 04-23-52](/images/poseidon/forensics/90012279-f215c580-dc57-11ea-8c1c-49343fd78678.png)

So this seems like a wordlist. Using this we managed to crack Database.kbdx through john.
After cracking database.kbdx and opening it we get to see 5 or 6 usernames and passwords again all of those passwords are links of pastebins. 
After checking all of them one with username "EY" gave us useful info. Using that as link we will get;

![Screenshot from 2020-08-12 04-49-13](/images/poseidon/forensics/90011892-3b194a00-dc57-11ea-8e0b-2759d52e672c.png)

This is base64 encoded image. We managed to get the image and it has the flag;
The flag is: **Poseidon{F1ref0x_P1ugIns_V0latI1Ity}**.

This is one of the best and cool challenges I've ever done.
Thanks to @iwd and @asis...
