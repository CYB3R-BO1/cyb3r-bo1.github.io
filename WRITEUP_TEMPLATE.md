---
title: <EVENT NAME> - Writeup
date: YYYY-MM-DD HH:MM:SS +ZZZZ
categories: [CTF, Others]
tags: [ctf, writeups, cybersecurity, <add relevant categories: web, pwn, crypto, forensics, reverse-engineering, osint, networking>]
description: A writeup for <EVENT NAME> challenges.
---

## TL;DR

* Participated in **<EVENT NAME>**, hosted by **<ORGANIZER>**
* Team rank: **<PLACEMENT> / <TOTAL TEAMS>**
* Personally solved **<N> <categories>** challenges
    * <one line per challenge/category actually written up below, no blanks, no filler>
* Key techniques: <comma-separated list, only techniques actually used>

Plus Ultra.

---

## Event Info

* **CTFtime:** [<link>](<link>)
* **Platform:** [<link>](<link>)

I participated in this CTF during the weekend of **<DATE RANGE>** with my teammates from **<TEAM>**.
Our team placed **<PLACEMENT> out of <TOTAL> teams**.

This post documents the challenges I solved.

---

## <Category, e.g. Web>

### <Challenge Name>

**Description:** <challenge prompt, verbatim or paraphrased>

#### Writeup

**Summary**
- Flag: `<FLAG>`
- Root cause: <the underlying bug/misconfiguration in one line>

**Approach**
1. <recon/first observation>
2. <how the bug was found>
3. <how it was exploited>

**Why it worked**
<the mechanism, why the fix, if any, would have prevented it>

**Proof**
```
<request/response, command output, or script confirming the flag>
```

---

## Closing Thoughts

<a few sentences in your own words: how the CTF felt overall, which challenge was the
highlight and why, anything you'd do differently.>

Not everything landed, though:

* **<category>/<challenge name>** - <how far you got and where you got stuck>
* <repeat per unsolved challenge worth mentioning; omit this list entirely if nothing to report>

<!--
Rules for this template:
- Every challenge listed in the TL;DR must have a matching section below with a Flag.
  If a challenge wasn't fully solved/written up, don't mention it in the TL;DR.
- No placeholder text ("The rev are", "Key techniques: ") - leave the post out of
  _posts/ (keep it as a draft elsewhere) until every claimed section is filled in.
- One flag block, one "why it worked" per challenge - skip the "We got the flag!"
  filler line.
- Date format is always `YYYY-MM-DD HH:MM:SS +ZZZZ` and the front-matter key is
  always `tags:` (plural), never `tag:`.
- Closing Thoughts goes once at the very end of the post (not per-challenge) - a
  short personal reflection plus any challenges attempted but not solved.
- No em dashes (—) anywhere - use a comma, period, or plain hyphen instead.
-->
