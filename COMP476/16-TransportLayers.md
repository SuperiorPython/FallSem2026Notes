# COMP476 — Networked Computer Systems
## Lecture 16 — The Transport Layer (slides: "Transport Layer," Wed Oct 14, 2026; reading 7.1)

**What's new vs. Lecture 14:** this is the first **end-to-end** layer. Covers **ports**, **UDP vs. TCP**, the **TCP handshake and teardown**, the **Two Armies problem**, **flow control**, **reliable delivery** (sequence numbers, ACKs, retransmission, SACK), **congestion control**, and a **Wireshark trace** of a real TCP session.

---

### 1. Where the Transport layer sits
```
Application
Transport      ← you are here
Internet
Logical Data Link
Media Access Control
Physical
```
- Provides services **to the application layer**.
- Uses the **Network (Internet) layer** to move packets to the destination.
- Programs reach it directly through **sockets** (next lecture, Mon Oct 19).
- **First end-to-end layer:** only the two **end hosts** run Transport; the routers/links in between only go up to the Internet layer.

### 2. Addressing applications: ports ★
| Layer | Gets the packet to… |
|---|---|
| **Internet** (IP) | the correct **host** |
| **Transport** (port) | the correct **application** on that host |

- Many programs share **one network interface** at once; the Transport layer **separates the traffic flows**.
- A **port** is a **16-bit number** identifying the application tied to a network connection (range 0–65,535).
- Ports have **nothing to do with physical outlets** on the computer.
- Apps pick their own port as long as it's **not already in use**.
- **First 1024 ports are reserved** for common network services.

**Well-known ports ★**
| Port | Service |
|---|---|
| 20, 21 | **FTP** (File Transfer) |
| 22 | **SSH** (Secure Shell) |
| 23 | **Telnet** |
| 25 | **SMTP** (mail sending) |
| 53 | **DNS** |
| 80 | **HTTP** |
| 110 | **POP3** |
| 123 | **NTP** (time) |
| 143 | **IMAP** |
| 443 | **HTTPS** |

### 3. Listening & client ports
- A **server** **listens** on a chosen port and waits for clients.
- A **client** opens a port on its own computer; the **OS assigns a temporary port**.
- A connection is identified by both ends: **(client IP, client port, server IP, server port)**. *(4-tuple — not on slides.)*

---

### 4. The two Transport protocols ★
| | **UDP** (User Datagram Protocol) | **TCP** (Transmission Control Protocol) |
|---|---|---|
| Connection | **Connectionless** | **Connection oriented** |
| Extras | Almost no added functionality | **Reliable byte stream** |

### 5. UDP
- Sends **individual packets (datagrams)** to an application on another computer.
- Packets may be **lost, delayed, corrupted, or delivered out of order**.
- **Header:** contains **port numbers**, plus a **checksum** of header **and** data.
  - Checksum is **optional for IPv4**, **mandatory for IPv6**.
  - *(Not on slide: UDP header is only 8 bytes — src port, dst port, length, checksum.)*
- **Use cases:** simple/fast; **quick requests that can be repeated** (e.g. DNS); **streaming and real-time** protocols (a lost packet doesn't stall the others); **multicasting**.

### 6. TCP
- Carries about **85% of Internet traffic** (per slide).
- Provides a **reliable byte stream**: IP sends *packets*, but TCP **merges the data into a stream**.
- Runs on top of **IP (connectionless, unreliable)** and **corrects**: lost, corrupted, out-of-order, and delayed packets.

**TCP header fields ★**
| Field | Size | Purpose |
|---|---|---|
| **Sequence number** | 32 bits | **Ordering** packets + error detection. **Initial** value is **randomly chosen** and sent during connection setup |
| **Acknowledgment number** | 32 bits | Sequence number of received data |
| **Window** | 16 bits | **Flow control** |
| **Checksum** | 16 bits | Detects errors in **header and data** |

*(Also in the header, not detailed on slides: source/dest ports, flags SYN/ACK/FIN/PSH, options like SACK.)*

### 7. TCP vs. UDP summary ★ (slide)
| TCP | UDP |
|---|---|
| Connected | Connectionless |
| State memory | Stateless |
| Byte stream | Packet/datagram |
| Ordered data delivery | No sequence guarantee |
| Reliable | Lossy |
| Error free | Error packets discarded |
| Handshake | No handshake |
| Flow control | No flow control |
| Relatively slow | Relatively fast |
| Point to point | Supports multicast |
| Security: **SSL/TLS** | Security: **DTLS** |

---

### 8. In-class exercise: reliable delivery over an unreliable medium
- Teams of 3: **A**, **B**, **C**. A and C each send a sentence (8–10 words, **one word per slip of paper**) to each other **through B**.
- B acts as the network: passes a slip along, **discards** it (lost packet), passes a **blank** (corrupted packet), or **delays** it.
- Decide **before starting**: how to **detect** a missing packet, what to do when one is **missing**, what to do when one is **received correctly**.
- *Takeaway:* you end up inventing **sequence numbers, ACKs, and retransmission** — exactly what TCP does (§12–§16).

---

### 9. TCP connection setup: 3-way handshake ★
```
Client                     Server
  | ---- SYN (seq=x) ------> |
  | <--- SYN+ACK (seq=y) --- |
  | ---- ACK --------------> |
```
1. Client sends a packet with the **SYN** flag.
2. Server replies with **SYN and ACK** set.
3. Client sends an **ACK**.
- **Sequence numbers are established** during this exchange; parameters are negotiated.
- IP is connectionless, so you must connect to the other device's **TCP layer** first.

### 10. Closing the connection ★
- Each **direction** closes **independently**.
- To end one direction, the sender sends a packet with the **FIN** bit; that direction is closed when the FIN is **ACKed**.
- Connection is closed when **both directions** are terminated.
- Usually, after one side sends a FIN, the other replies with **ACK + FIN** to close both sides, then a final ACK.

### 11. Byzantine Generals / Two Armies problem ★
**Setup**
- Two blue armies want to attack a third (white) army.
- **Together** they win; **alone**, either loses.
- Only communication: **messengers through white territory** — they may be **captured** (unreliable channel).
- General 1 wants to attack at dawn, and army 2 **must** attack too.

**Endless confirmations**
1. G1: "Attack at dawn" → G1 can't know G2 got it.
2. G2 ACKs → G2 can't know G1 got the ACK.
3. G1 ACKs the ACK → G1 can't know G2 got *that*.
… forever.

**Proof it can't work**
- Assume some message sequence gives **shared certainty**.
- Consider the **last message**. If it's lost, the receiver won't attack.
- The **sender** of the last message can't tell if it arrived, so it attacks alone → **contradiction**. No finite protocol guarantees agreement.
- **Why it matters for TCP:** no way to *guarantee* both sides agree the connection is closed; TCP uses ACK/FIN plus timers as a practical compromise.
- **Blockchains:** the multi-party version (some nodes may be **malicious**) is **Byzantine fault tolerance**. Needed because blockchains are **decentralized with no central authority**, so nodes must reach **consensus** even when some fail or lie.

---

### 12. Flow control: sliding window ★
- **End-to-end**: stops the sender from overwhelming the **receiver**.
- Each message from the receiver states **how many bytes it can accept** (the **Window**, 16 bits).
- Sender **may not exceed** that amount.
- **Slide example:** sender sends **2K**, told **2K** available → sends another **2K** → window full, **sender stops** → continues when the receiver frees space.

### 13. Sequence numbers & ACKs ★
- Sender puts a **sequence number** in each packet = **number of bytes sent** (byte-counting, not packet-counting).
- A packet with a **previously seen** sequence number is a duplicate → **discard**.
- **ACKs** carry a sequence number telling the sender data was received.
- **Initial sequence number (ISN)** is exchanged in the SYN/SYN-ACK handshake.
  - Should be **random**, not always starting at 1.
  - Unpredictable ISNs defend against **TCP sequence prediction attacks**.

### 14. Cumulative ACKs ★
- An ACK acknowledges **all data up to** that number.
- If packet **#100 is lost**, the receiver **can't ack anything above it** → it **re-ACKs 99**.
- **Example:** receiver gets 6,000–7,999 but missed 5,000–5,999 → ACK **4,999** → sender retransmits **5,000–7,999** (wasteful).

### 15. Stop-and-wait vs. pipelining
- **Stop-and-wait:** send one, wait for ACK, send next → **low throughput**.
- **TCP sends multiple packets before waiting for an ACK** (**overlapping transmissions**, slide diagram: left = one at a time, right = several in flight with ACKs returning).

### 16. Retransmission ★
**A. Duplicate-ACK retransmission (fast retransmit)**
- Sender gets ACKs with the sequence number of the last in-order packet.
- **3 duplicate ACKs with the same number → retransmit** the last unacknowledged packet **without waiting for a timer**.

**B. Timeout retransmission**
- On send, start a timer with a **conservative estimate** of ACK arrival time.
- **Timer expires → assume lost → retransmit** with the **original sequence number**.
- New timer set to **2× the previous value** (**exponential backoff**).

**C. Getting the timeout right**
- **Too short:** unnecessary retransmits; the sender may **mistake the original's ACK for the retransmission's** ACK and wrongly think later packets were lost.

**D. Selective ACK (SACK)**
- Lets the receiver ACK **discontinuous blocks**.
- **Example:** ACK **4,999** + SACK **6,000–7,999** → sender retransmits **only 5,000–5,999**.

---

### 17. Congestion control ★
- **Flow control** = receiver's capacity. **Congestion control** = the **network's** capacity.
- Heavier load → more loss. **Most common cause of loss: overwhelmed routers.**
- TCP **regulates its sending rate** to avoid swamping the network.

**Rate adjustment (as on slide)**
1. **Slow start:** rate **increases by two every RTT** up to a **threshold** *(really doubling; see note)*.
2. **After the threshold: additive increase** — **+1 every RTT**.
3. On **ACK timeout or triple duplicate ACK** → **cut the threshold in half** (**multiplicative decrease**).

**Congestion window graph (slide figure)**
- y-axis: congestion window (KB or packets, 0–40). x-axis: transmission rounds (RTTs, 0–48).
- **Slow start**: steep climb to **threshold ≈ 32** at ~RTT 6.
- **Additive increase**: linear climb to ≈ **39**, then **packet loss** at ~RTT 14.
- **Fast recovery**: window drops to the **new threshold ≈ 20** (about half), then linear growth again.
- Next loss at ≈ **36** (~RTT 34) → **multiplicative decrease** → threshold ≈ **17**.
- Overall shape: the **sawtooth** (AIMD: additive increase, multiplicative decrease).

> ⚠️ **Slide vs. textbook:** "increases by two every RTT" in slow start is loosely worded; in standard TCP the window **doubles** each RTT (exponential). On a **timeout**, classic TCP also resets the window to **1 segment** and re-enters slow start; the **triple-dup-ACK** path is the one that does fast recovery (window ≈ half), as drawn. Go with the slide wording on the exam unless the professor says otherwise.

### 18. OS timers
- Computers have a **real-time clock that interrupts at a constant interval**.
- The OS handles the interrupt, **increments the time counter**, and **releases any process waiting** for that time. (This is how TCP's retransmission timers are implemented.)

---

### 19. Wireshark trace: one HTTP request over TCP ★
Client = "User" (win 65535/65280); Server = web server (win 64240/64256). Sequence numbers are **relative**.

| # | From | Len (frame) | Flags | seq | ack | Payload | Meaning |
|---|---|---|---|---|---|---|---|
| 1 | User | 66 | SYN | 0 | – | 0 | Handshake step 1 |
| 2 | Server | 66 | SYN, ACK | 0 | 1 | 0 | Step 2 (acks the SYN → ack 1) |
| 3 | User | 54 | ACK | 1 | 1 | 0 | Step 3 — **connected** |
| 4 | User | 133 | PSH, ACK | 1 | 1 | **79** | **HTTP request** |
| 5 | Server | 60 | ACK | 1 | **80** | 0 | Acks 79 bytes (1 + 79 = 80) |
| 6 | Server | 1514 | ACK | 1 | 80 | **1460** | Data bytes 1–1460 |
| 7 | Server | 1514 | PSH, ACK | 1461 | 80 | 1460 | Next 1460 |
| 8 | Server | 1514 | ACK | 2921 | 80 | 1460 | Next 1460 |
| 9 | User | 54 | ACK | 80 | **4381** | 0 | **Cumulative ACK** of all 3 (3 × 1460 = 4380, +1) |
| 10 | Server | 1514 | PSH, ACK | 4381 | 80 | 1460 | More data |
| 11 | Server | 162 | PSH, ACK | 5841 | 80 | **108** | Last piece: 4381 + 1460 + 108 → **5949** |
| 12 | User | 54 | ACK | 80 | **5949** | 0 | Acks everything (response = **5,948 bytes**) |
| 13 | User | 54 | FIN, ACK | 80 | 5949 | 0 | Client closes its direction |
| 14 | Server | 60 | FIN, ACK | 5949 | **81** | 0 | Server acks the FIN (80 + 1) and closes its direction |
| 15 | User | 54 | ACK | 81 | **5950** | 0 | Final ACK (5949 + 1) |

**Things to notice**
- **Ack number = next byte expected** (seq of last byte + 1).
- **SYN and FIN each consume one sequence number** (that's the +1 at setup and teardown).
- **1460** = max payload per packet (MSS); frame **1514** = 14 Ethernet + 20 IP + 20 TCP + 1460. A bare ACK = 54 (no data). Server's 60-byte frames are padded to the Ethernet minimum.
- Server sent **3 segments before one ACK** → **pipelining + cumulative ACK** in action.
- Server's window (~64 K) bounds how much it can have in flight; the client advertises ~65 K.
- **PSH** flag = deliver to the application now.

---

### 20. Course logistics
| Date | Topic | Reading |
|---|---|---|
| **Wed Oct 14** | **Transport layer** | 7.1 |
| Mon Oct 19 | Sockets | 5.2 – 5.4 |
| Wed Oct 21 | Online APIs | 6.5 |

- Slide reminder: **check voter registration at vote.gov**; early voting **Oct 15 – Oct 31**; election day **Nov 3**.

### 21. Quick fact sheet
| Concept | Key fact |
|---|---|
| Port size | **16 bits**; **0–1023 reserved** (well-known) |
| IP vs. Transport addressing | IP → **host**; port → **application** |
| UDP | Connectionless, unreliable, fast; checksum optional (IPv4) / mandatory (IPv6) |
| TCP | Connection-oriented, reliable ordered byte stream, ~85% of traffic |
| Handshake | **SYN → SYN+ACK → ACK** |
| Teardown | **FIN/ACK per direction**; both must close |
| Two Armies | **No protocol** can guarantee agreement over a lossy channel |
| Flow control | **Window** field (16 b), receiver-driven sliding window |
| ACK type | **Cumulative** by default; **SACK** for gaps |
| Fast retransmit | **3 duplicate ACKs** |
| Timeout | Retransmit, **timer doubles** |
| Congestion | Slow start → **+1/RTT** after threshold → **threshold halves** on loss |
| Flow vs. congestion | Receiver capacity vs. **network** capacity |
| ISN | **Random** (stops sequence prediction attacks) |
