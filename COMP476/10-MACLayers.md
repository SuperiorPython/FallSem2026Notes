# COMP476 — Networked Computer Systems
## Lecture 10 — Media Access Control Sub Layer (slides: "MAC layer," Mon Sept 21, 2026)

> "When Steve Jobs toured Xerox PARC and saw computers running the first operating system that used windows and a mouse, he assumed he was looking at a new way to work a personal computer. He brought the concept back to Cupertino and created the Mac, then Bill Gates followed suit, and the rest is history." — Douglas Rushkoff

### 1. Where we are in the stack
The data link layer is split into two sublayers. This lecture covers the **lower** one, **Media Access Control (MAC)**.

```
┌──────────────────────┐
│     Application      │
├──────────────────────┤
│      Transport       │
├──────────────────────┤
│       Internet       │
├──────────────────────┤
│  Logical Data Link   │  ← LLC sublayer
├──────────────────────┤
│ Media Access Control │  ◄── YOU ARE HERE
├──────────────────────┤
│       Physical       │
└──────────────────────┘
```

### 2. Core functions of the MAC layer
- **Channel access management:** decides **when** devices may transmit on a shared medium.
- **Collision handling:** when devices try to transmit at the same time, **collision detection** or **collision avoidance** protocols manage access.
- **Physical addressing:** unique hardware identifiers (**MAC addresses**) are assigned and used.
- **Framing:** wraps data from upper layers into **frames**.

### 3. Network types (by size)
Nested from largest to smallest: **WAN ⊃ RAN ⊃ MAN ⊃ CAN ⊃ LAN ⊃ PAN ⊃ BAN ⊃ Nano**

| Type | Name | Range | Key traits | Example |
|---|---|---|---|---|
| **WAN** | Wide Area Network | Long distances | **Propagation delay is significant**; usually owned by a **common carrier/telephone company**; line speeds at various cost/bandwidth levels | — |
| **MAN** | Metropolitan Area Network | A city, **~50 km** diameter | Similar to LANs but larger; owned by a common carrier **or** a single organization | — |
| **CAN** | Campus Area Network | A campus | — | — |
| **LAN** | Local Area Network | **~1 km** | Usually **high speeds**; usually owned by the **organization that owns the computers** | Ethernet, Token Ring |
| **DAN** | Desk Area Network | **~1 m** | **Very high speeds** | USB |
| **PAN** | Personal Area Network | Devices you carry | May connect to outside networks | Bluetooth earbuds ↔ laptop; phone ↔ car |
| **BAN** | Body Area Network | On/in the body | Devices may be **implants or pills**, surface-mounted, or carried; mainly **healthcare**; **privacy and security** are concerns | Medical sensors |

### 4. Why a MAC sublayer is needed
Many communication systems **share the medium** with other users:
- radio, satellite, cell phones, bus networks, ring networks
- the COMP476 Transmission team assignment

When the medium is shared, something has to decide who talks when.

### 5. Network topology
Topology = the **connection arrangement** of the stations.

- **Star:** all stations connect to a **central switch** (or hub).
- **Ring:** computers connect in a **circle**. Each computer **sends only to its left neighbor** and **receives only from its right neighbor**.
  - **Ring with wiring hub:** physically wired like a star, but the hub routes the signal in and out of each station so it still behaves like a logical ring.
- **Bus:** all stations connect to a **single wire**, usually **coax**.
  - **Dual bus:** all stations connect to **two wires transmitting in opposite directions**.

```
STAR                 RING                  BUS
 o  o  o  o           o → o                o    o    o    o
  \ |  | /            ↑   ↓                |    |    |    |
  [switch]            o ← o              ━━━━━━━━━━━━━━━━━━━━
```

### 6. LAN point-to-point communication (full mesh)
- **Advantages:** each connection can use its **own preferred hardware**; can run at **high speeds**.
- **Disadvantage:** needs a link between **every pair** of stations, which is **O(N²)** connections.
  - Exact count: `N(N−1)/2` links. *(Example: 10 stations → 45 links.)*

### 7. Ways to share common media
1. **Frequency division multiplexing** (FDM)
2. **Time division multiplexing** (TDM)
3. **Code Division Multiple Access** (CDMA)
4. **Carrier Sense Multiple Access** (CSMA)
5. **Token based**
6. **Anarchy** *(that is, Aloha: transmit whenever you want)*

**Shared media assumptions** (used in all the analysis below):
- Each user sends a **fixed-size packet**.
- A user's need to send occurs **randomly**.
- Packets sent at the same time **collide and are unreadable**.
- Users may be far enough apart that **propagation time matters**.

### 8. LAN performance goals
- **High throughput:** deliver packets successfully at as high a rate as possible.
- **High utilization:** maximize `total throughput / channel data rate`.
- **Queuing theory says these two goals conflict:** high utilization means long queues and delay (see Lecture 7).
- **The LAN challenge:** let every node transmit at maximum speed **without interfering** with the others.

### 9. TDM/FDM analysis: splitting the channel hurts
Setup: the full channel sends one packet in **s** seconds; packets arrive at **λ** packets/sec. Split it into **N** sub-channels, so each sub-channel takes **Ns** seconds per packet and gets **λ/N** of the traffic.

Using the M/M/1 time-in-system formula from Lecture 7:

$$T_q = \frac{s}{1-\lambda s} \qquad\qquad T_{q,sub} = \frac{Ns}{1 - Ns\cdot\frac{\lambda}{N}} = \frac{Ns}{1-\lambda s} = N\,T_q$$

**Result:** the more stations share a channel by static division, the longer it takes to send data, **even if every other station is idle**. That is because idle sub-channels' capacity is wasted.

### 10. Polling
- A **central controller** asks (**polls**) each station whether it wants to send.
- When polled, a station sends a **data packet** or a **short "nothing to send" packet**.
- Polling order: **round robin** or **priority**.
- **Downside:** a lot of time can go to asking stations if they have anything to send, especially when **propagation time is large**.

### 11. Reservation system
- A **common poll** is sent to **all** stations at once.
- Each station indicates, **in order**, whether it wants to send.
- Once everyone knows who wants to transmit, they **transmit in order**.
- **Downsides:** working out who wants to transmit can take a lot of time; **all stations must receive all reservation messages**.

### 12. Aloha network
- Developed at the **University of Hawaii** to communicate with **remote islands**.
- Two channels between the terminals and a **central computer**:
  - **Channel B (uplink):** all terminals transmit to the central computer.
  - **Channel A (downlink):** the central computer sends **acknowledgements** back.
- If two stations transmit at the same time, the messages **collide** and the central receiver gets nothing correctly.
- **No ACK, so the node resends after a random wait.**
- Distant stations might **not hear each other**, but the central system **hears everyone**.

| Advantages | Disadvantages |
|---|---|
| Efficient for a **small number of users** | **Collisions drain channel capacity** |
| **No coordination** needed between senders | |
| **Simple** and easy to implement | |

### 13. Aloha analysis
- **Throughput** = the amount of data that actually reaches its destination.
- A transmission succeeds only if **no other node transmits during it**.
- **Vulnerable period (pure Aloha) = 2T:** an interfering packet that started up to **1T before** overlaps the tail of yours, and one starting up to **1T after** overlaps the front.

```
   ├──── T ────┼──── T ────┤
   [interferer starts before]
               [ ORIGINAL  ]
                     [interferer starts after]
   ◄──── 2T vulnerable ────►
```

**Parameters:**
- **X** = fraction of stations that want to transmit at a given time, `0 ≤ X < 1`.
- **Low load** (X ≈ 0): few collisions, so network load **λ ≈ X**.
- **Heavy load:** many collisions mean many retransmissions, so **λ > X**.
- **Throughput = λ × P(no other station transmits)**

**Poisson arrivals.** If arrivals are exponentially distributed at rate λ, the probability of exactly **k** arrivals in time **t** is:

$$P_k(t) = \frac{(\lambda t)^k}{k!}\,e^{-\lambda t}$$

### 14. Pure Aloha throughput
No other transmission during **2 time periods** (k = 0, t = 2):

$$Prob_0(2) = \frac{(2\lambda)^0}{0!}e^{-2\lambda} = e^{-2\lambda} \quad\Rightarrow\quad \boxed{\text{Throughput} = \lambda e^{-2\lambda}}$$

- Peaks at **λ = 0.5** with throughput **1/(2e) ≈ 0.184 (18%)**.
  - *Derivation:* `d/dλ[λe^(−2λ)] = e^(−2λ)(1 − 2λ) = 0 → λ = ½ → ½·e^(−1) = 1/(2e)`

### 15. Slotted Aloha
- Packets may only be sent **at the start of fixed time slots**.
- Packets **can't partly overlap**: either a full collision or none. This cuts the **vulnerable period to 1T**.

$$Prob_0(1) = \frac{\lambda^0}{0!}e^{-\lambda} = e^{-\lambda} \quad\Rightarrow\quad \boxed{\text{Throughput} = \lambda e^{-\lambda}}$$

- Peaks at **λ = 1** with throughput **1/e ≈ 0.368 (37%)**, **double** pure Aloha.

| | Vulnerable period | Throughput | Peak at | Max throughput |
|---|---|---|---|---|
| **Pure Aloha** | 2T | λe^(−2λ) | λ = 0.5 | **1/(2e) ≈ 18%** |
| **Slotted Aloha** | 1T | λe^(−λ) | λ = 1 | **1/e ≈ 37%** |

**Shape of the curve:** as load increases, throughput **rises at first**, then **falls** as collisions take over and approaches 0 at high load.

### 16. Idle Aloha & exponential backoff
- If **only one** station wants to transmit, it gets the **full speed**.
- Throughput only degrades when **multiple stations compete**.
- So stations must **regulate the overall load** to stay near the optimal λ.

**Exponential backoff (general idea):**
- A collision means the station assumes the **load is too high**.
- After a **failed** send, **halve** the probability of sending.
- After a **successful** send, **double** the probability (up to the maximum).

### 17. CSMA/CD (old Ethernet)
**C**arrier **S**ense **M**ultiple **A**ccess with **C**ollision **D**etect.
- Like Aloha, except the station **listens to the line before transmitting**.
- Better throughput than Aloha because it **tries to avoid collisions**.
- Ethernet used to run as a **bus on coax**, or as a **star with twisted pair or fiber**.

**Protocol:**
1. **Sense** the line to see if someone else is transmitting.
2. **Idle:** transmit the frame.
3. **Busy:** **wait** until the current transmission ends.
4. **While transmitting,** check that you receive **exactly the same signal** you're sending.
5. **Collision detected:** **stop**, wait a **random** time, and try again.

**Ethernet exponential backoff:**
- The **8-byte preamble** helps detect collisions.

| Collision # | Random wait (in transmission times) |
|---|---|
| 1st | 0 – 1 |
| 2nd | 0 – 3 |
| 3rd | 0 – 7 |
| n-th | 0 – (2ⁿ − 1) |

- **Each collision doubles the maximum wait.**

### 18. Collision detection & cable length
- **Traditional coax Ethernet** used a shared coax cable, and the **cable length affected CSMA/CD**.
- To detect a collision, a node must **still be transmitting** when the other node's signal reaches it.

**Failure scenario on a very long cable (A … B … C … D):**
1. A transmits; the signal travels down the cable toward D.
2. **Just before** A's signal reaches D, D senses an idle line and starts transmitting.
3. D, C, then B see the collision, but **A doesn't yet**, because D's signal hasn't reached it.
4. A **finishes** transmitting **just before** D's signal arrives.
5. **A never sees a collision**, which defeats the collision-detect protocol.

**Rule to guarantee detection:**

$$\boxed{\text{Transmission time} > 2 \times \text{propagation time}}$$

(×2 = worst-case **round trip**: your signal goes to the far end and the collision comes back.)

**Worked example:** minimum Ethernet frame **72 bytes**, **100 Mbit/s**, signal speed **2.0 × 10⁸ m/s**:

$$\frac{72 \times 8\ \text{bits}}{10^8\ \text{bits/s}} > 2\cdot\frac{L}{2.0\times10^8\ \text{m/s}}$$
$$5.76\ \mu s > \frac{L}{10^8} \;\Rightarrow\; L < 576\ \text{m}$$

**Maximum cable length = 576 m.**

### 19. Network interface (NIC)
- The network interface connects the **network cable (or antenna)** to the computer's **I/O controller** on the system bus (alongside CPU/cache and memory).

**Network Interface Chip** manages the **data link protocol**:
- **Receiving:** gets packets, **checks for errors**, decides if the packet is **for this computer**, and **saves the data to memory**.
- **Sending:** builds the **header and trailer** and transmits the bits.
- **Interrupts the CPU** when data is received or sent.

**CPU involvement:**
- The CPU **doesn't do much** network processing; the NIC runs **in parallel** with the CPU.
- Why: at **1 Gbit/s**, a **4.0 GHz** CPU running one instruction per cycle has only **4 cycles per bit**, which is far too few to process each bit in software.
  - *Note:* the slide calls 1 Gbit/s "Fast Ethernet"; strictly, Fast Ethernet is 100 Mbit/s and 1 Gbit/s is **Gigabit Ethernet**. The point about CPU load holds either way.

### 20. Ethernet wiring
- **Originally coax:** the wires were **expensive**, and routing them through a building as a **bus** was hard.
- **Now twisted pair**, with **RJ-45** connectors.
- Each computer connects by twisted pair to a **central hub or switch** (a physical **star**).

**Central connection:**
- Twisted-pair Ethernet connects to an Ethernet **hub, switch, or router**.
- Switches usually connect **4 to 48 nodes**.
- **Single point of failure:** if the switch breaks, the network stops.

### 21. LAN addressing
- Each node on a LAN has an **address** identifying it to the other nodes on **that** network.
- Packets carry the LAN address of the node that should **receive** them.
- **LAN addresses mean nothing outside the LAN.**

**Three types of LAN address:**

| Type | Who sets it |
|---|---|
| **Static** | **Manufacturer** assigns a unique address to each network interface |
| **Configurable** | **Customer** sets the address |
| **Dynamic** | Assigned **automatically** when the station **first boots** |

### 22. Ethernet frame & addresses

```
┌──────────┬───────────┬───────────┬──────┬────────────────┬─────┐
│ Preamble │ Dest Addr │ Src Addr  │ Type │ Data in Frame  │ CRC │
│    8     │     6     │     6     │  2   │   46 – 1500    │  4  │
└──────────┴───────────┴───────────┴──────┴────────────────┴─────┘
            ◄──────── Header ──────────►◄──── Payload ────►   (bytes)
```

- Ethernet uses **static 48-bit (6-byte)** addresses.
- Every Ethernet interface has a **unique, manufacturer-set** address.
- **Both** sender and receiver addresses are in the header.
- **Total overhead = 26 bytes** (8 + 6 + 6 + 2 + 4), with **up to 45 bytes of padding**.
  - Padding brings tiny payloads up to the **46-byte minimum**: 1 byte of data + 45 bytes padding = 46.
  - **Minimum frame** = 26 + 46 = **72 bytes** (the number used in the cable-length example).
  - **Maximum frame** = 26 + 1500 = **1526 bytes**.

### 23. Broadcasting
- **Broadcast** = one packet sent to **all nodes** on the network.
- A special address tells every node to accept it.
- **Ethernet broadcast address = all 1 bits:** `FF:FF:FF:FF:FF:FF`

### 24. Identifying packet contents
- The header's **frame type** field says **which protocol** should handle the packet.
- One type code means the "data" is an **Internet Protocol (IP)** packet.

**Selected Ethernet type values:**

| Value | Meaning |
|---|---|
| 0000–05DC | Reserved for IEEE LLC/SNAP |
| **0800** | **Internet IP Version 4** |
| 0805 | CCITT X.25 |
| 6559 | Frame Relay |
| 8035 | Internet Reverse ARP |
| 809B | Apple AppleTalk |
| 8137–8138 | Novell IPX |
| FFFF | Reserved |

*(The full slide table also lists vendor codes: Ungermann-Bass, Banyan VINES, Berkeley UNIX Trailer, DEC LAT/LANBridge, HP, AT&T, SGI, Stanford V Kernel, IBM SNA, Wellfleet, Motorola.)*

**Packet identification in the data:**
- Some packets have **no type field** in the header.
- Instead, the **first few bytes of the data** say what to do with the packet.
- The **IEEE LLC/SNAP** header is often used for this:
  - **LLC** = Logical Link Control
  - **SNAP** = **S**ub **N**etwork **A**ttachment **P**oint

### 25. Network analyzers & security
- **Network analyzers** can show **every packet on the wire**.
- To do that, the software puts the NIC into **promiscuous mode**, so it accepts **all** frames, not just ones addressed to this computer.
- **Security implications:**
  - Anyone can read all packets that reach their computer, **even ones not addressed to them**.
  - **Messages on a LAN are not guaranteed to be private.**

### 26. Wi-Fi networks & the hidden station problem
- **CSMA/CD doesn't work well in wireless LANs** because transmitters have **limited range**.
- A receiver more than **δ** from a transmitter **won't hear it**, so it **can't detect a collision**.

**Missed collisions (hidden station problem):**

```
computer 1 ◄── δ ──► computer 2 ◄── δ ──► computer 3
```

- Computer 3 is sending to computer 2. Computer 1 is out of range of 3, so its **carrier sense says the line is idle**.
- If computer 1 also transmits to 2, the frames **collide at computer 2**, and neither sender knows.
- *(Wi-Fi handles this with collision **avoidance**, CSMA/CA, instead of collision detection; see "Core functions" §2.)*

### 27. Token-based LANs
- **Token Ring** and **Token Bus** have **no collisions** like Aloha's.
- There is exactly **one logical token**, and **only the station holding it may transmit**.
- The token cycles through the stations until it reaches the next one waiting to send.

| Advantages | Disadvantages |
|---|---|
| Good when the network is **very busy** | At **low loads**, a station still has to **wait its turn** for the token |
| Stays **efficient at high loads** | Each station is **limited to a set transmission time** |

**Token Ring operation:**
- Each station **receives a bit and sends a bit** (stores it and forwards it to the next station).
- The **transmitting station doesn't forward** the incoming bits (its own frame coming back around is removed).
- The **receiving station adds a bit after the last bit** if it received the message correctly, which acts as a built-in ACK.

**Token Ring protocol:**
1. A logical **token** circulates around the ring continuously.
2. The station holding the token may send **a few frames** if it has any.
3. Every other station stores each bit and **forwards** it to the next station.
4. The sending node **doesn't forward** bits.
5. When done, the sender **passes the token** to the next station.

**Contention (Aloha/CSMA) vs. token, in one line:** contention is best at **low load** (no waiting) and token is best at **high load** (no collisions).

### 28. In-class practice question (from slides)
**If you have a 1 Mb/s wireless network using Pure Aloha, what is the maximum amount of data you can transmit in one minute?**
A. 1.35 MByte  B. 2.78 MByte  C. 10.8 MByte  D. 60.0 MByte

**Answer: A, 1.35 MByte**

$$\text{data} = \frac{1\times10^6\ \text{bits/s}\times 60\ \text{s/min}}{8\ \text{bits/byte}}\times 0.18 = 7.5\times10^6 \times 0.18 = 1.35\times10^6\ \text{bytes}$$

- **Slotted Aloha** allows about **twice** as much: the slide says **2.7 MB**. Using the exact `1/e = 0.368` gives `7.5 × 10⁶ × 0.368 ≈ 2.76 MB`, which is answer choice **B (2.78 MB)**, so B is the slotted-Aloha distractor.
- *Exam tip:* the professor uses **0.18** for pure Aloha. Use the same rounding to match graded answers.

### 29. Quick formula sheet
| Concept | Formula / value |
|---|---|
| Full-mesh links | N(N−1)/2 → O(N²) |
| Split-channel delay | T_q,sub = N · T_q |
| Poisson | P_k(t) = (λt)^k e^(−λt) / k! |
| Pure Aloha | S = λe^(−2λ), max 1/(2e) ≈ 0.18 at λ = 0.5 |
| Slotted Aloha | S = λe^(−λ), max 1/e ≈ 0.37 at λ = 1 |
| CSMA/CD condition | t_transmit > 2 · t_propagation |
| Max cable length | L < (frame bits / rate) · v / 2 |
| Ethernet overhead | 26 bytes; data 46–1500; min frame 72 B |
| Ethernet address | 48 bits; broadcast FF:FF:FF:FF:FF:FF |
| IPv4 type code | 0x0800 |
| Ethernet backoff | after n collisions wait 0…(2ⁿ−1) slots |
