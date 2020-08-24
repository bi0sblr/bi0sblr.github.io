---
layout : post
title: Inctf 2020
description: 
date: 2020-08-15
image:
tags: Forensics
author: s0ur_bl00d
categories: Forensics
---

# Investigation

## Chall Description

![description](/images/inctfi/1.png)

## Writeup

As we were given a windows memory dump file I've started to analyze it with volatility.
So 1st we go with finding image profile using imageinfo command in volatility.
But this takes a little while...

![Screenshot from 2020-08-06 06-45-55](/images/inctfi/2.png)

Now knowing profile we can proceed to find what we need.

The main objectives are:
- To find when is the last time that caluclator is opened.
- To find how many times Chrome was used.

### Part-1

To find when an app is last opened, there are multiples way to proceed with. Those are,
- Prefetch files
  Windows store prefetch files and these prefetch files give details about every file that is opened and details like when its last modified, what version it is, and all details.
  All we have to do is get that prefetch file which is related to calc.exe and open it using **IEF** tool or **CROWDSTRIKE CROWDRESPONSE** tool.
  But issue with this method is you should be able to extract and open .pf files properly. Unfortunately the timestamp mentioned in this process is incorrect.
  Prefetch files can be found under the `C:\Windows\Prefetch` folder.
- Super Prefetch files
  Also in the Prefetch directory can be some files which are related to Superfetch. One of these files is AgAppLaunch.db which contains information about the       executed files. I found this file so I used **CrowdStrike’s CrowdResponse** tool to parse it. I could successfully parse the file and get a file for Adobe Reader execution but unfortunately, this value is also incorrect.
- MFT Parser;
  I've used mftparser and copied the output to a text file. I've tried mactime plugin also which gave me 4 outputs i.e, 4 time stamps. But all ofthese are incorrect again.
-  User assist;
  Similar to prefetch there will be a folder called userassist which has crucial data like time stamps and run count and version etc... details. I got to know there is a plugin for getting info about userassist directory.

Command will be;    `volatility -f windows.vmem --profile=Win7SP1x64 userassist`

From the outputs searching for calc.exe, the result is as follows.
![calc exe-userassist](/images/inctfi/3.png)
We can see the timestamp of when calculator is last updated: `21-07-2020_18:21:35`

### Part-2

I have already mentioned that we can get run count using userassist plugin. So from results,I searched for Chrome and the output is as follows;
 

![chrome-userassist](/images/inctfi/4.png)
  
The count is 19 as mentioned.

Wrapping the time stamp and count in flag format gives; `inctf{21-07-2020_18:21:35_19}`

While doing this challenge during ctf, I have started with part2 so I am so sure that count is 19 and was able to identify which timestamp is correct and which is not.

Thanks to stuxn3t for such intresting chall.


# Investigation Continues:

## Description

![inv cont description](/images/inctfi/5.png)


## Writeup

Current challenge is the continution of Investigation Challenge. So skipping some basic steps and directly moving to the main part.
The objectives of this challenge as per description suggests are;
- We are asked to find when is the last time Adam entered incorrect password.
- We have to find when is the last time the file 1.jpg opened.
- We have to find when is the last time when chrome is launched through task bar.

I'm writing this writeup in the order I got the flags.

### Part-3
Last time when chrome is launched through taskbar.

To find that I have used userassist to check if there is any info related to taskbar/..../google chrome or similar.

And my guess was right. I got the timestamp of it and it is as follows.
  
[taskbar-chrome](/images/inctfi/6.png)

So from the pic we see the timestamp of last opened through taskbar is `2020-07-21 17:37:18`

### Part-2

Generally, 1.jpg is a file. So I thought it can be in filescan. But it's not there. So We see that it might be deleted or Inserted into some other file.

Now I have only option left and that is to check mftparser. Then I made a text file and copying output of mftparser.
For minimizing my burden, I have tried `strings` command with searching for 1.jpg as follows;
  
  ![Screenshot from 2020-08-06 21-17-08](/images/inctfi/7.png)


So we can be sure that we found 1.jpg somewhere. Mind that the output file which has contents of mftparser is 16MB.

Make sure that you have good ram or good text opener with many features and requires less ram.
Searching for 1.jpg, we can find it is not exactly the file. But it is stored in 1.lnk. I was sure that this is the correct time stamp because I didnot find any other 1.jpgs.

![WhatsApp Image 2020-08-02 at 05 51 48](/images/inctfi/8.jpeg)

From the snap, we see that timestamp is `2020-07-21 18:38:33`
  
### Part-1

This is the tough part of the chall. And I had no idea how to deal with this. So after so much googilng and learning new things I got to know that we can use event logs to get those timestamps.

But evtlogs plugin only works for windowsXP and windows2003 profiles only :(

Again after googling and reading various volatility plugins got to know that we have to do registry analysis.

There are 2 ways which are extracting ntuser.dat or SAM registries and we have to open it using some registry exlorer or can be decoded from hex which is similar to decoding hex in userassist.

I tried extracting SAM registry for knowing where exactly it is we can use **Hivelist** plugin. From the output obtained we can see both as follows;

![Screenshot from 2020-08-06 22-49-05](/images/inctfi/9.png)

Now we have to extract any of those registries with **Dumpregistry** plugin. I have extracted SAM registry.

Then opened it with Registry explorer and inspecting on users carefully gives us the timestamp **2020-07-22 09:05:11**.

By the time I extracted this registry the ctf has ended and it really hurts when we miss something big when we lack time of 5 minutes. :(
  
Wrapping the flags we got as mentioned gives us;

  `Flag1; 22-07-2020_09:05:11`

  `Flag2; 21-07-2020_18:38:33`
  
  `Flag3; 21-07-2020_17:37:18`
  
  Joining these 3 flags, we get `inctf{22-07-2020_09:05:11_21-07-2020_18:38:33_21-07-2020_17:37:18}`.
  
But I enjoyed the challenge and learnt many things about analyzing registries and related info. Thanks to stuxn3t.

# LOGarithm:

## Description

![description](/images/inctfi/10.png)

## Writeup;

We're given a .vmem and a pcap file.
Let's start with .vmem file using volatility and we can find profile using **imageinfo** plugin in volatility.


![imageinfo](/images/inctfi/11.png)

We can use any of the profiles listed above in the pic.
Now let us observe what processes are running to know what is suspicious in memory dump.
We can do this using **pstree** plugin and I'm focusing on main observation we are supposed to observe. 
Well we can see there is cmd.exe in explorer.exe as there in pic below.

![pstree](/images/inctfi/12.png)

Along with that there is some pythonw.exe. Lets start analysing cmd.exe
By using **cmdscan** we can know what command is executed last in terminal.

![cmddownload](/images/inctfi/13.png)

As we see we couldn't find any commands we can see "Downloads" and "Desktop" maybe a clue to proceed.
So now lets check if there are any suspicious files in Desktop or Downloads. 
We can find it using Filescan as shown above in the pic.
Well we see a .py in Downloads. Lets go see what it does.

![pycode](/images/inctfi/14.png)

So after understanding the code, we can understand its prime motive is to xor a message with variable `t3mp`.
And it is recieving message from specific ip address and port number.
So we have a pcap let's try to get the message from it using wireshark.

![wireshark](/images/inctfi/15.png)

We see there is some encoded data lets get that using exfiltration in wireshark or we can use scapy module in python. So from scapy, we have to code and extract.
Now I thought it would be xor and xoring again with "t3mp" could give flag But its not. It's base64 encoded though.
Now as "t3mp" isnt working and base64 decoding is not giving anything useful we have to search for more details.
After using **cmdline, consoles, envars...**. I encountered some data in envars which is as follows;

![envar](/images/inctfi/16.png)

Now lets try xor this msg with "t3mp", but still that isn't useful.
So I tried xoring the data we got in envars with data we got in wireshark.
Well finally this gave the flag. And the flag is; `inctf{n3v3r_TrUs7_Sp4m_e_m41Ls}`


