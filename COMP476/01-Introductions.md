# COMP 476 — Introduction

> Networked Computer Systems — Lecture 1 Notes

---

## Why Networks Matter

- **Core idea**: People buy computers to *network*, not just to compute. Email, web browsing, and connected devices are the dominant uses of computers today.
- **Internet of Things (IoT)**: sensors + everyday devices networked to computers that analyze data.
  - Examples: doorbell cameras, remote thermostats, smart speakers, wireless sensor networks.
- **Scale of the Internet**:
  - IPv4 host count has grown from a handful in 1970 to roughly a billion+ hosts (log-scale growth curve) — note this undercounts IPv6 devices.
  - As of 2025, **~74% of the global population** uses the Internet.
  - **English** is the most used language on the Internet (~55%), followed by Russian, German, Spanish, Chinese, French, Japanese, Arabic, Portuguese.

## Course Scope (What COMP 476 Covers)

The course builds up the network stack from the physical layer upward, answering:
1. How do we send bits physically (wires, fiber, wireless)?
2. How fast can we send bits (bandwidth/speed limits)?
3. How do modems/LANs work, and how are networks interconnected (switches, hubs)?
4. How does the phone network work?
5. How does data find its way across the Internet (routing)?
6. What is TCP? IPv4 vs IPv6? What other protocol options exist?
7. What does HTTP do (application layer, foundation of the Web)?
8. How do you secure a network?

**Important distinction**: COMP 476 does **not** teach web page development (HTML/CSS) — that's COMP 322 (Internet Systems). This course covers the *networks* that support the web.

## Brief History of Networking

| Year | Event |
|------|-------|
| 1793 | Claude Chappe develops the **optical telegraph** |
| 1844 | Samuel Morse develops the **electrical telegraph** |
| 1850 | First undersea telegraph cable laid |
| 1876 | Alexander Graham Bell invents the **telephone** |
| 1969 | **ARPANET** created at UCLA, connected at 50 Kbps |
| 1970 | AlohaNet operational |
| 1971 | 15 nodes on ARPANET |
| 1973 | Bob Metcalfe's PhD thesis outlines **Ethernet** |
| 1974 | Vint Cerf & Bob Kahn publish the **TCP** design |
| 1978 | TCP split into **TCP** and **IP** |
| 1981 | BITNET & CSNet created; TCP/IP becomes the Internet standard |
| 1983 | Name server (DNS precursor) developed at U. of Wisconsin |
| 1986 | **IETF** (Internet Engineering Task Force) created |
| 1991 | **WWW** released by CERN (Tim Berners-Lee) |
| 1993 | **W3C** (World Wide Web Consortium) created |
| 1995 | Sun launches **Java** |

**Key people to remember**: Chappe (optical telegraph), Morse (electrical telegraph), Bell (telephone), Metcalfe (Ethernet), Cerf & Kahn (TCP/IP).

## Public vs. Private Networks

- **Public network**: a service available to any paying subscriber; the company offering it is a **service provider**.
- **Private network**: restricted to one particular group/organization; may include leased circuits from a provider.

## Network Speed

- Measured in **bits per second** (bps), not bytes.
- Gigabit Ethernet = high speed for home use.
- Analogy: think of network "speed" as network **volume/flow rate**, like a river — the Mississippi River flows ~3 mph but moves about 1 billion cubic inches of water per second. A network's bps is analogous to that volumetric flow, not necessarily how fast any single bit "travels."

## Network Design Goals (4 Major Goals) — *likely testable*

1. **Reliability**
   - Make the network work correctly even though built from unreliable components.
   - **Error detection**: finds errors in received data.
   - **Error correction**: recovers/corrects the incorrect bits.
   - **Routing**: finds a working path through the network automatically.

2. **Resource Allocation**
   - **Scalability**: design continues to work as the network grows large.
   - **Statistical multiplexing**: sharing resources based on statistics of demand (not fixed dedicated slots).
   - **Flow control**: keeps a fast sender from overwhelming a slow receiver.
   - **Congestion**: occurs when too many senders want to push more traffic than the network can deliver.
   - **Quality of Service (QoS)**: reconciles competing demands for bandwidth/priority.

3. **Evolvability**
   - Networks must grow and change over time; new tech needs to interoperate with the old.
   - **Protocol layering** is the key structuring mechanism — it divides the overall problem and hides implementation details so layers can evolve independently.

4. **Security** — remember with mnemonic **C.I.A. + Authentication**:
   - **C**onfidentiality — defends against eavesdropping.
   - **I**ntegrity — prevents surreptitious (sneaky) changes to messages.
   - **A**ccessibility — available when you want to use it.
   - **Authentication** — prevents someone from impersonating someone else.

---

## Quick-Reference Summary

- **4 Design Goals**: Reliability, Resource Allocation, Evolvability, Security (CIA + Authentication)
- Network speed = **bits per second**, think "volume" not "velocity"
- COMP 476 ≠ web dev (that's COMP 322)
- Know the history timeline, especially: ARPANET (1969), Ethernet thesis (1973), TCP design (1974), TCP/IP split (1978), WWW (1991)

---

*Notes generated from: "Introduction" (COMP476/467) lecture slides.*
*Reading assigned: Sections 1.5, 1.6 (scan rest of Ch.1) of the course textbook.*

**Related**: [Protocol Layers Notes →](../02-Protocol-Layers/README.md)
