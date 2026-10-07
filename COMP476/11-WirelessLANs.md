# COMP476 — Networked Computer Systems
## Lecture 11 — Wireless LANs (slides: "Wireless LANs," Wed Sept 23, 2026)

> "Wireless is freedom. It's about being unleashed from the telephone cord and having the ability to be virtually anywhere when you want to be." — Martin Cooper (placed the first public call from a handheld portable cell phone)

### 1. IEEE 802 LAN standards
| Standard | Network |
|---|---|
| **802.3** | **Ethernet** |
| 802.4 | Token Bus |
| 802.5 | Token Ring |
| 802.6 | Distributed Queue Dual Bus |
| **802.11** | **Wireless LAN (Wi-Fi)** |
| 802.15 | Wireless PAN (Bluetooth, Zigbee) |

- **Today, only Ethernet and wireless LANs are in use.**
- Ethernet itself has changed a lot (coax bus → twisted-pair star with switches).

### 2. Token-based LANs (recap from Lecture 10)
- **Token Ring** and **Token Bus** have **no collisions** like Aloha.
- Exactly **one logical token**; **only the holder may transmit**.
- The token cycles through stations until it reaches the next one waiting to send.

| Advantages | Disadvantages |
|---|---|
| Good when the network is **very busy** | At **low loads**, a station must still **wait its turn** |
| Stays **efficient at high loads** | Each station is limited to a **predetermined transmission time** |

**Token Ring:**
- Each station **receives a bit and sends a bit** (store-and-forward, one bit at a time).
- The **transmitting station does not forward** the incoming bits (its frame is removed when it comes back around).
- The **receiving station adds a bit after the last bit** if it received the message correctly (built-in ACK).
- After sending its frames, the station **passes the token** to the next station.

**Token Bus:**
- Physically wired to a **coax cable** like traditional Ethernet.
- Uses the Token Ring protocol, but passes the token to the next station in a **logical ring** (the ring order is defined in software, not by the wiring).

**In-class question:** *What happens if there is a transmission error when sending the token to the next computer?*
A. The next computer continually sends packets  B. The previous computer continually sends packets  C. All computers send packets  **D. No computers send packets**

**Answer: D.** If the token is lost, nobody holds it, so nobody is allowed to transmit. (Real token networks need a token-recovery/regeneration mechanism to fix this.)

### 3. Wireless LAN (WLAN) basics
- Wireless stations connect to an **access point (AP)**.
- The **AP follows the same protocol rules** as the wireless stations (it has no special priority).
- APs are usually connected to the **Internet** (through a wired network).

### 4. Wireless LAN standards (802.11 family)
| Gen | IEEE | Adopted | Speed (Mbps) | 2.4 GHz | 5 GHz | 6 GHz |
|---|---|---|---|---|---|---|
| 1 | 802.11 | 1997 | 1–2 | ✔ | | |
| 2 | 802.11b | 1999 | 1–11 | ✔ | | |
| 3 | 802.11a | 1999 | 6–54 | | ✔ | |
| 3 | 802.11g | 2003 | 6–54 | ✔ | | |
| **Wi-Fi 4** | 802.11n | 2009 | 6.5–600 | ✔ | ✔ | |
| **Wi-Fi 5** | 802.11ac | 2013 | 6.5–6,933 | | ✔ | |
| **Wi-Fi 6** | 802.11ax | 2021 | 0.4–9,608 | ✔ | ✔ | |
| **Wi-Fi 6E** | 802.11ax | 2021 | 0.4–9,608 | ✔ | ✔ | ✔ |
| **Wi-Fi 7** | 802.11be | 2024 | 0.4–23,059 | ✔ | ✔ | ✔ |
| **Wi-Fi 8** | 802.11bn | — | — | ✔ | ✔ | ✔ |

*Pattern:* each generation adds more bandwidth/frequency bands; **6E** is just Wi-Fi 6 extended into the **6 GHz** band.

### 5. Multiple frequencies and speeds
- **Higher frequencies are faster** but **don't go through walls easily**.
  - This can be a **security advantage** (signal stays in the room/building).
- On initial connection, devices **negotiate speed based on channel noise**.
  - **Shannon:** lower signal-to-noise ratio → lower maximum data rate (`C = B·log₂(1 + S/N)`, Lecture 5).
- Several **modulation techniques** are used (QAM levels drop as the signal gets noisier).

**In-class question:** *Some WLANs use QAM with 4,096 different values. How many bits are sent per signal?*
A. 10  **B. 12**  C. 1024  D. 4096

**Answer: B.** `bits per symbol = log₂(M) = log₂(4096) = 12` (since 2¹² = 4096). This is **4096-QAM**, used by Wi-Fi 7.

### 6. Why there's no collision detection in Wi-Fi
- Wireless devices **send and receive on the same frequency**.
- A station's **own transmitted signal is far stronger** than anything it receives from other stations, so it drowns them out.
- So, **unlike CSMA/CD**, wireless devices **cannot detect a collision** while transmitting.

### 7. CSMA/CA
**C**arrier **S**ense **M**ultiple **A**ccess with **C**ollision **A**voidance.
- **Avoids** collisions but **doesn't always prevent** them.
- **Every correctly received message is immediately acknowledged (ACK).**

| | CSMA/CD (wired Ethernet) | CSMA/CA (Wi-Fi) |
|---|---|---|
| Listen before sending | Yes | Yes |
| Detect collision while sending | **Yes** (compare sent vs. received signal) | **No** (can't hear others over itself) |
| How collisions are found | Detected on the wire | **Missing ACK** |
| Random wait | **After** a collision | **Before** every transmission (backoff counter) |

### 8. Backoff counter
1. Each station sets a **backoff counter** to a **random value from 0 to the current max** (e.g., **15** in some systems).
2. When it has something to send, it **decrements the counter by 1 for every idle transmission slot** (a slot where it senses no one else transmitting).
3. If it **senses another station transmitting**, it **stops decrementing** (freezes the counter).
4. When the counter hits **0**, the station **transmits**.

*(Slide shows the CSMA/CA timeline diagram from the Tanenbaum textbook: stations count down only during idle slots and freeze while someone else is on the air.)*

### 9. Resolving and reducing collisions
**Resolving:**
- ACKs are sent for correctly received messages.
- **No ACK ⇒ assume a collision** (could be another problem, but collision is assumed).
- The station **doubles its maximum backoff value**, picks a **new random value from 0 to the new max**, and tries again. *(Same exponential-backoff idea as Ethernet, Lecture 10 §17.)*

**Reducing:**
- **Random delays before sending** help prevent collisions.
- When collisions do happen, the random delays **get longer**, which **lowers the chance** of another collision.

### 10. RTS / CTS
1. A station's backoff counter reaches 0 → it sends a short **Request To Send (RTS)** to the AP.
2. The AP replies with a **Clear To Send (CTS)** addressed to that station.
3. **Only a station that receives a CTS addressed to it may start transmitting.**

```
Station A ──RTS──► AP
Station A ◄──CTS── AP ──CTS──► (everyone in AP's range hears it, incl. hidden nodes)
Station A ──DATA─► AP
Station A ◄──ACK── AP
```

*Why it helps:* RTS/CTS are tiny, so if they collide, little time is wasted, and the CTS from the AP reaches stations that can't hear the sender.

### 11. Hidden node problem
- Some stations **can't receive the signal** from other stations:
  - **obstacles** in the way
  - **low-powered** stations with **short range**

```
   A  ─────────►  B (access point)  ◄─────────  C
   A and C can both reach B, but cannot hear each other
```

- A and C both sense an idle channel, both transmit, and the frames **collide at B**. (Same scenario as Lecture 10 §26.)

### 12. Virtual sensing (NAV)
- A station that **can't sense** another's transmission will likely **transmit at the same time** and cause a collision.
- Channel sensing can be **real** (actually hearing the signal) or **virtual**.
- RTS and CTS messages carry a **Network Allocation Vector (NAV)**: roughly **how long the station will transmit**.
- Every other station that hears the RTS **or** CTS knows the channel will be **busy for that period**, even if it can't hear the data itself.

**Rule:** a station **stops decrementing its backoff counter** when it **actually senses** a transmission **or** is **waiting out a NAV period**.

*(Slide shows the Tanenbaum "virtual channel sensing" timeline: C hears A's RTS → sets NAV; D hears only B's CTS → sets NAV; both stay quiet through the data + ACK.)*

### 13. LAN addressing (recap)
- Each node has an **address** identifying it to other nodes **on that network**.
- Packets carry the LAN address of the node that should **receive** them.
- **LAN addresses mean nothing outside the LAN.**

### 14. Small (short-range) network systems
Inexpensive, short-range wireless for **sensing and connectivity**: **Bluetooth, NFC, Zigbee, RFID**.

### 15. Zigbee
- Standard for **low-power, low-data-rate, close-proximity (PAN)** wireless **ad hoc** networks.
- **Simpler and cheaper** than other wireless PANs.
- **2.4 GHz** band, **20–250 kbps**, **10–100 m**.
- **Self-organizing** ad hoc digital radio network.
- Uses **4-value QAM → 2 bits per signal** (`log₂ 4 = 2`).
- Topologies: **star, tree, and mesh**.

**Device types:**
| Device | Role |
|---|---|
| **Zigbee Coordinator (ZC)** | **Exactly one per network** (required) |
| **Zigbee Router (ZR)** | **Passes data** to other devices |
| **Zigbee End Device (ZED)** | Just enough functionality to **talk to its parent node** |

**Use:**
- Popular for **IoT**.
- **Low power** → ideal for **battery-operated devices and sensors**.
- Popular for projects: **self-configuring**; **authentication supported but not required**.

### 16. Bluetooth
- Short-range wireless for **PANs**.
- Power limited to **2.5 mW** for distances **< 10 m**.
- Frequencies **2.402–2.480 GHz**.
- Common uses: phone ↔ **earbuds**, phone ↔ **car**.
- Named after Danish king **Harald "Bluetooth" Gormsson**.

**Transmission slots (piconet):**
- **Controller/Responder** architecture: one controller talks to **up to 7 devices** in a **piconet**.
- The controller **chooses which device to address**.
- The controller's **master clock** defines transmission slots.
- Controller **transmits in even slots**, **receives in odd slots**.
- Packets can be **1, 3, or 5 slots** long.

**Spread spectrum:**
- **Adaptive frequency-hopping spread spectrum (FHSS)** with QAM.
- Data split into packets; each packet sent on one of **79 channels**, each **1 MHz** wide (79 × 1 MHz ≈ the 2.402–2.480 GHz range).
- **Adaptive:** uses only "good" frequencies, avoids "bad" (noisy) ones.
- Usually **1600 hops per second**.

**Pairing:**
- Devices must be **bonded/paired** to communicate; usually needs **user permission**.
- Devices may have **pairing restrictions** (e.g., the professor's hearing aids only pair within 3 minutes of leaving the charger).
- Pairing creates a shared secret called a **link key**.
- Slide: "communication is **public key encrypted**." *(Strictly: public-key cryptography (ECDH) is used during pairing to agree on the key; the actual data is then encrypted with a symmetric cipher (AES). For the exam, go with the slide.)*

### 17. Near Field Communication (NFC)
- Protocols for communication over **≤ 4 cm (1½ in)**.
- **106–424 kbit/s**.
- Used in **electronic ID cards, contactless payment, e-tickets, smartphones**.

**Inductive coupling:**
- Based on coupling between **two electromagnetic coils**.
- A **changing current** in one coil **induces a current** in an adjacent coil.
- Used for NFC **communication** and **wireless charging**.
- Same principle as **induction stoves** (induced current in iron cookware heats it through resistance).

**In-class question:** *Your Aggie OneCard probably uses…*
A. Wi-Fi  B. Zigbee  C. Bluetooth  **D. NFC**

**Answer: D.** Contactless tap cards use NFC (13.56 MHz, a few cm).

### 18. Wireless charging
- Two major standards, both **inductive within 2–4 cm**:
  - **NFC Wireless Charging (WLC):** up to **1 W**
  - **Qi:** up to **25 W**
- **Medical implants** can be charged wirelessly.

**Efficiency:**
- **Less efficient** than wired charging.
- Uses about **15%–39% more energy** to charge a phone. *(Slide wording is garbled: "uses about 15% to 39% for energy"; "more energy than wired" is the intended meaning.)*
- A **misaligned** phone may use **80% more** energy.

### 19. RFID
- **Radio Frequency Identification:** **tags** + a **two-way radio reader**.
- Tags are **passive** or **battery-assisted**.
- Usually transmit an **ID value up to 96 bits**.

**Active Reader, Passive Tag (ARPT):**
- A **powered reader** reads a tag that has **no battery**.
- The tag has an **antenna** (receive + transmit) and an **IC** that processes info and **modulates/demodulates** RF signals.

**Operation:**
1. Reader transmits a radio signal at the tag's frequency (continuously or periodically).
2. Tag's antenna receives it; **the signal powers the tag**.
3. Tag sends its **ID by modulating the radio signal**.
4. Reader receives and **decodes the ID**.

**Uses:** pet "chips," inventory/tracking, race timing.

### 20. Small network summary
| | **Zigbee** | **Bluetooth** | **NFC** | **RFID** |
|---|---|---|---|---|
| **Frequency** | 2.4 GHz, 868 MHz, 915 MHz | 2.402–2.480 GHz | 13.56 MHz | 13.56 MHz, 120–150 kHz, others |
| **Distance** | 100 m | 10 m | 4 cm | 10 cm – 1 m |
| **Power** | 1–100 mW | 2.5 mW | < 15 mA | none (passive) |
| **Speed** | 250 kbit/s | up to 2 Mbps | 106–424 kbit/s | — |
| **Standard** | IEEE 802.15.4 | Bluetooth SIG | ISO/IEC 14443 | many |

### 21. Logical Data Link sublayer (LLC)
Now moving **up** one sublayer from MAC.

```
┌──────────────────────┐
│     Application      │
├──────────────────────┤
│      Transport       │
├──────────────────────┤
│       Internet       │
├──────────────────────┤
│  Logical Data Link   │  ◄── YOU ARE HERE
├──────────────────────┤
│ Media Access Control │
├──────────────────────┤
│       Physical       │
└──────────────────────┘
```

**Functions:**
- **Encapsulate** Network-layer packets into **frames**.
- **Detect transmission errors.**
- **Regulate data flow** (flow control).
  - Error detection and flow control are **also done at other layers** (e.g., Transport).

**Service types:**
| Service | Notes / example |
|---|---|
| **Unacknowledged connectionless** | **Most common**; **Ethernet** |
| **Acknowledged connectionless** | **Wi-Fi** (every frame ACKed, §7) |
| **Acknowledged connection-oriented** | Connection set up first, frames numbered and ACKed |

**Single link:**
- The data link layer sends frames to the data link layer on an **adjacent** machine only.
- It **doesn't worry about routing**.
- **Only the Physical layer actually sends bits**; the data link "talks" to its peer **virtually** (slide figure: (a) virtual communication peer-to-peer vs. (b) actual path down through the physical layer and back up).

**Link for each interface:**
- There may be a separate data link interface (from the Internet layer) for **each network device** (e.g., one for Ethernet, one for Wi-Fi).
- A data link process communicates on **only one network**.

### 22. Framing
- The data link layer groups the physical layer's bits into **frames**.
- Frames usually have a **header** and an **error-detecting trailer**.
- **Error detected → usually discard the frame.**

**Four framing techniques:**
1. **Byte count:** frame starts with a count of its bytes.
2. **Flag bytes with byte stuffing:** a special **flag** byte marks frame boundaries; an **escape** byte stops data from looking like a flag.
3. **Flag bits with bit stuffing:** same idea at the **bit** level.
4. **Physical-layer coding violations:** a sequence that is **not legal as data** marks a new frame.

#### 22a. Byte count
```
(a) no errors:   [5|1 2 3 4] [5|6 7 8 9] [8|0 1 2 3 4 5 6] [8|7 8 9 0 1 2 3]
(b) one error:   [5|1 2 3 4] [7|6 7 8 9 0 1] [2|3 ...  ← out of sync from here on
```
- **Weakness:** if the **count itself** is corrupted, the receiver **loses sync** and every frame after it is misread. Rarely used alone.

#### 22b. Flag bytes with byte stuffing
- Each frame starts and ends with a **FLAG** byte.
- If FLAG or ESC appears in the data, the sender inserts an **ESC** byte before it; the receiver removes it.

| Original data | After stuffing |
|---|---|
| A FLAG B | A **ESC** FLAG B |
| A ESC B | A **ESC** ESC B |
| A ESC FLAG B | A **ESC** ESC **ESC** FLAG B |
| A ESC ESC B | A **ESC** ESC **ESC** ESC B |

#### 22c. Bit stuffing
- Like byte stuffing but adds an extra **bit**.
- Data **doesn't have to be 8-bit bytes**.
- Also prevents **long runs of 1s or 0s**, which helps **clock synchronization**.
- Electrical systems: can reduce overall **DC current**.

**HDLC (High-level Data Link Control):**
- Frames begin and end with the flag **`01111110` (0x7E)**.
- **Sender:** after **five consecutive 1s** in the data, **stuff a 0**.
- **Receiver:** after **five consecutive 1s**, **remove the following 0**. *(Slide says "outgoing data" here; on the receive side it's the incoming data.)*
- So the data can never contain six 1s in a row, and the flag can't appear by accident.

**Worked example:**
```
Original:  011011111111111111110010
Stuffed:   011011111 0 11111 0 11111 0 10010
         = 011011111011111011111010010
```
(Each `0` after five 1s is stuffed; the receiver strips them back out.)

#### 22d. Coding violations (4B/5B)
- Some physical layers **don't allow every bit pattern**.
- **4B/5B** sends **5 signal bits for every 4 data bits** (80% efficiency).
- 32 possible 5-bit codes, only 16 used for data → the **unused (prohibited) patterns** serve as **frame delimiters**.

| Data | Sent | Data | Sent |
|---|---|---|---|
| 0000 | 11110 | 1000 | 10010 |
| 0001 | 01001 | 1001 | 10011 |
| 0010 | 10100 | 1010 | 10110 |
| 0011 | 10101 | 1011 | 10111 |
| 0100 | 01010 | 1100 | 11010 |
| 0101 | 01011 | 1101 | 11011 |
| 0110 | 01110 | 1110 | 11100 |
| 0111 | 01111 | 1111 | 11101 |

*Notice:* no code has more than one leading 0 or more than two trailing 0s, so there are never long runs of 0s (keeps the clock in sync).

**Invisible to the Network layer:**
- All framing is **invisible to the Internet layer**.
- Frames are built by the sending data link layer and removed by the receiving one.
- **Stuffed bytes/bits are removed** before data goes up to the Network layer.

### 23. LAN length limits & bits on the wire
- LANs have a **maximum length**: an **Ethernet segment ≤ 100 m**.
- Limits come from **power (attenuation)** and **propagation delay** (light/electricity isn't infinitely fast).

**Bits on the wire (worked example):**
- 100 Mbit/s → one bit every `1 / 10⁸ = 10 ns`.
- At **2.0 × 10⁸ m/s**, one bit occupies `2.0×10⁸ × 10×10⁻⁹ = 2.0 m` of wire.
- Two computers **16 m** apart → `16 / 2 = 8 bits` in flight between them.
- The first bit arrives just as the sender sends the 8th bit.

$$\text{bit length (m)} = \frac{v}{R} \qquad \text{bits in flight} = \frac{d}{v/R} = \frac{d \cdot R}{v}$$

### 24. Interconnecting networks
| Layer | Device |
|---|---|
| Application | Router *(gateway)* |
| Transport | — |
| **Network** | **Router** |
| **Data Link** | **Bridge** (switch) |
| **Physical** | **Repeater** (hub) |

**Repeaters (Physical layer):**
- Copy **individual bits** between cable segments.
- Copy **every** bit to all segments, **including collisions**.
- Act as an **amplifier**; **invisible** to computers.
- Ethernet can be extended to **1500 m** with **no more than 4 repeaters** between hosts.

**Bridges (Data Link layer):**
- **Store and forward frames** between LANs: receive the whole frame, then retransmit it on the other side.
- Retransmitting **adds delay**.
- **Invisible** to computers on the network.
- **Frame filtering:** only forward frames that **need to go to the other side**.
- **Broadcasts always go through** a bridge.
- **Learn** where hosts are.
- **Long-distance bridging:** special bridges can be joined by a **point-to-point link** (fiber, leased phone line, satellite).

**Routers (Network layer):**
- Connect networks of **different types** (e.g., **Ethernet ↔ WLAN**).
- Provide **routing**.

### 25. Learning bridges
- **No configuration needed**; work straight out of the box.
- Learn **which side** each computer is on by looking at **every frame's source address**.
- **Forwarding rule:**
  - Destination **known to be on the same side** as the source → **don't forward** (filter).
  - Destination **on the other side** → forward to that side only.
  - Destination **unknown** (or broadcast) → **flood** to both sides.
  - *(The slide says "if the bridge does know the destination is on the same side… it will forward the frame"; that's a typo. It should be **does not** forward, as the walkthrough below shows.)*

**Walkthrough** (A, B on the left; X, Y on the right):

| Step | Frame | Bridge knows dest? | Action | Learns | Left table | Right table |
|---|---|---|---|---|---|---|
| 0 | — | — | — | — | {} | {} |
| 1 | A → B | No | **Flood** to left and right | A is left | {A} | {} |
| 2 | B → A | A is left (same side) | Delivered on left only, **not forwarded** | B is left | {A, B} | {} |
| 3 | X → Y | No | **Flood** to right and left | X is right | {A, B} | {X} |
| 4 | X → A | A is left | **Forward to left** | — | {A, B} | {X} |
| 5 | Y → X | X is right (same side) | **Not forwarded** to left | Y is right | {A, B} | {X, Y} |

**In-class question:** *If Y sends a frame to X, will it be sent to the left side?* (Left: A B; Right: X Y)
A. Yes  **B. No**  C. Can not be determined  D. All of the above

**Answer: B.** The bridge already learned X is on the right (step 3), so the frame stays on the right.

### 26. Quick formula & fact sheet
| Concept | Formula / value |
|---|---|
| Bits per QAM symbol | `log₂(M)` → 4096-QAM = 12 bits, 4-QAM = 2 bits |
| Shannon capacity | `C = B·log₂(1 + S/N)` |
| Bit time | `1 / R` (100 Mbps → 10 ns) |
| Bit length on wire | `v / R` (2×10⁸ / 10⁸ = 2 m) |
| Bits in flight | `d·R / v` |
| CSMA/CA backoff | random 0…max; decrement on idle slots; freeze when busy/NAV; send at 0 |
| No ACK | double max backoff, pick new random value |
| HDLC flag | `01111110` (0x7E); stuff 0 after five 1s |
| 4B/5B | 5 bits per 4 data bits → 80% efficient |
| Ethernet segment | ≤ 100 m; ≤ 4 repeaters → ≤ 1500 m |
| Bluetooth | 79 × 1 MHz channels, 1600 hops/s, ≤ 7 responders, 2.5 mW / 10 m |
| Zigbee | 802.15.4, 20–250 kbps, 10–100 m, 4-QAM |
| NFC | 13.56 MHz, ≤ 4 cm, 106–424 kbps |
| RFID | ID up to 96 bits; passive tag powered by reader |
| Wi-Fi standard | 802.11; Ethernet = 802.3; WPAN = 802.15 |

**In-class answers:** Token error → **D** · 4096-QAM → **B (12)** · Aggie OneCard → **D (NFC)** · Y → X across bridge → **B (No)**
