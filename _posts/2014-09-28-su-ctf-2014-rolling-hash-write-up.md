---
id: 13
title: 'SU CTF 2014 &#8211; Rolling Hash Write up'
date: 2014-09-28T19:26:44+00:00
author: mashhadimj
layout: post
guid: http://mjafar.me/blog/?p=13
permalink: /blog/:year/:month/su-ctf-2014-rolling-hash-write-up/
tag:
  - CTF Writeup
---
```python
flag="*********"
def RabinKarpRollingHash( str, a, n ):
        result = 0
        l = len(str)
        for i in range(0, l):
                result += ord(str[i]) * a ** (l - i - 1) % n
        print "result = ", result


RabinKarpRollingHash(flag, 256, 10**30)
```

<span style="color: #333333;">output is
1317748575983887541099
What is the flag?</span>

----

Well, the point is 10**30 is a very large number and calculating the modulus has no effect on output. also 256 is a power of 2.

Given code calculates ASCII code for each character of input and multiplies it in 256 ** (l-i-1) that makes their bits shift (8*k) then adds all results together. So representation of <span style="color: #333333;">1317748575983887541099 in binary contains concatenated binary codes of flag in reverse order. 
I used  [this site](http://www.onlineconversion.com/base.htm) to convert this number to binary.</span>


```python
chars = [
  0b01000111,
  0b01101111,
  0b01101111,
  0b01100100,
  0b00100000,
  0b01001100,
  0b01110100,
  0b00000000,
  0b00000000,
]
flag = ''
for ch in chars:
  flag = flag + chr(ch)

print flag
```

Well, that site failed in last bits and this code outputs **'Good Lt'** but it's very easy to guess that flag was **Good Luck**

----

**EDIT:** a better solution for the last step mentioned in comments:

```python
flag = ''
number = 1317748575983887541099 
p = number
while p > 0:
  flag = chr(p%256) + flag
  p = p / 256

print flag
```
