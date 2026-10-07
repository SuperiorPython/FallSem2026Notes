# COMP476 — Networked Computer Systems
## Lecture 13 — Network Layer Routing (slides: "Network Layer Routing," Wed Sept 30, 2026)

> "Power corrupts, and PowerPoint corrupts absolutely." — Vinton Cerf, inventor of TCP

**What's new vs. Lecture 12:** the **/21 address-mask example** (§4), the **Internet transmission** walk-through (§9), and all of **DHCP** (§12–§17) including the **`ipconfig` breakdown** (§18). Everything else is a condensed recap; see Lecture 12 for the long versions.

---

### 1. Where we are
```
Application
Transport
Internet / Network   ◄── YOU ARE HERE
Logical Data Link
Media Access Control
Physical
```
- **Diverse networks:** the Internet is many networks of **different types**, each with its **own addressing scheme**. On a Wi-Fi LAN, an Ethernet MAC is **meaningless**.
- **Overlaid network:** the Internet layer gives every device a **unique address independent of its hardware address**, so packets route over any network type using **IP addresses**.

### 2. Network identifiers (recap)
| Identifier | Used by | Example |
|---|---|---|
| **Internet name** | Humans | `www.ncat.edu` |
| **Internet (IP) address** | Routing (binary, written in decimal) | `152.8.240.16` |
| **Hardware (MAC) address** | Network hardware (hex) | `00-e0-63-03-76-c0` |

### 3. Addressing recap
- **IPv4 classes:** host address is A, B, or C; **prefix = network**, **suffix = host**; class set by the **first bits** (A `0` 0–127, B `10` 128–191, C `110` 192–223).
- **Classless (CIDR):** classes were **wasteful**; a 12-host network only needs a **4-bit hostid**. Write `ddd.ddd.ddd.ddd/m`, **m = netid bits**.
- **IPv6 unicast:** 128 bits = **routing prefix (48+)** · **subnet ID (≤16)** · **interface identifier (64)**.
- **Link-local:** every IPv6 host needs one, prefix **`fe80::/10`** + usually **54 zero bits**; interface ID often a **hash of the MAC**. Example: `fe80::cdac:fa7:98f4:b1cb%20` (`%20` = interface index).

### 4. IPv4 address mask ★ (new example)
- A **32-bit integer ANDed** with an IP address to separate netid from hostid.
- Mask for **`/x`** = **x one-bits, left-justified**, rest zeros.
- **Same-network test:** AND **both** addresses with the mask. **Same result → same network.**

**Example — is `213.5.23.149` on the same network as `213.5.18.51/21`?**

Mask `/21` = 21 ones + 11 zeros = **`255.255.248.0`** (third octet `11111000` = 248).
```
    11010101.00000101.00010010.00110011   213.5.18.51
AND 11111111.11111111.11111000.00000000   255.255.248.0
    11010101.00000101.00010000.00000000   213.5.16.0

    11010101.00000101.00010111.10010101   213.5.23.149
AND 11111111.11111111.11111000.00000000   255.255.248.0
    11010101.00000101.00010000.00000000   213.5.16.0
```
**Both → `213.5.16.0` → YES, same network.**

*Shortcut:* only the octet where the mask is "partial" matters. 18 AND 248 = 16; 23 AND 248 = 16. Match.

| Property (213.5.16.0/21) | Value |
|---|---|
| Host bits | 32 − 21 = **11** |
| Total addresses | 2¹¹ = **2048** |
| Usable hosts | 2048 − 2 = **2046** |
| Network address | 213.5.16.0 |
| Broadcast | 213.5.23.255 |
| Host range | 213.5.16.1 – 213.5.23.254 |

*Quick block-size trick:* partial octet mask 248 → block size `256 − 248 = 8`, so networks start at third-octet multiples of 8 (…, 8, **16**, 24, …). 18 and 23 both fall in the 16–23 block.

### 5. IPv4 & IPv6 headers (recap)
| Field | IPv4 | IPv6 |
|---|---|---|
| Version | 4 bits, always **4** | 4 bits, always **6** |
| Length | **Total Length** 16 bits → max **64 KB** | **Payload Length** 16 bits; **Jumbograms up to 4 GB** |
| Hop counter | **TTL** (8 bits), decremented per router, drop at 0 | **Hop Limit** (8 bits), same job |
| Upper protocol | **Protocol** (8 bits) → transport protocol | **Next Header** (8 bits) → TCP header or **extension header** |
| Checksum | **Header only**; IP does **not** check data | None |
| Addresses | 32 bits each | 128 bits each |
| Fragmentation fields | Identification (16), Flags (3, incl. **More Fragments**), Fragment Offset (13) | In an extension header |

- **IPv4 fragmentation:** MTU limits packet size (**Ethernet = 1500 bytes** data/frame). A **router** splits a too-big packet; **More Fragments = 1** on all but the last; the **destination reassembles**.
- **IPv6 extension headers** (zero or more after the fixed header): **routing**, **fragmentation**, **Encapsulating Security Payload**.
- **No IPv6 router fragmentation:** router sends an **ICMP message back to the source**; the **source** must fit the MTU. Simplifies routing.

### 6. ICMP (recap)
- Supporting network-layer protocol for **errors and operational info**; **not** for user data.
- Informs the source of: **Destination Unreachable**, **TTL time exceeded**, **Better gateway**, **Echo** (used by `ping`).

### 7. Address mapping (recap)
```
Internet name ──DNS──► IP address ──ARP (IPv4) / ND (IPv6)──► MAC address
```
- **DNS:** distributed database of names ↔ addresses; hosts query it; **hosts and DNS servers cache** results.
- **MAC:** hardware only understands MACs; an Ethernet interface knows **nothing** about IP. To send on the **same** network you need the MAC (or broadcast).
- **ARP:** source **broadcasts** "who has IPv4 X?"; owner **replies with its MAC**; everyone else ignores.
- **ND:** same job, uses **ICMP**, **Neighbor Solicitation via multicast**. **Secure ND** uses **public-key crypto**.
- **Caching:** IP↔MAC pairs and name↔IP pairs are cached (separately), so ARP/DNS are needed **once per destination**.
- **ARP poisoning / spoofing:** attacker sends fake ARP replies binding **their MAC** to a legit IP (often the **router**). Victims' caches update; traffic goes to the attacker, who can **view & forward, modify, or discard** it.

**In-class question:** *MAC addresses are…* → **B. Built into the network interface hardware.** (Not bigger for IPv6, not DHCP-assigned, not used to route across networks.)

### 8. IP encapsulation & data envelopes (recap)
- The **entire IP datagram** sits in the **frame data area**.
- **Frame dest = next hop's MAC**; IP dest = **final destination**.
- Each hardware type defines its **own envelope**. Router opens the envelope, takes out the IP packet, and if it must go further puts it in the **next network's envelope**.

### 9. Internet transmission ★
When a datagram arrives in a frame, the receiver **extracts the datagram and discards the frame header**.

```
Source host      [            datagram ]
     │  Net 1    [header 1 |  datagram ]   ← framed for Net 1
Router 1         [            datagram ]   ← header stripped
     │  Net 2    [header 2 |  datagram ]   ← re-framed for Net 2
Router 2         [            datagram ]
     │  Net 3    [header 3 |  datagram ]
Destination host [            datagram ]
```
- The **datagram is unchanged** end-to-end (apart from TTL/checksum); the **frame header is different on every network**.
- Same rule of thumb as before: **MACs change every hop; IP src/dst stay the same.**

### 10. Local routing decision (recap)
- Source must decide: **direct to destination** or **to a router/gateway**.
- Each host must know its **local DNS** and **default gateway** addresses.
- Routers are **visible**; the **source** decides to send to the router.

| | Local | Global |
|---|---|---|
| Test | netid(dest) **=** netid(src) | netid(dest) **≠** netid(src) |
| Frame to | Destination | **Gateway** |
| ARP/ND for | Destination's MAC | Gateway's MAC (*may be preconfigured*) |
| IP dest | Destination | **Still the final destination** |

**Procedure (A → B):** (1) DNS request for B → (2) DNS returns B's IP → (3) **B's IP AND A's subnet mask** → (4) same netid → send directly → (5) different → send to gateway → (6) gateway forwards toward B's domain → (7) B's domain gateway delivers.

### 11. Routing examples (same network as Lecture 12 §37)
| Host | IP | HW | Gateway | DNS |
|---|---|---|---|---|
| a.ncat.edu | 152.8.244.55 | 5 | 152.8.254.254 | 152.8.244.1 |
| DNS.ncat.edu | 152.8.244.1 | 3 | 152.8.254.254 | — |
| b.ncat.edu | 152.8.247.77 | 7 | 152.8.254.254 | 152.8.244.1 |
| gate.ncat.edu (router, ncat side) | 152.8.254.254 | 4 | | |
| router.acme.com (same router, acme side) | 176.5.4.3 | 2 | | |
| www.acme.com | 176.5.6.9 | 9 | 176.5.4.3 | 234.6.7.14 |
| www.b.com | 176.5.6.17 | 17 | 176.5.4.3 | 234.6.7.14 |

**Ex 1 — a.ncat.edu → b.ncat.edu (cold start)**
| # | Step | Src HW | Dst HW | Src IP | Dst IP |
|---|---|---|---|---|---|
| 1 | ARP request for DNS | 5 | broadcast | 152.8.244.55 | 152.8.244.1 |
| 2 | ARP reply DNS → A | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| 3 | DNS query for b | 5 | 3 | 152.8.244.55 | 152.8.244.1 |
| 4 | DNS reply | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| — | netid 152.8 = 152.8 → **local** | | | | |
| 5 | ARP request for B | 5 | broadcast | 152.8.244.55 | 152.8.247.77 |
| 6 | ARP reply B → A | 7 | 5 | 152.8.247.77 | 152.8.244.55 |
| 7 | **A → B datagram** | 5 | 7 | 152.8.244.55 | 152.8.247.77 |

**Follow-on (cached):** only step 7.

**Ex 2 — a.ncat.edu → www.acme.com (cold start)**
| # | Step | Src HW | Dst HW | Src IP | Dst IP |
|---|---|---|---|---|---|
| 1–4 | ARP DNS, DNS query/reply (same as Ex 1) | | | | |
| — | netid 176.5 ≠ 152.8 → **global** | | | | |
| 5 | ARP request for gateway | 5 | broadcast | 152.8.244.55 | 152.8.254.254 |
| 6 | ARP reply gateway → A | 4 | 5 | 152.8.254.254 | 152.8.244.55 |
| 7 | **A → gateway data** | 5 | **4** | 152.8.244.55 | **176.5.6.9** |
| 8 | Gateway ARP request for WWW | 2 | broadcast | 176.5.4.3 | 176.5.6.9 |
| 9 | WWW ARP reply | 9 | 2 | 176.5.6.9 | 176.5.4.3 |
| 10 | **Gateway → WWW data** | 2 | 9 | **152.8.244.55** | 176.5.6.9 |

**Traps (red on the slides):** step 7 Dst IP is the **final** destination, not the gateway; step 10 Src IP is still **A**. Router changes **MACs**, not IPs.

> ⚠️ **Netid in these examples:** the slides treat `152.8` / `176.5` as the netids (Class B, /16). The real campus `ipconfig` in §18 uses a **/22** mask, so on a real problem **use whatever mask you're given**. If no mask is given, fall back to the class.

---

### 12. DHCP ★
- **Dynamic Host Configuration Protocol:** gives a computer its **IP configuration when it boots**.
- With DHCP you **don't** manually configure the IP address etc. when installing TCP/IP.

### 13. DHCP servers
- Hand out configuration **created by the domain administrator**.
- The DHCP service can run on a machine that is **also the DNS**.
- All DHCP traffic uses **UDP ports 67 and 68**. *(67 = server, 68 = client.)*

### 14. DHCP address types
| Type | How it works | Used for |
|---|---|---|
| **Reserved / known** | Admin stores **[HW address : IP address]** pairs. A known MAC **always gets the same IP**. | **Web servers** (must have a fixed IP), faculty machines |
| **Pool** | Unknown MACs get **any available IP** from a pool; addresses are **recycled when released**. | Most **ISPs**, dorm students |

**A&T example:**
- **Faculty** computer boots → **broadcasts** DHCP request → its MAC is **registered** → server **always sends the same IP**.
- **Dorm student** computer boots → **broadcasts** DHCP request → server sends **an available address from the pool**.

### 15. Using DHCP — the 4-step lease (DORA)
| # | Step | Who | How | What happens |
|---|---|---|---|---|
| 1 | **IP lease request** *(Discover)* | Client | **Broadcast** | Client starts a **limited version of IP** and asks where a DHCP server is |
| 2 | **IP lease offer** *(Offer)* | **All** DHCP servers | To client | Every server that hears it sends an offer |
| 3 | **IP lease selection** *(Request)* | Client | **Broadcast** | Picks the **first offer received**, broadcasts a request to lease that IP |
| 4 | **IP lease acknowledgement** *(Ack)* | The offering server | To client | Confirms. **All other servers withdraw** their offers |

*Why step 3 is a broadcast:* it tells the **losing** servers to withdraw. *Mnemonic:* **D-O-R-A** = Discover, Offer, Request, Acknowledge (standard names; slides use "request / offer / selection / acknowledgement").

### 16. Renewing the lease
- IPs are **leased** for a limited time.
| Time into lease | Action |
|---|---|
| **½** | Try to renew with **the same server** that granted it |
| **⅞** | No renewal yet → **broadcast** a renewal request to **any** DHCP server |
| **End** (or refused) | **Immediately stop** using the IP address |

### 17. Without DHCP
You must manually configure:
- **IP name**
- **IP address**
- **Default gateway IP address**
- **DNS IP address**
- **Subnet mask**

DHCP can also provide extras like **WINS** (Windows Internet Name Service: NetBIOS name ↔ IP).

### 18. `ipconfig` output (campus machine) ★
```
Connection-specific DNS Suffix  : ncat.edu
Physical Address. . . . . . . . : 00-1E-C9-44-1D-21
DHCP Enabled. . . . . . . . . . : Yes
Autoconfiguration Enabled . . . : Yes
Link-local IPv6 Address . . . . : fe80::286:fbdb:816a:a134%10
IPv4 Address. . . . . . . . . . : 152.8.110.47(Preferred)
Subnet Mask . . . . . . . . . . : 255.255.252.0
Lease Obtained. . . . . . . . . : Wed, Sept 25, 2026 8:36:49 AM
Lease Expires . . . . . . . . . : Thurs, Sept 26, 2026 12:36:49 AM
Default Gateway . . . . . . . . : 152.8.111.254
                                  fe80::3698:b5ff:fed6:c6be%20
DHCP Server . . . . . . . . . . : 152.8.200.110
DNS Servers . . . . . . . . . . : 152.8.200.110
                                  2603:6080:b903:964f:3698:b5ff:fed6:c6be
```

**Reading it:**
| Line | Meaning / tie-in |
|---|---|
| Physical Address | The **MAC** (48-bit, hex) |
| Link-local IPv6 | `fe80::` prefix (§3); `%10` = interface index |
| IPv4 Address | 152.8.110.47 → first byte 152 = **Class B** |
| Subnet Mask | **255.255.252.0 = /22** (252 = `11111100`) → campus is **subnetted**, not a plain /16 |
| DHCP Server = DNS Server | Same box (`152.8.200.110`): DHCP **can run on the DNS** (§13) |
| Default Gateway | IPv4 gateway + an IPv6 **link-local** gateway |

**Worked: what subnet is this host on?**
```
110 = 01101110
252 = 11111100
AND = 01101100 = 108
```
→ Network **152.8.108.0/22**, range **152.8.108.0 – 152.8.111.255** (block size 256 − 252 = 4: 108, 109, 110, 111).
- Gateway **152.8.111.254** is inside that range ✔ (gateway must be local).
- Broadcast = 152.8.111.255; usable hosts = 2¹⁰ − 2 = **1022**.
- DHCP/DNS server 152.8.200.110 → 200 AND 252 = 200 → **different subnet**, reached through the router.

**Worked: lease timing**
- Obtained **8:36:49 AM** → Expires **12:36:49 AM** next day = **16-hour lease**.
- ½ (8 h) → renew with same server at **4:36:49 PM**.
- ⅞ (14 h) → broadcast to any server at **10:36:49 PM**.

> ⚠️ Slide says "Wed, Sept 25, 2026"; Sept 25, 2026 is actually a **Friday**. Doesn't affect the math.

### 19. Course logistics (from slides)
- **Queuing questions** were due 5:00 pm Wed Sept 30 (Canvas).
- **IP packet address questions** due 5:00 pm "Monday, October 4, 2026" (Canvas). *(Oct 4, 2026 is a Sunday; same conflict as Lecture 12.)*
- **Exam 2: Wed Oct 7, 2026**; returned Wed Oct 14. **Fall Break** Mon Oct 12. **Last day to drop:** Oct 19.

| Date | Topic | Reading |
|---|---|---|
| Wed Sept 30 | Routing | 5.5 |
| Mon Oct 5 | Routers & review | — |
| Wed Oct 7 | **Exam 2** | — |
| Mon Oct 12 | Fall Break | — |
| Wed Oct 14 | Transport layer | 7.1 |
| Mon Oct 19 | Sockets | 5.2 – 5.4 |
| Wed Oct 21 | Online APIs | 6.5 |

**Final exam policies:**
- **Conflict resolution:** not required to take **more than two finals in a day**; notify the instructor **as soon as** the conflict is found with your **complete exam schedule**; work out an alternative.
- **Optional final:** if the instructor determines the final is **statistically unlikely to change your grade**, it's optional. You can **always** choose to take it. If you skip, grade = **weighted average of all other graded work**.

### 20. Quick formula & fact sheet
| Concept | Formula / value |
|---|---|
| /x mask | x ones left-justified, rest zeros |
| Same network? | (IP₁ AND mask) == (IP₂ AND mask) |
| Block size (partial octet) | 256 − mask octet |
| /21 | 255.255.248.0 · 2046 hosts |
| /22 | 255.255.252.0 · 1022 hosts |
| Usable hosts | 2^(32−m) − 2 |
| Datagram vs. frame | Datagram unchanged; frame header replaced every network |
| Frame dest / IP dest | Next hop's MAC / final destination |
| DHCP | Boot-time IP config; **UDP 67/68** |
| DHCP types | Reserved (MAC→fixed IP, web servers) vs. pool (recycled, ISPs) |
| Lease steps | Request (bcast) → Offer (all servers) → Selection (bcast, first offer) → Ack (others withdraw) |
| Renewal | ½ same server · ⅞ broadcast any · expire/refused → stop using IP |
| Manual config without DHCP | name, IP, gateway, DNS, subnet mask |
