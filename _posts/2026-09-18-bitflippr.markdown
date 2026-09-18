---
layout: post
title: "Priya and the Cosmic Ray: Squatting S3 Buckets One Flipped Bit at a Time"
date: 2026-09-18 10:00:00 +0100
categories: cloud s3 aws bitflip cosmic-rays theory
permalink: /bitflippr/
---

*Featuring a particle from a dying star, a hyphen with ambitions, one coffee mug of prior renown, and a heist that only works while you're asleep.*

**TL;DR.** A hyphen and a forward slash are one bit apart in binary. In S3, a forward slash is the wall between a bucket and a key. So flip a single bit in the right spot and the one bucket named `really-popular-bucket-123456789` quietly becomes `really-popular-bucket/123456789`. S3 no longer reads that as one name. It reads a bucket called `really-popular-bucket` holding a key called `123456789`. Register that shorter bucket, switch on logging, and wait for the sky to mail you someone else's traffic. It is beautiful. It almost works. Here's why the universe says no, and what to do with the idea instead.

```bash
# bitflippr emits every single-bit-flip neighbour of a string.
# The ones with a fresh "/" are where a bucket just split into bucket + key.
go install github.com/cybercdh/bitflippr@latest
echo really-popular-bucket-123456789 | bitflippr | grep /
```

It lives at https://github.com/cybercdh/bitflippr

Priya Parity could not sleep. This was not unusual. What was unusual was the reason, which was a hyphen.

She had spent the day on a cloud engagement, the kind where you stare at S3 URLs until they stop looking like words, and somewhere around the third coffee a thought had wandered in, sat down, and refused to leave. It concerned cosmic rays, which is not a phrase you want rattling around your skull at 2am, and it concerned Gerald.

Gerald, for the uninitiated, is a coffee mug. Ceramic, and a veteran. He has witnessed schemes before, in other rooms, belonging to other people, and he has held the coffee through all of them with the professional detachment of a mug who has seen things. Priya had inherited the family trait of explaining her worst ideas to him out loud.

"Gerald," she said, "did you know the sky can flip a bit?"

Gerald, being a mug, said nothing. But he was listening, in the way that mugs do.

**The part where the sky reaches into your computer**

Here is the uncomfortable truth that keeps hardware engineers reaching for the good whisky. Every so often, a high-energy particle, born in some supernova that died before the dinosaurs, finishes a journey across the galaxy by slamming into a transistor in your RAM and knocking a bit from a 1 to a 0. No malware. No exploit. Just physics, arriving uninvited, editing your memory on the way through. Engineers call it a single-event upset. Everyone else calls it "why did that crash, I changed nothing."

This is not a campfire story. In 2003, in the Belgian municipality of Schaerbeek, an electronic voting machine handed a candidate an extra 4,096 votes out of a clear sky. The number is the giveaway. 4,096 is two to the twelfth, exactly one bit, flipped high, in exactly the wrong register. The leading theory, once everyone had stopped panicking, was a cosmic ray. A particle from space had voted, once, very hard.

The gamers have their own gospel. A Super Mario 64 speedrunner, mid-jump, was allegedly yanked upward through the level by a single flipped bit in Mario's height value, saving a fraction of a second that no human input could explain. Nobody has ever reproduced it, which is exactly what you'd expect from a bug you cannot summon, only be visited by. Legend, probably. But a good one.

Priya loved these stories the way other people love ghost stories, which is to say she wanted one of her own.

"So the sky edits memory," she told Gerald. "The question is whether we can be standing underneath when it does."

**The hyphen with ambitions**

Then came the hyphen.

Everything a computer holds is numbers, and every character is a number in a coat. The humble hyphen, `-`, is ASCII 45, or `0x2D` in hex. Its neighbour the forward slash, `/`, is ASCII 47, or `0x2F`. Two apart in the character table, which sounds like a comfortable distance right up until you write both bytes out in binary and line them up.

```
-   0x2D   0 0 1 0 1 1 0 1
/   0x2F   0 0 1 0 1 1 1 1
```

Look at the two rows. Count the differences. There is exactly one, the bit worth 2, sitting quietly in the middle. Flip it, and a hyphen becomes a forward slash. One particle. One bit. One punctuation mark, promoted.

Most of the time this would be a shrug. A slash where a hyphen should be is usually just a typo the size of an atom. But Priya did cloud for a living, and in her world the forward slash is not punctuation. It is architecture.

Because in S3, when you address a bucket the old path-style way, the URL looks like this.

```
https://s3.amazonaws.com/really-popular-bucket-123456789/some/object.json
                         └────────── bucket ───────────┘ └──── key ─────┘
```

The first slash after the host is the border between the bucket's name and the object's key. It is the wall. And Priya had just realised that the wall and the hyphen were one bit apart.

"Gerald," she whispered, "what happens if the sky moves the wall?"

Watch what a single flip does to that URL.

```
before:  s3.amazonaws.com/really-popular-bucket-123456789/some/object.json
after:   s3.amazonaws.com/really-popular-bucket/123456789/some/object.json
```

S3 does not see a mistake. S3 sees a perfectly valid request for a different bucket. The name is now `really-popular-bucket`, and the key is `123456789/some/object.json`. The request has quietly changed address. And if you happen to own a bucket called `really-popular-bucket`, that misdirected request, meant for something enormous and popular, lands in your lap instead.

This is where `bitflippr` came from. Feed it a string and it walks every bit, flips each one, and prints every neighbour that is still legal printable ASCII. It does not even need convincing. Its own front-page example, the word `foo`, quietly produces `f/o` and `fo/`, letters spontaneously becoming slashes.

```
$ bitflippr foo
goo  doo  boo  noo  voo  Foo  &oo
fno  fmo  fko  fgo  fOo  f/o
fon  fom  fok  fog  foO  fo/
```

If a plain `o` will turn into a slash under one flip, a hyphen, which sits even closer, does it gladly. So point it at a bucket name and sieve the output for the ones that grew a wall.

```
$ echo really-popular-bucket-123456789 | bitflippr | grep /
really/popular-bucket-123456789     bucket: really
really-popular/bucket-123456789     bucket: really-popular
really-popular-bucket/123456789     bucket: really-popular-bucket
```

Three new buckets, born from one string, each waiting for a particle to send it some post. Run that across a corpus of the internet's most-hammered bucket names, keep the flips that decompose into a shorter name nobody has claimed yet, register those, switch on access logging, and sit back. Somewhere out there, across trillions of requests, the sky is flipping bits all day long. Surely, Priya thought, some of them are mine.

"We wait for cosmic rays to deliver traffic," she said, "and we log what falls out of the sky."

Gerald steamed gently, which could have meant agreement or could have just been the coffee. It usually is.

**The part where the universe says no**

Priya slept on it. In the morning, sober and caffeinated, she did the thing that separates a good hacker from a good story, which is she tried to break her own idea. It broke.

The first wall is TLS. Yes, the sky flips bits, but if a bit flips inside an encrypted connection while it's crossing the wire, the maths at the other end simply refuses it. TLS carries an integrity check, and a single corrupted bit fails that check and tears the whole connection down rather than delivering the mangled version. So the flip cannot happen in transit and still arrive looking legitimate. It has to happen earlier, in memory, in the string, before the request is sealed for sending. Which is possible. Which brings us to the second wall.

The second wall is ECC. The memory in a phone, a cheap router, a bargain-bin laptop, is often unprotected, and that is exactly the population that made the sky's mischief measurable in the first place. But the machines that actually talk to S3 at scale, and the machines inside S3 that route your request once it arrives, live in data centres, and data centre memory is ECC. Error-correcting. It is built specifically to catch a single flipped bit and put it back before anyone notices. The one weapon Priya's whole scheme depended on is the exact thing the cloud is engineered to disarm. The house does not just win, it installed the felt on the table.

The third wall is the cruellest. You cannot aim a cosmic ray. This is not an exploit. There is no button. You cannot make the particle arrive, cannot choose the machine, cannot choose the bucket, cannot choose the moment. The natural follow-up is whether you could induce a flip yourself, and there is real research here, the Rowhammer family of tricks that hammer memory rows until a neighbour buckles, and Flip Feng Shui, which used it to reach into a co-hosted virtual machine's memory. But none of it puts your hands anywhere near S3's own routing plane, because you do not get to run code there. You are not a tenant of the wall. You are just someone standing in a field, holding a bucket, hoping for weather.

Priya added it up. A flip that survives TLS, in a machine without ECC, that happens to be constructing a path-style request, to a bucket whose name flips into one you were clever enough to have already registered, at a scale where "astronomically rare" finally meets "astronomically many requests" and coughs up a single hit you might not even be able to attribute. It is not impossible. Nothing is impossible. It is a lottery where the sky buys your ticket, on a night of its choosing, and forgets to tell you the draw date.

"It's a cosmic dream, Gerald," she said. "A beautiful one. But you can't heist the weather."

**The grown-up version, which actually pays**

Here is the turn, though, because the idea is not worthless. It is just pointed at the wrong prize.

Strip away the cosmic ray and one solid thing remains standing. Right now, today, deterministically, the bit-flipped sibling of some important bucket is very likely sitting unregistered. And an unregistered name in front of critical infrastructure is not a lottery ticket, it is a door left open. You do not need a particle from space to walk through a door.

This is not a new lesson, it is just a new dialect of an old one. Back in 2011, Artem Dinaburg registered the bit-flipped cousins of popular domains, the DNS version of this exact trick, switched on logging, and over a few months caught tens of thousands of misdirected requests from real machines whose memory really had glitched. And more recently, Aqua's Bucket Monopoly research showed that when AWS services created buckets with predictable names, an attacker could pre-register those names in an unused region and quietly intercept the traffic, which is why AWS now sprinkles random suffixes into them. Same family. Own the name something trusted will one day reach for, and catch what arrives.

So the useful thing `bitflippr` does is not summon rays. It generates the siblings. You bring the corpus of bucket names worth worrying about, you pipe them through `bitflippr`, you grep for the ones that grew a slash, and then you check which of those shorter names nobody has claimed yet. From there the honourable move is the boring one. You tell whoever depends on that bucket that its one-bit shadow is unclaimed, so they can register it themselves and close the door. Defensive registration. Unglamorous. Genuinely useful.

And here is the line you do not cross, the bit the fun version of this post would happily skip. Finding a claimable sibling and reporting it is research. Registering it and logging strangers' traffic is harvesting other people's requests, which can carry tokens and identifiers and things that are none of your business, and serving content back from it is a supply-chain attack with a bow on it. You discover, you disclose, you register defensively, and you stop. The interesting part was always the idea, not the interception.

**The part worth keeping**

Strip the romance off it and a few things stay true. A hyphen and a slash are one bit apart, and in S3 that bit is the wall between a bucket and a key, which quietly makes bucket names load-bearing. The trick only bites path-style addressing, because a virtual-hosted flip breaks the hostname and fails closed, so it dies rather than redirects. The sky genuinely does flip bits, and TLS and ECC are both the reason your provider gets away with shrugging at that and the reason you cannot build anything on it yourself. You cannot aim a cosmic ray, and you cannot Rowhammer your way onto S3's routing plane, so any plan that needs the weather to play along is a daydream and not an exploit. The thing that actually earns is the boring deterministic cousin, the unclaimed bit-flip sibling sitting in front of something important today, waiting to be found and handed back to whoever should have owned it in the first place.

Priya filed the idea under "beautiful, mostly untrue," which is a folder every good researcher keeps and nobody admits to. She registered nothing. She caught no rays. She did, however, ship a tool that finds the doors people left open, which is a more reliable way to get paid than waiting for a dying star to take an interest in your bounty targets.

"Goodnight, Gerald," she said, and tipped the last cold inch of coffee down the sink.

Gerald, exercising the only power a mug truly has, said nothing at all, and kept the warmth a moment longer, which on a night like this was the more useful of the two. Somewhere overhead a particle from the far side of the galaxy was still falling, aimed at nobody, arriving anyway. Priya chose not to think about where it would land. Some bits are better left unflipped.

> "You can't heist the weather. But you can find every door it might one day blow open."
>  -  Priya Parity, filed under dreams, 2am
