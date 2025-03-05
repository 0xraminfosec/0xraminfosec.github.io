---
title: TFC-CTF 2022
published: 2022-07-31
description: Challenge walkthrough's
image: ./TFC.png
tags: [ctf, web, ruby, ssti]
category: CTF
draft: false      # Set only if the post's language differs from the site's language in `config.ts`
---

## CALENDAR - WEB (Medium)
### Description
Are online calendars trusty?
Note: The website displays "Error establishing a database connection" at startup. Please wait ~10 seconds and refresh
The flag is format :TFCCTF{FOUNDPASSWORD}

### Solution

On visiting the challenge url,it was the simple website which was using wordpress.
So, i started searching about wordpress CVE's based on calenders.We ended up with this <a href="https://www.exploit-db.com/exploits/50084">CVE</a>

![Screenshot](https://raw.githubusercontent.com/0xRamInf0sec/TFC-CTF/refs/heads/main/Calendar/cve.png "chall")

Here, what is the vulnerability is , It did not properly restrict access to the export files,
allowing unauthenticated users to exports all events data in CSV or XML format.

![Screenshot](https://raw.githubusercontent.com/0xRamInf0sec/TFC-CTF/refs/heads/main/Calendar/CVEout.png "chall")

Using this exported CSV file, we can able to find username and password. And the password is the FLAG!!!

## PONG - WEB (WARMUP)
### Description
Some random website that can ping hosts.

### Solution

It was the easy command injection challenge.

#### payload 
> /index.php?host=127.0.0.1|cat%20../../../flag.txt

## Diamonds - WEB (MEDIUM)
### Description
Write something nice that passes our filter.

The webpage shows only one input box which can allow to enter only alphabets and numbers.

![Screenshot](https://raw.githubusercontent.com/0xRamInf0sec/TFC-CTF/refs/heads/main/Calendar/diamonds.png "chall")

If we enter anything other than this it shows like `That is far away from nice !! `
So,here we have to bypass the regex which were written on the server side.

`There were a lot of payloads online on SSTI . But before that can be passed through to read the flag.txt file. After a time of research,Knowing that the regex can be bypassed with newlines, we can start looking for an exploit to read flag.txt `

### Solution 

```
<%= File.open('flag.txt').read %>
```

 But we need to url encode this payload
 
![Screenshot](https://raw.githubusercontent.com/0xRamInf0sec/TFC-CTF/refs/heads/main/Calendar/diamondsout.png "chall")



