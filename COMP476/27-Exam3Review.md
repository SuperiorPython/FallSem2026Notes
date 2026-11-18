# COMP476 — Networked Computer Systems
## Lecture 27 — Exam 3 Review (slides: "Exam 3 Review," file dated Nov 16, 2026)

**What this deck is:** a 57-slide recap of everything on Exam 3, mostly pulled from Lectures 16–26 (Transport Layer, HTTP/RPC, Email, Cookies, Encryption, Hashing/Certs, Web Attacks). The last slide lists the **likely exam questions** — study those first. ★ = most likely exam-worthy.

**★★ Slide 57 — "Likely Questions":**
1. **Write a socket program**
2. **Select the best encryption for a situation**
3. **HTTP details**
4. **Cookies**

*Numbering note: filed as Lecture 27 (the 27th note), following Lecture 26 (TLS). Rename if your numbering differs.*

---

## Part 1 — Transport Layer & Ports

### 1. Transport layer ★
- **First end-to-end layer.** Stack: Application → **Transport** → Internet → Logical Data Link → Media Access Control → Physical. Only the end hosts run the Transport layer; routers in the middle stop at Internet.

### 2. Ports ★
- A **port is a 16-bit number** identifying the application tied to a network connection (so 0–65,535).
- **Nothing to do with physical outlets** on the back of the computer.
- Applications can pick their own port **as long as it isn't already in use**.
- **The first 1024 port numbers are reserved** for common network services.

### 3. TCP vs UDP ★ (slide is a two-column graphic)
| TCP | UDP |
|---|---|
| **Connected** | **Connectionless** |
| **State memory** | **Stateless** |
| **Byte stream** | **Packet / datagram** |
| **Ordered data delivery** | **No sequence guarantee** |
| **Reliable** | **Lossy** |
| **Error free** | **Error packets discarded** |
| **Handshake** | **No handshake** |
| **Flow control** | **No flow control** |
| **Relatively slow** | **Relatively fast** |
| **Point to point** | **Supports multicast** |
| **Security: SSL/TLS** | **Security: DTLS** |

---

## Part 2 — TCP Reliability

### 4. Sliding window flow control ★
- Receiver advertises how much space it has; sender can only send that much.
- Slide example: sender sends **2K**, told **2K available**. After another **2K** the window is full → **sender stops** until space frees up, then continues.

### 5. Reliable delivery ★
- TCP runs on **IP, which is connectionless and unreliable.**
- TCP adds a **connection-oriented** service that corrects **lost, corrupted, out-of-order, and delayed** packets.

### 6. Sequence numbers & ACKs ★
- Sender puts a **sequence number in each packet = number of bytes sent.**
- A packet arriving with a **previously received sequence number can be discarded** (duplicate).
- The receiver sends **ACKs with a sequence number** to confirm receipt.

### 7. Cumulative ACKs ★★
- Default ACK = **cumulative**: acknowledges **everything up to** that sequence number.
- If packet **#100 is lost, the receiver can't ack anything above it → it re-ACKs #99.**
- **Worked example (slide):** receiver got bytes **6,000–7,999** but missed **5,000–5,999** → ACK **4,999** → sender **retransmits 5,000–7,999** (even the parts that arrived).

### 8. Selective ACK (SACK) ★★
- SACK option lets the receiver **acknowledge discontinuous blocks.**
- Same example: **ACK 4,999 + SACK 6,000–7,999** → sender **only retransmits 5,000–5,999.**

---

## Part 3 — Sockets ★★★ (exam question #1)

### 9. Socket program flow (slide diagram)
| Step | **Client side** | **Server side** |
|---|---|---|
| 1 | **socket** | **socket** |
| 2 | **connect** | **bind** |
| 3 | **send** | **listen** |
| 4 | **recv** (loops back to send) | **accept** |
| 5 | **close** | **recv** (loops with send) |
| 6 | | **send** |
| 7 | | **close** → loops back to **accept** (next client) |

- **Memory aid:** server = **S**ocket, **B**ind, **L**isten, **A**ccept, then **R**ecv/**S**end. Client = socket, connect, send/recv.
- Server **binds** to a port, **listens**, **accepts** (blocks until a client connects, returns a **new connection socket + client address**).

### 10. Example server (Python, from slide)
```python
import socket
HOST = "127.0.0.1"   # This host's IP address, localhost
PORT = 47600         # Port where the program will listen

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.bind((HOST, PORT))
    sock.listen()
    conn, addr = sock.accept()
    with conn:
        print(f"Connected by {addr}")
        data = conn.recv(1024)
        print(f"Received {data!r}")
        conn.sendall(data)      # echoes back what it received
```

### 11. Example client (Python, from slide)
```python
import socket

HOST = "whatever.ncat.edu"   # The server's hostname or IP address
PORT = 47600                 # The port used by the server

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.connect((HOST, PORT))
    sock.sendall(b"Hello, world\n")
    data = sock.recv(1024)

print(f"Received {data!r}")
```

**Key details to memorize ★:**
- **`AF_INET`** = IPv4; **`SOCK_STREAM`** = **TCP** (UDP would be `SOCK_DGRAM`).
- Data is **bytes** (`b"..."`), not str.
- `recv(1024)` = read **up to** 1024 bytes.
- **Server: `bind` → `listen` → `accept`. Client: `connect`.**
- Client and server must use the **same port**.

---

## Part 4 — HTTP ★★★ (exam question #3)

### 12. Request format
- HTTP is sent in **readable ASCII**.
- **The request is always terminated by a blank line.**
- Format:
  ```
  Method filename HTTP/1.1
  options
  (blank line)
  ```
  Each line ends with a **carriage return / end-of-line** character.

### 13. Example GET
```
GET /mypage.html HTTP/1.1
HOST: comp476.online
```
- Asks the server to send **mypage.html**; browser says it's using **HTTP version 1.1**.
- **The `HOST` option is required** (in 1.1).
- Request ends with **two end-of-line characters** (the blank line).

### 14. Real browser GET (slide, Chrome)
Headers shown: `Host`, `Sec-Ch-Ua*` (browser/platform hints), `Accept-Language`, `Upgrade-Insecure-Requests`, **`User-Agent`**, **`Accept`** (content types the browser takes), `Sec-Fetch-*`, **`Accept-Encoding: gzip, deflate, br`**, `Priority`, **`Connection: keep-alive`**.

### 15. Server response ★
```
HTTP/1.1 statuscode reason
response options
(blank line)
file contents
```
- Status line = **protocol version + status code + reason**; then headers; then the content.
- **Slide example:** `HTTP/2 200 OK`, `Server: openresty`, `Date`, `Content-Type: text/html`, `Vary: Accept-Encoding`, `Last-Modified`, **`Cache-Control: max-age=500, public, proxy-revalidate`**, `Expires`.

---

## Part 5 — Cookies ★★★ (exam question #4)

### 16. What a cookie is
- **Small piece of data stored by the browser** on the client.
- **Sent and received with no interaction or knowledge of the user**; **not displayed** by the browser.

### 17. Why: stateless servers ★
- **Web servers and REST API servers are stateless:** each request is independent, **though the server may save info about previous requests.**
- **Cookies let the server connect saved info to a client on a second request.**
- Server code can save **session information**, found later through the cookie.
- **Frequently the cookie only contains a database key** to look up the real data.

### 18. Session vs persistent ★★
| | **Session cookie** | **Persistent cookie** |
|---|---|---|
| Stored | **Temporary memory** only | **File system** on the client |
| Lifetime | **Deleted when browser closes** | **Has an expiration date** |
| Expiry | **No expiration date** | `expires=` date or **`Max-Age`** (seconds until expiry) |
| Other | **May be safer** (only the browser can access) | Called **tracking cookies** — advertisers use them to record browsing habits |

**Example header (slide):**
```
Set-Cookie: insomnia_cookie=eyJp…3D; expires=Thu, 18-Feb-2023 18:17:15 GMT;
Max-Age=3600; path=/; HttpOnly; Secure; Domain=.insomniacookies.com
```
- **`Secure`** = only sent over HTTPS; **`HttpOnly`** = not readable by JavaScript; **`Domain`/`path`** = where it's sent. *(The slide only shows these flags; the meanings are from earlier lectures / standard behavior.)*

---

## Part 6 — Email & SMS

### 19. Email overview
- Email is **a file with a specific header format** moved to the recipient's mailbox.
- **Two parts:** **sending** the email, and **viewing/editing** it.

### 20. Standards ★
| Standard | Job |
|---|---|
| **X.400** | Defines the **email file format** |
| **SMTP** (IETF **RFC 2821**) | How mail is **sent between mail servers** |
| **POP and IMAP** | **Retrieving** mail from your mailbox |

### 21. SMS ★
- Runs on the **telephone system**, based on **SS7 (Signaling System No. 7)** — 1970s telephony signaling used to **set up and tear down calls** on the public switched telephone network.
- Sends data on the phone's **signaling channel** (the one used for network control); uses **unused resources** → minimal cost.
- **7-bit default character set** (good for English/western languages).
- **160 characters × 7 bits = 140 bytes.**

---

## Part 7 — Encoding & Encryption ★★★ (exam question #2)

### 22. Base64 ★
- Purpose: send **binary data over networks that expect regular characters.** Some binary values look like punctuation/control chars (**return, comma, ETX, NAK, null**) and confuse systems. *(Slide: 256 possible byte values vs ~92 keyboard characters.)*
- **Converts any data to alphanumeric characters.**
- **Encoding is NOT encryption — there is no key.**
- **How:** **3 bytes (24 bits) → four 6-bit values.** 2⁶ = **64** values: **26 uppercase + 26 lowercase + 10 digits + "+" and "/"**.
- **Slide example:** bytes `X`, `CR`, `LF` = `01011000 00001101 00001010` → regroup as `010110 | 000000 | 110100 | 001010` = 22, 0, 52, 10 → **W A 0 K**.

### 23. Letter frequency
- Frequency of letters in English (also letter pairs, triples, short words) is public → **breaks simple substitution ciphers** (frequency analysis).

### 24. Symmetric vs asymmetric ★★
| | **Symmetric (secret key)** | **Asymmetric (public key)** |
|---|---|---|
| Keys | **Same key** encrypts and decrypts | Decrypt key is **different but related** |
| Key distribution | **Must be shared securely first** (before communication) | **Encryption key can be public**, decryption key stays secret |
| Examples | **DES, AES** | **RSA** |
- Asymmetric flow (slide): User1 looks up **User2's public key** in a **public key directory**, encrypts; User2 decrypts with **their private key**.

### 25. Hybrid cryptography ★★ (this is what TLS does)
- **Step 1:** **AES key is encrypted with the receiver's public key** (RSA) and sent → both sides now share the AES key.
- **Step 2:** **All following messages are encrypted with AES**; the **AES key is discarded after the session.**
- *Why:* asymmetric is slow, symmetric is fast.

### 26. Block cipher modes ★
| Mode | How it works | Notes |
|---|---|---|
| **ECB** (Electronic Codebook) | Data divided into blocks, **each encrypted separately** | Identical plaintext blocks → identical ciphertext blocks |
| **CBC** (Cipher-Block Chaining) | Blocks are **chained** (XOR with previous ciphertext block before encrypting) | **A one-bit change in a plaintext block affects all following ciphertext blocks** |
| **GCM** (Galois/Counter) | **Encrypts an incremented IV**; result is **XORed with the plaintext** | **Easily parallelized; used by Zoom**; OWASP says to prefer GCM (Lecture 26) |

### 27. Message Authentication Code (MAC) ★★
- Confirms **a message came from the stated sender.**
- **Created by hashing the message, encrypting the hash, and appending it to the packet.**
- **Checking:** receiver **hashes the message, decrypts the received hash, and compares** the two (slide diagram: MSG → Hash → Encrypt → send MSG + Hash; receiver: Hash the MSG, Decrypt the hash, **compare**).
- **Encrypting the hash prevents anyone from forging it without the key.**

### 28. Digital certificates ★
- Slide diagram (left → right): **Checksum of Public Key** (encrypted with the **CA's private key**) | **Alice's Public Key** | **Checksum of data** (encrypted with **Alice's private key**) | **data**.
- The **CA's signature** vouches that the public key is Alice's; **Alice's signature** over the data proves it came from her.

### 29. Diffie–Hellman key exchange ★★ (numbers are on the slide — know how to compute)
- **Public:** modulus **p = 23**, base **g = 7**.
- **Alice** secret **a = 4** → sends **A = gᵃ mod p = 7⁴ mod 23 = 9**
- **Bob** secret **b = 5** → sends **B = gᵇ mod p = 7⁵ mod 23 = 17**
- **Alice computes** s = Bᵃ mod p = **17⁴ mod 23 = 8**
- **Bob computes** s = Aᵇ mod p = **9⁵ mod 23 = 8**
- **Shared secret = 8.** **Private:** a, b, s. **Public:** p, g, A, B.
- *(I checked the arithmetic: 7⁴=2401≡9, 7⁵=16807≡17, 17⁴≡8, 9⁵≡8 mod 23.)*

### 30. "Select the best encryption" cheat guide ★★★
*(My summary of the slide content, not a slide.)*
| Situation | Best choice |
|---|---|
| Two parties who **already share a secret key**, bulk data | **Symmetric (AES)** — fast |
| Parties who **have never met / no shared key** | **Asymmetric (RSA)** to exchange a key, or **Diffie–Hellman** |
| Large amount of data over an open network with no prior key | **Hybrid:** RSA/DH to share an AES key, then **AES** for the data |
| Need to prove a **message is from the claimed sender / unaltered** | **MAC** (hash + encrypt the hash) |
| Need to prove a **public key belongs to someone** | **Digital certificate** (signed by a CA) |
| Encrypting many blocks, want **speed/parallelism** | **GCM** mode (not ECB — it leaks patterns) |
| Just need binary → text for transport | **Base64** (not a security measure) |

---

## Part 8 — Network Attacks ★

### 31. ARP poisoning
- Malicious system sends an **ARP reply with an incorrect MAC address for an IP**; the victim then **sends packets to the attacker** thinking they go to the right IP.
- **Defenses:** **static, read-only ARP entries** for critical services; **check MAC addresses against the DHCP server**; **SEND (Secure Neighbor Discovery)** in IPv6 — uses **encryption and certificates**.

### 32. DNS attacks & DNSSEC
- Original DNS had **no security.** An attacker **masquerading as a DNS server** sends false info → **cache poisoning** → users sent to malicious sites.
- **DNSSEC:** **cryptographic authentication of data, authenticated denial of existence, and data integrity**, using **digital signatures.**

### 33. Ping flood ★
- Attacker **broadcasts an ICMP echo packet** to all computers on a network with the **return address forged as the victim's.** Every computer **sends an ICMP echo-reply to the victim**, flooding its network resources.
- **Mitigation:** **don't accept ICMP echo sent to a broadcast address**; **many firewalls (including A&T's) block ICMP echo.**

### 34. SYN flood ★
- TCP connection = **three messages** (SYN, SYN-ACK, ACK), then data.
- Attacker **sends many SYNs and never sends the final ACK.** The **half-open connections tie up ports until the victim times out**, so **others can't connect.**

### 35. SQL injection ★★
- Vulnerable pattern: user input is **concatenated directly into the query string.**
```php
$name = $HTTP_POST_VARS["user"];
$pwd  = $HTTP_POST_VARS["password"];
$query = "select * from login where username = '" . $name . "' and password = '" . $pwd . "'";
$result = mysql_query($query);
if ($result->num_rows > 0)   // allow access
```
- Normal: `... username = 'KenW' and password = 'Secret'`
- **Attack:** password = **`x' or 'a' = 'a`** → `... and password = 'x' or 'a' = 'a'` → **`'a'='a'` is always true**, so the query returns rows. **The username doesn't even have to match anyone in the database.**
- *Fix (my addition, not on these slides):* **parameterized / prepared statements** instead of string concatenation.

### 36. Misleading links & phishing ★
- A link has **both a URL and display text; the text can lie** (shows `www.warmandfuzzy.com`, goes to `www.spidersandsnakes.com`).
- **Phishing** = fraudulently getting sensitive info **by masquerading as a trustworthy entity.** **Spear phishing** = the **targeted** version.

---

## Quick fact sheet ★
| Concept | Key fact |
|---|---|
| Likely exam Qs | **Socket program, best encryption for a situation, HTTP details, cookies** |
| Port | **16-bit**; **first 1024 reserved** for common services |
| TCP / UDP | TCP: connected, reliable, ordered, flow control, slower. UDP: connectionless, lossy, fast, multicast |
| TCP ACK | **Cumulative** (re-ACKs last in-order byte); **SACK** acks gaps |
| Socket server | **socket → bind → listen → accept → recv/send → close** |
| Socket client | **socket → connect → send/recv → close** |
| HTTP request | `Method file HTTP/1.1` + options + **blank line**; **HOST required** |
| HTTP response | `HTTP/1.1 code reason` + headers + blank line + content |
| Cookies | Browser-stored, **server uses them to remember stateless clients**, often just a **DB key** |
| Session cookie | Memory only, deleted at browser close, no expiry |
| Persistent cookie | File system, **expires / Max-Age**, tracking |
| Email | **X.400** format, **SMTP** (RFC 2821) server-to-server, **POP/IMAP** retrieval |
| SMS | **SS7** signaling channel; **7-bit**, **160 chars = 140 bytes** |
| Base64 | **3 bytes → 4 six-bit chars**; **encoding ≠ encryption** |
| Symmetric / asymmetric | **AES, DES** / **RSA** |
| Hybrid | **RSA shares AES key → AES for data → discard key** |
| Modes | **ECB** separate blocks; **CBC** chained; **GCM** counter, parallel (Zoom) |
| MAC | **Hash → encrypt hash → append → receiver recomputes & compares** |
| Diffie–Hellman | p=23, g=7, a=4, b=5 → A=9, B=17 → **s=8** |
| ARP poisoning | Fake MAC reply; **static ARP, DHCP check, SEND** |
| DNS attack | **Cache poisoning**; fix = **DNSSEC** (digital signatures) |
| Ping flood | Forged-source **ICMP echo to broadcast**; block echo to broadcast |
| SYN flood | **SYNs with no final ACK** exhaust half-open connections |
| SQL injection | `x' or 'a' = 'a` → always true |
| Phishing | Masquerading; **spear phishing** = targeted |

---

## Slide errata / cautions (flagged, not changed above unless noted)
- **CBC slide wording is backwards-ish.** It says "each block of ciphertext is XOR with the *next* block of plaintext before encryption." *My correction:* it's the **previous ciphertext block XORed with the next plaintext block, then encrypted** (what the table above says). Same idea, clearer direction.
- **Cookie slide is truncated** ("Frequently cookies only contain a database key to look…" cuts off). Completed as "to look up saved data," which matches the lecture on stateless servers.
- **Ping flood slide** says the **attacker broadcasts the echo**; this is the classic **Smurf-style amplification** (my label, not on the slide).
- **Socket examples** bind to `127.0.0.1` (localhost) — a server meant to take outside connections would bind to **`0.0.0.0`** or its real address (my addition).
- **Password slide in the SQL example** mixes `$user`/`$username`; the idea is identical.
- **SQL slide** shows the old PHP `mysql_query` API (long deprecated) — only the **concept** matters for the exam.
- **Image-only slides** (quote title slide with Melinda French Gates quote, TCP vs UDP, socket flow, letter frequency chart, Base64 bit diagram, hybrid/asymmetric diagrams, MAC process, digital certificate, TCP handshake, SYN flood) are **described in words** above; the quote is not exam material.
- **Scope:** this deck covers material from earlier lectures. TLS (Lecture 26) and ICMP/firewalls are **not** on these review slides, so lean on Lecture 26 notes for those if the exam touches them.
