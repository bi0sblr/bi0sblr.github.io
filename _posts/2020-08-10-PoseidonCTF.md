---
layout : post
title: PoseidonCTF
description: Rev writeups
date: 2020-08-10
image: /images/poseidon/logo.png
tags:
   - z3
   - ctf
   - rev
author: thatloststudent
categories: Reversing
---

![logo](/images/poseidon/logo.png)
# Intro

Well, I screwed up. I was only able to solve only one rev chall, their warmup. I guess it's normal for a n00b like me, though.


## The Large Cherries

We're given a file called Lao-Tzu. 

![ptrace reeee](/images/poseidon/rev/1.png)

There's a ptrace at the very top, so if you bypass it, you will see that:

It asks for two kinds of inputs, and for the first time, it asks for 9 characters and 10 characters the second time.

The first input is sent to this function, called `magic_word`.

![magick](/images/poseidon/rev/2.png)

The function essentially makes sure that certain conditions are met for input[0] to input[7], even though the input asks for 9 characters. This is our key to solving this chall. 

```
from z3 import Solver, Xor, BitVec

a = [BitVec('a%s' % i,64 ) for i in range(9)]

s = Solver()
s.add(a[3] + a[0] == 0xab)
s.add(a[3] == 0x37)
s.add(a[2] ^ a[1] == 0x5d)
s.add(a[4] - a[2] == 5)
s.add(a[6] + a[4] == 0xa2)
s.add(a[5] == a[6])
s.add(a[6] == 0x30)
s.add(a[7] == 0x7a)
s.check()
m = s.model()
print(s.model())
b= []
for i in range(8):
    b.append(chr(int(m[a[i]].as_string())))
c = ''.join(b)
print(c)
```
Now, this piece of code gives out `t0m7r00z`, which is only 8 characters. This will be important later.

The second piece of input is a 10 character input, that checks if all of the characters add up to `0x294`, i.e. `660`. That happens to be the sum of digital sum of `t0m7r00z`. 

Therefore, we need to add something extra to the mix. This character can be quite literally anything, and I just typed `t0m7r00za` as both inputs.

Flag: `Poseidon{Its_L3_t3Mp5_DeS_C3r1s35}`


