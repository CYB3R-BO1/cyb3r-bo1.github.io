---
title: gaslightCTF 2026 - Writeup
date: 2026-08-19 11:30:00 +0530
categories: [CTF, Others]
tags: [ctf, writeups, cybersecurity, forensics, web, crypto, osint]
description: A writeup for gaslightCTF 2026 challenges.
---

## TL;DR

- Participated in **gaslightCTF 2026**, hosted by **gaslighting**
- Team rank: **7 / 299** in OPEN Division and **17 / 688** in ALL Division.
- Personally solved **5 misc, 4 forensics, 2 web and 1 crypto** challenges
    - Misc: Sanity Check, odyssey, quack, meow, speedy
    - Forensics: good-luck!, blackout, icon-sketch, layered-pages
    - Web: crawl, corridors
    - Crypto: NEWJEANS IS FIVE

- Key techniques: OSINT/reverse image search geolocation, audio spectrogram analysis, PDF hidden-text recovery, EXIF metadata extraction, base64/hex/Atbash decoding, file carving with binwalk and dd, robots.txt enumeration, path-brute-force scripting, GF(2) linear algebra cryptanalysis

Plus Ultra.

---

## Event Info

- **CTFtime:** [https://ctftime.org/event/3181](https://ctftime.org/event/3181)
- **Platform:** [https://play.gaslightctf.cooking/](https://play.gaslightctf.cooking/)

I participated in this CTF during the weekend of **Fri, 14 Aug. 2026 - Mon, 17 Aug. 2026** with my teammates from **O.W.C.A** (Organization Without a Cool Acronym, part of \$cr1pt_K1dd13\$).

My teammates were `themanwiththegreyhat`, `xecho1337`, `am.i.the.monster` (me), and `amnesia21`.

Our team placed **17th out of 688 teams** in **ALL** Division and **7th out of 299** in **OPEN** Division.

This post documents the challenges I solved.

---

## Misc

### Sanity Check (100)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/sanity-check.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{w3lc0me_2_g4sl1ghtCTF!}`
- Root cause: the flag is printed directly on the challenge webpage, no digging required

**Proof**

![Sanity Check PoC](/assets/img/posts/gaslightCTF-2026/sanity-check-poc.png)

### odyssey (354)

![challenge info](/assets/img/posts/gaslightCTF-2026/odyssey.png)

![challenge image](/assets/img/posts/gaslightCTF-2026/02.jpg)

#### Writeup

**Summary**
- Flag: `gaslightCTF{37.973,23.725}` (there may be a ±0.001 tolerance)
- Key clue: a historic landmark and a distinctive elevator/house combo in the image were enough to pin down the exact spot

**Approach**
1. Reverse image search identified the historic landmark in the background as the Parthenon on the Acropolis of Athens.
2. Walking the area in Google Maps street view and matching the elevator and the iconic house from the image against the surroundings pinned down the exact location.

**Why it worked**
This one was a matter of combining two clues from the same scene: the landmark in the background and the distinctive building/elevator details in the foreground. Greece has plenty of ruins, but once both matched the Acropolis area, the location was narrowed down to a single corner pretty quickly.

**Proof**

![odyssey-poc](/assets/img/posts/gaslightCTF-2026/odyssey-poc.png)

### quack (436)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/quack.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{52.195,0.114}`
- Key clue: a specific set of public toilets in the image was already indexed online with coordinates attached

**Approach**
1. Given an image, I focused on three details: the public toilets, the river, and the bridge.
2. Reverse image search on the toilets turned up several photos of the same location.
3. One result, [Public Toilets, Lammas Land](https://www.geograph.org.uk/photo/5331751), showed the exact same toilets and had coordinates attached to it.

**Why it worked**
The toilets themselves weren’t special, but this exact set had already been photographed and geotagged elsewhere online. Once that photo showed up, the location was basically handed to me.

**Proof**

![quack poc](/assets/img/posts/gaslightCTF-2026/quack-poc.png)

### meow (443)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/meow.png)
![Challenge Image](/assets/img/posts/gaslightCTF-2026/IMG_4731.jpeg)

#### Writeup

**Summary**
- Flag: `gaslightCTF{37.447,25.327}`
- Key clue: a seafood market's signage visible in the background gave away the location

**Approach**
1. Three things stood out in the given image: the van, the bench/table, and the cat.
2. The van had Greek text painted on it, so the shot had to be somewhere in Greece. Reverse image search on the van and the cat didn't lead anywhere.
3. Reverse image search on the bench turned up `Ψαραγορά Μυκόνου`, a seafood market in Greece, which pinned down the location.

**Why it worked**
The van and the cat were just distractions. The real clue was the seafood market sign behind the bench, which was distinctive enough to pull up the exact location in Greece.

**Proof**

![meow-poc](/assets/img/posts/gaslightCTF-2026/meow-poc.png)

### speedy (460)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/speedy.png)
![Challenge Image](/assets/img/posts/gaslightCTF-2026/IMG_5441.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{22.279,114.181}` (there may be a ±0.001 tolerance)
- Key clue: the image matched a frame from IShowSpeed's own broadcast footage

**Approach**
1. Reverse image search identified the person in frame as streamer IShowSpeed, wearing the same red jersey he wore during a stream filmed in Hong Kong.
2. Found the full 9-hour stream on YouTube. The image showed a security guard and a flyover in the background, which suggested it was from early in the stream, so I skipped ahead instead of watching the whole thing.
3. About 5 minutes into the stream near the flyover, the footage passed by Times Square in Hong Kong, right by the bridge shown in the challenge image, which gave the coordinates.

    ![speedy-poc](/assets/img/posts/gaslightCTF-2026/speedy-poc.png)

4. The green building and the others in the background of the challenge image line up with this spot, just in front of the flyover, confirming the match.

**Why it worked**
This one basically turned into a search through a 9-hour stream. Once I matched the jersey and the flyover in the background, the location was already there in the footage; it was just a matter of finding the right minute.


## Forensics

### good-luck! (100)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/good-luck.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{c4n_u_s33_me?_u4ya}`
- Root cause: the flag was drawn directly into the audio's frequency data, visible only as a spectrogram

**Approach**
1. My usual checklist for an audio forensics challenge is to check the metadata, listen to the file, and then look at the spectrogram. If none of that turns up anything, there are a few other tricks to fall back on.
2. Metadata and a straight listen-through didn't reveal anything.
3. I loaded the audio into Audacity and switched to spectrogram view. The flag was right there, drawn into the frequencies.

**Why it worked**
It sounded like noise at first, because the flag wasn’t being encoded in a way that could be heard. Once I switched to a spectrogram, it was just text drawn into the frequency data, which made the solve straightforward.

**Proof**

![Challenge poc](/assets/img/posts/gaslightCTF-2026/good-luck-poc.png)

### blackout (251)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/blackout.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{c0w4bung4_f1le_4ev3r}`
- Root cause: the PDF's text layer was still intact underneath the black redaction boxes

**Approach**
1. I was given a PDF with every page fully covered in black redaction boxes. Metadata didn't have anything useful.
2. Some PDF redactions only paint over the text visually and leave the actual text layer selectable underneath, so I tried Ctrl+A to select everything on the page. The first line selected read "Grep me - flag", and the rest was filler/repetitive text.
3. Searching (Ctrl+F) for the flag format `gaslightCTF` inside that selected text surfaced the flag directly.
4. So the trick is just: Ctrl+A to grab the hidden text layer, then Ctrl+F for the flag format.

**Why it worked**
The black boxes were only hiding the text visually. The actual content stream was still there underneath, so selecting all the text and searching for the flag pattern was enough to recover it.

**Proof**

![Challenge PoC](/assets/img/posts/gaslightCTF-2026/blackout-poc.png)

### icon-sketch (339)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/icon-sketch.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{i5_th4t_supp0s3d_2b_p1ss?}`
- Root cause: the flag was split across three EXIF metadata fields, each layered with a different encoding

**Approach**
1. I was given an image containing the gaslightCTF banner. I checked the metadata and found three base64 strings tucked into the EXIF fields:
    - Document Name: `dGhlIHBlZSBwZW9wbGUgc2FpZCB0byBkZWNvZGUgYW5kIHB1dCB0aGUgdGl0bGVzIHRvZ2V0aGVy`
    - Artwork Title: `MmUgMmUgMmUgMmUgMmUgMmUgMmUgMmUgNDMgNTQgNDYgMmUgMmUgMmUgNWYgMmUgNjggMzQgMmUgNWYgMmUgMmUgNzAgNzAgMzAgNzMgMzMgMmUgMmUgMzIgNjIgNWYgMmUgMzEgNzMgMmUgMmUgN2Q=`
    - Title: `dHpob3J0c2cuLi57cjUuZy4uZy5oZi4uLi4ud18uLi5rLi5oPy4=`
2. Decoding all three from base64 gives:
    - "the pee people said to decode and put the titles together"
    - `2e 2e 2e 2e 2e 2e 2e 2e 43 54 46 2e 2e 2e 5f 2e 68 34 2e 5f 2e 2e 70 70 30 73 33 2e 2e 32 62 5f 2e 31 73 2e 2e 7d`
    - `tzhortsg...{r5.g..g.hf.....w_...k..h?.`
3. So the "Document Name" is basically the instruction: decode the other two fields and stitch them together for the flag. Those two fields are "Artwork Title" and "Title".
4. "Artwork Title" decodes from base64 into a hex string, which decodes again into `........CTF..._.h4._..pp0s3..2b_.1s..}`. Since the flag format is already known to be `gaslightCTF{...}`, no further decoding was needed, this is the second half of the flag.
5. "Title" decodes to `tzhortsg...{r5.g..g.hf.....w_...k..h?.`. Lining up the first 8 characters against the expected `gaslight` prefix (`t<->g`, `z<->a`, and so on) is the signature of an Atbash cipher. Running it through Atbash gives `gaslight...{i5.t..t.su.....d_...p..s?.`, the first half of the flag.
6. Merging the two halves gives the complete flag.

**Why it worked**
This one was mostly about following the clue the file gave me. The first EXIF field told me exactly how to interpret the other two, and once I decoded them and recognized the Atbash pattern, the flag fell into place.

### layered-pages (384)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/layered-pages.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{c4rv3_1t_0ut}`
- Root cause: 9 separate JPEG/PNG images were concatenated into a single file, and the flag was spread across the hidden layers

**Approach**
1. Given an image with the text "gas" on it. Metadata didn't have much, but running `binwalk` against the file turned up 9 embedded image signatures (JPEG/PNG) stacked inside the single file.
    ![pages-poc-1](/assets/img/posts/gaslightCTF-2026/pages-poc-1.png)
2. Binwalk's built-in extraction pulled out some extra junk alongside the actual images, so instead I carved each layer out manually with `dd`, using the offsets binwalk reported:

```bash
dd if=123456789.jpg of=layer1.jpg bs=1 skip=$((16#00000)) count=$((16#055F7-16#00000)) status=none
dd if=123456789.jpg of=layer2.jpg bs=1 skip=$((16#055F7)) count=$((16#09891-16#055F7)) status=none
dd if=123456789.jpg of=layer3.jpg bs=1 skip=$((16#09891)) count=$((16#0E05E-16#09891)) status=none
dd if=123456789.jpg of=layer4.jpg bs=1 skip=$((16#0E05E)) count=$((16#126B4-16#0E05E)) status=none
dd if=123456789.jpg of=layer5.png bs=1 skip=$((16#126B4)) count=$((16#17D93-16#126B4)) status=none
dd if=123456789.jpg of=layer6.jpg bs=1 skip=$((16#17D93)) count=$((16#1B6A4-16#17D93)) status=none
dd if=123456789.jpg of=layer7.png bs=1 skip=$((16#1B6A4)) count=$((16#1FD12-16#1B6A4)) status=none
dd if=123456789.jpg of=layer8.jpg bs=1 skip=$((16#1FD12)) count=$((16#2520F-16#1FD12)) status=none
dd if=123456789.jpg of=layer9.png bs=1 skip=$((16#2520F)) status=none
```

3. Running the script splits the file into its 9 layers, and the flag was spread across them.

    ![pages-poc-2](/assets/img/posts/gaslightCTF-2026/pages-poc-2.png)

**Why it worked**
The file was just several valid images stitched together. Once I found the offsets, each layer could be carved out separately and the flag was spread across them in plain sight.

## Web

### crawl (416)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/crawl.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{LLM_1nduc3d_4r4chn0ph0b1a_fc4c22d85154}`
- Root cause: a sensitive endpoint was disclosed through a `Disallow` rule in `robots.txt`

**Approach**
1. Opened the website and checked `/robots.txt`, as usual.
    ![robots.txt endpoint](/assets/img/posts/gaslightCTF-2026/crawl-poc-1.png)
2. It had a disallowed endpoint listed.
    ![disallowed endpoint, part 1](/assets/img/posts/gaslightCTF-2026/crawl-poc-2.png)    
3. Visiting that endpoint led to another page which pointed at yet another endpoint, `/_flag`.
    ![disallowed endpoint, part 2](/assets/img/posts/gaslightCTF-2026/crawl-poc-3.png)
4. Upon visiting the newly found endpoint: 
    ![flag endpoint](/assets/img/posts/gaslightCTF-2026/crawl-poc-4.png)
5. That gave the flag. Surprisingly easy for 416 points, I'm guessing the AI-assisted solves didn't think to check robots.txt either. The page also had an LLM system prompt baked into it, apparently there to catch anyone throwing an LLM at the challenge.

**Why it worked**
This was a classic "check robots.txt first" challenge. The path was listed as disallowed to crawlers, but that doesn’t stop a person from opening it directly, and that was enough to get the flag.

### corridors (458)

![Challenge Info](/assets/img/posts/gaslightCTF-2026/corridors.png)

#### Writeup

**Summary**
- Flag: `gaslightCTF{fr33d0m_4t_l4st_9d11a422952f}`
- Root cause: the correct path through a binary `l`/`r` maze encoded the flag itself, in binary

**Approach**
1. The site gives two links at each step, `l` and `r` (left and right), like doors in a corridor. Pick the right one and the page prints "correct", pick the wrong one and it prints "nope". Hitting "nope" just means backing up and trying the other option.
2. So the intended solve is to keep following the "correct" path all the way down.
3. Doing that by hand wasn't realistic with what turned out to be hundreds of levels, so I wrote a script to automate it: try `l`, fall back to `r` if `l` comes back "nope", and stop as soon as the page shows something other than "correct" or "nope".

    ```python
    import re
    import requests

    BASE_URL = "https://...play.gaslightctf.cooking:1337/"
    session = requests.Session()

    def get_h1(url):
        r = session.get(url, timeout=15)
        r.raise_for_status()
        m = re.search(r"<h1[^>]*>(.*?)</h1>", r.text, re.I | re.S)
        return m.group(1).strip() if m else None, r.text

    moves = []
    while True:
        found = False
        for choice in ("l", "r"):
            url = BASE_URL + "".join(m + "/" for m in moves + [choice])
            h1, body = get_h1(url)

            if h1 == "correct":
                moves.append(choice)
                found = True
                break
            if h1 == "nope":
                continue

            print("flag page:", url)
            print(body)
            raise SystemExit

        if not found:
            print("dead end at", "/".join(moves))
            raise SystemExit
    ```

4. Running it walked all the way down and returned the full correct path:

    ```plaintext
    /l/r/r/l/l/r/r/r/l/r/r/l/l/l/l/r/l/r/r/r/l/l/r/r/l/r/r/l/r/r/l/l/l/r/r/l/r/l/l/r/l/r/r/l/l/r/r/r/l/r/r/l/r/l/l/l/l/r/r/r/l/r/l/l/l/r/l/l/l/l/r/r/l/r/l/r/l/r/l/l/l/r/l/l/l/r/r/l/l/r/r/r/r/l/r/r/l/r/r/l/l/r/r/l/l/r/r/r/l/l/r/l/l/l/r/r/l/l/r/r/l/l/r/r/l/l/r/r/l/r/r/l/l/r/l/l/l/l/r/r/l/l/l/l/l/r/r/l/r/r/l/r/l/r/l/r/r/r/r/r/l/l/r/r/l/r/l/l/l/r/r/r/l/r/l/l/l/r/l/r/r/r/r/r/l/r/r/l/r/r/l/l/l/l/r/r/l/r/l/l/l/r/r/r/l/l/r/r/l/r/r/r/l/r/l/l/l/r/l/r/r/r/r/r/l/l/r/r/r/l/l/r/l/r/r/l/l/r/l/l/l/l/r/r/l/l/l/r/l/l/r/r/l/l/l/r/l/r/r/l/l/l/l/r/l/l/r/r/l/r/l/l/l/l/r/r/l/l/r/l/l/l/r/r/l/l/r/l/l/l/r/r/r/l/l/r/l/l/r/r/l/r/l/r/l/l/r/r/l/l/r/l/l/r/r/l/l/r/r/l/l/r/r/r/r/r/l/r/
    ```

5. Opening that final URL just showed the text "freedom" and an unrelated stock image, a dead end on its own.
6. That made me look at the path itself instead of the page content: mapping `l` to `0` and `r` to `1` and stripping the slashes turns the whole path into one long binary string.

    ```plaintext
    0110011101100001011100110110110001101001011001110110100001110100010000110101010001000110011110110110011001110010001100110011001101100100001100000110110101011111001101000111010001011111011011000011010001110011011101000101111100111001011001000011000100110001011000010011010000110010001100100011100100110101001100100110011001111101
    ```

7. Decoding that binary string straight to ASCII gives the flag.

**Why it worked**
The visible page text was basically a red herring. The real data was hidden in the path itself: the sequence of left/right choices encoded the binary string, and once I converted that back to ASCII, the flag was there.

**Proof**

![corridors poc](/assets/img/posts/gaslightCTF-2026/corridors-poc.png)

## Crypto

### NEWJEANS IS FIVE (100)

![Challenge info](/assets/img/posts/gaslightCTF-2026/nj-is-five.png)


#### Writeup

**Summary**
- Flag: `gaslightCTF{newj34ns-nv-d!es}`
- Root cause: `SubBytes()`/`SubWord()` are identity functions, so the AES-like cipher collapses to an affine function of the 128-bit key, solvable with linear algebra over GF(2)

**Approach**
1. I read through `chall.py`. It encrypts a known plaintext ("incomprehensible") and the flag with an AES-128-like construction, and both ciphertexts get dumped to `output.txt`. Comparing it against real AES, `SubBytes()` and `SubWord()` just return their input unchanged, everything else (`ShiftRows`, `MixColumns`, `AddRoundKey`, key schedule) is untouched.
2. That removes the only non-linear piece of AES. XOR, ShiftRows, MixColumns, and the key expansion are all linear over GF(2), so with the fixed Rcon constants folded in, the whole cipher collapses to an affine function of the key for a fixed plaintext: $C(K) = A·K \oplus b$. So instead of attacking AES, I just had to recover this affine map.
3. I pulled `chall.py`'s own functions (`KeyExpansion`, `GenerateRoundKeys`, `ShiftRows`, `MixColumns`, `GMul`, etc.) into my script so the forward direction matched the challenge exactly, then:
    - Encrypted the known plaintext under the all-zero key to get `b`.
    - Encrypted the same plaintext under each of the 128 single-bit keys, XORing each result with `b` to build the 128 columns of `A`.
    - Solved `A·K = C_real ⊕ b` over GF(2) with Gaussian elimination to recover the master key.
4. Regenerated the round keys from the recovered key and ran the cipher backwards on the second ciphertext to get the flag back out.

**Why it worked**
The trick is that the challenge isn’t really AES anymore. Once I compared the implementation to real AES, the substitution step was just `return word` and `return state`, so the non-linear piece was gone entirely.

That leaves only XOR, shifts, mix columns, and the key schedule, all of which are linear over GF(2). Once that clicked, the whole thing collapsed into a system of the form $C(K) = A·K \oplus b$, and recovering the key was just Gaussian elimination instead of trying to break AES itself.

```python
from pwn import xor

Rcon = [
    "01000000", "02000000", "04000000", "08000000",
    "10000000", "20000000", "40000000", "80000000",
    "1b000000", "36000000",
]


def SubWord(word):
    return word


def RotWord(word):
    return word[2:] + word[:2]


def KeyExpansion(key):
    w = [key[i:i+8] for i in range(0, 32, 8)]
    w += [0] * 40
    for i in range(4, 44):
        temp = w[i-1]
        if i % 4 == 0:
            temp = xor(bytes.fromhex(SubWord(RotWord(temp))), bytes.fromhex(Rcon[(i//4)-1])).hex()
        w[i] = xor(bytes.fromhex(w[i-4]), bytes.fromhex(temp)).hex()
    return w


def GenerateRoundKeys(w):
    return ["".join(w[4*r:4*(r+1)]) for r in range(11)]


def ToMatrix(s):
    m = [[0] * 4 for _ in range(4)]
    for i in range(0, 32, 2):
        row = (i // 2) % 4
        col = (i // 2) // 4
        m[row][col] = s[i:i+2]
    return m


def FromMatrix(m):
    return "".join(m[i][j] for j in range(4) for i in range(4))


def AddRoundKey(state, rk):
    return ToMatrix(xor(bytes.fromhex(FromMatrix(state)), bytes.fromhex(rk)).hex())


def ShiftRows(state):
    state[1] = state[1][1:] + state[1][:1]
    state[2] = state[2][2:] + state[2][:2]
    state[3] = state[3][3:] + state[3][:3]
    return state


def GMul(a, b):
    p = 0
    for _ in range(8):
        if b & 1:
            p ^= a
        a <<= 1
        if a & 0x100:
            a ^= 0x11b
        b >>= 1
    return p


def MixColumns(state):
    out = [[0] * 4 for _ in range(4)]
    for col in range(4):
        vals = [int(state[r][col], 16) for r in range(4)]
        out[0][col] = hex(GMul(0x02, vals[0]) ^ GMul(0x03, vals[1]) ^ vals[2] ^ vals[3])[2:].zfill(2)
        out[1][col] = hex(vals[0] ^ GMul(0x02, vals[1]) ^ GMul(0x03, vals[2]) ^ vals[3])[2:].zfill(2)
        out[2][col] = hex(vals[0] ^ vals[1] ^ GMul(0x02, vals[2]) ^ GMul(0x03, vals[3]))[2:].zfill(2)
        out[3][col] = hex(GMul(0x03, vals[0]) ^ vals[1] ^ vals[2] ^ GMul(0x02, vals[3]))[2:].zfill(2)
    return out


def AES_128(x, round_keys):
    state = ToMatrix(x)
    state = AddRoundKey(state, round_keys[0])
    for r in range(1, 10):
        state = ShiftRows(state)
        state = MixColumns(state)
        state = AddRoundKey(state, round_keys[r])
    state = ShiftRows(state)
    return FromMatrix(state)


def recover_key(plaintext, ciphertext):
    zero_key = "00" * 16
    base = bytes.fromhex(AES_128(plaintext, GenerateRoundKeys(KeyExpansion(zero_key))))
    target = bytes.fromhex(ciphertext)
    rhs = int.from_bytes(xor(target, base), "big")

    columns = []
    for bit in range(128):
        key = (1 << (127 - bit)).to_bytes(16, "big").hex()
        diff = xor(bytes.fromhex(AES_128(plaintext, GenerateRoundKeys(KeyExpansion(key)))), base)
        columns.append(int.from_bytes(diff, "big"))

    rows = []
    for out_bit in range(128):
        row = 0
        for in_bit in range(128):
            if (columns[in_bit] >> (127 - out_bit)) & 1:
                row |= 1 << (128 - in_bit)
        if (rhs >> (127 - out_bit)) & 1:
            row |= 1
        rows.append(row)

    # Gaussian elimination over GF(2)
    pivot = 0
    for col in range(128):
        if pivot >= 128:
            break
        r = next((i for i in range(pivot, 128) if (rows[i] >> (128 - col)) & 1), None)
        if r is None:
            continue
        rows[pivot], rows[r] = rows[r], rows[pivot]
        for i in range(128):
            if i != pivot and ((rows[i] >> (128 - col)) & 1):
                rows[i] ^= rows[pivot]
        pivot += 1

    key_int = 0
    for r in range(128):
        if rows[r] & 1:
            key_int |= 1 << (127 - r)

    return key_int.to_bytes(16, "big").hex()
```


**Proof**
```
$ python3 decrypt.py 
[+] Master key: d58d0b337eecb2e23c11436c8b2dcde2 
[+] Flag: newj34ns-nv-d!es
```

---

## Closing Thoughts

Great weekend, landing 17th out of 688 teams. NEWJEANS IS FIVE was the highlight for me. It looked like a plain AES-128-ECB black box at first, right up until diffing `chall.py` against real AES showed `SubBytes`/`SubWord` doing nothing at all. Once that clicked, it stopped being a crypto challenge and turned into a 128x128 linear algebra problem over GF(2), which was a fun "wait, that's actually it?" kind of solve. The OSINT set (odyssey, quack, meow, speedy) were easy for me, so I was able to move through those fairly quickly.

It was a fun CTF, especially because using AI was against the rules, and several teams were caught and banned for automation. We stayed near the top of the leaderboard the whole time because we played by the rules, and the hard work of everyone on the team helped us reach this position. This was my personal second-best rank, and my first time finishing this high in an international CTF.

Thanks for reading. Plus Ultra. 🍀