---
title: scriptCTF 2026 - Writeup
date: 2026-08-10 18:00:00 +0530
categories: [CTF, Others]
tags: [ctf, writeups, cybersecurity, reverse-engineering, osint, web]
description: A writeup for scriptCTF 2026 challenges.
---

![scriptCTF](/assets/img/posts/scriptCTF-2026/scriptCTF.jpg)

## TL;DR

* Participated in **scriptCTF 2026**, hosted by **ScriptSorcerers**
* Team rank: **55 / 778**
* Personally solved **1 web, 1 reverse and 1 osint** challenges
    * Web/404 Found
    * Reversing/Diabolical
    * Geo-OSINT/Midnight Snack
* Key techniques: robots.txt enumeration, Go binary reverse engineering, ELF section header analysis, `/proc/pid/mem` memory inspection, anti-debug patching, OSINT geolocation

Plus Ultra.

---

## Event Info

* **CTFtime:** [https://ctftime.org/event/3052](https://ctftime.org/event/3052)
* **Platform:** [https://play.scriptsorcerers.xyz/](https://play.scriptsorcerers.xyz/)

I participated in this CTF during the weekend of **8-10 August** with my teammates from **$cr1pt_K1dd13$**.
Our team placed **55 out of 778 teams**.

This post documents the challenges I solved.

---

## Web

### 404 Found

![Challenge Info](/assets/img/posts/scriptCTF-2026/404-Found.png)

#### Writeup

**Summary**
- Flag: `scriptCTF{r0b07s_4r3_t4k1ng_0v3r_e2ccd3fd696e}`
- Root cause: the disallowed endpoint `/the-best-robot` in `/robots.txt`

**Approach**
1. Launched the instance and it is a shopping website.
2. Following the years old tradition, I visited `/robots.txt` and saw a disallowed endpoint `/the-best-robot`
3. I visited the disallowed endpoint, and the flag is there in plain sight

**Why it worked**
The web admins often add endpoints to the robots.txt to disallow bots visiting the endpoint. But humans can easily figure out the disallowed endpoints by visiting robots.txt.

**Proof**
```
GET /robots.txt      -> Disallow: /the-best-robot
GET /the-best-robot  -> scriptCTF{r0b07s_4r3_t4k1ng_0v3r_e2ccd3fd696e}
```

---

## Reverse

### Diabolical

![Challenge Info](/assets/img/posts/scriptCTF-2026/Diabolical.png)

#### Writeup

**Summary**
- Flag: `scriptCTF{n0t_s0_h4rd_4ft3r_4ll}`
- Root cause: the flag was just base64 text tacked onto the end of `.shstrtab`, never loaded, never touched by the binary at all.

**Approach**
1. Loaded `vault` into Ghidra. It's a stripped, statically-linked Go binary run through garble (encrypted strings, scrambled symbol names), and Ghidra's Go analyzer came up empty because the `pclntab` header magic had been patched.
2. Fixed those bytes in Ghidra's hex editor and re-ran Auto Analysis, which recovered enough of the function table to get readable names back on the std-lib crypto calls (AES-GCM, HMAC-SHA256) in the decompiler. From there, cross-checked against a live `strace`/gdb session and read the real key material, nonce, and ciphertext straight out of `/proc/<pid>/mem` while the binary sat waiting for input.
3. Decrypted the "vault" contents offline (GCM tag checked out fine), then read through the decompiled password check. It hashes a 96-byte value and compares it to the hash of an HMAC output, which is always 32 bytes. Those can never match, so the whole "solve the vault" path is a dead end by construction.
4. Once that was confirmed, opened the binary in a hex editor and found the flag base64-encoded, sitting right after the real section names in `.shstrtab`, exactly where `objdump`/`readelf` had been complaining the string table was "corrupt."

**Why it worked**
The whole reversing challenge was a red herring. All that obfuscation and the fake password check exist purely to burn time, while the actual flag was parked outside any loadable segment, somewhere the program itself never looks.

**Proof**
```
$ python3 -c "
d=open('vault','rb').read()
print(d[0x30c000:0x30c0aa])"
b'\x00.text\x00.rodata\x00.gopclntab\x00.typelink\x00.itablink\x00.go.buildinfo\x00.go.fipsinfo\x00.go.module\x00.noptrdata\x00.data\x00.bss\x00.noptrbss\x00.shstrtab\x00c2NyaXB0Q1RGe24wdF9zMF9oNHJkXzRmdDNyXzRsbH0='

$ echo "c2NyaXB0Q1RGe24wdF9zMF9oNHJkXzRmdDNyXzRsbH0=" | base64 -d
scriptCTF{n0t_s0_h4rd_4ft3r_4ll}
```

---

#### Deep dive

The short version above skips a lot of dead ends. Here's roughly how it actually went.

**Getting Ghidra to cooperate.** `vault` is a Go binary run through garble (encrypted strings,
scrambled names), and `strings` confirms it before anything else runs:

```
$ file vault
vault: ELF 64-bit LSB executable, x86-64, statically linked, stripped
```

Loading it into Ghidra, Auto Analysis came back with nothing under `main`, no symbols worth
looking at. The `pclntab` header magic Ghidra's Go loader checks for had been patched to
garbage. Fixing those four bytes in Ghidra's hex editor and re-running Auto Analysis was enough
to get the standard-library crypto calls named again in the decompiler (`aes.NewCipher`,
`cipher.NewGCM`, `hmac.New`, `sha256.Sum256`), even though `main`'s own functions stayed garbled.
That was enough to navigate from.

**The anti-debug check.** Running it under `strace` shows the obvious:

```
$ echo AAAA | strace -f -e trace=openat,ptrace ./vault
openat(AT_FDCWD, "/proc/self/status", ...) = 3
ptrace(PTRACE_TRACEME)                     = -1 EPERM
```

Ghidra's decompiler shows the same thing: check `TracerPid`, try `PTRACE_TRACEME`, return a
bool. A debugger doesn't get blocked, it just silently flips one byte of the AES key later on,
so decryption fails and the program takes a wrong-but-plausible path. Patched the jump in Ghidra
(Patch Instruction) so it always takes the clean branch.

**Cracking the "vault."** With the crypto calls named, the decompiler view of the vault function
reads like AES-GCM decrypt: build a key from a couple of byte slices, decrypt a ciphertext blob
with a nonce, return the plaintext. All of those inputs live in package globals filled at
startup, so instead of untangling garble's encrypted initializer, I just read them straight out
of `/proc/<pid>/mem` while the binary sat waiting at the `key>` prompt. Decrypting offline, the
GCM tag verified, which confirmed the key was right, and produced a 96-byte plaintext.

**Why the check can never pass.** The password check hashes that 96-byte plaintext and compares
it against the hash of an HMAC derived from whatever you type. HMAC-SHA256 output is always 32
bytes. `sha256(96 bytes)` can never equal `sha256(32 bytes)`, so there's no input that opens the
vault, full stop. Forcing the comparison to pass anyway (NOP'ing the jump in Ghidra) just prints
a "gate released" message with no flag behind it, confirming it really was a dead end.

**Finding the actual flag.** `objdump` and `readelf` had been complaining since the very first
run:

```
objdump: vault: string table [13] is corrupt
```

That's `.shstrtab`. Opened the binary in a hex editor and jumped to that section: it's declared
as `0xaa` bytes, but the real section names only use `0x7e` of them. The leftover 44 bytes are
base64:

```
offset 0x30c07e:  c2NyaXB0Q1RGe24wdF9zMF9oNHJkXzRmdDNyXzRsbH0=
                  -> scriptCTF{n0t_s0_h4rd_4ft3r_4ll}
```

**Takeaways:** fix the loader before fighting the binary by hand (one patched byte was the whole
blocker), take toolchain warnings seriously (`objdump` named the exact section from the first
run), and check whether a check is even satisfiable before grinding on it.

---

## Geo-OSINT

### Midnight Snack

![Challenge Info](/assets/img/posts/scriptCTF-2026/Midnight-Snack.png)

![tacobell](/assets/img/posts/scriptCTF-2026/tacobell.jpg)

#### Writeup

**Summary**
- Flag: `scriptCTF{9900_W_Parmer_Ln}`
- Root cause: a half-visible sign in the dark half of the image gave away the cross street.

**Approach**
1. Left half of the image is a Taco Bell drive-thru menu, right half is mostly dark except for some faint silver text that reads "Parmer" plus more I couldn't make out.
2. Reverse image search turned up a similar menu on a Facebook post (FP, no leads in the comments) and the same menu on a Yelp page for a Taco Bell in South Milwaukee, WI (FP, didn't even bother submitting it).
3. Realized "Parmer" is short for Parmer Lane in Austin, TX. Searched Google Maps for "Taco Bell Parmer" and got three hits: 1825 W Parmer Ln, 9900 W Parmer Ln, 1548 E Parmer Ln.
4. Walked all three in street view and found it on the second one, "Parmer Eye Care" next door matches the partial text in the image (TP).

**Why it worked**
The dark half of the photo wasn't actually empty, there was just enough of a neighboring sign visible to narrow "Parmer" down from a random street name to one exact intersection.

**Proof**
```
Facebook (FP): https://www.facebook.com/100072391852009/posts/its-midnight-and-your-at-taco-bell-what-you-getting/1032913352465050/
Yelp (FP):     https://m.yelp.com/biz/taco-bell-south-milwaukee?dd_referrer=https%3A%2F%2Fwww.google.com%2F
Google Maps (TP): "Taco Bell Parmer" -> 9900 W Parmer Ln, Austin, TX (confirmed via "Parmer Eye Care" sign in street view)
```

---

## Closing Thoughts

Fun weekend overall couldn't spend much time on this CTF though. Diabolical was the highlight for me, mostly because I spent way too long convinced the vault was actually crackable before sitting down and doing the math on the hash lengths, should've caught that sooner. 404 Found and Midnight Snack were quicker wins in between. But for real I thought the web challenge is a unintended solve, cause which `robots.txt` challenge worth 328 points even after the CTF ended? lmao.

I didn't get everything though, tried couple challenges made progress but couldn't solve. Here are some:

* **Geo-OSINT/Titan** - had to figure out where the "deepest underwater product photoshoot" happened. Got as far as a Titan watches India shoot at Barracuda Point/rock in Thailand, but couldn't nail down coordinates precise enough to match the flag format before time ran out. But after CTF ended I got to know the coords have been in front of my eyes all the time. 
* **misc/flagchecker67** got close but at the end couldn't solve this and a couple others, just didn't get anywhere with these in time.

After the CTF ended, the orgs dq'ed multiple team (a lot) as the rules say "ai ASSISTANCE was allowed, not ai-automated solving" then in the final scoreboard we stood at 55th position :yay:

Thanks for Reading. Plus Ultra. 🍀