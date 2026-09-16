# COMP476 — Networked Computer Systems
## Lecture 6 — Multiplexing, Transmission Timing, and Error Correction (slides: "Multiplexing, Transmission Timing and Error Correction", dated 2026-09-02)

### 0. Admin notes from this lecture — READ THIS FIRST
- **⚠️ Exam 1 is scheduled for TODAY, Wednesday, September 16, 2026**, per the "Upcoming Schedule" slide at the end of this deck. The schedule slide also shows: Mon Sept 7 = Labor Day (no class); Wed Sept 9 = Queuing Theory (HW due/assigned); Mon Sept 14 = Telephones & review (read 2.5, HW due).
- **Transmission Team assignment date conflict — flagged, not resolved:** this deck's "Transmission Team Assignment" slide states the one-word-message assignment is due **"by midnight on Sunday, September 7, 2025."** That's inconsistent on two counts: (1) the year doesn't match this Fall 2026 course, and (2) Sept 7, 2026 is a Monday (Labor Day) per this same deck's schedule slide, not a Sunday. This looks like a stale date left over from reusing a prior semester's slide. The Lecture 5 notes already on file recorded the deadline as **"due noon, Wednesday Sept 9, 2026"** — treat that as more likely correct, but confirm the real due date on Blackboard rather than trusting either slide blindly.

### 1. Goals of the lecture
- Finish multiplexing: legacy telephone TDM, SONET, Code-Division Multiplexing (CDMA), and real-world multiplexed systems (ADSL, cable).
- Learn how to calculate total transmission time (transmission time + propagation delay).
- Compare circuit switching, message switching, and packet switching, including timing formulas.
- Understand packet structure and typical packet sizes.
- Learn sources of transmission errors and the difference between error *detection* and error *correction*.
- Learn parity, checksums, CRC, Hamming distance, and how many bits are needed to correct errors.

### 2. Multiplexing review (carried over from Lecture 5)
Four multiplexing types: **FDM** (Frequency-Division), **WDM** (Wavelength-Division), **TDM** (Time-Division — synchronous/asynchronous), **CDM** (Code-Division).

### 3. Legacy Telephone TDM
Digital voice channels are combined hierarchically using synchronous TDM:

| Level | Definition | Rate |
|---|---|---|
| **DS-0** | one digital voice channel | 64 Kbps |
| **DS-1** | 24 × DS-0 channels, often sent over a **T-1** carrier | 1.544 Mbps |
| **DS-2** | 4 × DS-1 signals | 6.312 Mbps |
| **DS-3** | 7 × DS-2 signals | 44.736 Mbps |
| **DS-4** | 6 × DS-3 signals (shown in the hierarchy diagram) | 274.176 Mbps |

Each stage multiplexes several lower-level streams into the next higher-capacity stream (24 DS-0 → 1 DS-1 → ... → 1 DS-4).

**Quiz callback:** "Legacy TDM systems are irrelevant now that most people use cell phones" — **False.** Cell phones are wireless only from the phone to the cell tower antenna; from the tower onward, the call is carried over the same wired phone network (including legacy TDM systems) to the receiver's cell tower.

### 4. SONET / SDH
- **SONET** (Synchronous Optical Networking) and **SDH** (Synchronous Digital Hierarchy) send multiple bit streams over fiber optic cable; SONET has been used for most fiber optic telephone cabling.
- Unlike Ethernet (header before data), **SONET interspersess header bits with the data** and is synchronous — it sends a bit every time period even with no data to send.
- **STM-1** (Synchronous Transport Module, level 1) frame = **810 bytes**, sent every **125 μsec**.
- Frames are drawn as a rectangle (9 rows × 90 columns: 3 columns overhead + 87 columns payload) but transmitted byte-at-a-time, left to right, then top to bottom. Overhead types: Section, Line, Path, plus the SPE (Synchronous Payload Envelope) payload region.

**Worked example — STM-1 transmission speed:**
```
transmission speed = (810 bytes × 8 bits/byte) / (125×10⁻⁶ sec)
                    = 51.84×10⁶ bits/second = 51.84 Mbps
```

**SONET Optical Carrier (OC) levels:**

| OC level | Payload (Kbps) | Line rate (Kbps) |
|---|---|---|
| OC-1 | 50,112 | 51,840 |
| OC-3 | 150,336 | 155,520 |
| OC-12 | 601,344 | 622,080 |
| OC-24 | 1,202,688 | 1,244,160 |
| OC-48 | 2,405,376 | 2,488,320 |
| OC-192 | 9,621,504 | 9,953,280 |
| OC-768 | 38,486,016 | 39,813,120 |

### 5. Code-Division Multiplexing (CDM / CDMA)
- Sends many signals or **"chips"** per bit; each sender has a unique chip pattern. Sending multiple chips per bit spreads the signal across more bandwidth (**spread spectrum**).
- Common in wireless systems, especially 3G cell phones.
- **CDMA analogy:** many people in a room talking at once — take turns (time division), speak at different pitches (frequency division), or **speak different languages (code division)** — you understand your language and treat others as noise.
- Each bit time is split into (usually) **64 or 128 chips**. Each user has a **chip code** (vector) **orthogonal** to every other user's code. Two vectors are orthogonal if their dot product is zero: `A·B = Σaᵢbᵢ`. Chip codes use values **1 and −1**.

**Example chip codes (2 chips/bit):**
- User A: code `(1,-1)` = 0, `(-1,1)` = 1
- User B: code `(1,1)` = 0, `(-1,-1)` = 1
- Check orthogonality: `A·B = (1,-1)·(1,1) = 1·1 + (-1)·1 = 1 - 1 = 0` ✓

**Transmission example:**
| | Sender A | Sender B |
|---|---|---|
| Code / data | (1,-1), data=(1,0,1,1) | (1,1), data=(0,0,1,1) |
| Zeros → -1 | encoded = (1,-1,1,1) | encoded = (-1,-1,1,1) |
| Signal = encoded ⊕ code | (1,−1,−1,1,1,−1,1,−1) | (−1,−1,−1,−1,1,1,1,1) |

Both signals are summed onto the shared medium: `(1,−1,−1,1,1,−1,1,−1) + (−1,−1,−1,−1,1,1,1,1) = (0,−2,−2,0,2,0,2,0)`

**Decoding (both senders present):** split the received value into code-sized pairs, multiply by each receiver's own code, and interpret values `> 0` as 1 and `< 0` as 0. Receiver A recovers `(1,0,1,1)`; Receiver B recovers `(0,0,1,1)`.

**Decoding (only sender A transmits):** Receiver A still correctly recovers `(1,0,1,1)`; Receiver B computes all zeros → correctly detects **no message** for it.

**Pros/cons of CDMA:**
- No coordination needed between senders.
- Spread spectrum is hard to jam and resistant to noise.
- Costs more bandwidth, and sending multiple chips per bit **slows the bit rate**.

**Quiz callback:** Is CDMA's baud rate higher, the same as, or lower than its bit rate? **Higher** — baud rate = signals/second, and CDMA sends many chip-signals per data bit.

**Historical fact:** Actress **Hedy Lamarr** and composer **George Antheil** patented a spread-spectrum communication system in **1942**.

### 6. ADSL (home broadband over phone lines)
- **Asymmetric Digital Subscriber Line** (ADSL/DSL) delivers high-speed data to homes; "asymmetric" because upload and download rates differ.
- Requires being within **18,000 feet** of the central office with good wiring — not every line qualifies.
- Uses **FDM** on the phone line: Voice (POTS) at 0–4 KHz, Upstream at 25–160 KHz, Downstream at 240 KHz–1.5 MHz. The central office modulates data to the correct frequency bands.
- A **splitter** at the home separates the conventional voice wiring from the new DSL wiring going to the DSL modem.

### 7. Cable television multiplexing
- Cable TV can carry up to ~500 channels plus Internet.
- Analog TV used FDM, 6 MHz per channel. Modern (digital, compressed) transmissions have a varying bit rate; digital cable channels are FDM-divided, then multiplexed with **Asynchronous TDM**.
- Each channel uses one of 16-QAM, 128-QAM, 512-QAM, 1024-QAM, 2048-QAM, or 4096-QAM.
- **Cable modems** modulate the digital signal to a specified frequency; **statistical multiplexing** separates multiple users.
- **DOCSIS** (Data Over Cable Service Interface Specification) adds high-bandwidth data to existing cable TV. **DOCSIS 4.0** supports up to **10 Gbps download / 6 Gbps upload**; channels are encrypted with **AES**.

### 8. Moving energy & transmission timing
- Communication requires moving energy (light or electricity). The **speed of light is the max speed data can travel**:
  - **3×10⁸ m/sec** in a vacuum (exactly 299,792,458 m/s)
  - **2×10⁸ m/sec** in glass (fiber)
  - **2×10⁸ m/sec** electrical propagation in copper wire

**Two components of "time to send X bytes":**
- **Transmission time** — time to push the bits out the transmitter (depends on the system's bit rate).
- **Propagation delay** — time for the signal to physically travel down the wire/fiber/air.

**Total transmission time formula** (X bytes, D meters, transmission rate B bits/sec):
```
Time = (X bytes × 8 bits/byte) / (B bits/sec)  +  (D meters) / (2.0×10⁸ m/s)
```

**Calculating hints:** watch your bits vs. bytes (8 bits/byte); if your units don't cancel correctly, the answer is wrong; watch significant digits.

**Worked example 1 — 30K bytes at 10 Mbps across a 4,000 km ocean cable:**
```
Time = (30,000 bytes × 8) / (10×10⁶ bits/sec) + (4×10⁶ m) / (2.0×10⁸ m/s)
     = 0.024 sec + 0.02 sec = 0.044 seconds
```

**Worked example 2 — 30 KB file over a 10 Mbit/sec line to a server 20 m down the hall:**
```
Time = (30,000 bytes × 8) / (10×10⁶ bits/sec) + (20 m) / (2.0×10⁸ m/s)
     = 0.024 sec + 1.0×10⁻⁷ sec ≈ 0.024 seconds
```

**Rule of thumb ("Nearby"):** propagation time is often trivial next to transmission time. **If a problem doesn't state a distance, you can ignore propagation delay.**

### 9. Switching methods
Three ways to move data from source to destination:

| Method | Description |
|---|---|
| **Circuit switching** | A switch electronically connects the wires of the two computers together (e.g., telephone system for local calls). Requires setup time (like dialing), but once connected, data flows with **no further delay**. |
| **Message switching** | "Store-and-forward" — the entire message is sent as one unit; each intermediate node must receive the **whole message** before forwarding it. May need large buffers. |
| **Packet switching** | Like message switching, but data is divided into **packets**; intermediates only need the **entire packet** (not the whole message) before forwarding — this pipelines transmission across hops and is faster than message switching. Each packet needs its own header. **The Internet uses packet switching.** |

Variables used in the timing formulas below: **K** = number of hops, **D** = average propagation delay per hop, **R** = circuit request size, **S** = circuit/switch setup time, **H** = header size (bits), **X** = total data size (bits), **B** = transmission rate (bits/sec), **P** = bits per packet.

**Circuit switching time:**
```
circuit = K·(R/B + D + S) + R/B + K·D + X/B + D·K
```
(send the request → send back connection acknowledgement → send the data)

**Message switching time:**
```
msg = K · ( (X+H)/B + S + D )
```

**Worked example — message switching:** 1 MB of data, 10 Mbps line, 4 hops, 16-byte headers, negligible propagation/setup:
```
msg = 4 × ( (1×10⁶ bytes + 16 bytes) × 8 bits/byte / (10×10⁶ bits/sec) + 0 + 0 )
msg = 3.2 seconds
```

**Packet switching time:**
```
packet = ⌈X/P⌉ · ( (P+H)/B + S ) + D + (K-1) · ( (P+H)/B + S + D )
```
(number of packets × time to send one packet, plus time for intermediates to resend packets)

**Worked example — packet switching:** same 1 MB of data, but sent as **0.5 KB packets**, 10 Mbps line, 4 hops, 16-byte headers, negligible propagation/setup:
```
1 MB = 8×10⁶ bits; 0.5 KB packet = 4000 bits; 16-byte header = 128 bits; P+H = 4128 bits

Time = (8×10⁶ bits / 4000 bits/pkt) × (4128 bits / 10×10⁶ bits/sec) + 3 × (4128 bits / 10×10⁶ bits/sec)
     = 2000 × 4.128×10⁻⁴ sec + 3 × 4.128×10⁻⁴ sec = 827 milliseconds
```
**Key takeaway: packet switching (827 ms) beat message switching (3.2 sec) for the identical data/network — this is why the Internet uses packet switching.**

**Simple packet transmission time** (used when propagation/setup are insignificant and hop delays are insignificant for large data amounts):
```
time = ⌈dataBits / PktSize⌉ × (PktSize + headerSize) / transmissionRate
```

**Comparison of methods:**
- Circuit switching works well when the data-transfer time is long compared to circuit setup time.
- Message switching may need large buffers and takes longer than packet switching.
- Packet switching easily lets multiple independent data streams share one channel.
- **The Internet uses packet switching.**

### 10. Packets
**Standard packet structure:** `Header | Data | Trailer`
- Header: destination address, maybe source address, other parameters.
- Data: sent without start/stop/parity bits.
- Trailer: error-checking values.
- For timing equations, header + trailer size are combined into one "H."

**Ethernet frame format** (bytes per field):

| Preamble | Destination | Source | Type | Data | CRC |
|---|---|---|---|---|---|
| 8 | 6 | 6 | 2 | 46–1500 | 4 |

**Packet sizes** are usually much smaller than total data size, and can be fixed or variable: **ATM = 48 bytes**, **Ethernet ≤ 1500 bytes**, **frame relay up to ~8K bytes**.

### 11. Errors — why we bother
- The networking field spends a lot of effort correcting errors (unlike ordinary arithmetic, where we rarely doubt the computer added correctly).
- Data sent between computers can be **corrupted, lost, delayed, or duplicated**.

**Sources of signal errors:**
- **Interference** — electromagnetic radiation from the environment.
- **Distortion** — capacitance and induction of wires distort the signal.
- **Attenuation** — signal slowly loses energy as it travels through the media.

**The most common reason packets are lost on the Internet:** a router receives more input than it can process (e.g., two 10 Mbps inputs into a router with only a 16 Mbps output — the excess is dropped).

### 12. Error detection vs. correction
- Most networking systems focus on **detecting** an error, then **retransmitting** the data.
- Some systems instead try to identify and **correct** the specific incorrect bits.

**Error detection concept:** the sender computes a function of the data bits and sends the result after the data; the receiver computes the same function and compares. A mismatch means a transmission error occurred.

**Historical note:** to avoid copying errors in the Hebrew Bible, Jewish scribes counted words per line; between the 7th–10th centuries CE this was formalized into the **Numerical Masorah**, which counted words per line, section, book, and groups of books.

**Error bursts:** errors may hit a single bit or many bits at once. An **error burst** is when an electrical disturbance causes several bits in a row to be corrupted.

### 13. Parity
- Simplest error-detection method: add one extra **parity bit**.
- Parity bit = **XOR of the data bits**, set so the total count of 1-bits (data + parity) is **even**.
- Sender computes and appends parity; receiver recomputes parity on the received data and compares to the received parity bit. Mismatch = error. Match = data **might** be OK (not guaranteed, since 2 flipped bits cancel out).

### 14. Checksum
- Alternative to a parity bit: when a packet of bytes is sent, the **arithmetic sum of the bytes** is sent at the end.
- Receiver re-sums the received bytes and compares to the received checksum; a mismatch = error. **XOR can be used instead of addition.**

**Accuracy comparison:**
- A parity bit has only 2 values — with multiple errors, it only has a **50% chance** of catching the error.
- A 16-bit checksum has 1 correct value and 65,535 wrong ones — with multiple errors it's wrong (fails to detect) only about **1 in 64K times**.

### 15. Cyclic Redundancy Check (CRC)
- An improvement over a plain checksum, based on **polynomial division**.
- An **n-bit CRC detects any single error burst no longer than n bits**.
- CRCs are commonly **16 or 32 bits** (can be other sizes) and are the **most commonly used error detection scheme**. Can be computed in hardware via shift registers with XOR gates (e.g., a 3-bit CRC circuit for the polynomial x³ + x¹ + 1).

### 16. Error correction
- **Forward Error Correction (FEC)** / **Hamming codes** can reconstruct a few incorrect bits without retransmission. Most systems still just retransmit the packet instead.
- **Parity error correction:** using *multiple* parity bits lets you identify *which* bit is wrong — and since a bit only has two possible values, once you know it's wrong you automatically know the correct value.

**Block parity:** arrange data in rows/columns, computing a parity bit for every row and every column (plus a parity-of-parities corner cell). If exactly one bit flips, the row parity and column parity that no longer match pinpoint the intersection cell that's wrong — allowing correction, not just detection.

**Hamming distance:** the number of differing bits between two code words. Considering only single-bit errors, if legal ("good") code words are spaced so that changing any *one* bit always produces an illegal word, single-bit errors are always detectable (illustrated with 3-bit and 4-bit cube diagrams of the data space, and a "1-bit correcting space" cube where 000 = logical 0, 111 = logical 1, and the six 1-bit-away words are unambiguously closer to one or the other).

**Number of correction bits needed:**
```
d + p + 1 ≤ 2^p        (approximately: p = log₂(d) + 1)
```
where **d** = number of data bits, **p** = number of check bits. Example: **32 data bits need 6 check bits.** (A chart in the slides shows this relationship is roughly linear/logarithmic — e.g., 8 data bits need 4 check bits, growing to 20 check bits somewhere past 262,144 data bits.)

**Quiz callback:** How many check bits are needed to correct a single error in an 8-bit byte? Using `d + p + 1 ≤ 2^p` with d=8: p=4 satisfies `8+4+1=13 ≤ 16=2⁴` (p=3 fails: `8+3+1=12 > 8=2³`). **Answer: 4.**

### 17. Deep-space communications
- Most terrestrial systems just resend a packet when an error is detected.
- Probes like **Voyager 1 and Voyager 2** are so far away that propagation time is enormous — retransmission is infeasible, so **error correction codes** are used instead of retransmission.
- **Voyager 1 is projected to be one "light day" from Earth this November** (i.e., November 2026, per this lecture).

### 18. Quick reference — formulas from this lecture
| Formula | Use |
|---|---|
| `speed = 810 bytes×8 / 125×10⁻⁶ sec = 51.84 Mbps` | SONET STM-1 frame transmission speed |
| `Time = X·8/B + D/(2.0×10⁸)` | Total transmission time (transmission + propagation) |
| `circuit = K·(R/B+D+S) + R/B + K·D + X/B + D·K` | Circuit switching total time |
| `msg = K·((X+H)/B + S + D)` | Message switching total time |
| `packet = ⌈X/P⌉·((P+H)/B+S) + D + (K-1)·((P+H)/B+S+D)` | Packet switching total time |
| `time = ⌈dataBits/PktSize⌉ × (PktSize+headerSize)/transmissionRate` | Simple packet time (no propagation/setup) |
| `d + p + 1 ≤ 2^p` (≈ `p = log₂(d)+1`) | Error-correction bits needed for d data bits |
