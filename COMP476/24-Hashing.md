# COMP476 — Networked Computer Systems
## Lecture 24 — Hashing, Digital Signatures & Digital Certificates (slides: "Hashing and Digital Signatures," file dated Nov 9, 2026)

**Relationship to earlier lectures:** continues the security thread (Election Security → Encryption → **this**). Lecture 23 gave the toolbox (symmetric, asymmetric, hybrid, modes, a first look at hashes). This lecture finishes hashes, then builds **MACs → digital signatures → key exchange → certificates**. It is the trust machinery behind **TLS/HTTPS**. Items marked 🆕 are new topics. ★ = most likely exam-worthy.

*Numbering note: filed as Lecture 24 because Lecture 23 (Encryption) is dated Nov 4 and the schedule slide lists Nov 11 as Network Security (next). Rename if your numbering differs.*

*Overlap note: the first ~30 slides (crypto vocabulary, symmetric vs. asymmetric, hybrid, ECB/CBC/IV/CFB/GCM, hash basics) are a **recap of Lecture 23**. Only the genuinely new points are expanded here; see Lecture 23 for the rest.*

---

## Part 1 — Recap (see Lecture 23 for detail)

### 1. Quote
- **Esther Dyson:** encryption is a powerful defensive weapon for free people — a **technical guarantee of privacy regardless of who runs the government**.

### 2. Quick recap ★
- **Plaintext →(key)→ Ciphertext →(key)→ Plaintext.** Ciphertext looks random. **Compression and encoding (Base64) are not encryption** — no key needed.
- **Symmetric** (same key; **DES, AES**; fast; must share key securely) vs. **Asymmetric** (different but related keys; **RSA**; slower; public key can be published).
- **Hybrid:** **Step 1** — encrypt an **AES key** with the receiver's **public key** and send it. **Step 2** — all later messages use the AES key; **discarded after the session**.
- **Usage review:** Symmetric works for two-party talk, group members sending to the group, saving for yourself; **difficulty = every pair needs a unique key**. Asymmetric works for **everyone → leader (confidentiality)** and **leader → everyone (integrity)**; poor for saving to read later and group-to-group.
- **Block sizes:** AES **128 bits**, DES **64 bits**, RSA **(key size − 1)/8** *(slide says "255 bits" for a 2K key — likely **bytes**)*.
- **Modes:** **ECB** (blocks separate), **CBC** (chained; one-bit change affects all following blocks; decrypt can be parallelized), **CFB** (IV encrypted → random stream XOR plaintext; decryption uses the **encryption** algorithm), **GCM** (encrypts an incrementing IV; parallelizable; **used by Zoom**).
- **IV:** dummy first ciphertext block; **not secret, should be random**; new IV each time → identical messages encrypt differently; often **prepended** to the ciphertext; **protect with a MAC**.
- *Instructor aside:* "Tales of Alice and Bob" illustrations at **comp476.online**.

---

## Part 2 — Cryptographic Hashes (extended) ★

### 3. What a hash is ★
- Result of a calculation over **all bytes of a file**. Simple hash = **checksum** (sum of bytes).
- **Cryptographic hash:** (1) **easy to generate**, (2) **infeasible to generate a file that produces a given hash**, (3) **changes extensively if any part of the file changes**.
- Built on a **one-way function:** **hash = f(text)**. Very hard to find text for a given hash. *Not mathematically proven* to be non-invertible — it just **appears difficult**.

### 4. Why a plain sum is weak ★
- `12 02 17 24 09` → **64**. Change a byte (`12 02 17 13 09`) → **53** (detected).
- But `12 04 15 24 09` → also **64** → **collision**; attacker alters message, hash still matches.

### 5. Standards ★
| Algorithm | Notes |
|---|---|
| **MD5** | Found **less secure** |
| **SHA-1** | **1993**, **160-bit** |
| **SHA-3** | Newer, more secure; **224 / 256 / 384 / 512 bits** |
| **BLAKE3** | **2020**; efficient and **parallelizable** |

### 6. 🆕 Hashing for integrity ★
- Compute the hash of a file/packet and **append it**. Receiver recomputes and compares; mismatch → file changed.
- **Weakness:** **without encryption, an attacker can change the file and recalculate the hash.** → motivates MACs.

---

## Part 3 — 🆕 Message Authentication Codes (MACs)

### 7. What a MAC is ★★
- Confirms a message **came from the stated sender** (and was not modified).
- **Per the slides:** **hash the message → encrypt the hash → append it to the packet.**
- **Creating:** compute hash → **encrypt hash** → append to file.
- **Checking:** compute hash → **encrypt your hash** → compare with the one in the file.
- Because the hash is encrypted, **nobody can make a false MAC without the key**.
- *(Real-world note: this "encrypt the hash" description is the lecture's model; in practice a MAC is a **keyed hash such as HMAC**, which Lecture 23 mentioned for WPA3.)*

### 8. MAC encryption ★
- Provides **data integrity and authenticity**. **Only someone with the proper key** can create a correct MAC.
- Can use **symmetric** (key must be **securely shared** with both parties) **or asymmetric** (receiver must be able to **get the correct public key**).

### 9. Integrity but **not** confidentiality ★
- MAC proves the message came from someone who knows the key and wasn't modified. It **does not encrypt or modify the message itself** — the content is still readable.

---

## Part 4 — 🆕 Chaffing and Winnowing

### 10. Idea ★
- Achieves **confidentiality over an insecure channel without encrypting the data**. **Conceived by Ron Rivest.** Possibly useful where **encryption is not allowed**.
- **"Hiding in plain sight":** embed the real message in a **large stream of irrelevant packets**; only the receiver can tell which are real.

### 11. Example ★
- **Alice** splits her message into **bits, one bit per packet**. Each real packet = **data bit + serial number + MAC**, where the MAC = **symmetric-key encryption of the hash of (data + serial number)**.
- Alice also sends many **fake ("chaff") packets**: **random data bit + serial number + random number as fake MAC**.
- **Bob** has the key → **authenticates real packets** (wheat) and rebuilds the message; **packets with an incorrect MAC are ignored** (winnowing).

---

## Part 5 — 🆕 Hashing Passwords ★

- A **database of plaintext passwords is vulnerable to disclosure** → store the **hash** instead.
- Login: **hash the candidate password and compare** to the stored hash.
- Since it is **infeasible to craft a password that hashes to a specific value**, knowing the hashes does an attacker **little good**.

---

## Part 6 — 🆕 Digital Signatures ★★

### 12. Goals (like a handwritten signature) ★
1. **Difficult to forge** 2. **Easily verifiable** 3. **Not deniable** 4. **Easy to implement** 5. **Differs from document to document**

### 13. How they work ★★
- Message body can be **clear plaintext** — the objective is **verifying the sender**, not hiding content.
- With **RSA, either key can encrypt or decrypt**. A digital signature = a **MAC encrypted with asymmetric encryption**.

**Signing process (know the order):**
1. **Hash the data**; **append the sender's name** to the hash.
2. **Encrypt the hash with the sender's PRIVATE key.**
3. **Append the encrypted hash to the data** and transmit together.
4. **Receiver decrypts the hash with the sender's PUBLIC key.**
5. **Receiver calculates their own hash of the message and compares.** Match = authentic and unmodified.

> Contrast with Lecture 23 *encryption*: **encrypt with receiver's public key** (confidentiality). **Sign with sender's private key** (authenticity/integrity/non-repudiation).

### 14. Uses ★
- **Digitally signed email** verifies the sender.
- **Signed applets/programs** come from a known source and **can't be modified or infected without detection**.
- **Signed documents** can't be changed without detection. **Adobe PDF and Microsoft Word** documents can be signed.
- Many web programs are signed so users have some trust to run them; **a program with an invalid signature should be treated with great caution.**

### 15. What is NOT a digital signature ★
- **Handwritten-signature systems** for documents.
- An **email "signature" text block** (name, title, contact info).
- **Electronic signature = just a picture of a handwritten signature.**
- **Rule: if it doesn't involve a digital certificate, it doesn't count.**
- The **Gmail web interface does not support digital signatures.**

---

## Part 7 — 🆕 Nonces and Randomness

### 16. Nonce ★
- Used to **avoid replay attacks**: a **random number identifying a message**, used **only once** in a protocol.
- Must be generated so the attacker **can't predict it**. **Repeated use of a pseudo-random number generator is not secure.**

### 17. Cryptographically random numbers ★
- **Pseudo-random generators** produce an apparently random stream by an algorithm: **know one value → calculate the next.**
- **True randomness** requires measuring a **naturally random event**: **radioactive decay, thermal noise, avalanche noise in Zener diodes, clock drift, hard-disk read/write head timing, radio noise**.

---

## Part 8 — 🆕 Key Exchange / Key Distribution

### 18. The problem ★
- Receiver must be confident the sender is who they claim. **If Cindy convinces Bob that her public key belongs to Alice, Cindy can send signed messages masquerading as Alice.**
- **Symmetric:** both sides need the key **before** encrypting. *Fun fact:* for years the government had a **Marine guard carry the key** to a remote ambassador.
- **Asymmetric:** you must be assured the **public key you hold is authentic**.
- **Reliable distribution methods:** **Key servers**, **Digital certificates**, **Human interaction**.

### 19. Trusted key server ★
- Every user has a **separate key known only to them and the key server**. In the literature **Cathy** is the trusted friend.
- **Public key request** (everyone knows the server's public key):
  1. **Alice → Server:** `Bob, nonce1`
  2. **Server → Alice:** `{Key_Bob, nonce1}` encrypted with **server's private key**
- Alice trusts message 2 because it was **encrypted with the server's private key** and **contains the correct nonce** (proves freshness).

### 20. Session keys vs. interchange keys ★
- **Interchange key** = tied to an **entity** (person/system). **Session key** = **short-term key for one communication session**.
- Short use → **less exposure if compromised**.

### 21. Needham–Schroeder key exchange ★★
- Lets Alice and Bob get a **session key through trusted third party Cathy** (assume **Cathy has a key for each person**).
1. **Alice → Cathy:** `Alice, Bob, nonce1`
2. **Cathy → Alice:** `{Alice, Bob, nonce1, key_session, {Alice, key_session}key_Bob}key_Alice`
3. **Alice → Bob:** `{Alice, key_session}key_Bob`
4. **Bob → Alice:** `{nonce2}key_session`
5. **Alice → Bob:** `{nonce2 − 1}key_session`
- Steps 4–5 are a **handshake proving both hold the session key** (the **−1** shows Alice actually decrypted nonce2 and did a calculation, not a replay).

### 22. Diffie–Hellman key exchange ★★ (be able to compute)
- Two parties derive a **shared secret over an insecure channel**; **eavesdroppers watching everything still can't get it**. The shared number becomes a **symmetric key**.
- *(Slide credits **Ralph Merkle**; it's usually called Diffie–Hellman(–Merkle).)*

**Algorithm and slide example (verified):**
| Step | Value |
|---|---|
| **Public:** modulus **p = 23**, base **g = 7** | |
| Alice picks secret **a = 4**, sends **A = gᵃ mod p** | 7⁴ mod 23 = **9** |
| Bob picks secret **b = 5**, sends **B = gᵇ mod p** | 7⁵ mod 23 = **17** |
| Alice computes **s = Bᵃ mod p** | 17⁴ mod 23 = **8** |
| Bob computes **s = Aᵇ mod p** | 9⁵ mod 23 = **8** |

- **Private:** a, b, s. **Public:** p, g, A, B. Both get **s = 8**.
- *Check:* both equal g^(ab) mod p = 7²⁰ mod 23 = 8.

### 23. Forward search attack ★
- **Paul** will send **George** one of two messages: **"by land"** or **"by sea."**
- **Benedict** encrypts **both** messages with **Paul's public key** → when Paul sends, Benedict **compares and learns which was sent**.
- Works because the message space is tiny and the public key is public. **Session keys prevent forward searches.**

---

## Part 9 — 🆕 Digital Certificates ★★

### 24. What a certificate is ★
- Contains a **user's public key** plus **information about the user** (email address or server name).
- **Digitally signed by a Certificate Authority (CA).** CAs are **vendors of certificates**.
- **Clients must already know the CA's public key** — this is the trust anchor.
- Solves the Part 8 problem: the CA's signature binds **public key ↔ identity**.

### 25. Chain of trust ★
- **University example:** a chain of trust can start from a **single point in an organization**; **only the top public key is needed by everyone**.

### 26. X.509 ★
- **Standard format for public-key certificates**, defined by the **ITU-T** (International Telecommunications Union Standardization sector).
- Used in many Internet protocols, **including TLS → the basis of HTTPS**.

### 27. Buying certificates
- Several companies sell them; **usually expensive for servers**.
- **Free certificates for personal email** exist (**cacert.org**, **Comodo** free email certificate).
- Slide shows the **list of CAs trusted on the instructor's computer** (built-in trust store).

### 28. Revocation ★
**Reasons to stop using a certificate:**
- **Key protecting the certificate compromised**
- **CA compromised**
- **Entity no longer belongs to the organization**
- **Certificate issued to the wrong organization**
- **Privilege to use certificates withdrawn**

**Revocation lists ★**
- CAs keep a **revocation list** of issued certs that should no longer be used/accepted. **Does not include certs that merely expired.**
- Software accepting a certificate **should check the list — but checking is not mandatory.**

**Potential issues ★**
- Contacting the CA is **another step** in connection setup.
- It's **another step the programmer must code — and some skip it.**
- The **CA could be hit by a denial-of-service attack.**

---

## Part 10 — 🆕 Creating and Using Certificates (practical)

### 29. Java security tools (JDK 25)
| Tool | Purpose |
|---|---|
| **keytool** | Manage keystores and certificates |
| **jarsigner** | Generate and verify **JAR signatures** |
| **policytool** | GUI for managing policy files |
- Command-line tools with many options (Oracle docs, Java SE 25 man pages).

### 30. KeyStore
- Securely holds **key entries** and **trusted certificate entries**. **Can be encrypted.** Implemented as a **file** (on Windows it may have **no extension**).

### 31. keytool: self-signed certificate ★
```
keytool -genkeypair -alias YourLastName -keystore classcert -storepass secret -keyalg rsa
```
- **alias** = name shown with the certificate; **classcert** = file holding it; **secret** = password protecting the keystore.

### 32. Importing your certificate
- **Outlook:** File → Options → Trust Center → Trust Center Settings → Email Security → Import.
- **Chrome:** … → Settings → Privacy and Security → Security → **Manage Device Certificates**.

### 33. Microsoft Word signing
- Create a signing certificate with **Selfcert.exe** (typically in `C:\Program Files\Microsoft Office\root\Office16`).
- Program claims Office only trusts a self-signed cert **on the machine that created it**; instructor says **experience shows otherwise**.

---

## Part 11 — 🆕 PGP and Legal History

### 34. Pretty Good Privacy (PGP) ★
- Provides **cryptographic privacy and authentication**. **Developed by Phil Zimmermann, 1991.** Available as **OpenPGP** and **Gpg4win**.
- Can **encrypt and/or sign email or files**. Commonly uses **AES + RSA** (hybrid).
- **PGP-signed email:** the signature block (`-----BEGIN PGP SIGNATURE-----` … `-----END PGP SIGNATURE-----`) is **base64 encoded**.

### 35. Legal restrictions ★
- **Up to 1999**, the US treated **encryption devices/software as munitions**. **Couldn't export software with a key over 40 bits.**
- **Zimmermann published a book with PGP's full source code** — protected by the **First Amendment**, so it could be **sold overseas**. An **electronic copy of the source could not be exported.**

---

## Part 12 — Course Schedule (from last slide)

| Date | Topic | Reading |
|---|---|---|
| **Mon Nov 9** | Digital Certificates | **8.12.3** |
| **Wed Nov 11** | Network Security | **5.12** |
| **Mon Nov 16 (online only)** | Firewalls & review | **8.3** |
| **Wed Nov 18** | **Exam 3** | |
| **Mon Nov 23** | Backups | |
| **Wed Nov 25** | Thanksgiving — no classes | |
| **Mon Nov 30** | Gestalt | |
| **Wed Dec 2** | Final review | |

- ⚠️ **Exam 3 is dated Wed Nov 18 — today.** This lecture's material is on the Exam 3 side of the line (Encryption → Hashing → Signatures → Certificates).

---

## Quick fact sheet ★
| Concept | Key fact |
|---|---|
| Crypto hash | **Easy to compute; infeasible to invert/forge; changes extensively** with any edit |
| Simple-sum hash | **Collisions trivial** (`…17 24…` vs `…15 24…` both = 64) |
| SHA-1 / SHA-3 / BLAKE3 / MD5 | **160-bit (1993)** / **224–512-bit** / **2020, parallelizable** / **weak** |
| Hash alone | Detects change, but **attacker can recompute** → need MAC |
| MAC | **Hash → encrypt → append**; gives **integrity + authenticity**, **no confidentiality** |
| Chaffing & winnowing | **Rivest**; **one bit per packet**; valid MAC = wheat, **fake MAC = chaff**; **confidentiality without encryption** |
| Password storage | **Store the hash**, compare hash of the candidate |
| Digital signature | **Hash + name → encrypt with sender's PRIVATE key → append; receiver decrypts with PUBLIC key and compares** |
| NOT digital signatures | **Email text signature, scanned/pictured signature, electronic signature** (no certificate) |
| Nonce | **Random, used once**, stops **replay**; **PRNG reuse insecure** |
| True randomness | **Radioactive decay, thermal/avalanche noise, clock drift, disk timing, radio noise** |
| Key distribution | **Key server, digital certificate, human interaction** |
| Needham–Schroeder | **5 messages**; Cathy issues **session key**; **nonce2 / nonce2−1** handshake |
| Diffie–Hellman | **p=23, g=7, a=4, b=5 → A=9, B=17, s=8** |
| Forward search | Attacker **pre-encrypts all possible messages**; **session keys** prevent |
| Certificate | **Public key + identity, signed by CA**; clients must know **CA public key** |
| X.509 | **ITU-T** certificate format; basis of **TLS/HTTPS** |
| Revocation | CRL excludes **expired** certs; checking **not mandatory**; **DoS risk** |
| keytool | `keytool -genkeypair -alias … -keystore … -storepass … -keyalg rsa` |
| PGP | **Zimmermann, 1991**; **AES + RSA**; signature is **base64** |
| Export law | **Until 1999 encryption = munition; 40-bit export limit**; source published as a **book** |

---

## Slide errata / cautions (flagged, not changed above unless noted)
- **RSA block size** "255 **bits**" for a 2K key probably means **bytes** (carried over from Lecture 23).
- **CBC slide wording reversed** ("ciphertext XOR next plaintext"); correct is **plaintext XOR previous ciphertext**, then encrypt.
- **MAC slide** describes **"encrypt the hash"**; real-world MACs are **keyed hashes (HMAC)**. Use the lecture's description for the exam, know the distinction.
- **Diffie–Hellman** slide credits **Ralph Merkle**; commonly **Diffie–Hellman(–Merkle)**.
- **Exponents** in the DH slide (gᵃ, 7⁴, 17⁴, etc.) lost superscripts in the PDF text; I restored them and **re-verified all four values (9, 17, 8, 8)**.
- **Needham–Schroeder** message 2 lost its nested braces in extraction; I restored the nesting from the standard protocol — **check against the original slide** if the exam asks for exact format.
- Several slides are **diagram-only** (MAC process, digital signature flow, certificate and university-chain figures, trusted CA list screenshot, Alice/Bob figures); described in words here.
