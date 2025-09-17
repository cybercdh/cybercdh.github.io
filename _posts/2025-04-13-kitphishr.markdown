---
layout: post
title:  "The Curious Case of Mr. Phisherton's Left-Behind Zip File"
date:   2025-04-13 16:22:00 +0100
categories: phishing recon kitphishr reginald
permalink: /kitphishr/
---
*Featuring open directories, forgotten credential dumps, and a security researcher with way too much disk space.*

**TL;DR:** enumerate obviously open directories and pull exposed phishing kits for research/IOC extraction — do not access anything non-public.

```bash
cat hosts.txt | kitphishr -c 30 -out loot | tee findings.csv
```

Use it: https://github.com/cybercdh/kitphishr

Reginald Q. Phisherton III (self-appointed) makes phishing kits the way a first-year student makes spaghetti: in bulk, with enthusiasm, and directly into the sink. He posts whole webroots to `/var/www/html` like it’s 2002, names his credential logs `creds.txt`, and zips the entire mess without a password because why not let fate decide.

On a wet afternoon I pointed [kitphishr][gh-kitphishr] at a dozen suspicious hosts. Think of it as “museum night security for bad webservers”: it walks predictable paths like `/login/`, `/admin/`, `/panel/`, peeks for open directories, and sniffs out `.zip` files left lying around like gym bags. If there’s a kit, it fetches it; if there’s a `creds.txt`, it gently screams.

I found `pp_secure_login_v2.zip`. Inside: pixel-approximate PayPal HTML, a `process.php` that logs to plaintext, and yes, our old friend `creds.txt`:

```php
$file = fopen("creds.txt", "a");
fwrite($file, "Email: ".$email." Password: ".$password."\n");
fclose($file);
```

Open directories are the cyber equivalent of leaving your front door open with a sign that says “We trust the universe.” Here’s the minimum viable curiosity you can apply without any bespoke tooling:

```bash
# Safe recon on obviously open directories (no auth bypasses)
base=https://suspicious.example
wget --recursive --no-parent --no-host-directories --level=1 \
     --accept ".zip,.tar,.gz" "$base/" -P loot/

# List interesting zips and fingerprint common kit files
fd -e zip -a loot | while read z; do
  echo "--- $z"; unzip -l "$z" | rg -i "(creds\.|process\.php|sendmail|config|result|panel)" || true
done
```

Prefer a single-file fetch against a known open path? Respect robots.txt and local law, but when a directory is deliberately exposed:

```bash
curl -fsSL "$base/paypal/credentials.zip" -o loot/credentials.zip || echo "nope"
```

Copy me:

```bash
# Minimal pipeline: hosts -> kits -> IOCs
cat hosts.txt | kitphishr -c 30 -out loot | tee kits.csv
fd -e zip loot -x unzip -l {} \; | rg -i "(creds\.|sendmail|process\.php|/admin)" | tee iocs.txt
```

Once you’ve got a kit, quick triage in Python helps separate “cringe” from “court exhibit”:

```python
import sys, zipfile, hashlib, re

z = zipfile.ZipFile(sys.argv[1])
paths = z.namelist()

# detect credential sinks
creds = [p for p in paths if re.search(r"creds?\.(txt|log)$", p, re.I)]
print("creds files:", creds)

# fingerprint core templates for reuse analysis
hashes = {}
for p in paths:
    if p.endswith(('.php','.html','.js','/login','/index')):
        h = hashlib.sha1(z.read(p)).hexdigest()
        hashes.setdefault(h, []).append(p)

print("common code hashes:", len(hashes))
```

The joy isn’t just in the single kit; it’s in the patterns. Same footer typos, same `sendmail.php`, same admin panel at `/admin/`, sometimes even the same hardcoded panel password left in `config.php`. Collect three kits and you’re basically doing sloppy forensics with a punchline.

kitphishr makes this civilized. You feed it hosts or let it use an OSINT feed; it pokes, downloads, tags, and stashes. You get a tidy folder instead of a browser history that looks like evidence.

Practical pipeline:

```bash
# 1) Seed with suspicious domains
cat hosts.txt \
| kitphishr -c 30 -out loot \
| tee findings.csv

# 2) Walk the loot and triage
fd -e zip loot \
| while read z; do python triage.py "$z"; done

# 3) Pull IOCs for defenders (URLs, hashes, panel endpoints)
rg -NI "(action=|/process\.php|/sendmail\.php|/admin)" -g "loot/**" | tee iocs.txt
```

Analogy time: these webroots are messy dorm rooms. You are not “breaking in”; you are standing in the corridor, looking through the open door at the pile of pizza boxes labelled “passwords.txt” and taking notes so you can tell the RA.

Rules that keep you safe and useful:

- Only collect what is publicly, obviously exposed. No guessing, no bypassing.
- Don’t touch victim data; notify the host/abuse contact and the platform (if any).
- Prefer hashes/paths over contents in reports; redact aggressively.

[Kitphishr][gh-kitphishr] is open source and enjoys long walks through open directories. Reginald, if you’re reading this: stop calling it `creds.txt`. Call it `totally_not_credentials.txt` like a professional. Kidding. Please don’t.

[gh-kitphishr]:https://github.com/cybercdh/kitphishr.git

Report snippet (to hosting/abuse or platform):

```
Title: Publicly exposed phishing kit directory and plaintext credential sink

Host: suspicious.example (/paypal/)
Evidence:
  - Directory listing publicly accessible at https://suspicious.example/paypal/
  - Zip present: credentials.zip (hash: <SHA256>)
  - Kit files include process.php and creds.txt (listing only; contents not accessed)
Impact: Live credential capture and reuse by third parties; brand abuse; user compromise.
Actions taken: Collected file names and hashes only; no victim data accessed.
Recommended fix: Disable directory listing; remove kits; rotate any compromised credentials.
```
