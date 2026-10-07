# COMP476 — Networked Computer Systems
## Exam 2 Review (slides: "Review for Second Exam," Mon Oct 5, 2026 — exam Wed Oct 7, 2026)

### 0. What the professor said to expect
**"Likely Questions" slide:**
1. **Give IP addresses and mask for a subnet** → §5, Problems 1–3
2. **HW and IP addresses to send a packet from A to B** → §4, Problems 4–5
3. **Queuing theory** → §6, Problems 6–7

Everything else on the review (network types, Aloha, CSMA/CA, RTS/CTS, small wireless systems, DHCP) is fair game for short-answer / multiple-choice.

### 1. Study map (full notes already in the repo)
| Review topic | Full notes |
|---|---|
| Queuing theory, M/M/1, M/D/1, M/M/N, Little's formula | `07-QueueingThoery.md` |
| MAC layer, Aloha, static TDM/FDM problem | Lecture 10 (MAC Layer) |
| Wi-Fi, CSMA/CA, RTS/CTS, NAV, Bluetooth/Zigbee/NFC/RFID | Lecture 11 (Wireless LANs) |
| IP addressing, ARP, DHCP, routing decision | Lectures 12–13 (Network Layer / Routing) |
| Subnets, CIDR, mask design, DNS | Lecture 14 (Subnets) |

---

### 2. MAC layer & wireless — quick review

**Network types (largest → smallest):** WAN (Wide) → RAN (Regional) → MAN (Metropolitan) → CAN (Campus) → LAN (Local) → DAN (Desk) → PAN (Personal) → BAN (Body) → Nano.

**Why static TDM/FDM fail for a LAN**
- Splitting capacity into **N channels makes each transmission take N× longer**.
- That penalty applies **even when every other station is idle** — wasted capacity for bursty LAN traffic.

**Aloha** (Univ. of Hawaii, terminals on remote islands → central computer; Channel A down, Channel B up)
| Advantages | Disadvantages |
|---|---|
| Efficient for a small number of users | **Collisions drain channel capacity** |
| No coordination between senders | |
| Simple, easy to implement | |

**Pure vs. Slotted Aloha** (throughput S vs. load G/λ)
| | Formula | Peak throughput | Peak at load |
|---|---|---|---|
| Pure Aloha | `S = G·e^(−2G)` | `1/(2e) ≈ 0.184` (18.4%) | G = 0.5 |
| Slotted Aloha | `S = G·e^(−G)` | `1/e ≈ 0.368` (36.8%) | G = 1.0 |

Slotted doubles the peak because frames can only start on slot boundaries → the vulnerable period is halved (1 frame time instead of 2).

**CSMA/CA (Wi-Fi) — reading the timing diagram**
- Station ready to send while channel busy → **wait for idle**.
- When channel goes idle → **random backoff** countdown.
- Shortest backoff wins (C in the slide) and sends Data → receiver returns **Ack**.
- Losers **freeze their backoff counter** while the channel is busy, then finish the **rest of their backoff** (B in the slide) — they don't restart from scratch.

**RTS / CTS**
1. Backoff hits 0 → station sends short **RTS** (Request To Send) to the AP.
2. AP replies with **CTS** (Clear To Send).
3. **Only a station that receives a CTS addressed to it may transmit.**

**Virtual carrier sense — NAV**
- Stations that overhear an RTS or CTS set a **NAV** (Network Allocation Vector) timer for the rest of the exchange (through the ACK).
- C hears A's RTS → NAV starts after RTS. D hears only B's CTS → NAV starts after CTS (D is hidden from A).
- **While sensing a transmission OR waiting out a NAV, a station stops decrementing its backoff counter.**

**Small (short-range) network systems**
| System | Key facts |
|---|---|
| **Zigbee** | Low-power, low-data-rate PAN, ad hoc, self-organizing; cheaper/simpler than other PANs; **2.4 GHz**, **20–250 Kbps**, **10–100 m**; **4-QAM → 2 bits/signal** |
| **Bluetooth** | Short-range PAN; **2.5 mW**, **< 10 m**; **2.402–2.480 GHz**; **79 channels × 1 MHz**; frequency hopping **~1600 hops/sec**, avoids "bad" frequencies; phones ↔ earbuds/cars |
| **NFC** | **≤ 4 cm (1½ in)**; **106–424 Kbit/s**; ID cards, contactless payment, e-tickets, phones; **inductive coupling** between two coils |
| **RFID** | Tags + two-way radio reader; tags **passive or battery-assisted**; ID up to **96 bits**; reader's signal **powers** the tag; tag replies by **modulating** the radio signal |

---

### 3. Address conversion & DHCP

```
Internet name ──DNS──► IP address ──ARP (IPv4) / ND (IPv6)──► MAC address
 (for humans)          (for routing)                          (for the wire)
```

**DHCP** (Dynamic Host Configuration Protocol): hands a computer its IP configuration at boot.
Without DHCP you must manually set: **IP name, IP address, default gateway IP, DNS IP, subnet mask**. DHCP can also supply extras like **WINS**.

---

### 4. Sending a packet A → B ★ (likely question #2)

**The two rules that answer every one of these:**
1. **IP addresses are end-to-end** — on the data packet, source IP = original sender, dest IP = final destination, on **every hop**.
2. **HW (MAC) addresses are hop-to-hop** — they change at every router, and you can only send to a HW address on **your own** network.

**Decision procedure for a host:**
1. Know only a name? → need DNS's IP (from config) → **ARP for DNS's HW** → query DNS → get dest IP.
2. Compare NetIDs (dest AND mask vs. own IP AND mask).
   - **Same** → ARP for the **destination's** HW, send directly.
   - **Different** → ARP for the **gateway's** HW, send to gateway with **dest IP unchanged**.
3. Gateway repeats step 2 on the far network using **its interface on that network**.
4. ARP requests go to **broadcast**; ARP replies go **unicast** back to the asker.

**Slide network**
| Host | IP | HW | Gateway | DNS |
|---|---|---|---|---|
| a.ncat.edu | 152.8.244.55 | 5 | 152.8.254.254 | 152.8.244.1 |
| DNS.ncat.edu | 152.8.244.1 | 3 | 152.8.254.254 | — |
| b.ncat.edu | 152.8.247.77 | 7 | 152.8.254.254 | 152.8.244.1 |
| gate.ncat.edu (router, NCAT side) | 152.8.254.254 | 4 | | |
| router.acme.com (router, acme side) | 176.5.4.3 | 2 | | |
| www.acme.com | 176.5.6.9 | 9 | 176.5.4.3 | 234.6.7.14 |
| www.b.com | 176.5.6.17 | 17 | 176.5.4.3 | 234.6.7.14 |

**Full trace — a.ncat.edu sends one packet to www.acme.com (cold start, all caches empty)**
| # | Step | Src HW | Dest HW | Src IP | Dest IP |
|---|---|---|---|---|---|
| 1 | ARP request: who has the DNS? | 5 | **broadcast** | 152.8.244.55 | 152.8.244.1 |
| 2 | ARP reply DNS → A | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| 3 | DNS query: IP of www.acme.com? | 5 | 3 | 152.8.244.55 | 152.8.244.1 |
| 4 | DNS reply → A (176.5.6.9) | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| — | *Decision: dest NetID 176.5 ≠ source NetID 152.8 → go via gateway* | | | | |
| 5 | ARP request: who has the gateway? | 5 | **broadcast** | 152.8.244.55 | 152.8.254.254 |
| 6 | ARP reply gateway → A | 4 | 5 | 152.8.254.254 | 152.8.244.55 |
| 7 | **Data A → gateway** | 5 | **4** | 152.8.244.55 | **176.5.6.9** |
| 8 | Gateway ARP request: who has 176.5.6.9? | 2 | **broadcast** | 176.5.4.3 | 176.5.6.9 |
| 9 | ARP reply www → gateway | 9 | 2 | 176.5.6.9 | 176.5.4.3 |
| 10 | **Data gateway → www** | 2 | 9 | **152.8.244.55** | 176.5.6.9 |

**What to notice (these are the points a grader checks):**
- Row 7: dest HW = **gateway (4)** but dest IP = **www (176.5.6.9)** — not the gateway's IP.
- Row 10: src HW = **router's acme-side interface (2)** but src IP = **still A (152.8.244.55)**.
- Row 8: on the acme side the router uses **HW 2 / IP 176.5.4.3**, not its NCAT-side 4 / 152.8.254.254.
- The DNS is on A's own network, so A ARPs for it directly — no gateway needed for steps 1–4.

> ⚠️ **Slide quirk:** slide titles for rows 8–9 say "ARP request for WWW's **IP** address" / "ARP reply sending www **IP** address." The router already knows the IP — it's asking for www's **HW** address (9). Write "HW address" on the exam.

---

### 5. Subnets ★ (likely question #1)

**Core facts**
- IP address = **NetID + HostID**. Mask = 1s for NetID, 0s for HostID; **IP AND mask = network address**.
- Classful masks: **A = 255.0.0.0 (/8)**, **B = 255.255.0.0 (/16)**, **C = 255.255.255.0 (/24)**.
- CIDR `ddd.ddd.ddd.ddd/m` → mask has **m leading 1s**. Mask and CIDR do exactly the same job.
- Subnetting = borrow upper HostID bits as a **local** NetID extension. The **outside world still routes by the original NetID**; subnets are joined internally by **routers**.
- For **N subnets**, borrow **⌈log₂N⌉** bits. All addresses in one physical subnet share those upper bits.

**Recipe — "give the mask and address ranges for N subnets of network X/M"**
1. Bits borrowed `b = ⌈log₂N⌉`; new prefix = `/(M + b)`.
2. Write the mask: M + b ones, then zeros → convert the partial octet to decimal (table below).
3. **Block size** = 256 − (partial mask octet).
4. Subnets start at 0, block, 2·block, … in that octet.
5. Each range: first address = network, last = broadcast; **usable = first+1 … last−1**.
6. Hosts per subnet = `2^(32 − prefix) − 2`.

**Partial-octet cheat table**
| Bits of 1s | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| Octet value | 128 | 192 | 224 | 240 | 248 | 252 | 254 | 255 |
| Block size | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

**Slide example — 152.8.0.0/16 into 4 subnets** → 2 bits → **/18 = 255.255.192.0**, block 64
| Subnet | Address block | Usable hosts |
|---|---|---|
| 1 | 152.8.0.0 – 152.8.63.255 | 152.8.0.1 – 152.8.63.254 |
| 2 | 152.8.64.0 – 152.8.127.255 | 152.8.64.1 – 152.8.127.254 |
| 3 | 152.8.128.0 – 152.8.191.255 | 152.8.128.1 – 152.8.191.254 |
| 4 | 152.8.192.0 – 152.8.255.255 | 152.8.192.1 – 152.8.255.254 |

---

### 6. Queuing theory ★ (likely question #3)

**8-step process:** (1) what's asked (2) server (3) queued items (4) model (5) s (6) λ (7) ρ (8) compute.
**Units must match** for λ and s. Round only at the end.

| | M/M/1 (random service) | M/D/1 (constant service) |
|---|---|---|
| Utilization | `ρ = λs` | `ρ = λs` |
| Time in system | `Tq = s/(1−ρ)` | `Tq = s(2−ρ) / [2(1−ρ)]` |
| Time waiting | `Tw = sρ/(1−ρ)` | `Tw = sρ / [2(1−ρ)]` |
| Number in system | `Q = ρ/(1−ρ)` | `Q = ρ²/[2(1−ρ)] + ρ` |
| Number waiting | `W = ρ²/(1−ρ)` | `W = ρ²/[2(1−ρ)]` |

**Always true:** `Tq = Tw + s` · Little: `Q = λTq`, `W = λTw` · `a = 1/λ`
**Poisson:** `Pk(t) = (λt)^k / k! · e^(−λt)` · expected arrivals = `λt`
**Queue size:** `P[Q=N] = (1−ρ)ρ^N` · `P[Q>N] = 1 − Σ(i=0..N)(1−ρ)ρ^i`
**M/M/N:** `ρ = λs/N`; compute K and C (see `07-QueueingThoery.md`); `Tq = Cs/[N(1−ρ)] + s`
**Series queues:** total time = sum of each stage's Tq. **Reused server** (e.g. network both ways) → **2λ** at that server.

---

### 7. Worked practice problems

**Problem 1 — subnet range from a CIDR address (slide's A&T example)**
*An A&T address with 8 subdomains is 152.8.12.34/19. Give the mask, the subnet it's in, and its usable range.*
- Setup: /16 class B + log₂8 = 3 bits → /19 ✓ matches slide.
- Mask: 16 + 3 ones → third octet `11100000` = 224 → **255.255.224.0**
- Block size: 256 − 224 = **32** → subnets at 152.8.0, .32, .64, .96, .128, .160, .192, .224
- Which subnet: 12 AND 224 = **0** → **152.8.0.0/19**
- Range: **152.8.0.0 – 152.8.31.255**; usable **152.8.0.1 – 152.8.31.254**
- Hosts: 2^(32−19) − 2 = 8192 − 2 = **8190**
- Verify: 8 subnets × 8192 = 65,536 = 2¹⁶ ✓ (covers the whole /16)

**Problem 2 — design a mask, non-power-of-2 subnet count**
*Split 192.168.10.0/24 into 6 subnets. Give the mask and every range.*
- Bits: ⌈log₂6⌉ = ⌈2.585⌉ = **3** → /27
- Mask: fourth octet `11100000` = **255.255.255.224**
- Block: 256 − 224 = **32** → 2³ = 8 subnets available (6 used, 2 spare)

| Subnet | Range | Usable |
|---|---|---|
| 1 | .0 – .31 | .1 – .30 |
| 2 | .32 – .63 | .33 – .62 |
| 3 | .64 – .95 | .65 – .94 |
| 4 | .96 – .127 | .97 – .126 |
| 5 | .128 – .159 | .129 – .158 |
| 6 | .160 – .191 | .161 – .190 |
| (spare) | .192 – .223, .224 – .255 | |
- Hosts each: 2⁵ − 2 = **30**
- Verify: 32 × 8 = 256 ✓. Rounding **down** to 2 bits would only give 4 subnets — always round **up**.

**Problem 3 — same subnet or router? (mask 255.255.192.0)**
*Are 152.8.130.7 and 152.8.200.9 on the same subnet?*
- Only the third octet matters: 130 AND 192 = **128**; 200 AND 192 = **192**
- 152.8.128.0 ≠ 152.8.192.0 → **different subnets → through a router**
- Verify: 130 is in the 128–191 block; 200 is in the 192–255 block ✓ (matches §5 table)

**Problem 4 — local delivery (no gateway)**
*Using the slide network, a.ncat.edu already knows b.ncat.edu's IP (152.8.247.77). List the frames to send one packet.*
- Decision: NetID 152.8 = 152.8 → **local** → ARP for B directly (gateway not involved)

| # | Step | Src HW | Dest HW | Src IP | Dest IP |
|---|---|---|---|---|---|
| 1 | ARP request | 5 | broadcast | 152.8.244.55 | 152.8.247.77 |
| 2 | ARP reply | 7 | 5 | 152.8.247.77 | 152.8.244.55 |
| 3 | Data | 5 | 7 | 152.8.244.55 | 152.8.247.77 |
- Verify: even with the /18 subnets, 244 AND 192 = 192 and 247 AND 192 = 192 → still same subnet ✓
- If A only knew the **name**, prepend the 4 DNS rows (ARP DNS, reply, query, reply) from §4.

**Problem 5 — reply path (reverse direction)**
*www.acme.com replies to a.ncat.edu. Gateway and A's HW are already cached. Give the data frames.*
- Decision at www: 152.8 ≠ 176.5 → send to its gateway 176.5.4.3 (HW 2)

| # | Hop | Src HW | Dest HW | Src IP | Dest IP |
|---|---|---|---|---|---|
| 1 | www → router | 9 | 2 | 176.5.6.9 | 152.8.244.55 |
| 2 | router → A | 4 | 5 | 176.5.6.9 | 152.8.244.55 |
- Verify: IPs identical on both rows ✓; HW changes per hop and the router uses its **NCAT-side** HW (4) on hop 2 ✓

**Problem 6 — M/M/1 router**
*A router forwards a packet in an average of 4 ms (exponential). 150 packets/sec arrive. Find Tq, Tw, Q, W.*
- Server = router; items = packets; model = **M/M/1**
- `s = 0.004 sec`, `λ = 150 /sec`, `ρ = 150 × 0.004 = 0.6`
- `Tq = 0.004 / 0.4 = 0.010 sec = 10 ms`
- `Tw = 0.004 × 0.6 / 0.4 = 6 ms`
- `Q = 0.6 / 0.4 = 1.5`
- `W = 0.36 / 0.4 = 0.9`
- Verify: `Tq = Tw + s = 6 + 4 = 10 ms` ✓; Little `Q = λTq = 150 × 0.010 = 1.5` ✓; `W = λTw = 150 × 0.006 = 0.9` ✓

**Problem 7 — same router, fixed-size packets (M/D/1)**
*Same numbers, but every packet takes exactly 4 ms.*
- `ρ = 0.6` (unchanged)
- `Tw = 0.004 × 0.6 / (2 × 0.4) = 3 ms`
- `Tq = 0.004 × (2 − 0.6) / (2 × 0.4) = 7 ms`
- `W = 0.36 / 0.8 = 0.45`
- `Q = 0.45 + 0.6 = 1.05`
- Verify: `Tq = Tw + s = 3 + 4 = 7 ms` ✓; M/D/1 wait is exactly **half** the M/M/1 wait (3 vs 6 ms) ✓ — constant service = less randomness = less queuing

**Problem 8 — Aloha throughput**
*Pure Aloha at load G = 1. What's the throughput? What would slotted Aloha give?*
- Pure: `S = 1 × e^(−2) = 0.135`
- Slotted: `S = 1 × e^(−1) = 0.368`
- Verify: both match the slide's graph at λ = 1 (pure ≈ 0.135 and falling; slotted at its peak) ✓

---

### 8. One-page note sheet (condensed)
| Topic | Must-know |
|---|---|
| Name → IP → MAC | DNS → ARP/ND |
| Data packet | IP end-to-end, HW hop-to-hop |
| Local test | (dest AND mask) == (own AND mask) → ARP dest; else ARP gateway |
| ARP | request → broadcast; reply → unicast |
| Router | uses the interface (HW + IP) on the network it's sending onto |
| Subnet bits | ⌈log₂N⌉; new prefix M + bits |
| Block size | 256 − partial mask octet |
| Usable hosts | 2^(32−prefix) − 2 |
| Class masks | A /8 · B /16 · C /24 |
| Without DHCP | name, IP, gateway, DNS, mask |
| M/M/1 | ρ=λs · Tq=s/(1−ρ) · Tw=sρ/(1−ρ) · Q=ρ/(1−ρ) · W=ρ²/(1−ρ) |
| M/D/1 | Tq=s(2−ρ)/[2(1−ρ)] · Tw=sρ/[2(1−ρ)] · W=ρ²/[2(1−ρ)] · Q=W+ρ |
| Little | Q=λTq · W=λTw · Tq=Tw+s |
| Aloha | pure Ge^(−2G), max 18.4% @0.5 · slotted Ge^(−G), max 36.8% @1 |
| Static TDM/FDM | N channels → N× slower, even if others idle |
| CSMA/CA | wait idle → backoff → send → ACK; backoff freezes while busy/NAV |
| RTS/CTS | send only after CTS addressed to you; overhearers set NAV |
| Zigbee | 2.4 GHz · 20–250 Kbps · 10–100 m · 4-QAM 2 bits |
| Bluetooth | 2.402–2.480 GHz · 79×1 MHz · 1600 hops/s · 2.5 mW · <10 m |
| NFC | ≤4 cm · 106–424 Kbps · inductive coupling |
| RFID | passive/battery-assisted · ≤96-bit ID · reader powers tag |

### 9. Course logistics
| Date | Topic | Reading |
|---|---|---|
| **Wed Oct 7** | **Exam 2** | — |
| Mon Oct 12 | Fall Break (no classes) | — |
| Wed Oct 14 | Transport layer | 7.1 |
| Mon Oct 19 | Sockets | 5.2 – 5.4 |
| Wed Oct 21 | Online APIs | 6.5 |

- Spring/Summer advisement opened Oct 5; **registration Nov 2 – 23**. Make an advisor appointment and come prepared.
- NC voting: register by **5 p.m. Fri Oct 9** for mail/Election Day (Nov 3); early voting **Oct 15 – 31** with same-day registration; photo ID required (license or A&T ID works).
