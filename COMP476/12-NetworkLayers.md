# COMP476 — Networked Computer Systems
## Lecture 12 — Network Layer (slides: "Network Layer," Mon Sept 28, 2026)

> "The question of whether a computer can think is no more interesting than the question of whether a submarine can swim." — Edsger Dijkstra

### 1. Where we are
```
┌──────────────────────┐
│     Application      │
├──────────────────────┤
│      Transport       │
├──────────────────────┤
│ Internet / Network   │  ◄── YOU ARE HERE
├──────────────────────┤
│  Logical Data Link   │
├──────────────────────┤
│ Media Access Control │
├──────────────────────┤
│       Physical       │
└──────────────────────┘
```
- The layer above Data Link is the **Network layer**, also called the **Internet layer**.
- **Primary purpose: routing.**
- Sends a packet **across multiple networks** to its **final destination**.
- Physical and Data Link layers involve only a **single network**; the Internet layer creates an **interconnected internet**.

### 2. Interconnecting networks (recap from Lecture 11 §24)
| Layer | Connector |
|---|---|
| Application | Router *(gateway)* |
| Transport | — |
| **Internet** | **Router** |
| **Data Link** | **Bridge** |
| **Physical** | **Repeater** |

- **Repeaters (Physical):** copy individual bits between segments, **including collisions**; act as amplifiers; **invisible**; Ethernet → **1500 m** with **≤ 4 repeaters** between hosts.
- **Bridges (Data Link):** **store-and-forward frames**; add delay; **invisible**; **frame filtering**; **broadcasts always pass**; **learn** host locations; long-distance bridges can be joined by a **point-to-point link** (fiber, leased line, satellite).
- **Learning bridges:** no configuration; learn sides from **source addresses**; if the bridge **doesn't know** whether the destination is on the same side, it **forwards** (floods). *(This deck has the wording right; the Lecture 11 deck had the typo.)*
- **Routers (Network):**
  - Connect networks of **different types** (Ethernet ↔ WLAN).
  - Provide **routing**.
  - **Are visible** to computers on the network: **computers must send messages to a router**.

### 3. Connection visibility
| Device | Visible to hosts? | How hosts use it |
|---|---|---|
| Repeater | **No** | Works "magically"; sender doesn't know it's there |
| Bridge | **No** | Same: moves frames to other segments silently |
| Router | **Yes** | Host must **explicitly address frames to the router**; hosts **must know their routers** |

### 4. Packet switches & terminology
- A small **WAN** is formed by interconnecting **packet switches**.
- Links **between** packet switches usually run **faster** than links to individual computers.
- WAN interconnect devices are often called **switches**, but by our hierarchy they're **really routers**. *Terminology is not always consistent.*

### 5. Network model (graph)
- Each **node = a packet switch**; each **edge = a connection** between switches.

```
   (1)                 (2)
     \               /  |
      \            /    |
       \         /      |
        (3) ────────── (4)
```
Edges: **1–3, 3–2, 3–4, 2–4**.

### 6. Next-hop forwarding
- A switch only knows the **next place (hop)** to send a packet so it eventually reaches its destination.
- **Each switch has different next-hop information.**

**Example — table in switch 2** (addresses are `[switch, port]`; A,B on switch 1; C,D on switch 3; E,F on switch 2):

| Destination | Next hop |
|---|---|
| [1,2] | interface 1 (→ switch 1) |
| [1,5] | interface 1 |
| [3,2] | interface 4 (→ switch 3) |
| [3,5] | interface 4 |
| [2,1] | computer E (local) |
| [2,6] | computer F (local) |

*Key idea:* the table only depends on the **switch number** part of the address, not the port, until the packet reaches its own switch.

### 7. Next-hop routing tables (graph in §5)
| node 1 | | node 2 | | node 3 | | node 4 | |
|---|---|---|---|---|---|---|---|
| **dest** | **next** | **dest** | **next** | **dest** | **next** | **dest** | **next** |
| 1 | – | 1 | 3 | 1 | 1 | 1 | 3 |
| 2 | 3 | 2 | – | 2 | 2 | 2 | 2 |
| 3 | 3 | 3 | 3 | 3 | – | 3 | 3 |
| 4 | 3 | 4 | 4 | 4 | 4 | 4 | – |

- Node 1 has only one link (to 3), so **everything goes to 3**.
- `–` means "that's me."

### 8. Routing tables with alternatives
- Tables can hold a **second-choice route**.
- If the first route is **unavailable or congested**, send on the **alternate interface**.

| node 1 | | | node 2 | | | node 3 | | | node 4 | | |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **dest** | **next** | **alt** | **dest** | **next** | **alt** | **dest** | **next** | **alt** | **dest** | **next** | **alt** |
| 1 | – | – | 1 | 3 | 4 | 1 | 1 | – | 1 | 3 | 2 |
| 2 | 3 | – | 2 | – | – | 2 | 2 | 4 | 2 | 2 | 3 |
| 3 | 3 | – | 3 | 3 | 4 | 3 | – | – | 3 | 3 | 2 |
| 4 | 3 | – | 4 | 4 | 3 | 4 | 4 | 2 | 4 | – | – |

- Node 1 has **no alternates** (only one link). Node 3 → 1 has no alternate (3 is node 1's only neighbor).

### 9. Hierarchical addressing / routing
- **Hierarchical addresses simplify routing.**
- **Local routing** happens **without knowledge of distant nodes**.
- Packet for a **distant** node → handed to a **higher-level node**.
- Higher-level nodes get it to the **right group**; **local routing** there gets it to the **right node**.
- *(Slide figure: three clusters of hosts, each hanging off a yellow higher-level router; the routers are linked by thick backbone edges.)*
- Real-world analogy: the IP address's **netid** picks the network, the **hostid** picks the host (§16).

### 10. Route generation
- **Global routing is done by local decisions** (each node just picks a next hop).
- A routing table can be created:
  1. By a **central system** and **distributed** to nodes
  2. By the **sending node** for **each packet**
  3. **"Learned"** by each node **from its neighbors**

### 11. Optimal routes
Algorithms for the optimal path:
- **Dijkstra's algorithm**
- **Floyd–Warshall algorithm**
- **Distributed algorithm**

**A good but not necessarily optimal path can generally be used.**

### 12. Dijkstra's algorithm
- Popular method; finds the **shortest path from a source** to a destination.
- Uses **edge weights** as distance.
- **The path with the fewest edges may not be the path with the least weight.**

**Example graph** (edge weights):
```
 (1)        (2) ──3── (3) ──11── (4)
   \9      6/   \8      |2        |3
    \      /     \      |         |
     (5) ─        (6) ──5────── (7)
```
Edges: 1–5 = 9, 5–2 = 6, 2–3 = 3, 2–6 = 8, 3–6 = 2, 3–4 = 11, 6–7 = 5, 7–4 = 3.

**Shortest path 4 → 5:** `4 → 7 → 6 → 3 → 2 → 5`
`3 + 5 + 2 + 3 + 6 = 19`

*Why not the "obvious" route?* `4 → 3 → 2 → 5` has only 3 edges but costs `11 + 3 + 6 = 20`. Fewer hops ≠ shorter path.

**How Dijkstra works (quick version):**
1. Set dist(source) = 0, all others = ∞.
2. Pick the **unvisited node with the smallest** dist; mark it visited.
3. For each neighbor: `dist[n] = min(dist[n], dist[cur] + w(cur, n))`.
4. Repeat until the destination is visited.

### 13. Floyd's (Floyd–Warshall) algorithm
Computes **all-pairs** shortest paths. For every intermediate `k`, check whether going **through k** is shorter than the known path.

```
for k from 1 to |V|
   for i from 1 to |V|
      for j from 1 to |V|
         if dist[i][j] > dist[i][k] + dist[k][j]
            dist[i][j] = dist[i][k] + dist[k][j]
```

**Example graph** (from the starting table): 1–3 = 4, 2–3 = 5, 2–4 = 1, 3–4 = 7.

**Step 0 — start:** neighbors get link distance, everyone else ∞.

| src\dst | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| **1** | 0 | ∞ | 4 | ∞ |
| **2** | ∞ | 0 | 5 | 1 |
| **3** | 4 | 5 | 0 | 7 |
| **4** | ∞ | 1 | 7 | 0 |

**Step 1 — 1 → 2 via 3:** `4 + 5 = 9`.

**Step 2 — 1 → 4 via 3:** `4 + 7 = 11` (also fills 2→1 = 9, 4→1 = 11 by symmetry).

| src\dst | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| **1** | 0 | 9 | 4 | 11 |
| **2** | 9 | 0 | 5 | 1 |
| **3** | 4 | 5 | 0 | 7 |
| **4** | 11 | 1 | 7 | 0 |

**Step 3 — iterate until no changes.** 3 → 4 via 2: `5 + 1 = 6 < 7` ✔. That also improves 1 → 4: `4 + 6 = 10 < 11` ✔.

**Final:**
| src\dst | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| **1** | 0 | 9 | 4 | 10 |
| **2** | 9 | 0 | 5 | 1 |
| **3** | 4 | 5 | 0 | 6 |
| **4** | 10 | 1 | 6 | 0 |

**Keeping the next hop:** a small modification stores the **next node** on the shortest path with each distance (written `distance next-hop`):

| src\dst | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| **1** | 0 | 9 ³ | 4 ³ | 10 ³ |
| **2** | 9 ³ | 0 | 5 ³ | 1 ⁴ |
| **3** | 4 ¹ | 5 ² | 0 | 6 ² |
| **4** | 10 ² | 1 ² | 6 ² | 0 |

→ This **is** a next-hop routing table for each node (read a row).

**Why Floyd can't route the Internet** — it's **O(n³)**. At 1.0 ns per step with 1.5 billion nodes in the U.S.:
```
Time = (1.5 × 10⁹)³ × 1.0 × 10⁻⁹ = 3.375 × 10¹⁸ s
     3.375 × 10¹⁸ s / 3600     = 9.38 × 10¹⁴ hours
     9.38 × 10¹⁴ h  / 24       = 3.9 × 10¹³ days
     3.9 × 10¹³ d   / 365.25   ≈ 107 billion years
```

### 14. Distributed routing
1. Each node computes the **time to send a packet to each neighbor**.
2. **Periodically**, each node **shares its routing table with its neighbors**.
3. Each node keeps:
$$\text{dist}(dest) = \min\big(\text{current},\ \text{time to neighbor} + \text{neighbor's time to dest}\big)$$

*(This is the distance-vector / Bellman-Ford idea; it's how tables get "learned from neighbors" in §10.)*

### 15. Diverse networks & the overlay
- The Internet is **many networks of different types**, each with its **own addressing scheme**.
- On a Wi-Fi LAN, an **Ethernet MAC address is meaningless**.
- **Overlaid network:** the Internet layer gives every device a **unique address independent of its hardware address**, so packets can be routed over any network type using **Internet addresses**.

### 16. Network identifiers
Every **host** has **at least three** identifiers:

| Identifier | Used by | Example |
|---|---|---|
| **Internet name** | Humans | `www.ncat.edu` |
| **Internet (IP) address** | Routing (binary, written in decimal) | `152.8.240.16` |
| **Hardware (MAC) address** | Network hardware (hex) | `00-e0-63-03-76-c0` |

**Internet names:**
- Hierarchical, **read from the right**: `host.subnet.organization.type`
- Rightmost = **type, organization, or country**:
  - `edu, com, gov, mil, org, net`
  - `us, ca, de, uk`
  - `biz, ai, online, io`

**Internet addresses:**
- Names map to addresses.
- Two parts: **netid (prefix)** + **hostid (suffix)**.
  - **netid** = which **network** the host is on.
  - **hostid** = which **host** on that network.
- A computer on **two networks needs two IP addresses**.

### 17. Address formats
- **IPv4: 32-bit** addresses. **IPv6: 128-bit** addresses.
- IPv4 written as **four decimal numbers**, one per byte: `152.8.110.47`.
- **Be able to think of it as a binary number.**

**Writing IPv6:** 8 groups of 4 lowercase hex digits separated by colons.
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334   full
2001:db8:85a3:0:0:8a2e:370:7334           drop leading zeros
2001:db8:85a3::8a2e:370:7334              :: replaces consecutive zero groups (ONLY ONCE)
```
*Why only once:* with two `::` you couldn't tell how many zero groups each one stands for.

**Binary ↔ dotted decimal** (IP uses **Big Endian**: most significant byte first):

| 32-bit binary | Dotted decimal |
|---|---|
| 10000001 00110100 00000110 00000000 | 129.52.6.0 |
| 11000000 00000101 00110000 00000011 | 192.5.48.3 |
| 00001010 00000010 00000000 00100101 | 10.2.0.37 |
| 10000000 00001010 00000010 00000011 | 128.10.2.3 |
| 10000000 10000000 11111111 00000000 | 128.128.255.0 |

Bit weights per octet: `128 64 32 16 8 4 2 1`.

### 18. IPv4 address classes
- Host addresses are **Class A, B, or C**. Class is set by the **first bits**.

| First bits | First byte | Class | netid / hostid bytes | Max hosts |
|---|---|---|---|---|
| `0` | 0 – 127 | **A** | 1 / 3 | 16 million (2²⁴) |
| `10` | 128 – 191 | **B** | 2 / 2 | 65,536 (2¹⁶) |
| `110` | 192 – 223 | **C** | 3 / 1 | 256 (2⁸) |

```
class   8 bits   16 bits   24 bits   32 bits
  A     NetID    hostID    hostID    hostID
  B     NetID    NetID     hostID    hostID
  C     NetID    NetID     NetID     hostID
```
*(Slide counts are raw 2ⁿ. Usable hosts are 2ⁿ − 2 because hostid all-0s = the network and all-1s = broadcast (§22), e.g., Class C → 254.)*

**Example:** `152.8.x.x` → first byte 152 is in 128–191 → **Class B** → netid = `152.8`. That's why NC A&T's addresses all start with 152.8.

### 19. Address mask
- A **32-bit value ANDed** with an IP address to **separate the netid from the hostid**.

| Class | Network mask |
|---|---|
| A | 255.0.0.0 |
| B | 255.255.0.0 |
| C | 255.255.255.0 |

**Worked example** (`152.8.251.41`, Class B):
```
    10011000.00001000.11111011.00101001    152.8.251.41
AND 11111111.11111111.00000000.00000000    255.255.0.0
    10011000.00001000.00000000.00000000    152.8.0.0   ← netid
```

### 20. Classless addressing (CIDR)
- Classes are **wasteful**: many domains never fill their address space.
- A network with **12 computers** only needs a **4-bit hostid** (2⁴ = 16).
- **CIDR (Classless Inter-Domain Routing):** you choose **how many bits are netid**.
- Written `ddd.ddd.ddd.ddd/m` where **m = number of netid bits**.

**CIDR example — `128.211.0.16/28`:**
| | Dotted | Last octet (binary) |
|---|---|---|
| Network prefix | 128.211.0.16 | 0001 **0000** |
| Address mask (/28) | 255.255.255.240 | 1111 **0000** |
| Lowest host | 128.211.0.17 | 0001 **0001** |
| Highest host | 128.211.0.30 | 0001 **1110** |
| *(Broadcast)* | *128.211.0.31* | *0001 **1111*** |

- `32 − 28 = 4` host bits → 16 addresses → **14 usable hosts** (.17–.30).

**CIDR recipe:**
1. host bits `h = 32 − m`
2. mask = `m` ones then `h` zeros (/28 → last octet `256 − 2⁴ = 240`)
3. network = IP AND mask
4. lowest host = network + 1; broadcast = network + 2ʰ − 1; highest host = broadcast − 1

### 21. IPv4 broadcast & special addresses
| Address | Meaning |
|---|---|
| **all 1s** (`255.255.255.255`) | Broadcast on the **local** network |
| **all 0s** | **This host** |
| hostid = all 0s | **The network** itself (not a host) |
| hostid = all 1s | **Broadcast** on that network |
| netid = all 0s | **This network** |
| netid = **127** | **Loopback** (never appears on the net) |

### 22. IPv6 address types
| Type | Delivered to |
|---|---|
| **Unicast** | A **single** host |
| **Anycast** | **One** member of a group, usually the **closest** |
| **Multicast** | **All members** of the group |

- **IPv6 does not support broadcasting.**

**IPv6 unicast parts (128 bits = 64 + 64):**
| Bits | 48 (or more) | 16 (or less) | 64 |
|---|---|---|---|
| Field | **Routing prefix** (network) | **Subnet ID** (subnet in network) | **Interface identifier** (host) |

**Link-local address:**
- **Every IPv6 host requires one**; prefix **`fe80::/10`**, usually followed by **54 zero bits** (10 + 54 = 64).
- Hosts can **assign their own** interface identifier, often a **hash of the MAC address**.
- Example: `fe80::cdac:fa7:98f4:b1cb%20`

**Interface index (`%n`):**
- Mostly seen on **link-local** addresses.
- Tells the OS **which network card/interface** to send on (`%20` = interface 20).

### 23. Assigning addresses
- **Domain names and netids** assigned by **ICANN** (Internet Corporation for Assigned Names and Numbers).
- **Local network admins** assign **hostids**.
- **All IP addresses must be unique.**
- All computers in the same domain have the **same netid**.
- The domain's **DNS must know the IP addresses** of all computers in the domain.

### 24. The Internet Protocol (IP)
- IP is the "IP" in **TCP/IP**.
- **Connectionless, best-effort, packet-switched.**
- **Does not recover from errors.** Packets may be **corrupted, delayed, lost, or out of order** (higher layers like TCP fix this).

### 25. IPv4 packet format
```
0       4       8              16                              31
┌───────┬───────┬──────────────┬───────────────────────────────┐
│Version│  IHL  │     TOS      │         Total length          │
├───────┴───────┴──────────────┼─────┬─────────────────────────┤
│        Identification        │Flags│     Fragment offset     │
├──────────────┬───────────────┼─────┴─────────────────────────┤
│     TTL      │   Protocol    │        Header checksum        │  20 bytes
├──────────────┴───────────────┴───────────────────────────────┤
│                        Source address                        │
├──────────────────────────────────────────────────────────────┤
│                     Destination address                      │
├──────────────────────────────────────────────────────────────┤
│                      Options (0–40 bytes)                    │
├──────────────────────────────────────────────────────────────┤
│                    Data (up to 65,515 bytes)                 │
└──────────────────────────────────────────────────────────────┘
```

| Field | Bits | Purpose |
|---|---|---|
| Version | 4 | Always **4** |
| Total Length | 16 | Total packet length → max **64 KB** (2¹⁶) |
| Identification | 16 | Identifies all **fragments in a group** |
| Flags | 3 | Includes **More Fragments** |
| Fragment Offset | 13 | Where this fragment goes in the original packet |
| **TTL** | 8 | **Decremented by each router**; packet dropped at **0** |
| Protocol | 8 | Which **transport-layer** protocol (TCP, UDP, …) |
| Header checksum | 16 | Detects **header** corruption only. **IP does not check the data.** |
| Source / Dest address | 32 each | IPv4 addresses |

*(65,535 total − 20-byte header = 65,515 bytes of data max.)*

### 26. Fragmentation (IPv4)
- Networks have an **MTU (Maximum Transmission Unit)**: **Ethernet = 1500 bytes** of data per frame.
- If a **router** gets a packet too big for the next network, **it splits it into fragments**.
- **More Fragments = 1** on every fragment **except the last**.
- **The destination reassembles** the fragments (not the routers).

### 27. IPv6 header
```
┌───────┬───────────────┬──────────────────────────────────┐
│Version│ Traffic Class │            Flow Label            │
├───────┴───────────────┴───────┬─────────────┬────────────┤
│        Payload Length         │ Next Header │ Hop Limit  │
├───────────────────────────────┴─────────────┴────────────┤
│                 Source Address (128 bits)                │
├──────────────────────────────────────────────────────────┤
│              Destination Address (128 bits)              │
└──────────────────────────────────────────────────────────┘
```

| Field | Bits | Purpose |
|---|---|---|
| Version | 4 | Always **6** |
| Payload Length | 16 | Size of the data; option for **Jumbograms up to 4 GB** |
| Next Header | 8 | Type of next header: a **TCP header** or an **IP extension header** |
| Hop Limit | 8 | Same job as IPv4 **TTL** |
| Source / Dest | 128 each | IPv6 addresses |

**Extension headers** (zero or more after the fixed header) may specify: **routing**, **fragmentation**, **Encapsulating Security Payload**.

**No router fragmentation in IPv6:**
- Too big for the MTU → router sends an **ICMP message back to the source**.
- The **source** must send packets that fit.
- **Simplifies routing.**

| | IPv4 | IPv6 |
|---|---|---|
| Address size | 32 bits | 128 bits |
| Hop counter | TTL | Hop Limit |
| Who fragments | **Routers** | **Source only** (router sends ICMP) |
| Broadcast | Yes | **No** (uses multicast) |
| Header checksum | Yes | No |
| Address → MAC | **ARP** | **Neighbor Discovery (ND)** |

### 28. ICMP (Internet Control Message Protocol)
- Supporting **network-layer** protocol for **error messages and operational info**.
- **Not** used to send user data.
- Tells a source about:
  - **Destination Unreachable**
  - **TTL time exceeded**
  - **Better gateway** (redirect)
  - **Echo** (used by **`ping`**)

### 29. Mapping between addresses
```
 Internet name ──DNS──► IP address ──ARP / ND──► MAC address
 (www.ncat.edu)         (152.8.240.16)            (00-e0-63-03-76-c0)
```

**Domain Name Servers (DNS):**
- Map **names → IP addresses** using a **distributed database**.
- Hosts send a request to the DNS to get a computer's IP.
- **Hosts and DNS servers cache** what they find.

**MAC addresses:**
- Network hardware only uses **MAC/physical addresses**; an Ethernet interface **knows nothing about IP**.
- To send a frame on the same network you need the destination's **MAC** (or broadcast).
- Each network type uses **different** addresses.

**In-class question:** *MAC addresses are…*
A. Larger for IPv6  **B. Built into the network interface hardware**  C. Assigned by a DHCP server  D. Used to route packets across different networks

**Answer: B.** MACs are burned into the NIC. They don't change with IP version (A), DHCP assigns **IP** addresses (C), and they're meaningless outside the LAN, so they can't route across networks (D).

### 30. ARP (Address Resolution Protocol) — IPv4
1. Source needs the **MAC** of a host on its **local** network.
2. Source **broadcasts an ARP request**: "who has IPv4 address X?"
3. The host with that address **replies with its MAC**.
4. All other hosts **ignore** it.

### 31. IPv6 Neighbor Discovery (ND)
- Same job as ARP, but uses **ICMP**.
- Sends a **Neighbor Solicitation** via **multicast** (no broadcast in IPv6).
- The owner replies; others ignore.
- **Secure Neighbor Discovery** uses **public-key cryptography** for security.

### 32. Caching
- After ARP/ND, a host **saves the IP ↔ MAC pair** (ARP cache).
- After a DNS lookup, it saves **name ↔ IP** in a **separate cache**.
- So ARP or DNS is only needed **once per destination**.

### 33. ARP poisoning (ARP spoofing)
- An attacker sends **fake ARP messages** on the LAN, associating **their MAC** with a **legitimate device's IP** (often the **router**).
- Other devices **update their ARP caches** with the false mapping.
- **Impact:** traffic meant for the real device goes to the attacker, who can:
  - **view** it and pass it along (man-in-the-middle),
  - **modify** it, or
  - **discard** it.
- Works because ARP has **no authentication** (compare Secure ND in §31).

### 34. IP encapsulation
- An IP **datagram** is **encapsulated in a frame** to cross a physical network. The **entire datagram sits in the frame's data area**.
- The **frame's destination address = the next hop**, not the final destination.

```
┌──────────────┬──────────────────────────────────┐
│ Frame header │  IP header  │      IP data        │
│ (dest = next │ (dest = final destination)        │
│  hop's MAC)  │                                   │
└──────────────┴──────────────────────────────────┘
```

**Data envelopes:**
- Each network type defines its **own envelope** (frame).
- The IP packet goes inside the envelope and is sent to a **router**.
- The router **takes the packet out**; if it must go further, puts it in the **next network's envelope** and sends it.
- At each hop the receiver **extracts the datagram and discards the frame header**.

> **Rule of thumb:** MAC addresses change **every hop**; IP source/destination stay the **same end-to-end**.

### 35. Local routing decision
- The **source** must decide: send **directly** to the destination, or to a **router/gateway**?
- Each host must know its **local DNS** and **default gateway** addresses.

| | Local destination | Global destination |
|---|---|---|
| Test | **netid(dest) = netid(source)** | **netid(dest) ≠ netid(source)** |
| Frame sent to | The **destination** directly | The **gateway** |
| ARP/ND for | Destination's MAC | Gateway's MAC (may be preconfigured) |
| IP dest address | Destination | **Still the final destination** |

### 36. IP routing procedure (A → B)
1. A sends a **DNS request** for B's IP.
2. DNS returns **B's IP**.
3. Extract B's netid: **B's IP AND A's subnet mask**.
4. Same netid → **send directly** to B.
5. Different → send to the **gateway**.
6. Gateway forwards to another gateway **closer to B's domain**.
7. The gateway at B's domain delivers the frame to **B**.

### 37. Routing example network
| Host | IP | HW | Gateway | DNS |
|---|---|---|---|---|
| **a.ncat.edu** | 152.8.244.55 | **5** | 152.8.254.254 | 152.8.244.1 |
| **DNS.ncat.edu** | 152.8.244.1 | **3** | 152.8.254.254 | — |
| **b.ncat.edu** | 152.8.247.77 | **7** | 152.8.254.254 | 152.8.244.1 |
| **gate.ncat.edu** (router, ncat side) | 152.8.254.254 | **4** | | |
| **router.acme.com** (same router, acme side) | 176.5.4.3 | **2** | | |
| **www.acme.com** | 176.5.6.9 | **9** | 176.5.4.3 | 234.6.7.14 |
| **www.b.com** | 176.5.6.17 | **17** | 176.5.4.3 | 234.6.7.14 |

*The gateway is one box with **two interfaces**, so it has two IPs and two HW addresses (§16: a computer on two networks needs two addresses).*

#### Example 1 — a.ncat.edu → b.ncat.edu (cold start, nothing cached)
| # | Step | Src HW | Dst HW | Src IP | Dst IP |
|---|---|---|---|---|---|
| 1 | A: ARP request for **DNS**'s MAC | 5 | **broadcast** | 152.8.244.55 | 152.8.244.1 |
| 2 | DNS: ARP reply to A | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| 3 | A: DNS query for b.ncat.edu | 5 | 3 | 152.8.244.55 | 152.8.244.1 |
| 4 | DNS: reply with B's IP | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| — | **Decision:** netid 152.8 = 152.8 → **local** | | | | |
| 5 | A: ARP request for **B**'s MAC | 5 | **broadcast** | 152.8.244.55 | 152.8.247.77 |
| 6 | B: ARP reply to A | 7 | 5 | 152.8.247.77 | 152.8.244.55 |
| 7 | **A sends datagram to B** | 5 | 7 | 152.8.244.55 | 152.8.247.77 |

**Follow-on (everything cached):** A sends another packet to B → **only step 7**: `5 → 7, 152.8.244.55 → 152.8.247.77`. No ARP, no DNS.

**Levels of communication:**
- **High level:** host talks to the **DNS**, then to the **destination or gateway**.
- **Low level:** before *any* message, check the cache for the target's **MAC**; if missing, use **ARP/ND** first.

#### Example 2 — a.ncat.edu → www.acme.com (cold start)
| # | Step | Src HW | Dst HW | Src IP | Dst IP |
|---|---|---|---|---|---|
| 1 | A: ARP request for **DNS**'s MAC | 5 | **broadcast** | 152.8.244.55 | 152.8.244.1 |
| 2 | DNS: ARP reply | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| 3 | A: DNS query for www.acme.com | 5 | 3 | 152.8.244.55 | 152.8.244.1 |
| 4 | DNS: reply with 176.5.6.9 | 3 | 5 | 152.8.244.1 | 152.8.244.55 |
| — | **Decision:** netid 176.5 ≠ 152.8 → **global**, send to gateway | | | | |
| 5 | A: ARP request for **gateway**'s MAC | 5 | **broadcast** | 152.8.244.55 | 152.8.254.254 |
| 6 | Gateway: ARP reply | 4 | 5 | 152.8.254.254 | 152.8.244.55 |
| 7 | **A sends data to gateway** | 5 | **4** | 152.8.244.55 | **176.5.6.9** |
| 8 | Gateway: ARP request for **WWW**'s MAC | 2 | **broadcast** | 176.5.4.3 | 176.5.6.9 |
| 9 | WWW: ARP reply | 9 | 2 | 176.5.6.9 | 176.5.4.3 |
| 10 | **Gateway sends data to www.acme.com** | 2 | 9 | **152.8.244.55** | 176.5.6.9 |

**Things to notice (these are the homework/exam traps):**
- **Step 7:** Dst HW = **gateway (4)** but Dst IP = **final destination (176.5.6.9)**, *not* the gateway's IP.
- **Step 10:** Src IP is still **A (152.8.244.55)**: the router changes the **MAC** addresses, **not** the IP addresses.
- ARP requests/replies use the **interface's own IP** (gateway uses 176.5.4.3 on the acme side, 152.8.254.254 on the ncat side).
- Every ARP **request** has Dst HW = **broadcast**; every **reply** is unicast.

**Recipe for "fill in the addresses" problems:**
1. Need a MAC you don't have cached? → ARP: `src HW = me, dst HW = broadcast, src IP = me, dst IP = the IP I'm resolving`.
2. Need a name you don't have? → DNS query (ARP the DNS first if needed).
3. Compare netids → local (ARP dest) or global (ARP gateway).
4. Data frame: **HW = this hop**, **IP = end-to-end**.

### 38. Assignments & schedule (from slides)
- **Queuing questions** due **5:00 pm Wed Sept 30, 2026** (Canvas).
- **IP packet address questions** due **5:00 pm "Monday, October 4, 2026"** (Canvas). *(Slide date conflict: Oct 4, 2026 is a Sunday; Monday is Oct 5. Check Canvas.)*
- **Exam 2: Wednesday, October 7, 2026**; returned Wed Oct 14.
- **Fall Break:** Mon Oct 12 (no class).
- **Last day to drop:** Oct 19.

| Date | Topic | Reading |
|---|---|---|
| Mon Sept 28 | Network layer | 5.1 – 5.2 |
| Wed Sept 30 | Routing | 5.5 |
| Mon Oct 5 | Routers & review | — |
| Wed Oct 7 | **Exam 2** | — |
| Mon Oct 12 | Fall Break | — |
| Wed Oct 14 | Transport layer | 7.1 |
| Mon Oct 19 | Sockets | 5.2 – 5.4 |
| Wed Oct 21 | Online APIs | 6.5 |

### 39. Quick formula & fact sheet
| Concept | Formula / value |
|---|---|
| Network layer job | **Routing** across multiple networks |
| Visible to hosts | **Routers only** (not repeaters/bridges) |
| Dijkstra | single-source shortest path by **edge weight**; fewest hops ≠ shortest |
| Floyd–Warshall | `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`, **O(n³)** |
| Distributed routing | `min(current, t(neighbor) + neighbor's t(dest))`, share tables periodically |
| IPv4 / IPv6 size | **32 / 128 bits** |
| Class by first byte | A 0–127 · B 128–191 · C 192–223 |
| Class masks | A /8 · B /16 · C /24 |
| netid | **IP AND mask** |
| CIDR /m | host bits = 32 − m; addresses = 2^(32−m); usable = 2^(32−m) − 2 |
| /28 mask | 255.255.255.240 |
| Broadcast | all 1s (local) or hostid all 1s (that network) |
| Loopback | netid 127 |
| IPv6 link-local | `fe80::/10` |
| `::` | Use **once** to replace consecutive zero groups |
| IPv4 max packet | 64 KB (16-bit Total Length) |
| Ethernet MTU | 1500 bytes |
| More Fragments | 1 on all fragments except the last |
| IPv4 fragments at | **Routers**; reassembled at **destination** |
| IPv6 fragments at | **Source only** (router sends ICMP) |
| TTL / Hop Limit | decremented per router; drop at 0 |
| IP checksum | **header only** |
| Name → IP | **DNS** |
| IP → MAC | **ARP** (IPv4) / **ND** (IPv6, ICMP + multicast) |
| Frame dest | **Next hop**'s MAC; IP dest = **final** destination |

**In-class answers:** MAC addresses are… → **B (built into the NIC hardware)**
