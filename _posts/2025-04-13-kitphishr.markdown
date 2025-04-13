---
layout: post
title:  "The Curious Case of Mr. Phisherton's Left-Behind Zip File"
date:   2025-04-13 16:22:00 +0100
categories: phishing recon kitphishr reginald
permalink: /kitphishr/
---
*Featuring open directories, forgotten credential dumps, and a security researcher with way too much disk space.*

I’d like to introduce you to a man named **Reginald Q. Phisherton III**. Reg, as he insists his criminal forums call him, is a career phishing kit author with a fondness for three things: generic PayPal login pages, uploading his source code to webroots like it’s 2002, and never-*ever*-zipping with passwords.

It was a rainy afternoon when I first pointed [kitphishr][gh-kitphishr] at a few dozen freshly harvested subdomains. The tool had just received its latest OSINT feed scrape, and I was curious to see if the shady corners of the internet had been generous.

They had.

Kitphishr's approach is simple: feed it a list of domains or let it auto-scrape known badlands, then have it quietly poke through the URL paths - `example.com/phishkit/`, `example.com/login/zip/`, `example.com/paypal/credentials.zip` - you get the idea. If there’s an open directory, it peeks inside. If it finds a `.zip`, it grabs it. If that zip contains the exact kind of vintage, amateur phishing setup you’d expect to find on a C-tier Telegram group - jackpot.

Within minutes, I had a neat little zip file titled `pp_secure_login_v2.zip`. Inside: a full phishing site kit pretending to be PayPal, complete with:

- pixel-perfect HTML clone of the PayPal login page (with typos in the footer)
- a `process.php` file that didn’t process anything securely
- and a surprise bonus: `creds.txt`

The golden rule of phishing kits is that the bad guys are often worse at OPSEC than their victims. In this case, `creds.txt` was a plain text dump of every email and password entered into the fake login form. It was just sitting there. Public. Unencrypted. In 2025.

```php
$file = fopen("creds.txt", "a");
fwrite($file, "Email: ".$email." Password: ".$password."\n");
fclose($file);
```

This exact kind of find is what kitphishr was built for. Not just to collect kits for research and signature building, but to expose the many, many Reggies of the phishing world who treat their webservers like personal Dropbox accounts. I’ve collected over 60GB of kits using this method - dozens with hardcoded credential logging paths, admin panel credentials in config files, even one with screenshots of the developer’s desktop (Reginald, buddy, no).

What makes this interesting isn’t just the kits themselves, but the patterns you start to see: reused code, reused directories, reused typos. The same developer might post their kit under different aliases, but forgets to change the copyright year in the footer. It’s like phishing kit forensics, but with more comedy and less lab coats.

[Kitphishr][gh-kitphishr] is open source, and extremely good at what it does—quietly pulling back the curtain on the half-hearted operational security of cybercriminals who think `chmod 777` is a lifestyle choice.

If you're interested in exploring what’s hiding in the dark, dusty corners of shady webservers, or you just want to marvel at how little effort some criminals put into hiding their tracks, check out the project:

[https://github.com/cybercdh/kitphishr][gh-kitphishr]

And Reginald, if you're reading this—please stop naming your credential dumps `creds.txt`. You're making it too easy.

[gh-kitphishr]:https://github.com/cybercdh/kitphishr.git