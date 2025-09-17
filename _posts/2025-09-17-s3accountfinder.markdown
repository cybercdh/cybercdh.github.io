---
layout: post
title: "Bucket Bingo, But With Owners: Attributing S3 Buckets Cleanly"
date: 2025-09-17 10:40:00 +0100
categories: cloud s3 recon attribution buckets
permalink: /s3accountfinder/
---

*Featuring: stray buckets, mysterious owners, and a surprisingly helpful error message or two.*

**TL;DR:** find bucket names, verify existence, get regions, and attribute AWS buckets to account IDs with S3AccountFinder — so you can notify the right humans without peeking inside.

```bash
bucket-finder < urls.txt | tee buckets.txt
cat buckets.txt | isbucket | tee live-buckets.txt
awk '$2=="aws"{print $1}' live-buckets.txt | s3-bucket-region | tee aws-regions.txt
cut -f1 aws-regions.txt | S3AccountFinder -c 20 | tee owners.txt
```

Use it: https://github.com/cybercdh/S3AccountFinder

Meet Becky Buckets. Becky has a list of URLs from a giant app and plays “is this a bucket?” without touching any objects. She wants to tell the right team that their marketing site still links to `totally-public-assets`, but she’d also like to know who “owns” that bucket so her email goes to a person and not to the void.

Copy me:

```bash
# 1) Crawl for bucket-like names (S3, GCS, Azure) from URLs/content
bucket-finder < urls.txt | tee buckets.txt

# 2) Check existence across clouds (no listing, just existence/metadata)
cat buckets.txt | isbucket | tee live-buckets.txt

# 3) Focus on AWS; discover regions to reduce false alarms
awk '$2=="aws"{print $1}' live-buckets.txt | s3-bucket-region | tee aws-regions.txt

# 4) Attribute AWS buckets to account IDs
cut -f1 aws-regions.txt | S3AccountFinder -c 20 | tee owners.txt
```

Why attribution matters: escalations go faster when you can say “belongs to AWS account `123456789012` (likely the marketing sandbox)” instead of “belongs to vibes.” Incident responders love specifics.

What S3AccountFinder does in practice (high level):

- Probes bucket endpoints politely and interprets ownership‑revealing responses where possible.
- Uses region‑aware endpoints to avoid noisy/ambiguous errors.
- Emits best‑effort mappings of bucket → AWS account ID. Some buckets won’t disclose enough — that’s expected.

Installation:

```bash
go install github.com/cybercdh/S3AccountFinder@latest
go install github.com/cybercdh/bucket-finder@latest
go install github.com/cybercdh/isbucket@latest
go install github.com/cybercdh/s3-bucket-region@latest
```

Sample owners output (illustrative):

```
product-assets           111122223333   us-east-1   ok
stage-artifacts          444455556666   eu-west-1   ok
legacy-public-cdn        unknown        us-west-2   insufficient-signal
```

Safety and etiquette:

- Do not attempt to list or read objects; existence/region/owner is enough for a report.
- Be gentle: concurrency 10–30 is civilized; back off on repeated 429/503.
- Some buckets are intentionally access‑controlled; “unknown” owner is a valid, respectful result.

If you need a quick sanity check using AWS CLI (for your own buckets only):

```bash
# For buckets you own (requires permissions):
aws s3api get-bucket-acl --bucket my-bucket | jq '.Owner'
```

Report snippet (to the right humans faster):

```
Title: Publicly referenced S3 bucket attribution and posture

Bucket: product-assets (AWS)
Region: us-east-1
Owner account: 111122223333 (inferred)
Evidence:
  - Existence and region confirmed via s3-bucket-region
  - Ownership inferred via S3AccountFinder (best‑effort; see attached output)
Impact: External references to bucket; verify policy to prevent unintended public access
Recommended:
  - Confirm bucket policy and block public access if not intended
  - Tag bucket with owner/application to aid future attribution
  - Remove stale links in codebases/marketing sites if deprecated
Notes: No object listing or content access attempted; metadata only.
```

Attribution isn’t just detective work, it’s kindness: it lands your heads‑up on the right desk. And it keeps you from spending your evening spelunking through a decade of git history trying to guess which team loved `legacy-public-cdn` in 2017.

