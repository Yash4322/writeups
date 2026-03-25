# Crack the Hash — Complete CTF Writeup (GitHub Ready)

## Overview

This writeup covers identification and cracking of multiple hash types from beginner (unsalted, fast) to advanced (salted, slow). The focus is on selecting the correct hash mode, minimizing keyspace, and using efficient attack strategies.

---

## Hashes Provided

```
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
1DFECA0C002AE40B8619ECF94819CC1B
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
e5d8870e5bdd26602cab8dbe07a942c8669e56d6
```

---

## Hash Identification

| Pattern | Algorithm   | Notes         |
| ------- | ----------- | ------------- |
| $2y$... | bcrypt      | very slow     |
| 64 hex  | SHA-256     | fast          |
| 32 hex  | NTLM        | very fast     |
| $6$...  | sha512crypt | salted + slow |
| 40 hex  | SHA-1       | fast          |

---

## Tools

* Hashcat
* John the Ripper
* rockyou.txt
* awk / grep

---

## Level 1 — Easy Hashes (Fast, Unsalted)

### MD5

```
098f6bcd4621d373cade4e832627b4f6
```

Cracked using rockyou or online DB → `test`

---

### SHA-1

```
e5d8870e5bdd26602cab8dbe07a942c8669e56d6
```

Result:

```
secret
```

Verify:

```bash
echo -n 'secret' | sha1sum
```

---

### SHA-256

```
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```

```bash
hashcat -m 1400 hash.txt rockyou.txt
```

If needed:

```bash
hashcat -m 1400 -a 3 hash.txt ?a?a?a?a
```

---

## Level 2 — Fast but Stronger (NTLM)

```
1DFECA0C002AE40B8619ECF94819CC1B
```

```bash
hashcat -m 1000 hash.txt rockyou.txt
```

Brute-force (very effective):

```bash
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a
```

---

## Level 3 — Slow Hashes (Keyspace Reduction Required)

### bcrypt

```
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```

```bash
LC_ALL=C awk 'length($0)==4' rockyou.txt > rockyou_4.txt
hashcat -m 3200 hash.txt rockyou_4.txt
```

---

### SHA512-crypt

```
$6$aReallyHardSalt$...
```

```bash
LC_ALL=C awk 'length($0)==6' rockyou.txt > rockyou_6.txt
hashcat -m 1800 hash.txt rockyou_6.txt
```

Targeted mask:

```bash
hashcat -m 1800 -a 3 hash.txt ?l?l?l?l?l?l
```

---

## Level 4 — Salted Hashes (Critical Concept)

### Salted SHA-1 (HMAC)

Format:

```
hash:salt
```

Example logic:

* hash = SHA1(password + salt)
* OR HMAC-SHA1 depending on challenge

Hashcat mode:

```bash
hashcat -m 160 hash.txt rockyou.txt
```

Key insight:

* Using wrong mode = guaranteed failure

---

## Key Concepts

### 1. Hashes are One-Way

Cannot be decrypted, only cracked.

---

### 2. Speed Determines Strategy

| Algorithm   | Speed     | Strategy         |
| ----------- | --------- | ---------------- |
| bcrypt      | very slow | small wordlist   |
| sha512crypt | slow      | reduced keyspace |
| SHA-256     | fast      | brute-force      |
| NTLM        | very fast | full brute-force |
| SHA-1       | fast      | wordlist         |

---

### 3. Keyspace Reduction

```bash
LC_ALL=C awk 'length($0)==4' rockyou.txt > rockyou_4.txt
```

---

### 4. Mask Attacks

```
?a?a?a?a
?l?l?l?l
?d?d?d?d
```

---

### 5. Salt Handling

| Type     | Example      | Handling     |
| -------- | ------------ | ------------ |
| Inline   | $6$salt$hash | auto handled |
| External | hash:salt    | special mode |

---

## Optimized Workflow

1. Identify hash
2. Check common passwords
3. Apply constraints (length/pattern)
4. Use filtered wordlists
5. Switch to mask attacks
6. Adjust hash mode if needed

---

## Final Notes

* Always assume password reuse in CTFs
* Constraints (length/pattern) are intentional hints
* Strategy > brute force

---

## Conclusion

Efficient hash cracking depends on:

* correct identification
* correct hash mode
* reduced keyspace
* smart attack selection

Mastering these steps drastically reduces cracking time and reflects real-world red team methodology.
