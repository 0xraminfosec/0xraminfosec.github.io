---
title: 1337UP CTF
published: 2022-03-13
description: Challenge walkthrough's
image: ./inti.png
tags: [ctf, web]
category: CTF
draft: false      # Set only if the post's language differs from the site's language in `config.ts`
---

## Getting admin access 

> Description :  Bling bling bling, I bet you're not 1337 enough to shop here! Wait, you need to change /etc/hosts to get this one working??? Yes! To become an admin, you should use /admin

As per description,we have to add the host in /etc/hosts file.First ping the 1337shop.intigriti.io ,we got the IP of that domain ,add that to /etc/hosts file.

![Screenshot](https://raw.githubusercontent.com/Ramalingasamy012/WebCTF-writeups/main/screenshots/1337shop1.png "Screenshot")

And we have the endpoint /admin. On visiting this endpoint it redirects to the login page and i tried lots of ways to bypass this login page but i can't !!.My teammate 0xGodson spoke about Nosqli for some other challs.

I searched for Nosqli payloads and i ends up with this payload
```
{"username": {"$gt":""}, "password": {"$gt":""}}
```

I got logged in !!.

![Screenshot](https://raw.githubusercontent.com/Ramalingasamy012/WebCTF-writeups/main/screenshots/1337shop2.png "Screenshot")

View the PageSource and there's  a flag.

```
FLAG{NOSQL_INJECTION_IS_AWESOME_IKODSD}
```
    
## Not my server 

>Description :     Wait, you're an admin? Well, what can you do now?

Well, It was the continuation of Getting admin access challenge.I logged in as admin and i noticed there was an functionality generates the PDF which contains the previous logins username,Date and time , User-Agent strings.

![Screenshot](https://raw.githubusercontent.com/0xRamInf0sec/WebCTF-writeups/refs/heads/main/screenshots/notmy1.png
 "Screenshot")

I crafted a html tags in User-agent strings while logging in and it reflects in the PDF and it also vulnerable to SSRF !!.

![Screenshot](https://raw.githubusercontent.com/0xRamInf0sec/WebCTF-writeups/refs/heads/main/screenshots/notmy2.png
 "Screenshot")
 
 I want to know which protocol they used to save files and i used this payload to print file location 
 
 ```
 <script>document.write(document.location.href)</script>
 
 Location : file:///tmp/wktemp-a5227fe0-e5d3-4ecb-a055-1f38964c02e2.html
 ```
 
 They used file:/// protocal and i tried iframe to render the /etc/passwd file but it failed.I used XHR requests but it also failed
 
 ```
 <script>exfil = new XMLHttpRequest();exfil.open("GET","file:///etc/passwd"); exfil.send();exfil.onload = function(){document.write(this.responseText);} exfil.onerror = function(){document.write('failed!')}</script>
 ```
 
 My teammate 0xGodson helped me to solve this chall.He sent a payload which was working like  a king!!
 
 ```
 <script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///etc/passwd");x.send();</script>
 ```

And I can able to read the /etc/passwd file!!!

![Screenshot](https://raw.githubusercontent.com/0xRamInf0sec/WebCTF-writeups/refs/heads/main/screenshots/notmy3.png
 "Screenshot")
 
 Finally, The flag is in file:///flag
 
 ```
 FLAG{SSRF_IS_AWESOME_UDKSO}
 ```

 ##  Quiz

> Description : Ready for a little quiz?

On visiting the challenge link,it shows like a normal quiz page which contains 3 questions each questions have 10 marks .Even if we answer correctly for all the questions ,we have only 30 points.But we need 100 points to buy the flag!!.

![Screenshot](https://raw.githubusercontent.com/Ramalingasamy012/WebCTF-writeups/main/screenshots/quiz1.png "Screenshot")

### Race Condition 
>A race condition attack happens when a computing system that's designed to handle tasks in a specific sequence is forced to perform two or more operations simultaneously. Eventually, the application is forced to perform unintended actions. This leads the application to security exploitation.

So,We have to perform race condition here to obtain extra points for each question's.Choose the answer correctly and 
intercept the request with burp and send that request to intruder and add the NULL payloads in Referrer header and start the attack,You can see the extra points added  !!!.

![Screenshot](https://raw.githubusercontent.com/Ramalingasamy012/WebCTF-writeups/main/quiz2.png "Screenshot")

![Screenshot](https://raw.githubusercontent.com/Ramalingasamy012/WebCTF-writeups/main/quiz3.png "Screenshot")

Repeat this steps for remaining two questions ,We obtain 100 points !!.

```
Flag : 1337UP{this_is_a_secret_flag}
```
    