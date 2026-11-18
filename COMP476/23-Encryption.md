# COMP476 — Networked Computer Systems
## Lecture 23 — Encryption (slides: "Encryption," file dated Nov 4, 2026)

**Relationship to earlier lectures:** a **concepts + algorithms lecture** that sits under the security thread (Election Security → Encryption). It is the toolbox behind **HTTPS/TLS** (earlier Cookies/HTTP lectures) and **WPA** (Wireless LANs lecture). Items marked 🆕 are new topics. ★ = most likely exam-worthy.

*Numbering note: I filed this as Lecture 23 because Lecture 22 (Election Security) is dated Nov 2 — rename if your numbering differs.*

---

## Part 1 — Core Ideas

### 1. Security tool ★
- Schneier quote: two kinds of encryption — one that stops **your sister** reading your diary, one that stops **your government**.
- Encryption is **not the complete answer** to security, but a useful tool. It helps with **confidentiality** and **integrity**.

### 2. Cryptography vocabulary ★
- **Plaintext** (any bits: text, graphics, programs) → **Encryption** (with a **key**) → **Ciphertext** → **Decryption** (with a key) → **Plaintext**.
- Ciphertext **looks like a random collection of bits**.
- **Not encryption** (no key needed to undo): **compression**, **encoding** (e.g., **Base64** — same Base64 as the API assignment magic number).

### 3. Compression (not encryption)
| Type | Meaning | Examples |
|---|---|---|
| **Lossless** | Removes statistical redundancy; decompressed = **identical** to original; used for text/programs | **Zip, PKZIP** |
| **Lossy** | Decompressed may differ; humans usually can't tell; graphics/audio | **JPEG, MPEG, MP3, MP4** |

- **Ken's phone-texting compression** (joke example): type `tec`, keyboard suggests 9 words, *Technology* is the 2nd → send `tec2` (**40% of original size**). Decompress by typing `tec` and picking suggestion 2.

### 4. Data in motion vs. at rest ★
- **In motion** = over a network. **At rest** = stored on a computer.

---

## Part 2 — 🆕 Historical / Classical Ciphers

### 5. Caesar (shift) cipher ★
- Each letter shifted by a fixed amount; **key = number of letters shifted**.
- **Brute force: only 26 possible shifts** → trivially broken.
- Slide example: `Frpsxwhuv duh ixq` → shift back 3 → **"Computers are fun"**.

### 6. Types of attacks ★
Assume the adversary **knows the algorithm** and has:
| Attack | Adversary has |
|---|---|
| **Ciphertext only** | Ciphertext samples, no info about content |
| **Known plaintext** | Ciphertext **and** matching plaintext examples |
| **Chosen plaintext** | Can get ciphertext for **plaintext of their choosing** |

### 7. Substitution cipher ★
- Each letter maps to a symbol/letter; **key = the mapping list**.
- **26! ≈ 4×10²⁶ possible keys** → brute force is hopeless…
- …but **letter frequency analysis** breaks it: English letter frequencies (plus frequent **pairs, triples, short words**) reveal the mapping. Slide demo: a symbol-cipher of the Chancellor's welcome message was **partially decrypted using only letter frequency**.
- Lesson: **big key space ≠ secure** if the cipher leaks statistical structure.

### 8. Vigenère cipher ★
- Described by **Giovan Battista Bellaso, 1553**. A **text key is repeated** for the length of the plaintext.
- **Encrypt:** Cᵢ = (Pᵢ + Kᵢ) mod 26  **Decrypt:** Pᵢ = (Cᵢ − Kᵢ) mod 26
- Slide example (verified): plaintext `welcometonorthcarolina`, key `informationprivacyinfo` → ciphertext `erqqfyemwbbgkpxatmtvso`.
- **Weakness:** for **long text + short key**, frequency analysis still works. Same plaintext letter sometimes meets the same key character; **statistics on frequency and distance between matches** reveal the **key size**.

### 9. Running key cipher
- Like Vigenère but the key is **long and non-repeating** (typically text from a **book/periodical**).
- Slide example using a well-known C book: plaintext `fleeatonceweardiscovered` ("flee at once we are discovered"), key `ERRORSCANOCCURINSEVERALPL`, ciphertext `JCVSRLQNPSYGUIMQAWXSMECTO` (verified).
- **Weakness:** human text is far from random. If you know **part of the plaintext**, subtract it along the whole string and look for **readable text** — probably part of the key.
- If the key text were **perfectly random** → it becomes a one-time pad.

### 10. One-time pad ★
- **XOR** the message bits with a **random key**.
- Key must be **at least as long as the message** (never repeated) and **truly random** (not pseudo-random).
- **Perfect encryption — cannot be broken**: any ciphertext can decrypt to **any** plaintext of the same length with the right key. Slide example: ciphertext `011001010` ⊕ key `001100000` = `010101010`; ⊕ key `011010101` = `000011111`.
- *(Practical catch: the key is as big as the message and must be shared securely and never reused.)*

### 11. Diffusion and confusion ★
- **Diffusion:** spread plaintext across ciphertext — one plaintext byte affects **many** ciphertext bytes. In **DES/AES**, flipping **1 plaintext bit changes about half of the 128 ciphertext bits**.
- **Confusion:** change plaintext bits according to some rule (substitution).

---

## Part 3 — Symmetric vs. Asymmetric ★

### 12. The two types
| | **Symmetric (secret key)** | **Asymmetric (public key)** |
|---|---|---|
| Keys | **Same** key encrypts and decrypts | **Different but related** keys |
| Sharing | Sender and receiver must **securely share** the key beforehand | **Public key** can be published; **private key** stays secret |
| Examples | **DES, AES** | **RSA** |
| Speed | **Fast** | **Slower** (RSA slower than DES/AES) |

- Asymmetric flow: User1 gets **User2's public key** (from a **remote public key directory**), encrypts; only User2's **private key** decrypts.

### 13. Why publish a standard? (DES/AES are public) ★
- To be useful, **others must implement it**.
- **Published standards are analyzed by experts.**
- A good algorithm lets **only key-holders** in — **knowing the algorithm doesn't give access** (no security through obscurity).

### 14. Performance
- **DES/AES easy in hardware**; **AES built into Intel, AMD, ARM** processors; AES also efficient in software.
- **Hybrid encryption** uses **both** (see Part 6).

### 15. Key strength and lengths ★
- **n-bit key → 2ⁿ guesses** to try all; you'd likely succeed in about **2ⁿ⁻¹** (half).
- **Asymmetric keys must have a mathematical relationship**, so not every bit string is a valid key.

| Algorithm | Key length |
|---|---|
| **DES** | **56 bits** |
| **Triple DES (3DES)** | 2 DES keys = **112 bits** (3 keys = 168) |
| **AES** | **128, 192, or 256 bits** |
| **RSA** | Variable, often **512, 1024, 2048 (2K)** bits |

---

## Part 4 — 🆕 Block Cipher Internals: DES and AES

### 16. Substitution–permutation (SP) ciphers ★
- Three phases, **often repeated** (rounds): **Substitution** (swap a bit pattern for another), **Permutation** (rearrange bits), **XOR with a key**. **DES and AES are SP ciphers.**
- **K box** = XOR input with that round's key. **S box** = substitution.
- **S box** is efficiently implemented as a **lookup table**.
  - 3-bit → 3-bit example: `000→101, 001→010, 010→110, 011→000, 100→111, 101→001, 110→011, 111→100`.
  - Input and output sizes **don't have to match** (e.g., 3-bit in → 2-bit out: `000→10, 001→01, 010→11, 011→01, 100→11, 101→00, 110→10, 111→00`).
- **Permutation can expand** — some input bits go to **multiple** output bits.
- **Key stages:** each stage's key is a **function of the original key** (modified so each stage is unique); some stages use only some key bits.

### 17. DES (Data Encryption Standard) ★
- Developed by **IBM**; standard in **1977**; once the most widely used cryptosystem.
- **56-bit key**, **64-bit blocks**, **16 stages**.
- Data split into **left and right 32-bit halves**, each handled differently.
- **Key schedule:** 56-bit key → two **28-bit halves**; each **rotated left 1 or 2 bits**; **48 of 56 bits** selected as the stage key; each key bit is used in **14 of the 16 stages**.
- **Weaknesses:** key **too short**. **EFF built a device in 1998 that brute-forced a DES key in a little over 2 days.** **Differential cryptanalysis** finds the key with **2⁴⁷ chosen plaintext/ciphertext pairs**.

### 18. Triple DES ★
- **ciphertext = E_Key3( D_Key2( E_Key1(plaintext) ) )** (encrypt–decrypt–encrypt).
- Keys: **all different** → best, **168-bit**; **Key1 = Key3** → **112-bit**, good; **all identical** → same as single DES.
- **Still not great:** encrypting known plaintext with Key1 then Key2 is **ineffective** — a **meet-in-the-middle** attack needs only about **2⁵⁷** work instead of 2¹¹² *(slide shows this as exponents that extracted garbled, so treat the exact numbers with care)*.
- **NIST deprecated DES and 3DES for new applications in 2017, and for all applications by 2023.**

### 19. AES (Advanced Encryption Standard) ★
- Also called **Rijndael**; selected **2000** after an **open international competition**; by Belgian researchers **Rijmen and Daemen**; **royalty-free worldwide**.
- **128-bit blocks**; keys **128, 192, or 256 bits** *(slide typo says "194")*.
- Operates on a **4×4 array of bytes = the "state."**
- **Rounds:** **10** (128-bit key), **12** (192), **14** (256).

**AES round structure ★**
| Stage | Steps |
|---|---|
| **Initial** | **AddRoundKey** |
| **Each round** | **SubBytes** → **ShiftRows** → **MixColumns** → **AddRoundKey** |
| **Final** | **SubBytes** → **ShiftRows** → **AddRoundKey** (**no MixColumns**) |

- **AddRoundKey:** XOR each byte with the stage key.
- **SubBytes:** each byte replaced via the **8-bit Rijndael S-box**.
- **ShiftRows:** **cyclically shift** bytes in each row by an offset.
- **MixColumns:** each column multiplied by a **fixed polynomial**; all 4 input bytes determine all 4 output bytes.
- **Implementation speedup:** combine SubBytes + ShiftRows + MixColumns into **table lookups** (**4 KB** of tables): a stage = **16 lookups + 12 XORs**, then **4 XORs** for AddRoundKey. **Intel/AMD have added instructions to do an AES stage.**

### 20. AES security
- **No published practical attack better than brute force.** One attack is **4× faster than brute force** (still impractical).
- **Snowden documents** claim the **NSA** is researching ways to defeat AES.

---

## Part 5 — 🆕 RSA (Asymmetric)

### 21. RSA background ★
- Developed **1977** by **Rivest, Shamir, Adleman** (Rivest wrote the COMP785 textbook).
- **Secretly developed by Clifford Cocks in 1973.**
- **Most commonly used public-key algorithm**; now **public domain**.

### 22. RSA algorithm ★★ (know the steps and be able to compute)
1. Pick **secret primes p and q**; **n = p·q**.
2. **Totient Φ(n) = (p−1)(q−1)**.
3. Choose **e** with **gcd(e, Φ(n)) = 1**.
4. Compute **d** with **d·e ≡ 1 (mod Φ(n))**, i.e., **d = e⁻¹ mod Φ(n)**.
5. **Public key = (n, e). Secret key = d.**

- **Encrypt:** **c = mᵉ mod n**. **Decrypt:** **m = cᵈ mod n**.

**Worked example 1 (verified):** p = 101, q = 113 → n = **11413**; Φ(n) = 100·112 = **11200**; e = **3533** (gcd = 1); d = e⁻¹ mod 11200 = **6597**; m = **9726**.
- Encrypt: 9726³⁵³³ mod 11413 = **5761**. Decrypt: 5761⁶⁵⁹⁷ mod 11413 = **9726**.

**Worked example 2 (verified):** p = 885320963, q = 238855417 → n = **211463707796206571**; e = 9007; d = **116402471153538991**; m = 30120 → c = **113535859035722866**.
- ⚠️ **Slide typo:** it writes "d = e⁻¹ mod **11200**" — that's copied from example 1. It should be **mod Φ(n)** (Φ(n) = 211463706672030192).
- **c is much larger than m** → ciphertext needs more space.

### 23. Sizes ★
- p and q are **large** so n is hard to factor; n is **log₂ n bits** long.
- Plaintext piece must satisfy **0 < m < n**; ciphertext can be up to **log₂ n bits**.
- **Text blocks should be the length of n or shorter.**

### 24. Fast modular exponentiation
- Naive: multiply m by itself e times then mod → **O(e)**. Better: **O(log e)** using **square-and-multiply**:
```
modular_pow(base, exponent, modulus):
    result = 1
    base = base mod modulus
    while exponent > 0:
        if exponent mod 2 == 1:
            result = (result * base) mod modulus
        exponent = exponent >> 1
        base = (base * base) mod modulus
    return result
```
- Each loop handles **one bit of the exponent**: always **square** the base; **multiply into result** when the current bit is 1.

### 25. Mathematical basis for RSA strength ★
- Breaking RSA requires **factoring n**, the product of two large primes — **very difficult**.
- For a **2048-bit key**, the best factoring algorithm needs about **10²⁰⁵ divisions** — infeasible.

### 26. Quantum computing
- **AES is believed quantum-resistant.** A quantum computer could, **theoretically, break RSA** in reasonable time.
- Opinion slide: **Mikhail Dyakonov** (IEEE Spectrum, Dec 2018) — **"Not in the foreseeable future"**; quantum computing has been promised for a long time but never delivered.

---

## Part 6 — Hybrid Encryption, SSL/TLS ★

### 27. Speed considerations → hybrid
- **AES is much faster than RSA.** Symmetric needs a **securely shared key**; the **asymmetric public key can be public**.

### 28. Hybrid cryptography ★★
- **Step 1:** Sender generates an **AES key**, **encrypts it with the receiver's RSA public key**, sends it. Receiver decrypts with its **private key** → **both share the AES key**.
- **Step 2:** **All following messages are encrypted with the AES key.** The AES key is **discarded after the session**.

### 29. SSL / TLS ★
- **Hybrid encryption** is used in **SSL and TLS** (SSL **deprecated June 2015**) and in **HTTPS** and other systems.
- **Identity of parties authenticated with public-key cryptography.** **Typically only the server is authenticated.**

### 30. Symmetric vs. asymmetric — usage review ★
| | Symmetric | Asymmetric |
|---|---|---|
| Who knows the key | **All communicators**; all must keep it secret | **Everyone knows public key**; **only owner** knows private |
| Can send/receive | All can send **or** receive | — |
| Speed | **Fast** | **Slower** |
| **Works well for** | Two-party communication; **any member of a group sending to the group**; **saving something to read later** | **Everyone sending encrypted messages to a leader** (**confidentiality**); **leader sending everyone a message** (**integrity**/authenticity) |
| **Poor for / difficulty** | **Each pair needs a unique key** | **Saving something to read later**; **group members sending to the group** |

---

## Part 7 — 🆕 Backdoors and Key Escrow

### 31. Backdoors ★
- A **backdoor** lets a **third party decrypt** a message. Often proposed by **governments**. Implemented by a **super key** (decrypts anything) or **key escrow** (a **database of all keys**).
- **Key escrow motivations:** business (key lost — forgotten, employee died); governments (read messages if they believe it necessary).

### 32. Skipjack / Clipper chip (1993) ★
- **Skipjack** developed by the **NSA in 1993** for the **Clipper chip**, to secure **voice and data**; meant for adoption by **telecom companies**; **algorithm initially classified**; **significant backlash**.
- **Clipper key escrow:** every device with a Clipper chip gets a key **provided to the government in escrow**; agencies get the key if they **"established their authority"**. **Many doubted** it would be used **only after a court order**.

### 33. FBI vs. Apple (2015–16)
- **December 2015:** two terrorists killed **14 people in California**; one had an **iPhone** the FBI believed held information on other terrorists. FBI wanted Apple to unlock it; **Apple refused**.

---

## Part 8 — Using Algorithms Correctly: Modes of Operation

### 34. Use it correctly ★
- A **good algorithm used badly is not secure.** Most algorithms encrypt **small blocks**, so a **mode** defines how to handle longer messages.
| Algorithm | Block size |
|---|---|
| **AES** | **128 bits** |
| **DES** | **64 bits** |
| **RSA** | **(key size − 1)/8** — slide: for a 2K key, "255 bits" *(that figure looks like bytes, not bits: (2048−1)/8 ≈ 255 bytes)* |

### 35. ECB — Electronic Codebook ★
- Blocks encrypted **separately**.
- **Problem:** **identical plaintext blocks → identical ciphertext blocks.** Slide: an image encrypted in ECB still **shows the original image's outline** (patterns leak).

### 36. CBC — Cipher-Block Chaining ★
- Each **plaintext block is XORed with the previous ciphertext block** *before* encryption *(slide wording says the ciphertext block is XORed with the next plaintext block)*. A **one-bit change in a plaintext block affects all following ciphertext blocks**.
- **Decryption needs only the previous ciphertext block** → **can be parallelized**.

### 37. Initialization Vector (IV) ★
- A **dummy ciphertext block to start the chain.**
- **IV need not be secret**, but **should be random**. A **new IV each time** → identical messages give **different ciphertext**.
- Often **prepended to the ciphertext**. Should be **protected with a MAC** for message integrity.

### 38. CFB — Cipher Feedback
- **Encrypt the IV** to make a **"random" stream of bits**, **XOR with plaintext**.
- **CFB decryption uses the *encryption* algorithm**, not the decryption algorithm.

### 39. GCM — Galois/Counter Mode ★
- **Encrypts an incremented IV (counter)** to produce a value **XORed with the plaintext**. **Easily parallelized.** **Used by Zoom.**
- Instructor's aside: putting the **password with the Zoom ID** **doesn't make the meeting any more secure.**

---

## Part 9 — Wi-Fi Encryption

### 40. WEP ★
- **Wired Equivalent Privacy**, **1997**, early Wi-Fi; **severe design flaw found in 2001**. A class exercise **recovered a WEP key in a few minutes**.

### 41. WPA ★
- **WPA available 2003; WPA3 in January 2018.**
- **WPA3: AES-256 in GCM mode with SHA-384 as HMAC.** Mandatory for the **"Wi-Fi CERTIFIED"** logo.
- Keys **entered manually or retrieved from a server**. Most computers support **WPA, WPA2, WPA3**.

---

## Part 10 — 🆕 Cryptographic Hashes

### 42. What a hash is ★
- A hash = result of a calculation over **all bytes of a file**. A simple hash is a **checksum** (sum of bytes).
- **Cryptographic hash:** **easy to generate**; **infeasible to generate a file matching a given hash**; **changes extensively if any part of the file changes**.
- Based on a **one-way function**: **hash = f(text)**; **very difficult to find text for a given hash** (not mathematically proven impossible, just appears hard).

### 43. Why a simple sum is weak ★
- Message `12 02 17 24 09` → hash **64**. Change one byte (`12 02 17 13 09`) → **53** (detected).
- But `12 04 15 24 09` → also **64** → **collision**; an attacker can alter a message and keep the same sum.

### 44. Hash standards ★
| Algorithm | Notes |
|---|---|
| **MD5** | Found **less secure** (slide shows MD5 of a mission statement as a visual) |
| **SHA-1** | **1993**, **160-bit** hash |
| **SHA-3** | Newer, more secure; **224, 256, 384, or 512 bits** |
| **BLAKE3** | **2020**; **efficient and parallelizable** |

*(Hashes provide **integrity**; a keyed version — **MAC/HMAC** — is used with IVs and in WPA3.)*

---

## Part 11 — Course Housekeeping

### 45. Tales of Alice and Bob
- Instructor drew illustrations of encryption concepts, at **comp476.online**.

### 46. COMP476 online
- Classes on **Zoom** (ID **941 8575 9617**) plus in person in **Graham 210**; **recordings on YouTube** via **Canvas → Modules / videos**.
- **Monday, Nov 16, 2026: Zoom only.** *(Date has passed as of Nov 18.)*

---

## Quick fact sheet ★
| Concept | Key fact |
|---|---|
| Not encryption | **Compression**, **Base64 encoding** (no key) |
| Caesar | **26 shifts** → brute force |
| Substitution | **26! ≈ 4×10²⁶** keys, broken by **letter frequency** |
| Vigenère | **Cᵢ = (Pᵢ+Kᵢ) mod 26**; repeating key → key size found via match distances |
| One-time pad | **XOR**, **truly random**, key ≥ message length → **perfect** |
| Diffusion / confusion | 1 bit change flips ~**half** of 128 output bits / substitution |
| Symmetric | Same key; **DES, AES**; fast; need **secure key sharing** |
| Asymmetric | Public/private; **RSA**; slower; math-related keys |
| DES | **56-bit key, 64-bit block, 16 rounds**; EFF cracked in **~2 days (1998)** |
| 3DES | **E-D-E**; **112/168-bit**; deprecated **2017 / 2023** |
| AES | **Rijndael, 2000**; **128-bit block**; **128/192/256 keys**; **10/12/14 rounds**; **4×4 state** |
| AES round | **SubBytes → ShiftRows → MixColumns → AddRoundKey** (last round **no MixColumns**) |
| RSA | **n=pq, Φ=(p−1)(q−1), gcd(e,Φ)=1, d=e⁻¹ mod Φ**; **c=mᵉ mod n, m=cᵈ mod n**; 0<m<n |
| RSA strength | **Factoring n** (2048-bit: ~**10²⁰⁵** divisions) |
| Hybrid | **RSA encrypts AES session key**, then **AES for data**; key discarded after session |
| TLS | Hybrid; **SSL deprecated 2015**; usually **server-only auth** |
| Backdoor | **Super key** or **key escrow**; **Clipper/Skipjack (NSA, 1993)** |
| ECB / CBC / GCM | ECB leaks patterns; CBC chains + **IV**; GCM = **counter**, parallel, **Zoom** |
| WEP → WPA | WEP flaw **2001**; WPA **2003**; WPA3 = **AES-256-GCM + SHA-384** |
| Hashes | One-way; **SHA-1 160-bit**, **SHA-3 224–512**, **BLAKE3 (2020)**, **MD5 weak** |

---

## Slide errata (flagged, not changed above unless noted)
- AES key size typo: "**194** bits" → should be **192**.
- RSA example 2: "d = e⁻¹ mod 11200" → should be **mod Φ(n)**.
- RSA block size "255 **bits**" for a 2K key likely means **bytes**.
- CBC slide wording reversed (it's **plaintext XOR previous ciphertext**).
- Exponents on slides (2ⁿ, 2⁴⁷, 2⁵⁷, 10²⁰⁵) lost their superscripts in the PDF text; I restored them from context — **check the originals if the exam asks for exact numbers**.
- Several slides are **diagram-only** (cipher-symbol image, S/K box diagram, DES stage, ECB image, hybrid step figures); they're described in words here.
