---
title: Clean Bowled🏏!! Personal numbers of CSK Superstars and a lot more leaked !!
published: 2023-05-30
description: AWS misconfiguration to PII leak.
image: ./csk.png
tags: [web,aws,PII,csk]
category: General
draft: false      # Set only if the post's language differs from the site's language in `config.ts`
---

**Hi Security researchers,**  
Hey fellow hackers and bug hunters 👋  
This is the story of how a simple revisit to a website turned into full AWS access, database backups, and accidental exposure of a cricket team's internal data - all because nobody bothered to fix the bugs I reported years ago.

Relax, sit back. This one’s wild. 🏏💥

All findings were responsibly reported and fixed.  
This write-up exists purely for educational and awareness purposes.

---

## 🕸️ Flashback: October 2021  
In October 2021, I reported some *solid* vulnerabilities on **chennaisuperkings.com**:

- Authentication bypass  
- SQL injection  
- PII exposure  

Serious stuff.  
I waited for a response…

And waited…

Absolutely nothing.  

![](https://media.tenor.com/e53PdvXHRswAAAAC/sad.gif)

---


## ⏳ May 29, 2022 — One Year Later  
Out of curiosity (*and maybe a little frustration*), I revisited the site.  
Guess what changed?

**NOTHING.**

While scrolling through Burp Suite traffic, something popped into my eyes like a flashbang - an API response casually leaking **full AWS credentials**.

Yup. Right there. In plaintext.

```json
{
  "result": {
    "AWS_S3_REGION": "ap-south-1",
    "IMAGE_NAME": "image.jpg",
    "AWS_S3_BUCKET": "XXXXXX",
    "AWS_S3_ACCESS_KEY_ID": "AKIXXXXXXXXXXXXXXXAQ",
    "AWS_S3_SECRET_ACCESS_KEY": "Secret-key",
    "AWS_S3_FAN_PAGES_PATH": "path"
  },
  "status": "SUCCESS"
}
```
![](https://media.tenor.com/ZIwshzy97nkAAAAC/shocked-vadivelu.gif)

This is never supposed to be exposed to the client.

---

## Exploring the AWS Account

With these leaked credentials, it became evident that the IAM policies were overly permissive.

I was able to list IAM users:

![](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/awsusers.png)

The AWS account contained multiple S3 buckets:

![](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/awsbuckets.png)

And several SQL database backup files:

![](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/awsdb.png)

![](https://media.tenor.com/YSHs-WfGMWQAAAAd/seena-thana-tamil.gif)

---

## Sensitive Data Exposure

Inside one of the database backups, sensitive personal information was exposed - including the contact details of players and staff.

![](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/cskalter.png)

The leaked phone numbers even matched public caller databases.

![](https://raw.githubusercontent.com/0xraminfosec/imagerepo/refs/heads/main/IMG_1305.jpg)

---

## Potential Impact

If discovered by a malicious actor, these vulnerabilities could have enabled:

- Full access to S3 buckets

- Download of complete database backups

- PII harvesting

- Targeted phishing/social engineering

- Tampering or deletion of data

- Uploading malicious files to S3

- Compromising site assets or user data

This wasn’t just a minor misconfiguration - It was a complete **cloud compromise risk**.

---

## Reach me

[@0xraminfosec](https://instagram.com/0xraminfosec)