# COMP476 — Networked Computer Systems
## Lecture 8 — Telephone Systems (slides: "Telephone Systems," Mon Sept 14, 2026)

> "You should not be a slave to your telephone. The technology is there to serve you, not the other way around." — Martin Cooper, placed the first public call from a handheld portable cell phone.

### 1. Telephone's quick popularity
- **Alexander Graham Bell** is credited with inventing the telephone after receiving the **first U.S. patent** for the device in **1876**.
- Telephones quickly became very popular, particularly in cities where it was easy to install them.
- There are approximately **8.3 billion mobile phones** in the world.

### 2. Politics of telephones
- Prior to **1984** the **Bell System** provided local and long-distance service for most of the United States.
- Before 1984, the **telephone company owned your phone**.
- The **"Modified Final Agreement"** split the system into **AT&T Long Lines** and **23 Bell Operating Companies**.
- **Local Exchange Carriers (LECs)** provided traditional phone service in an area.
- Long distance was from a **separate company**.
- In **1996**, Congress passed a law allowing the various telephone companies to enter each other's business.

### 3. Home land lines
- **Plain Old Telephone Service (POTS)** provides a **twisted pair** connection from your phone to the central office.
- You own your **home wiring**; the telephone company owns the wires **outside** your home.

### 4. Central office
- There is a central office for each local **three number prefix** (or subset).
- The central office has a **computer-controlled switch**.
- **Local calls** are connected within the switch.
- Calls to other switches are **digitized** using a **coder-decoder (codec)**.
- The U.S. has over **22,000 central offices**.

### 5. Telephone connections & local loop
- The first phones required a **human operator** to make a connection; you asked the operator to connect you to a destination — and the operator could **listen in** on your conversation.
- The wiring from your home phone to the central office is sometimes called the **local loop**.
- For calls within the same central office, the equipment connects one phone directly to another.

```
Phone A ──(local loop)──► Central Office ──(local loop)──► Phone B
                                │
                                └──► to another central office
```

### 6. Rotary dial → push button → touch tone
- **1919**: the **rotary dial** telephone enabled customers to make a connection themselves. As the dial rotated, it **clicked the voice line on and off**, and the telephone exchange rotated switches to make the connection.
- A 1920 ad promoted dial phones over operator-assisted connections as a **privacy** advantage ("secret service") — no operator listening in.
- **Early 1960s**: dial phones were replaced with **push-button phones**. Pressing a button generated a **sinusoidal tone** telling the switching equipment what number was pressed. The switching system also used other tones for **system control**.
- Later, phone companies switched to a more complex **Touch Tone** signal:
  - Each button's tone is a combination of a **row frequency** and a **column frequency** — think of the tone as a **chord**, not a single note.
  - In the **1980s**, the telephone company moved signaling to a **separate channel** (away from touch tone), in response to hacking (below).

### 7. Phone signal hacking / phreaking
- The original signaling design **assumed everyone would be well behaved**.
- **In-band signaling** used the regular voice channel to control the phone call — so users could create the signaling commands themselves to avoid paying for calls.
- A **2400 Hz tone** (the third D note above middle C) signaled the **end of a phone call**.
- A **toy whistle** that came in boxes of **Captain Crunch** cereal happened to create a 2400 Hz note — famously exploited by "phone phreaks."
- People built **"Blue Boxes"** that generated the tones required to make free phone calls.
- **Cybersecurity rule (the moral of the story):** never combine control information with the data channel — users should not be able to access the control information. *(This is why signaling was later moved out-of-band.)*

### 8. Analog and digital
- The **POTS twisted pair** line between your home land line phone and the central office runs an **analog** signal.
- Communications **between central offices** is done using **digital** lines.
- **Long distance** calls are carried over digital lines.

### 9. Frequency limits & sampling (Nyquist–Shannon)
- If an analog signal's frequency is too high, it cannot be converted properly to digital.
- **Nyquist–Shannon sampling theorem**: you must sample at a rate **greater than twice** the highest frequency present in the signal.
- Phones have a **4 KHz bandwidth**, so the voice signal must be sampled **8000 times a second**.

### 10. A/D conversion
- The analog voice signal is converted to a digital **stream of bytes**.
- **8000 times a second** (every **125 μsec**), an **8-bit** sample is taken.

### 11. POTS codec
- Sending 8000 eight-bit values every second requires **64 Kbits/sec** for a voice channel.
  - `8000 samples/sec × 8 bits/sample = 64,000 bits/sec = 64 Kbps`
- Some systems take **7-bit** samples to transmit data at **56K** (classic 56K modem limit).

### 12. Interoffice traffic & routing hierarchy
- Calls between central offices are transmitted **digitally** over **time division multiplexed (TDM)** lines.
- Long distance calls may travel through **many switches** and **several companies' equipment**, following a routing hierarchy of interconnected switches.

### 13. PBX (Private Business Exchange)
- A PBX is like a local office but **owned by the organization** that owns the phones.
- Companies can install a PBX, purchase a multiplexed line from the phone company, and **not pay for each individual telephone**.
- Businesses used to have to pay for **local calls**.

### 14. Multiplexed phone lines (T-carrier hierarchy)
- Digitized voice channels can be **combined on one line**.

| Line | Made from | Data rate |
|---|---|---|
| **T1** | 24 voice channels on a single line | **1.544 Mbits/sec** |
| **T2** | 4 × T1 | 6.3 Mbits/sec |
| **T3** | 7 × T2 | 44.7 Mbits/sec |
| **T4** | 6 × T3 | 274 Mbits/sec |

- Different countries use different multiplexing systems.

**T1 line structure:**
- **193-bit frame**, transmitted every **125 μsec** (same cadence as the sampling rate).
- Frame = 24 channels, each contributing **8 bits per sample**: **bit 1** of the channel's 8 is used for **signaling**, leaving **7 data bits** per channel per sample; plus **1 extra framing bit** at the start of the whole frame.
- `193 bits / (125 × 10⁻⁶ sec) = 1.544 Mbits/sec` ✓
- **Signaling bit rate for a single channel:** one signaling bit every 125 μsec → `1 bit / (125 × 10⁻⁶ sec) = 8000 bits/sec` (**8 Kbps**).

### 15. Mobile phones — cells
- Many different cell phone technologies exist; the world has moved from **analog** phones to **digital** cell phones.
- The world is divided into **cells**, each with an antenna.
- Cells range from **~20 meters** in diameter (indoors) to **~50 km** (rural areas).

### 16. Mobile phone central office (MTSO)
- A **Mobile Telephone Switching Office (MTSO)** receives radio signals from a cell phone and transmits the messages onto a land line.
- The MTSO **keeps track of where you are**.
- Every so often a cell phone **announces its location**.
- As you move to a different cell, the **next antenna handles your call**.

### 17. Cell phone codec & compression
- Like land lines, a mobile phone initially takes **8000 samples of 8 bits every second** for **64 Kbps**.
- The phone signal is then **compressed**, reducing the data rate to around **4.75–12.2 Kbps**.

### 18. Enhanced Voice Services (EVS)
EVS is a speech audio coding standard commonly used in cell phones, providing:
- **Source-controlled variable bit-rate** — the output bit rate varies depending on the instantaneous level of compression.
- **Voice/sound activity detector** — detects when you are not speaking.
- **Comfort noise generation** — synthetic background noise fills in the removed silence.

**EVS error concealment:**
- Packets of voice signal can be **lost in transmission**.
- EVS aims to **minimize the deterioration** of signals caused by packet loss.
- It can **replace lost packets** with copies of previously received packets.
- EVS may **interpolate** — making educated guesses about the nature of a missing packet, e.g. by following speech patterns in the audio.

### 19. Jitter
- **Jitter** = the variance in the rate at which packets are received.
- Jitter problems can be reduced by **buffering**.
- This is why you see or hear a short delay before music/video playback starts.

### 20. Cell phone identifiers
- **Electronic Serial Number (ESN)** — a unique **32-bit** number programmed into the phone when manufactured.
- **Mobile Identification Number (MIN)** — a **10-digit** number derived from your phone's number.
- **System Identification Code (SID)** — a unique **5-digit** number assigned to each carrier by the FCC.

### 21. Cell phone startup & handoff
**Startup:**
- When you power up the phone, it listens for a **SID** on the **control channel** from the MTSO.
- When it receives the SID, the phone **compares it** to the SID programmed into the phone. If they match, the phone knows the cell it's communicating with is part of its **home** system.

**Handoff:**
- When you move to a different cell, another MTSO must handle your call.
- When the signal from another MTSO is **stronger** than the one currently in use, the system performs a **handoff**.
- The new MTSO handles the communications and the previous MTSO stops communicating.
- The new MTSO needs to establish a connection over the land lines.

### 22. 5G service
- 5G cell service uses a **broader spectrum** of frequencies and **smaller cells** to provide faster service.
- 5G allows for many signals to be sent/received **simultaneously**, significantly increasing network capacity.

### 23. Voice over IP (VoIP)
**Advantages:**
- Use the **same network** for voice and data.
- **Routers** are cheaper than phone switches.

**Challenges:**
- **IP was not designed for real-time** traffic.
- Networks must **convert phone numbers to IP addresses**.

**VoIP telephone system components** (IP telephone ↔ Internet ↔ gateways ↔ PSTN ↔ analog telephone):
- **Media gateway** — handles the actual voice data.
- **Signaling gateway** — handles call setup.
- **Media gateway controller** — coordinates the other gateways.

### 24. Real-time communications
- Networks once designed to reliably get data to a destination *eventually* are now used for **real-time multimedia**.
- **Real-time communications** = data that must be reproduced at the **same rate** at which it was created.
- Telephone, audio, and video are examples of real-time multimedia.

### 25. Real-time Transport Protocol (RTP)
- RTP is a mechanism for transmitting real-time data across the Internet.
- The protocol actually sits **above the transport layer** (despite the name).
- RTP header fields include: VER, P (padding), X (extension), CC (CSRC count), M (marker), **PAYTYPE**, **SEQUENCE NUMBER**, **TIMESTAMP**, **SYNCHRONIZATION SOURCE IDENTIFIER**, and (optionally) **CONTRIBUTING SOURCE IDENTIFIER**.

**RTP encapsulation:**
- RTP uses **UDP** for message transport.
- Each RTP message is encapsulated in a UDP datagram for transmission over the Internet.
- Resulting messages can be sent via **broadcast** or **multicast**.

```
Frame Header │ IP Header │ UDP Hdr │ RTP Hdr │ RTP Payload │
              └────────────────────────────────────────────┘
                       (each layer wraps the one above)
```

### 26. Wireless home phones & cordless vs. cell
- **Wireless home phones** are wired land lines without the cord between the phone base and the handset.
- **Cordless phones are completely different from cell phones.**
- A cordless phone should only connect to **your own** phone line.
- Cordless phones are good for **tens of meters** of range.
- Cordless phones use a radio signal to connect to a **base station owned by the user**, and connect into the owner's **home phone wiring**.
- It's advantageous if different cordless phones are **incompatible** (prevents interference/crosstalk between neighbors' cordless phones).

### 27. Admin notes from this lecture
- **Solutions to the Exam 1 Review quiz** are posted on Canvas under **Files / Solutions and Information**.
- Per the schedule noted last lecture: **Exam 1 was Wednesday Sept 16, 2026.**

### 28. In-class practice questions (from slides)
1. **What else takes 125 μsec?** *(besides one 8-bit voice sample)* — Answer relates to the T1 frame: sending an entire **193-bit T1 frame** also takes 125 μsec. (Options given: circuit switching setup time / RS-232-C byte transmission time / send an STM-1 SONET frame / transmit 64 chips with CDMA — the T1-frame-adjacent answer is the SONET/T1 framing cadence.)
2. **What is the bit rate for signaling on a single channel** of a T1 line? → **8 Kbps** (B). *(1 signaling bit every 125 μsec = 8000 bits/sec.)*
3. **When a packet of cell phone voice is lost in transmission, the receiver might...** → play the previous packet again / interpolate based on speech patterns (EVS error concealment — no retransmission, since it's real-time).
4. **If a voice signal is encoded to 10 Kbps of voice data, VoIP will require...** → **more than 10 Kbps** (C) — because of RTP/UDP/IP/frame header overhead added on top of the payload (see §25 encapsulation diagram).
