# COMP476 — Networked Computer Systems
## Lecture 26 — Transport Layer Security (slides: "TLS," file dated Nov 16, 2026)

**Relationship to earlier lectures:** builds directly on Lectures 23–24 (Encryption, Hashing/Digital Certs) — TLS is where RSA + AES + certificates get used together — and on the TCP handshake from Lecture 16 (Transport Layer). It also touches Lecture 25 (SYN flood = abuse of the TCP handshake; HTTPS ≠ safe site). The last third of the deck is a short **ICMP** section (leftover network-layer material). ★ = most likely exam-worthy.

*Numbering note: filed as Lecture 26 (the 26th note) since Lecture 25 is the last one in the project. Rename if your numbering differs. The slide-schedule slot for Mon Nov 16 (online) lists "Firewalls & review," but this deck is TLS + ICMP.*

---

## Part 1 — What TLS Is

### 1. Purpose ★
- **TLS aims to provide privacy and data integrity between two or more communicating applications.**
- **Encrypts all data between client and server.**
- Widely used in **email, instant messaging, VoIP**.
- **Provides the security in HTTPS.**

### 2. Port 443 ★
| | HTTP | HTTPS |
|---|---|---|
| **Port** | **80** | **443** |
| Security | none | **encrypted connection** |

- A server **can support both**.
- Browser connects to **443** → **encrypted connection is established**.
- Connect to **80** → server may return **`301 Moved Permanently`** redirecting the browser to the secure connection.

### 3. SSL → TLS timeline ★
| Event | Date |
|---|---|
| **SSL** created at **Netscape**, released publicly | **Feb 1995** |
| **TLS** created | **Jan 1999** |
| **SSL deprecated** | **June 2015** |

- RFC 2246 quote on slide: differences between TLS and SSL 3.0 are **"not dramatic, but significant enough to preclude interoperability between TLS 1.0 and SSL 3.0."** (They can't talk to each other.)

---

## Part 2 — TCP Handshake (prerequisite) ★

- **TCP is connection-oriented:** sender and receiver must **agree to communicate**. **All packets are numbered and acknowledged.**
- **Three steps:**
  1. **SYN** — client sends a connection request
  2. **SYN-ACK** — server responds with connection parameters
  3. **ACK** — client acknowledges
- **Sequence numbers are established** during this handshake.
- *(This is exactly what the SYN flood in Lecture 25 abuses: SYNs sent, final ACK never arrives.)*

---

## Part 3 — How TLS Works

### 4. Basic concept — hybrid encryption ★★
**Hybrid = asymmetric (RSA) to exchange a secret, then symmetric (AES) for the bulk data.**

1. **Client sends a request to the server listing its options** (ciphers, etc.).
2. **Server picks the cipher and hash function** and **sends its digital certificate** to the client.
3. **Client generates a random number, encrypts it with the server's RSA (public) key,** and sends it to the server.
4. **Both sides derive the same AES key from that random number.**
5. **Data is encrypted with AES** and transmitted.

*Why hybrid:* RSA is slow, AES is fast → use RSA only for the one small key exchange.

### 5. The handshake diagram ★★
Slide shows a timeline with a **34 ms one-way delay** (so each round trip = 68 ms):

| Time | Phase | Messages |
|---|---|---|
| 0 ms → 68 ms | **TCP handshake (68 ms)** | **Connection Request → Connection Acknowledged** |
| 68 ms | TLS begins | Client → **ClientHello** |
| 102 ms | | Server → **ServerHello, Certificate, ServerHelloDone** |
| 136 ms | | Client → **ClientKeyExchange, ChangeCipherSpec, Finished** |
| 170 ms | | Server → **ChangeCipherSpec, Finished** |
| 204 ms | **TLS total = 136 ms** | Client → **Application Data** (first encrypted data) |
| 238–272 ms | | Server → **Application Data** (reply arrives at client by 272 ms) |

**Message meanings (what each does):**
- **ClientHello** — client's supported options/ciphers (+ a random value).
- **ServerHello** — server's chosen cipher/hash.
- **Certificate** — server's digital certificate (identity + public key).
- **ServerHelloDone** — server is finished with its part of the negotiation.
- **ClientKeyExchange** — the encrypted random number (pre-master secret) from Step 4 above.
- **ChangeCipherSpec** — "from now on I'm switching to the agreed encryption."
- **Finished** — first message sent under the new keys; confirms handshake wasn't tampered with.

**Takeaway ★:** before any web data moves, you pay **1 round trip for TCP (68 ms) + 2 round trips for TLS (136 ms) = 204 ms** → this latency cost is the motivation for QUIC (Part 4).

### 6. Authentication ★
- **Server sends its digital certificate;** it contains the **name of the server**.
- **Client must check the certificate belongs to that server.**
- **Client should also check whether the certificate has been revoked.**
- **Optional client authentication exists,** but is **rarely done in web browsing.**
- *(Ties to Lecture 24: the certificate is signed by a CA, which is how the client trusts the server's public key.)*

### 7. OWASP TLS suggestions ★
- Many **cipher suites** exist in TLS.
- **Where possible, enable only GCM ciphers.**
- **Use TLS for ALL pages, not just ones with sensitive data.**
- **Use the `Secure` cookie flag** (cookie is only sent over HTTPS — ties to Lecture 21 Cookies).

---

## Part 4 — QUIC ★

### 8. What it is
- **QUIC is a transport-layer protocol that uses UDP** (not TCP).

### 9. Protocol stack diagram
| | HTTP/1.1 stack | HTTP/2 stack | **HTTP/3 stack** |
|---|---|---|---|
| App | HTTP 1.1 | HTTP/2 | **HTTP/3** |
| Security | **TLS/SSL (optional)** | **TLS 1.2+** | **TLS 1.3 (built into QUIC)** |
| Transport | **TCP** | **TCP** | **QUIC** |
| Below that | | | **UDP** |
| Network | IPv4 / IPv6 | IPv4 / IPv6 | IPv4 / IPv6 |

- All three share the same **"HTTP semantics"** on top.
- In HTTP/3, **TLS 1.3 sits inside QUIC** — security is not a separate layer on top of the transport.

### 10. QUIC startup (diagram) ★★
- **QUIC sends the server certificate during initial setup** — it combines transport + crypto handshakes.
- **TCP + TLS 1.2 path:** SYN → SYN-ACK → ACK + ClientHello → ServerHello, Cert, ServerKeyExchange → ClientKeyExchange, ChangeCipherSpec, Finished → ChangeCipherSpec, Finished → **data** (many round trips).
- **QUIC path:** Client **Initial, Hello** → Server **Init., Hello, Cert, Fin** → Client **Fin, data** (**data goes after ~1 round trip**).
- **Takeaway:** QUIC **cuts connection-setup latency** versus TCP + TLS.

---

## Part 5 — IP Best-Effort Delivery & ICMP

### 11. Best-effort delivery ★
- **IP datagrams may be:** **lost, duplicated, delayed, delivered out of order, or delivered with corrupted data.**
- **IP does not attempt to correct problem packets.**
- **Higher-layer protocol software must handle each of these errors** (e.g., TCP).

### 12. ICMP — Internet Control Message Protocol ★
- **Error reporting and network-management system.**
- Goal: **keep the Internet running as efficiently as possible.**
- **Works to correct network problems, but does NOT try to correct individual packet problems.**

### 13. ICMP packets & encapsulation ★
- ICMP messages are **sent inside IP datagrams**, with **no special priority.**
- **An error in an IP packet never creates an error message**.
- **Two levels of encapsulation:**
  ```
  Frame header | Frame data
                 └─ IP header | IP data area
                               └─ ICMP header | ICMP data area
  ```
  - **ICMP message → inside an IP datagram → inside a frame** for the physical network.

### 14. Uses of ICMP ★★ (slide lists 7)
| # | Use | Plain English |
|---|---|---|
| 1 | **Echo (ping)** | Is the host reachable? |
| 2 | **Tells source a packet exceeded the TTL** | Packet died in transit (this is how traceroute works) |
| 3 | **Tells source a packet had to be fragmented** | Packet was too large for a link |
| 4 | **Redirect host to a better router** | "Send it via the other router next time" |
| 5 | **Controls the rate a host sends packets (Source Quench)** | "Slow down" |
| 6 | **Destination unreachable** | **Either the network or the host is down** |
| 7 | **Request subnet mask** | Host asks what its subnet mask is |

### 15. Multiple Router Network (diagram)
- Slide shows **three LAN segments**: **B1, B2** on one segment, **A1, A2** on a middle segment, **C1, C2** on a third. **Router1 connects the B segment ↔ the A segment; Router2 connects the A segment ↔ the C segment.**
- The slide has **no text**; it's setup for the **ICMP redirect** idea (a host on segment A could pick the wrong router for a destination, and a router tells it a better one). This reading is inferred from the order of the slides — the slide itself doesn't say.

---

## Quick fact sheet ★
| Concept | Key fact |
|---|---|
| TLS purpose | **Privacy + data integrity**; secures HTTPS, email, IM, VoIP |
| HTTP / HTTPS ports | **80 / 443**; port 80 may answer **301 Moved Permanently** |
| SSL / TLS dates | SSL **Feb 1995** (Netscape); TLS **Jan 1999**; SSL deprecated **June 2015** |
| TLS 1.0 vs SSL 3.0 | **Not interoperable** |
| TCP handshake | **SYN → SYN-ACK → ACK** |
| TLS = hybrid | **RSA** to share a random number, **AES** key derived from it for data |
| Server picks | **Cipher + hash function**; sends **digital certificate** |
| Client checks | Cert **belongs to the server** and **isn't revoked** |
| Client auth | **Optional, rarely used** on the web |
| Handshake cost (slide) | **TCP 68 ms + TLS 136 ms = 204 ms** before first data |
| OWASP | **GCM ciphers only; TLS on all pages; `Secure` cookie flag** |
| QUIC | **UDP-based**, TLS 1.3 built in, **cert sent in initial setup** → faster start; basis of **HTTP/3** |
| IP delivery | **Best effort** — lost / duplicated / delayed / out of order / corrupted |
| ICMP | **Error reporting + management**, rides in **IP datagrams**, an error in an IP packet never creates an error message |
| ICMP uses | **Echo, TTL exceeded, fragmentation, redirect, source quench, unreachable, subnet mask request** |

---

## Slide errata / cautions (flagged, not changed above unless noted)
- **"Establishing a TCP Connection" / "Connection Handshake" slides** have typos ("Users sends…"); the content is the standard three-way handshake.
- **RSA key exchange is the slide's simplification.** The slide says the client encrypts a random number with the server's RSA key. That is how **TLS 1.2 with RSA key exchange** works (matching the handshake diagram: ClientKeyExchange, ChangeCipherSpec). *My addition, not on the slides:* modern TLS 1.3 **removes RSA key exchange** and uses **ephemeral Diffie–Hellman**, and its handshake takes **1 round trip** instead of 2. Learn the slide version for the exam.
- **Handshake timing diagram:** the numbers (34 ms one-way) are a **teaching example**, not a measured result. The diagram also shows only "Connection Request / Connection Acknowledged" for TCP (the third handshake ACK is not drawn).
- **"Source Quench" is obsolete** — *my addition:* it was **deprecated** (RFC 6633) and modern routers/hosts ignore or don't send it; the slide still lists it, so know it for the exam.
- **QUIC slide credit is Wikipedia;** the HTTP/3 box shows **TLS 1.3 inside QUIC** — it is built in, not layered on top.
- **ICMP section** is leftover network-layer material, not TLS, and the **Multiple Router Network slide has no text** — described in words above.
- **Image-only slides** (title/quote slide with the Bruce Schneier password quote, handshake timeline, QUIC stack/startup diagrams, ICMP encapsulation figure, router diagram) are **described in words**; the quote on slide 1 is not exam material.
- **Exam logistics:** Exam 3 is **today, Wed Nov 18**; the TLS and ICMP topics above come from the deck dated Nov 16, so check whether the exam covers them (the schedule in Lecture 25 lists Nov 16 as "Firewalls & review").
