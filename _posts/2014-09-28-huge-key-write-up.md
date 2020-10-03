---
id: 24
title: 'SU CTF 2014 &#8211; Huge key write up'
date: 2014-09-28T21:16:39+00:00
author: mashhadimj
layout: post
guid: http://mjafar.me/blog/?p=24
permalink: /blog/:year/:month/huge-key-write-up/
categories:
  - CTF Writeup
---

Given a php file as coder and an encoded file, Find the flag.

------

In encipher code the $key is initially [0, 0]. After reading `Huge Key` file and the nested fors, the key will become (using python notation):

```python
# hk = huge key

key[0] = hk[-2] ** hk[-4] ** ... ** hk[0] ** key[0]
key[1] = hk[-1] ** hk[-3] ** ... ** hk[1] ** key[1]
```

Well, looks like neither key nor hk are not recoverable. But to decode the input we don't need the huge key, only key is enough.
With running this piece of php code we can find out the size of initialization vector:


```php
<?php
$iv_size = mcrypt_get_iv_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_CBC);
echo "$iv_size\n";
```

It will output 16, since the output file is prepended with iv this is very helpful in decoding.

Key has 256 * 256 possible values which can be found in a moderately good time with brute-forcing.

```php
<?php
$ciphertext = file_get_contents("ciphertext.bin");
$blocks = str_split($ciphertext, 16);
$iv = $blocks[0];
$message = '';
for($i=1; $i<count($blocks); ++$i) {
   $message .= $blocks[$i];
}
// I know this way of slicing input is inefficient
// and a little stupid, but come on! :D

for ($i = 0; $i < 255; ++$i) {
  for ($j = 0; $j < 255; ++$j) {
    $key[0] = chr($i);
    $key[1] = chr($j);
    $decypher = mcrypt_decrypt(MCRYPT_RIJNDAEL_128, $key, $message, MCRYPT_MODE_CBC, $iv);
    if (strpos($decypher, "flag") !== FALSE) {
        echo "key = ".ord($key[1]).",".ord($key[0])." = $key\n\"$decypher\"\n";
    }
  }
}
?>
```

Note that if you have php binaries installed on your OS, you don't need a webserver to run this code, simply run it in a terminal.

```bash
$ php decode.php
```
