---
layout: post
title: Arab Security Cyber Wargames
date: 2020-08-15
description : Crypto Writeups
image: /images/ascwg/ascscoreboard.png
tags: [crypto,ctf,rsa,des]
author_staff_member: Siddartha Malladi
---
![scoreboard](/images/ascwg/ascscoreboard.png)
First things first, Arab Security Cyber Wargames is a qualifiers CTF, Top 10 would be qualified for the finals at Egypt.
We [c0d3\_h4cki05\_](https://ctftime.org/team/72702)(aka bi0s|Bangalore) finished 10th globally, hence we qualified for finals!
Yay! 
 
In this blog post I will be discussing 2 crypto challenges from Arab Security Wargames CTF Quals. As there are some intial glitches with the server, they shared the challenges [repo](https://github.com/ascwg/Challenges) in the discord server, so we were able to work on the challenges even though there were some glitches.
 
|S.No.|Challenge                                                                            | Points |
|:---:|:-----------------------------------------------------------------------------------:|:------:|
| 1   | [Challenge 3](https://github.com/ascwg/Challenges/tree/master/Crypto/Challenge%203) | 600    |
| 3   | [Challenge 5](https://github.com/ascwg/Challenges/tree/master/Crypto/Challenge%205) | 300    |
 
## Challenge 3
> [output.txt](https://github.com/malladisiddu/Crypto-writeups/blob/master/ascwgctf/challenge3/output.txt)
 
Given `output.txt` consists of Public Key (`n` & `e`) and ciphertext `c`
```
n = 2318754427090927622417300593014303163027836982793164162950666250489681094136583599882469330682357229700000166714186122335692872792460409101465630110622887313064657894574037981904943176292533073634387002369380564791579428603519429963490374738649708747360755590037132507998435966068658633431918622092817702780128462915129741083129108481836485937804951555271147615962278158353917059561029043381242474374972583682945918237047674797098894662717409552897418650427548642489575961500481014997803061734956091625431696419759919121068387038071453059311371255995535187052409462363525765654622645413142987775053860188260137197659
e = 65537
c = 1852258477078452495280071169336816541669321769289372837198526954361460776833319048556839287633046754304414868057993901219892835088957705515939202089076460374548771033553266251154753679870528816210706553445963568771841753267644973871132621342897934474998162148362874305941012572949171990616677298854465965898581914403406403426504250013897086136105989549801404176555930509653029014518314103310549883855327513190607775750086851774949594618287441246861446444592130784569563671269161854267497652454746479173284327272563799067627736512266913669944284375302659511122504002144054772208775215907860036195680830269422876824977
```

I tried to factorize modulus `n` using [factordb](http://factordb.com/) which didn't help me, so I tried to find the [fermat factors](https://en.wikipedia.org/wiki/Fermat%27s_factorization_method), if you don't know the algo then no worry here is a simple function for Fermat factorization.
 
```python
def fermat_factor(n):
    assert n % 2 != 0
    a = gmpy2.isqrt(n)
    b2 = gmpy2.square(a) - n
    while not gmpy2.is_square(b2):
        a += 1
        b2 = gmpy2.square(a) - n
    return a + gmpy2.isqrt(b2), a - gmpy2.isqrt(b2)
``` 
yes, I found `p` and `q` using Fermat Factoriation,

```
p = 48153446679245376966822046985112099446617981034794594214042780096131516418638366375608599332095159143650219571976756039936351280836582867794175112625879990923510369077946617421338536566796348803001717218384229667003185508514134592197193786758239794011461538791978511429725895132475565257089664121103110770817                                                   
q = 48153446679245376966822046985112099446617981034794594214042780096131516418638366375608599332095159143650219571976756039936351280836582867794175112625874897500464997377986242441540940715154519674822662819026591330454041967249535003603147605312684911517825154805431323771837685531683672611660925609168788996827
```
 
Wait, challenge is not over yet. I tried decrypt the given ciphertext using these primes but couldn't retrieve the plaintext. then I checked whether `p` and `q` are primes or not, as we totient function should be calculated only by primes.
![partofchallenge](/images/ascwg/crypto/acscrypto3.png)
```
In [11]: isPrime(p)
Out[11]: 0

In [12]: isPrime(q)
Out[12]: 0
```
As the result shown above they aren't primes. So I further factorized `p` and `q` which got reduced to `p1`, `p2`, `q1`, & `q2`.
```
p1 = 6939268454184877330211144138413966814481101061382015473621711919814088916348213343387168181954880781520959109737312885406280110070698427014630125251118873
p2 = 6939268454184877330211144138413966814481101061382015473621711919814088916348213343387168181954880781520959109737312885406280110070698427014630125251119529
q1 = 6939268454184877330211144138413966814481101061382015473621711919814088916348213343387168181954880781520959109737312885406280110070698427014630125251118111
q2 = 6939268454184877330211144138413966814481101061382015473621711919814088916348213343387168181954880781520959109737312885406280110070698427014630125251119557
```
so now the Euler totient function is
```
phi = (p1-1)*(p2-1)*(q1-1)*(q2-1)
```
Combining everything what I've discussed above we get entire exploit script,
```python
from Crypto.Util.number import *
import gmpy2
def fermat_factors(n):
    assert n % 2 != 0
    a = gmpy2.isqrt(n)
    b2 = gmpy2.square(a) - n
    while not gmpy2.is_square(b2):
        a += 1
        b2 = gmpy2.square(a) - n
    return a + gmpy2.isqrt(b2), a - gmpy2.isqrt(b2)
n = 2318754427090927622417300593014303163027836982793164162950666250489681094136583599882469330682357229700000166714186122335692872792460409101465630110622887313064657894574037981904943176292533073634387002369380564791579428603519429963490374738649708747360755590037132507998435966068658633431918622092817702780128462915129741083129108481836485937804951555271147615962278158353917059561029043381242474374972583682945918237047674797098894662717409552897418650427548642489575961500481014997803061734956091625431696419759919121068387038071453059311371255995535187052409462363525765654622645413142987775053860188260137197659
e = 65537
c = 1852258477078452495280071169336816541669321769289372837198526954361460776833319048556839287633046754304414868057993901219892835088957705515939202089076460374548771033553266251154753679870528816210706553445963568771841753267644973871132621342897934474998162148362874305941012572949171990616677298854465965898581914403406403426504250013897086136105989549801404176555930509653029014518314103310549883855327513190607775750086851774949594618287441246861446444592130784569563671269161854267497652454746479173284327272563799067627736512266913669944284375302659511122504002144054772208775215907860036195680830269422876824977


p, q = fermat_factors(n)

p1, p2 = fermat_factors(p)

q1, q2 = fermat_factors(q)
phi = (p1-1)*(p2-1)*(q1-1)*(q2-1)
d = inverse(e,phi)
flag = long_to_bytes(pow(c,d,n))
print(flag)
```
Flag: `ASCWG{you_need_fermat_factorization_to_solve_RSA_Small_diffrince_Prime_Attack_12312}`

---------------

## Challenge5
> [challenge.py](https://github.com/malladisiddu/Crypto-writeups/blob/master/ascwgctf/challenge5/challenge.py) [output.txt](https://github.com/malladisiddu/Crypto-writeups/blob/master/ascwgctf/challenge5/output.txt)

This challenge was pretty easy, I didn't took much time to solve this chalenge as I was aware of this attack before. Given `challenge.py` is the encrypiton file and `output.txt` is the ciphertext,
```python
from Crypto.Cipher import DES
import base64
from FLAG import flag


def pad(plaintext):
    while len(plaintext) % 8 != 0:
        plaintext += "*"
    return plaintext

def enc(plaintext,key):
    cipher = DES.new(key, DES.MODE_ECB)
    return base64.b64encode(cipher.encrypt(plaintext))


key = "##############".decode("hex")
plaintext = pad(flag)
print enc(plaintext,key)
```
```
kIi6qSDhcSVErHbkpy/M1hRHfDpr8TiaGbAIrKUXooxSXwNnaeJgTQ==
```
When we look in to the `challenge.py` it consists of two functions `pad` for padding the flag and `enc` for encrypting in `DES` and that to in `ECB` mode.

### The Vulnerability
Here the vulnerability is the key. DES has specifically 4 weak keys. You can refer about weak keys [here](https://en.wikipedia.org/wiki/Weak_key). So I applied DES weak key attack. And the weak keys are

```
0x0000000000000000
0xFFFFFFFFFFFFFFFF
0xE1E1E1E1F0F0F0F0
0x1E1E1E1E0F0F0F0F
```
As I don't know the key with which they encrypted, I just bruteforced among the 4 possible keys and got the flag.
Here is my entire exploit,
```python
from Crypto.Cipher import DES
import base64
import binascii
cipher = base64.b64decode("kIi6qSDhcSVErHbkpy/M1hRHfDpr8TiaGbAIrKUXooxSXwNnaeJgTQ==")
"""
Check out the weak keys of DES. There are 4 possible weak keys
0x0000000000000000
0xFFFFFFFFFFFFFFFF
0xE1E1E1E1F0F0F0F0
0x1E1E1E1E0F0F0F0F
"""
key = binascii.unhexlify("0000000000000000")
des = DES.new(key,DES.MODE_ECB)
pt = des.decrypt(cipher)
print(pt)
key = binascii.unhexlify("FFFFFFFFFFFFFFFF")
des = DES.new(key,DES.MODE_ECB)
pt = des.decrypt(cipher)
print(pt)
key = binascii.unhexlify("E1E1E1E1F0F0F0F0")
des = DES.new(key,DES.MODE_ECB)
pt = des.decrypt(cipher)
print(pt)
key = binascii.unhexlify("1E1E1E1E0F0F0F0F")
des = DES.new(key,DES.MODE_ECB)
pt = des.decrypt(cipher)
print(pt)
```
Flag: `ASCWG{Welcome_to_des_weak_key_attack}`

You can find both the exploit scripts in my [github repo](https://github.com/malladisiddu/Crypto-writeups/tree/master/ascwgctf).
Please post your comments in the comment section or you can ping me via twitter [@st0ci3r](https://twitter.com/st0ic3r) for any queries, suggestions and feedback.
