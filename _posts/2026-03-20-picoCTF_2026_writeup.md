---
title: picoCTF 2026 - Writeup
date: 2026-03-19 21:30:00 +0530
categories: [CTF]
tags: [reverse-engineering, pwn, cryptography, forensics, web, networking, picoctf]
description: A writeup for picoCTF 2026 challenges.
---

## Summary

Compact writeups for picoCTF 2026 challenges. More entries will be added in the same format.

## Writeups

### Web

#### no-fa

- Description: "Seems like some data has been leaked! Can you get the flag?"
- Hint: rockyou and 2FA safety.
- Root cause: unsalted SHA-256 password storage plus OTP exposed in readable Flask client session.

Exploit flow:

1. Extract `admin` SHA-256 hash from `users.db`.
2. Crack the hash with a wordlist (for example, rockyou) to recover `apple@123`.
3. Log in as `admin` and capture the Flask `session` cookie.
4. Decode cookie payload, read `otp_secret`, and submit it to `/two_fa`.
5. Access `/` and retrieve the flag.

Flag:

```text
picoCTF{n0_r4t3_n0_4uth_41b9d45a}
```

Takeaway: 2FA fails if OTP material is client-readable, and unsalted hashes make account takeover easier.


#### ORDER ORDER

- Description: "Can you try to get the flag from our website. I've prepared my queries everywhere! I think!"
- Hint: "What does order in SQL Injection mean?"
- Root cause: second-order SQL injection via stored username reused during report generation.

Exploit flow:

1. Register with an injected username and log in.
2. Trigger report generation.
3. Open the newest CSV report from inbox.
4. Use UNION payloads to enumerate tables and extract the flag value.

Payloads used:

```sql
probe' UNION SELECT sqlite_version(),2,3-- -
probe' UNION SELECT group_concat(name,'|'),2,3 FROM sqlite_master WHERE type='table'-- -
x' UNION SELECT sql,2,3 FROM sqlite_master WHERE name='aDNyM19uMF9mMTRn'-- -
x' UNION SELECT value,2,3 FROM aDNyM19uMF9mMTRn WHERE name='flag'-- -
```

Flag:

```text
picoCTF{s3c0nd_0rd3r_1t_1s_3ad6ac82}
```

Takeaway: storing input safely is not enough if it is later concatenated into SQL.

#### sql-map1

- Description: authenticated search endpoint vulnerable to SQL injection.
- Hints: search box, MD5 passwords, SQLMap/manual SQLi.
- Root cause: unsafely interpolated search parameter allowed UNION extraction of credential hashes.

Exploit flow:

1. Register and log in to reach vulnerable search endpoint.
2. Confirm column count and perform UNION-based extraction.
3. Dump `users` table hashes and crack MD5 offline.
4. Log in with recovered credentials and retrieve flag.

Flag:

```text
picoCTF{F0uNd_s3cr3T_K3y_f0R_w3_<>}
```

Takeaway: parameterized queries and modern password hashing prevent this full attack chain.

### Reverse Engineering

#### binary-instrumentation-3

- Description: the executable should write the flag, but the runtime flow is broken.
- Hint: Frida is a great starting point.
- Root cause: packed PE payload with Base64 flag fragments initialized in static constructors.

Exploit flow:

1. Inspect PE sections and identify `.ATOM` as packed data.
2. Decompress `.ATOM` as LZMA (alone format).
3. Analyze the unpacked payload and static initializers.
4. Concatenate and decode the extracted Base64 fragments.

Flag:

```text
picoCTF{411_4r3_4p15_n07h1n9_3l53_4f70640e}
```

Takeaway: static unpacking is often faster than dynamic instrumentation when runtime setup is unstable.

#### binary-instrumentation-4

- Description: recover the flag from a staged PE that sends data after a key check.
- Hints: Frida is a great starting point; compare APIs too.
- Root cause: packed payload stores Base64 flag parts and checks a hardcoded key through compare logic.

Exploit flow:

1. Identify that `bin-ins.exe` is a loader with a large `.ATOM` section.
2. Decompress `.ATOM` and reverse the unpacked payload.
3. Follow key-check logic (`lstrcmpA`) and success path.
4. Reconstruct and decode the Base64 fragments from static initializers.

Flag:

```text
picoCTF{n3tw0rk_1s_4P1s_4S_W311_6ae41cdc}
```

Takeaway: loader stubs often hide simple logic in second-stage binaries.

#### bypass-me

- Description: password checker with misleading sanitization.
- Root cause: encoded password is XOR-decoded at runtime, while sanitized input is not used in the final comparison.

Exploit flow:

1. Enumerate symbols (`decode_password`, `sanitize`, `main`) from the unstripped ELF.
2. Recover encoded bytes from `decode_password` and XOR each byte with `0xaa`.
3. Obtain `SuperSecure` and provide it to the binary.

Flag:

```text
picoCTF{d3bugg3r_p0w3r_is_4w3s0m3_9d5f0f68}
```

Takeaway: debugging symbols and fake helper functions are common reverse-engineering misdirection.

#### secure-password-database

- Description: authenticate by submitting a hash value.
- Hint: understand the hashing algorithm.
- Root cause: deterministic secret generation (`XOR 0xAA`) plus predictable DJB2-style hash verification.

Exploit flow:

1. Reverse `hash` and identify `h = h * 33 + c` with seed `0x1505`.
2. Reverse `make_secret` and decode obfuscated bytes via XOR `0xAA`.
3. Compute required numeric hash and submit it.

Flag:

```text
picoCTF{d0nt_trust_us3rs}
```

Takeaway: custom auth primitives fail when secrets and transforms are statically reversible.

### Binary Exploitation

#### heap-havoc

- Description: heap overflow in a struct-based challenge with function pointers.
- Root cause: two `strcpy` calls into 8-byte heap buffers allow overwrite of adjacent struct fields.

Exploit flow:

1. Overflow `i1->name` into `i2` fields.
2. Set `i2->name` to writable `.bss` memory.
3. Set `i2->callback` to `winner()`.
4. Let the program invoke the overwritten callback.

Flag:

```text
picoCTF{h34p_0v3rfl0w_f810c23a}
```

Takeaway: function-pointer overwrites often require preserving intermediate pointers to avoid pre-trigger crashes.

#### offset-cycle

- Description: timed ret2win from a generated binary after running `./start`.
- Root cause: unsafe `gets()` in `vuln()` with non-PIE binary and reachable `win()`.

Exploit flow:

1. Run `./start` and inspect generated source.
2. Compute offset to saved return address (`67` bytes in solved instance).
3. Overwrite return address with `win()`.

Flag:

```text
picoCTF{u_Us3d_pwNt00L5_18428ce4}
```

Takeaway: in timed generators, fast triage and payload automation matter more than exploit complexity.

#### offset-cycleV2

- Description: generated binary with stack overflow and a predictable canary check.
- Hint: guessing the canary is easy.
- Root cause: canary derives from the beginning of the flag, making it predictable (`pico`).

Exploit flow:

1. Run `./start` and parse current source/binary pair.
2. Extract `BUFSIZE`, return offset, and `win()` address.
3. Craft payload: buffer fill + `pico` canary + padding + `win()` return address.
4. Send before timeout.

Flag:

```text
picoCTF{Y0U_AGa1n_Us3d_pwNt00L5_45fb3e0b}
```

Takeaway: canaries are ineffective when derived from predictable data.

### Cryptography

#### Secure Dot Product

- Challenge idea: oracle returns dot product of chosen vector with secret AES key bytes.
- Root cause: raw SHA-512 prefix MAC (`SHA512(secret || message)`) enables length extension.
- Extra weakness: parser strips non-digit characters, helping forged extended inputs remain valid.

Exploit outline:

1. Collect trusted vector/hash pairs from the service.
2. Forge extended payloads with SHA-512 length extension.
3. Query with controlled suffix values to isolate key-byte contributions.
4. Recover most bytes directly; solve remaining bytes with linear equations (for example with Z3).
5. Rebuild AES key and decrypt ciphertext.

Flag:

```text
picoCTF{n0t_so_s3cure_.x_w1th_sh@512_1bb6154f}
```

Takeaway: use HMAC for message authentication, never raw hash prefix constructions.

### Forensics

#### disko-4

- Description: recover a deleted flag file from a FAT32 image.
- Hint: check deleted files.
- Root cause: deleted directory entry remained recoverable.

Exploit flow:

1. Confirm filesystem type with `file` and `fsstat`.
2. List deleted entries using `fls -r -d`.
3. Recover `dont-delete.gz` with `icat` and decompress it.

Flag:

```text
picoCTF{d3l_d0n7_h1d3_w3ll_4fed4369}
```

Takeaway: on FAT filesystems, deleted artifacts can remain recoverable even when other entries are corrupted.

#### git-2

- Description: recover data from a partially damaged Git repository.
- Hint: object files were likely untouched.
- Root cause: refs metadata was damaged, but `.git/objects` still contained full history.

Exploit flow:

1. Locate recovered `.git` metadata.
2. Recreate minimal refs structure (`refs/heads`, `refs/tags`, `branches`).
3. Enumerate and inspect objects, then restore branch pointer.
4. Read deleted content from an earlier commit.

Flag:

```text
picoCTF{g17_r35cu3_16ac6bf3}
```

Takeaway: if object storage survives, Git history is usually recoverable with minimal ref repair.

#### Rogue Tower

- Description: identify rogue cell activity and recover exfiltrated data from PCAP.
- Hints: UDP/55000 broadcasts, HTTP User-Agent IMSI, key derived from victim IMSI.
- Root cause: exfil data split across POST chunks and weak XOR scheme.

Exploit flow:

1. Identify rogue broadcast (`PLMN=00101`, `CELLID=91043`).
2. Match victim via HTTP User-Agent fields (`IMSI`, `CELL`).
3. Reassemble POST data chunks.
4. Base64-decode and XOR with IMSI-derived key.

Flag:

```text
picoCTF{r0gu3_c3ll_t0w3r_7a06fd7c}
```

Takeaway: timeline correlation plus weak exfil encryption quickly exposes attacker workflow.

#### timeline-1

- Description: find hidden data by building and filtering an ext4 MAC timeline.
- Hints: recent activity, anti-forensics behavior, and `macb` filtering.
- Root cause: suspicious inode with synchronized timestamps revealed encoded payload.

Exploit flow:

1. Build timeline with `fls` + `mactime`.
2. Pivot on recent suspicious entries and verify with `istat`.
3. Extract inode content with `icat` and Base64-decode it.

Flag:

```text
picoCTF{573417h13r_7h4n_7h3_1457_58527bb222}
```

Takeaway: timestamp clustering is a reliable lead for hidden or tampered artifacts.

### General Skills

#### printer-shares-3

- Description: debug script left exposed through SMB shares.
- Hint: script runs every minute.
- Root cause: writable public SMB share + cron execution of shared script.

Exploit flow:

1. Enumerate shares on port 60023.
2. Confirm `script.sh` and `cron.log` exist in public share.
3. Replace `script.sh` to print the private flag file.
4. Wait for cron, then read `cron.log`.

Commands used:

```bash
smbclient -L //dolphin-cove.picoctf.net -p 60023 -N
smbclient //dolphin-cove.picoctf.net/shares -p 60023 -N -c 'ls'

cat > script.sh << 'EOF'
#!/bin/bash
echo "FLAG_START"
cat /challenge/secure-shares/flag.txt 2>&1
echo "FLAG_END"
EOF

smbclient //dolphin-cove.picoctf.net/shares -p 60023 -N -c 'put script.sh script.sh'
sleep 70
smbclient //dolphin-cove.picoctf.net/shares -p 60023 -N -c 'get cron.log cron.log.new'
tail -n 120 cron.log.new
```

Flag:

```text
picoCTF{5mb_pr1nter_5h4re5_r3v3r53_85690588}
```

Takeaway: never execute scheduled scripts from user-writable shares.

## Template For New Challenges

Use this block for each future entry:

~~~md
### <Category>

#### <Challenge Name>

- Description: <short description>
- Hint: <hint>
- Root cause: <one line>

Exploit flow:

1. <step>
2. <step>
3. <step>

Commands or payloads:

```bash
<commands>
```

Flag:

```text
<flag>
```

Takeaway: <one line lesson>
~~~