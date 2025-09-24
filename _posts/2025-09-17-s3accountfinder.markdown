---
layout: post
title: "Becky and the Haunted Bucket: Attributing S3 Without Touching a Single Object"
date: 2025-09-24 10:40:00 +0100
categories: cloud s3 recon attribution buckets
permalink: /s3accountfinder/
---

*Featuring: haunted microsites, a bucket with commitment issues, and an investigator who attributes ownership without touching a single object.*

**TL;DR:** Becky Buckets hunts a retired microsite wired to a now‑dead S3 website bucket. She attributes the bucket’s AWS account without listing anything, proves the impact, and gets it fixed before the weekend. You can, too:

```bash
bucket-finder < urls.txt | tee buckets.txt
cat buckets.txt | isbucket | tee live-buckets.txt
awk '$2=="aws"{print $1}' live-buckets.txt | s3-bucket-region | tee aws-regions.txt
cut -f1 aws-regions.txt | S3AccountFinder -c 20 | tee owners.txt
```

Use it: https://github.com/cybercdh/S3AccountFinder

Becky didn’t set out to fight a ghost; she set out to find a bounty. The scope said “include all subdomains.” The marketing team said nothing, because they were busy rebranding “Spring Savings 2019” into “Perpetual Value 2025.” Somewhere in the middle, a microsite forgot to retire properly and started rattling its chains.

The first knock came from a harmless reconnaissance flyover. Becky piped a stack of URLs through her usual sieve and, like a detective in a trenchcoat, circled the ones that smelled like storage.

“I don’t break in,” she told her coffee mug (named Gerald, for accountability). “I just read the building directory.”

She ran this, because of course she did:

```bash
# Scan page sources for bucket-shaped strings
bucket-finder < urls.txt | tee buckets.txt
cat buckets.txt | isbucket | tee live-buckets.txt
awk '$2=="aws"{print $1}' live-buckets.txt | s3-bucket-region | tee aws-regions.txt
cut -f1 aws-regions.txt | S3AccountFinder -c 20 | tee owners.txt
```

Four buckets appeared. Three were boring (owned, alive, named like spreadsheets). The last one: `spring-savings-2019.s3-website-eu-west-1.amazonaws.com`. The region check said “eu-west-1.” The website endpoint said “NoSuchBucket.” And Becky’s heartbeat said “bounty.”

Here’s the trick the ghost didn’t expect. S3 website endpoints are polite even when buckets are gone. They tell you the region they expect, sometimes even the nature of their disappointment. That’s enough to know “the house exists, the lights are off.” You don’t need to open the door to see that nobody’s home.

But Becky wanted more than vibes. She wanted an owner, because escalation without a recipient is just performance art. Enter: S3AccountFinder. It doesn’t list objects. It doesn’t read content. It pokes the right regional endpoints, listens to what S3 mutters under its breath, and (where possible) infers the AWS account that owns the bucket name.

“Account 111122223333,” S3AccountFinder whispered back. “Probably the marketing sandbox. The other three buckets? 999900001111.” A tale of two accounts. A tale of something that used to work and quietly didn’t anymore.

Becky checked the CNAME next. `events.example.com` pointed into the website endpoint. The site was gone; the CNAME remained, nobly insisting that the party was still at Spring Savings 2019. It was not. And because `s3-website-eu-west-1.amazonaws.com` is a public suffix boundary, the label before it—`spring-savings-2019`—was available for anyone with a credit card and a mischievous grin.

She verified the “NoSuchBucket” once more, like a burglar who only checks that the door is locked, then wrote a note that would make an SRE weep with relief.

“Gerald,” Becky said, “sometimes attribution is kindness.”

For the curious (and the auditors), here’s what she actually ran and why it works without rummaging:

```bash
# 1) Confirm the bucket-shaped thing is actually S3 and find the region
echo spring-savings-2019 | s3-bucket-region

# 2) Nudge the website endpoint (no auth, no listing) and watch the error
curl -sS http://spring-savings-2019.s3-website-eu-west-1.amazonaws.com \
  | rg -q "NoSuchBucket" && echo "dead website bucket" || echo "something else"

# 3) Attribute bucket → AWS Account (best-effort; metadata only)
echo spring-savings-2019 | S3AccountFinder -c 10
```

On a good day, S3 mutters the exact headers and ARN breadcrumbs you need. On a cautious day, it still gives you enough to infer who’s likely responsible. On a bad day, you accept “unknown” and keep your report helpful.

Becky’s report went out after lunch. The reply arrived before the kettle boiled.

“We didn’t know that CNAME still existed,” said someone named Alex who is either a real person or a marketing collective. “We deleted the bucket when we moved the campaign. We didn’t realize someone could… just make another one.”

“They can,” Becky said, in the gentle tone you use with people and also with DNS. “But you’re fixing it now, so they won’t.”

The fix was simple: remove the CNAME, or recreate the bucket in the right account, or move behind CloudFront with a proper origin and an Origin Access Control. The kindness was attribution. The bonus points were a list of other stale references Becky found along the way—because old campaigns never die, they just change URLs.

Lessons, muttered like a mantra while the tea steeps:

- If your subdomain points to an S3 website endpoint, ensure you actually own the bucket that name implies. “NoSuchBucket” is the ghost at the banquet.
- Tag buckets with owner and purpose. Tomorrow’s Becky is you.
- Keep bucket public access blocked by default; prefer CloudFront where possible.
- When you retire a microsite, retire the DNS too. DNS has the memory of an elephant and the bedside manner of a cat.

You can reproduce Becky’s whole evening in six lines:

```bash
bucket-finder < urls.txt | tee buckets.txt
cat buckets.txt | isbucket | tee live-buckets.txt
awk '$2=="aws"{print $1}' live-buckets.txt | s3-bucket-region | tee aws-regions.txt
cut -f1 aws-regions.txt | S3AccountFinder -c 20 | tee owners.txt
awk '/NoSuchBucket|unknown/ {print}' owners.txt
```

Report snippet (you know the drill, but here’s the good version):

```
Title: CNAME to S3 website endpoint with unclaimed bucket (potential takeover)

Host: events.example.com
Target: spring-savings-2019.s3-website-eu-west-1.amazonaws.com
Evidence:
  - Website endpoint returns NoSuchBucket (HTTP body)
  - Bucket region: eu-west-1 (s3-bucket-region)
  - S3AccountFinder infers owner account 111122223333 for bucket name
Impact: Attacker could create "spring-savings-2019" bucket and serve arbitrary content at events.example.com
Actions taken: No bucket creation/listing; metadata only
Recommended fix: Remove/replace CNAME or recreate and control the backing bucket; consider CloudFront + OAC; tag buckets with owners
```

Becky closed her laptop. The ghost stopped rattling. Gerald approved, silently, as mugs do. And somewhere, a marketing calendar finally let go of 2019.
