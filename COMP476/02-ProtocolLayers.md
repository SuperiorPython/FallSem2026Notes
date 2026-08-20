# COMP 476 — Protocol Layers

> Networked Computer Systems — Lecture 2 Notes

---

## Why Standards Matter

- The purpose of a network: allow two computers to communicate.
- **Standards** ensure sender and receiver agree on rules/format of data — just like North American electrical outlets standardize voltage (110V, 60Hz, up to 15A) so *any* compliant device works with *any* outlet.

## Standards Organizations (know the acronyms)

| Acronym | Full Name |
|---------|-----------|
| **ISO** | International Organization for Standardization |
| **IETF** | Internet Engineering Task Force |
| **IEEE** | Institute of Electrical and Electronics Engineers |
| **ITU** | International Telecommunication Union |

## Why We Use Layers

- Networks are almost always described using **layers** — concentrating on one layer at a time simplifies thinking about the whole system.
- **Layers are (mostly) independent**:
  - An HTTP request works the same whether it travels over Ethernet, fiber, or WiFi.
  - Occasionally a protocol at one layer *does* require specific support from the layer below it (a dependency).
- **US Mail analogy**:
  - You (the sender) don't need to know how the airline flies the plane.
  - The post office doesn't need to know how to fly a plane.
  - Each layer assumes the layer below provides certain functions, and adds its own functionality on top.
- **Programming analogy**: methodA calls methodB, which calls methodC/methodD. Upper-level code doesn't care *how* lower-level methods do their job — same idea as one network layer calling the layer below it.

## Three Core Layer Concepts

1. **Services** — each layer performs a well-defined function.
2. **Interfaces** — well-defined access point from the layer above.
3. **Protocols** — how a layer talks to its **peer layer** on a *different* machine.

## OSI Model vs. Internet Model

- **ISO OSI model** (created 1978) — the historical basis for almost all layered network models, but it is **rarely used directly** in practice.
- The **Internet protocol suite** follows the OSI model's spirit but **discards/merges some layers**.
- Per RFC 3439 (quoted on slides): avoid treating OSI/TCP-IP comparisons too strictly — TCP/IP layering wasn't a strict design principle.

| ISO OSI (7 layers) | Internet Model |
|---|---|
| Application | Application |
| Presentation | *(merged into Application)* |
| Session | *(merged into Application)* |
| Transport | Transport |
| Network | Internet |
| Data Link | Logical Link Control (upper half) |
| *(same)* | Media Access Control (lower half) |
| Physical | Physical |

## Layer-by-Layer Breakdown (bottom to top — this is the order the course follows)

**1. Physical Layer**
- Sends **bits** to the adjacent computer.
- Defines the transmission medium/hardware: electrical properties, frequencies, modulation, signals, connectors, media type.
- Examples: electrical connectors, cables, radio frequency, SONET, RS-232C.
- **This is the only layer that actually sends bits to another computer.**

**2. Data Link Layer** (split into two sublayers)
- **Media Access Control (MAC) sublayer** (lower half):
  - Frame synchronization; sends frames to an adjacent system.
  - Defines channel access for shared media (who gets to transmit).
  - Provides **physical addressing** (MAC addresses).
  - Examples: Ethernet, IEEE 802.11 (WiFi).
- **Logical Link Control (LLC) sublayer** (upper half), aka Network Interface Layer:
  - Performs error and flow control.
  - Encapsulates Network layer packets into **frames**.
  - Example: Point-to-Point Protocol (PPP).

**3. Internet / Network Layer**
- Routes packets across the Internet.
- Examples: **IP** (Internet Protocol), **ICMP** (Internet Control Message Protocol), IGMP, ARP, RARP.

**4. Transport Layer**
- **First end-to-end protocol** — communicates directly with the destination computer (not just the next hop).
- May provide error correction, flow control, and a reliable byte stream (**TCP**).
- Some transport protocols provide none of these (**UDP**) — just best-effort delivery.
- Routes data to a specific **application/port** on the destination computer.

**5. Session & Presentation Layers** (OSI only — mostly absorbed into Application in Internet model)
- **Session layer**: opens, closes, and manages sessions between end-user application processes.
- **Presentation layer**: formats received data to be compatible with the local system (e.g., encoding, encryption formatting).
- These provide little functionality on their own in the Internet model.

**6. Application Layer**
- Specifies how a pair of applications interact.
- Defines message format/meaning and the procedures applications follow.
- Examples: **HTTP**, SMTP, FTP, TFTP, DNS, SNMP, BOOTP.

## Common Internet Protocol Stack Diagram (memorize this!)

```
Application layer:  SMTP  FTP  TFTP  DNS  SNMP ... BOOTP  (also HTTP, Telnet, RIP)
Transport layer:     TCP              |         UDP
Network layer:      IGMP  ICMP        IP           ARP  RARP
Data link layer:              (Underlying LAN or WAN technology)
Physical layer:
```

## Byte Order: Big-Endian vs. Little-Endian — **exam-style question on slides**

> Q: The difference between Big Endian and Little Endian systems is:
> **Answer: C — The order of the bytes in an integer**

- Example: 32-bit integer `0A0B0C0D` stored at address `a`:
  - **Little-endian**: least significant byte first → `a:0D, a+1:0C, a+2:0B, a+3:0A`
  - **Big-endian**: most significant byte first → `a:0A, a+1:0B, a+2:0C, a+3:0D`
- This matters when systems with different byte orders communicate — protocols must define a **network byte order** (typically big-endian) so both ends interpret multi-byte values correctly.

## How Data Flows Through the Stack (OSI Flow Chart)

- Each layer communicates conceptually with its **matching peer layer** on the other computer.
- When a layer wants to send data to its peer, it calls a function in the layer **below** it to actually move the data.
- **Only the lowest layer (Physical) actually sends bits across the wire/air.**
- Sending computer: data flows *down* the stack (Application → ... → Physical).
- Receiving computer: data flows *up* the stack (Physical → ... → Application).

## Intermediate Devices (Routers)

- Lower layers (Physical, Data Link, Network/Internet) communicate with **adjacent neighbors** hop-by-hop.
- The **Network layer** decides where to send each packet next (routing decision) at every intermediate device.
- **Application and Transport layers** only communicate with their peer on the **final destination** system — intermediate routers don't process these layers.
- Visual: Host → [App/Transport/Internet/Link] → Router → [Internet/Link only] → Router → [Internet/Link only] → Host → [App/Transport/Internet/Link]

## Nested Protocol (Encapsulation) Headers

- As data passes down the stack, each layer adds its own **header** (and the Data Link layer often adds a **trailer** too):
  - Data → +AH (App Header) → +PH (Presentation) → +SH (Session) → +TH (Transport) → +NH (Network) → +DH (Data Link header) ... DT (Data Link trailer)
- The **Data Link trailer** typically contains a **Cyclic Redundancy Check (CRC)** used for error detection.
- It's the **bottom-most fully-wrapped frame** (all headers + trailer) that is actually transmitted across the network.
- At the receiving end, headers are **stripped off one by one** as the packet moves up the stack to the application.
- This is called **encapsulation** (going down) and **de-encapsulation** (going up).

## Stepping Through a Real Example (Python/Java HTTP request)

1. Your program calls an HTTP library method (e.g., Python's `requests.get(url)` or Java's `HttpClient`) → this is the **Application layer**.
2. The HTTP method calls **socket methods** at the **Transport layer** to format and send the request.
3. The Transport layer (e.g., using TCP) sends packets via **Network layer** methods.
4. The **Network layer** forwards the packet to the next adjacent computer closer to the destination; every intermediate router repeats this decision.
5. The **Data Link/MAC layers** format the packet into a frame for the local network segment; they only talk to *adjacent* systems.
6. The **Physical layer** actually transmits the bits.
7. On arrival, the process reverses — data is passed back up the stack, headers stripped at each layer, until the original response reaches the calling application.

- **General rule**: upper layers are usually implemented in **software**; lower layers are often implemented in **hardware**.
- The course will proceed **bottom-up**: physical layer first, moving up toward the application layer, so you understand what services each lower layer actually provides before studying what's built on top of it.

---

## Physical Media — Considerations & Comparison

### Seven Factors to Evaluate Any Transmission Medium (memorize the list!)

1. **Cost** — equipment purchase, installation, and operation.
2. **Mobility** — wired = not mobile; wireless varies from a couple meters to worldwide.
3. **Ease of installation and repair** — regulatory hurdles; some media (e.g., satellites) are very hard to repair.
4. **Attenuation** — loss of signal strength over distance; varies hugely by medium.
5. **Interference** — susceptibility to electromagnetic noise; impacts reliability.
6. **Security** — does the medium radiate signal (easy to intercept) or stay contained/private?
7. **Ability to cross public land** — in the US, only public utilities may string wire across a road; private wires need permission to cross others' property.

### Twisted Pair

- Pairs of copper wires **twisted together**; twisting reduces antenna-like interference pickup (more twists per length = less interference, but more wire needed).
- Used for landline telephones and Ethernet; 4 pairs of wires in an Ethernet cable.
- **Ethernet naming convention**: `speed` `BASE` `media`
  - `speed` = max Mbps; `media` = T (twisted pair) or F (fiber)
  - e.g., **10BaseT** = 10 Mbps copper; **100BaseF** = 100 Mbps fiber

| Category | Speed | Typical Use |
|----------|-------|-------------|
| Cat 3 | 16 Mbps | 10BaseT Ethernet |
| Cat 5 | 100 Mbps | 100BaseT Ethernet |
| Cat 5e | 1 Gbps | Gigabit Ethernet |
| Cat 6 | 10 Gbps | Gigabit Ethernet |
| Cat 7 | 10 Gbps | Gigabit Ethernet |
| Cat 8 | 25–40 Gbps | Datacenters |

- Cables differ in: wire thickness, shielding, twist rate, jacket type (plenum-safe plastic vs. underground-rated).
- **Crossover cables**: standard cables have two **T-568A** plugs; a device transmits on one pair, network equipment receives on that pair. A **crossover cable** connects two *user devices directly* (no switch/hub) by swapping transmit/receive pairs so each device's send signal lands on the other's receive pins.
- **Properties**: generally low cost; Cat 7 can run up to ~100m; ~55m runs can hit 40 Gbps; susceptible to interference (esp. telephone cable); easy to tap into (security risk); not mobile.

### Coaxial (Coax) Cable

- Used for cable TV and older Ethernet.
- Structure: inner conductor → insulation → foil shield → outer conductor (braided) → jacket.
- **Properties**: low cost (but more than twisted pair); can run hundreds of feet; more immune to interference than twisted pair; still easy to tap; not mobile.

### Fiber Optics

- Transmits data as **light** (infrared spectrum) through thin glass fibers.
- Two types:
  - **Multimode**: thicker, light bounces at multiple angles, cheaper, used for **short distances**.
  - **Single mode**: thinner, light travels in a straight path, faster, more expensive, used for **long distances**.
- **Attenuation varies by wavelength** — there are low-attenuation "bands" around 0.85µ, 1.30µ, and 1.55µ (the standard operating windows for fiber).
- **Repair methods**:
  - Connectors: lose ~10–20% of light energy.
  - Mechanical splices (align + clamp cut ends): ~10% energy loss.
  - **Fusion splicing** (melting two fiber ends together): minimal energy loss.
- **Undersea fiber cables**: connect continents; each cable bundles many fibers, each multiplexing many channels; signal must be **optically amplified roughly every 100 km**.
- **Properties**: moderately low cost; single-mode reaches tens of km, multimode only a few km; can reach **Terabit/second** speeds; lightweight; immune to electromagnetic interference; very hard to tap (no stray radiation = more secure); not mobile.

> **Slide question**: Compared to twisted pair, fiber optics are **more expensive and faster** (Answer A).

### Infrared

- Used by TV remote controls.
- Requires **line-of-sight** connection.
- Limited distance and transmission speed.
- Sunlight can interfere with signal.
- Inexpensive and mobile, but **not secure** (easily intercepted within line of sight).

### Quick Comparison Table

| Medium | Cost | Speed/Distance | Interference Immunity | Security | Mobile? |
|---|---|---|---|---|---|
| Twisted Pair | Low | Up to ~100m, 40 Gbps (short runs) | Low | Low (easy to tap) | No |
| Coax | Low–Med | Hundreds of feet | Medium | Low (easy to tap) | No |
| Fiber Optic | Med | Km range, up to Tbps | Very High (immune to EM) | High (hard to tap) | No |
| Infrared | Low | Short, line-of-sight only | Sensitive to sunlight | Low | Yes |

---

## Quick-Reference Summary

- **4 Standards orgs**: ISO, IETF, IEEE, ITU
- **3 Layer concepts**: Services, Interfaces, Protocols
- **5 Internet layers (bottom→top)**: Physical → Data Link (MAC + LLC) → Internet/Network → Transport → Application
- **Only Physical layer sends actual bits**
- **Transport layer = first end-to-end layer**
- **Big-endian vs little-endian = byte order of an integer**
- **7 media considerations**: cost, mobility, ease of install/repair, attenuation, interference, security, ability to cross public land
- **Fiber = more $ but faster than twisted pair**
- **Fusion splicing = best (lowest loss) fiber repair method**

---

*Notes generated from: "Protocol Layers" lecture slides.*
*Reading assigned: Sections 2.1 and 2.4 of the course textbook.*

**Related**: [← Introduction Notes](../01-Introduction/README.md)
