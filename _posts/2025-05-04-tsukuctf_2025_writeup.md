---
title: TsukuCTF 2025 writeups
date: 2025-05-04 11:00:00 +0530
categories: [CTF, TsukuCTF 2025]
tags: [ctf, tsukuctf, writeups, cybersecurity, osint, pwn, cryptography]     # TAG names should always be lowercase
description: This post consists of writeups of the challenges CYB3R_BO1 had solved in TsukuCTF 2025.
---

# **TsukuCTF 2025**

## **About TsukuCTF:** 

TsukuCTF is an annual CTF hosted by TaruTaru

This is a CTF with Japanese OSINT as the main genre. There are a few other genres mixed in as well..

- TsukuCTF 2025 will be held online (competition URL: https://tsukuctf.org/ ).
- The duration of the event is 24h00m from 12:00(JST) on 05/03/2025 to 11:59(JST) on 05/04/2025.
- Genres will include OSINT, Web, Pwn, Crypto, etc.
- Maximum number of players per team is 4
- The event was conducted online from Sat, 03 May 2025, 08:30 IST — Sun, 04 May 2025, 08:30 IST.

This is their CTF webpage - [https://tsukuctf.org/](https://tsukuctf.org/)

This is their CTFtime profile - [https://ctftime.org/event/2769/](https://ctftime.org/event/2769/)

## **tsukushi**

### **Welcome**

![Welcome](/assets/img/posts/tsukuCTF2025/Welcome_info.png)

**Translation:**

_Flags are listed in the "announcements" channel of the official TsukuCTF Discord. Flag Format: TsukuCTF25{}_

**Explanation:** 

As the description said the flag is in the anncouncement channel of the discord server. I head over to the discord server, checked out the announcements. I found the flag in the same message they have announced the CTF has started.

I got the flag!

**Flag:** `TsukuCTF25{welcome_to_TsukuCTF_2025!}`

## **OSINT**

### **curve**

![curve](/assets/img/posts/tsukuCTF2025/Curve_info.png)

**Translation:** 

_These are some of the famous places in Japan. Can you spot anything unusual about this photo? The flag is the website domain for this place. Example: TsukuCTF25{example.com}_

**Explanation:** 

After downloading, I used google lens on the image. After looking at few images I understood that the image is of an escalator in Landmark Tower (Yokohama), Japan. Searched for it in google, found it's official website

I got the flag!

**Flag:** `TsukuCTF25{yokohama-landmark.jp}`

### **destroyed**

![destroyed](/assets/img/posts/tsukuCTF2025/Destroyed_info.png)

**Translation:** 

_Identify the school in the photo of this Telegram post. The flag format should be the coordinates of the location rounded to the nearest 4 decimal places and written in the format TsukuCTF25{latitude_longitude} to the nearest 3 decimal places. Example: TsukuCTF25{12.345_123.456}_

_Warning: In the process of solving this problem, you may see direct images related to war._

_23:14 GMT+9 Update: Flag added_

**Explanation:** 

After visiting the telegram channel, looked over the images, checked the description of the post. Found that it is Stepne Community Gymnasium and from the flag emoji it is in Ukraine. So the war must be between Russia and Ukraine. We can see multiple news pages about the gymnasium when we use google lens on all the provided images.

After using google maps and searching over Stepne in Zaporizhzhia Oblast looking for the gymnasium for quite a period of time. Got the coordinated, rounded them.

I got the flag!

**Flag:** `TsukuCTF25{47.797_35.306}`

### **rider**

![rider](/assets/img/posts/tsukuCTF2025/Rider_info.png)

**Translation:**

_Footprints that walk far away and disappear into the evening darkness, The glittering streetlights decorate the city at night, A group of motorbikes pass by on the road nearby, Only the sound of the wind remains, In the light and shadow, I suddenly stop and wonder, Where am I now?_

_The flag format is TsukuCTF25{latitude_longitude} of the location where this person is standing. However, the latitude and longitude are rounded down to the fifth decimal place._

_View Hint: This poem has no meaning.__

**Explanation:** 

After looking at the image and using google lens, found some restaurant named "OTI fried chicken " which is a restaurant chain in Indonesia. 

Using google maps and the restaurant's official webpage, I looked over its branches. After looking at some of the branches came across a branch which has a panda logo on the side of it and the same street light from that in the image. Got the coordinated, rounded them.

I got the flag!

**Flag:** `TsukuCTF25{-7.3189_110.4970}`

### **buildings**

![buildings](/assets/img/posts/tsukuCTF2025/Buildings_info.png)

**Translation:**

_Once that building is built, the sky will probably get narrower again._

_The flag format is TsukuCTF25{latitude_longitude} of the location where this person is standing. Note that the latitude and longitude are rounded down to five decimal places._

**Explanation:** 

Used Google lens on the image. It provided me with two bulding names - Global Front tower and ロイヤルパークス品川 (Royal Parks Shinagawa). They are both from same place that is - Minato City, Tokyo, Japan.

Using Google Maps searched for the places. Found the buildings that resembles those in the image. As the photo is taken on a road, I followed the road and observed the buildings to position them just like how the image looks. I got to the position where the photo was taken as everything matches perfectly. Got the coordinated, rounded them.

I got the flag!

**Flag:** `TsukuCTF25{35.6318_139.7431}`

### **power**

![power](/assets/img/posts/tsukuCTF2025/Power_info.png)

**Translation:**

_I've felt the power._

_The flag format is TsukuCTF25{latitude_longitude} of the location where this person is standing. Note that the latitude and longitude are rounded down to five decimal places._

**Explanation:** 

Looking at the image, it is tactile map of someplace in Japan( as the image contains a japanese character). It contains a braille etched metal plate.

Used Google lens on the image, I searched the image by cropping it various ways after I cropped it full and added "Japan" to the search, looking over the results, I found the image which resembles the photo. The image is from a stockphoto website. After reading the description, it seems that the image is of "Taira no Masakado's Grave"

Using google maps went over to the location, looked for the maps, found them. Got the coordinated, rounded them.

I got the flag!

**Flag:** `TsukuCTF25{35.6872_139.7628}`

## **Crypto**

### **a8tsukuctf**

![a8tsukuctf](/assets/img/posts/tsukuCTF2025/A8tsukuctf_info.png)

**Translation:** 

_I created a suitable KEY and encrypted it, but the tsukuctf part remains the same..._

**Explanation:** 

After checking the encryption python file it seems that this challenge is based on custom autokey Vigenère cipher. Based on the functions and the contents of output.txt, I have written a decrpytion python code:

```python
import string

ciphertext = "ayb wpg uujmz pwom jaaaaaa aa tsukuctf, hj vynj? mml ogyt re ozbiymvrosf bfq nvjwsum mbmm ef ntq gudwy fxdzyqyc, yeh sfypf usyv nl imy kcxbyl ecxvboap, epa 'avb' wxxw unyfnpzklrq."

def f_inv(c, k):
    c = ord(c) - ord('a')
    k = ord(k) - ord('a')
    p = (c - k + 26) % 26
    return chr(ord('a') + p)

def decrypt(ciphertext, known_plaintext_segment, segment_position):
    idx = 0
    plain = []
    cipher_without_symbols = []
    decrypted_key = []

    letters_only = [c for c in ciphertext if c in string.ascii_lowercase]

    for i in range(len(known_plaintext_segment)):
        c = letters_only[segment_position + i]
        p = known_plaintext_segment[i]
        k = (ord(c) - ord(p)) % 26
        k = chr(ord('a') + k)
        decrypted_key.append(k)

    idx = 0
    key = decrypted_key
    for c in ciphertext:
        if c in string.ascii_lowercase:
            if idx < len(key):
                k = key[idx]
            else:
                k = cipher_without_symbols[idx - len(key)]
            p = f_inv(c, k)
            cipher_without_symbols.append(c)
            plain.append(p)
            idx += 1
        else:
            plain.append(c)

    return ''.join(plain), ''.join(decrypted_key)

lower_only = [c for c in ciphertext if c in string.ascii_lowercase]
segment = "tsukuctf"
segment_position = 30  # Based on the assert

plaintext, key = decrypt(ciphertext, segment, segment_position)

print("Recovered key:", key)
print("\nDecrypted plaintext:\n", plaintext)

```
The above code when ran, returns the output:

```txt
Recovered key: annzbwue

Decrypted plaintext:
 alo xok aqjoy this problem or tsukuctf, or both? the flag is concatenate the seventh word in the first sentence, the third word in the second sentence, and 'fun' with underscores.

```
Based on the decrypted plain text, the flag is seventh-word(firstSentence)_third-word(secondSentence)_fun.

I got the flag!

**Flag:** `TsukuCTF25{tsukuctf_is_fun}`

### **PQC0**

![PQC0](/assets/img/posts/tsukuCTF2025/PQC0_info.png)

**Translation:**

_I tried using PQC (Post-Quantum Cryptography)!_

**Explanation:** 

Looking at the given encryption source code and output.txt. The code uses ML-KEM-768 (Kyber768) for key encapsulation and AES for encrypting the flag. 

The provided prob.py script generates an ML-KEM-768 private key: _priv-ml-kem-768.pem_, derives the corresponding public key: _pub-ml-kem-768.pem_. Uses the public key to encapsulate a shared secret, producing: _ciphertext.dat_ and _shared.dat_( the derived shared secret ).

To decrypt it, we need the shared secret.
For that, from the output.txt the created three files:

- Private Key: Save the PEM-formatted private key between the -----BEGIN PRIVATE KEY----- and -----END PRIVATE KEY----- lines into a file named _priv-ml-kem-768.pem_.
- Ciphertext: Convert the hex string under ==== ciphertext(hex) ==== into binary and save it as _ciphertext.dat_.
- Encrypted Flag: Convert the hex string under ==== encrypted_flag(hex) ==== into binary and save it as _encrypted_flag.bin_.

Used OpenSSL to decapsulate the shared secret.

```bash
openssl pkeyutl -decap -inkey priv-ml-kem-768.pem -in ciphertext.dat -secret shared.dat
```

Used this python script to decrypt the flag:

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

with open("shared.dat", "rb") as f:
    shared_secret = f.read()

with open("encrypted_flag.bin", "rb") as f:
    encrypted_flag = f.read()

cipher = AES.new(shared_secret, AES.MODE_ECB)

decrypted_flag = unpad(cipher.decrypt(encrypted_flag), 16)

print(decrypted_flag.decode())
```

I got the flag!

**Flag:** `TsukuCTF25{W3lc0me_t0_PQC_w0r1d!!!}`