# COMP476 — Networked Computer Systems
## Lecture 14 — IP Subnets & DNS (slides: "Internet Protocol Subnets," Mon Oct 5, 2026)

**What's new vs. Lecture 13:** **subnetting** (§4–§8), the **subnet mask design formula** (§8), the **DNS hierarchy and search walk-through** (§11–§15), and **DNS resource records** (§16–§17). §1–§3 are a short recap.

---

### 1. Internet addresses (recap)
- An IP address = **NetID** (which network) + **HostID** (which host on that network).
- A computer **physically connected to two networks needs two IP addresses** (one per interface; routers are the usual example).

### 2. CIDR (recap)
- `ddd.ddd.ddd.ddd/m` where **m = number of NetID bits**.
- The subnet mask has **m leading one-bits**.
- **Subnet mask and CIDR notation do exactly the same job**: `/16` ≡ `255.255.0.0`.

### 3. Routing decision (recap)
Given a destination IP, a host:
1. **ANDs its mask with the destination IP**
2. **ANDs its mask with its own IP**
3. **Same result** → local → **ARP** for the destination's HW address
4. **Different** → send the packet to the **router**

**Slide example — 152.8.251.41/16 → 15.28.25.141**
```
    10011000.00001000.11111011.00101001   152.8.251.41
AND 11111111.11111111.00000000.00000000   255.255.0.0
    10011000.00001000.00000000.00000000   152.8.0.0

    00001111.00011100.00011001.10001101   15.28.25.141
AND 11111111.11111111.00000000.00000000   255.255.0.0
    00001111.00011100.00000000.00000000   15.28.0.0
```
**152.8.0.0 ≠ 15.28.0.0 → different networks → send to router.**

---

### 4. Hierarchical routing ★
- IP routes datagrams to the **destination domain/network**, not to the host.
- Once inside the destination network, the packet is assumed to reach the host over **bridges and repeaters** (layer 2).

### 5. IP subnets ★
- **Subnetting = dividing a large domain into smaller subdomains.**
- Packets route between subnets with **the same rules used for domains** (AND test → local or router).
- **Some upper HostID bits are used locally as an extension of the NetID.**
- **The outside world still routes by the normal NetID**; only **inside** is the NetID extended.
- Typical use: **each campus building on its own subnet.**

### 6. Extending the NetID
- A domain is **physically separated** into subdomains, **connected by routers**.
- For **N subdomains**, use the upper **log₂N bits of the HostID** with the NetID.
- **Every address in one physical subdomain must share the same log₂N upper bits.**

### 7. Example: 4 subnets of 152.8.0.0/16 ★
log₂4 = **2 extra bits** → /16 + 2 = **/18**
```
255.255.192.0 = 11111111.11111111.11000000.00000000
                                  ^^ subnet bits
```
| Subnet | Range | 3rd byte (binary) | Usable hosts |
|---|---|---|---|
| 1 | 152.8.**0**.0 – 152.8.**63**.255 | `00`000000 – `00`111111 | 152.8.0.1 – 152.8.63.254 |
| 2 | 152.8.**64**.0 – 152.8.**127**.255 | `01`000000 – `01`111111 | 152.8.64.1 – 152.8.127.254 |
| 3 | 152.8.**128**.0 – 152.8.**191**.255 | `10`000000 – `10`111111 | 152.8.128.1 – 152.8.191.254 |
| 4 | 152.8.**192**.0 – 152.8.**255**.255 | `11`000000 – `11`111111 | 152.8.192.1 – 152.8.255.254 |

- Block size = 256 − 192 = **64**. Each subnet: 2¹⁴ = 16,384 addresses, **16,382 usable** (first = network, last = broadcast — that's why the diagram ranges run .1 to .254).
- Slide diagram: the four subnet "clouds" are joined by **routers**; the outside world still sees one network, **152.8.0.0/16**.

**Subdomain membership test (mask 255.255.192.0)**

*Same subnet:* 152.8.251.41 vs 152.8.244.21
```
    10011000.00001000.11111011.00101001   152.8.251.41
AND 11111111.11111111.11000000.00000000
    10011000.00001000.11000000.00000000   152.8.192.0

    10011000.00001000.11110100.00010101   152.8.244.21
AND 11111111.11111111.11000000.00000000
    10011000.00001000.11000000.00000000   152.8.192.0
```
**Identical → same subdomain** (both in subnet 4).

*Different subnet:* 152.8.251.41 vs 152.8.47.14
```
    10011000.00001000.00101111.00001110   152.8.47.14
AND 11111111.11111111.11000000.00000000
    10011000.00001000.00000000.00000000   152.8.0.0
```
**152.8.192.0 ≠ 152.8.0.0 → different subdomains** → goes through a router even though both are "152.8".

*Shortcut:* only the partial octet matters. 251 AND 192 = 192; 244 AND 192 = 192; 47 AND 192 = 0.

### 8. Creating subnet masks ★ (formula)
- Network is `/M`, you want **N** subnets:
  - Extra bits = **⌈log₂N⌉** (always round **up**)
  - New mask = **M + ⌈log₂N⌉** leading ones

**Slide example — 125.47.13.8/18 into 12 subnets**
- log₂12 = 3.585 → **4 bits**
- 18 + 4 = **/22**
```
11111111.11111111.11111100.00000000 = 255.255.252.0
                    ^^^^ the 4 new subnet bits (bits 19–22)
```
**Extra detail (not on slide, useful for exam):**
| Property | Value |
|---|---|
| Original network | 125.47.**0**.0/18 (13 AND 192 = 0) → 125.47.0.0 – 125.47.63.255 |
| Subnets available | 2⁴ = **16** (12 used, 4 spare) |
| Block size (3rd octet) | 256 − 252 = **4** → subnets at .0, .4, .8, … .60 |
| Addresses per subnet | 2¹⁰ = 1024 → **1022 usable** |
| 125.47.13.8 is in | 13 AND 252 = 12 → **125.47.12.0/22** (125.47.12.0 – 125.47.15.255) |

> Trade-off: every extra subnet bit **doubles** the subnets and **halves** the hosts per subnet.

---

### 9. Mapping between addresses (recap)
```
Internet name ──DNS──► IP address ──ARP──► MAC address
```
- Humans use **names**; hardware uses **MAC addresses**.

### 10. Domain Name Servers
- Map **Internet names → IP addresses**.
- Maintain a **distributed database** of names and addresses.
- Hosts send a request to a DNS to get a computer's IP.
- **Hosts and DNS cache** addresses they've found.
- **DNS does NOT provide physical (MAC) addresses.** That's ARP's job.

### 11. `getbyname` / `gethostbyname`
```java
// Java
InetAddress hostent = java.net.InetAddress.getByName("www.acme.com");
```
```python
# Python
import socket
ip_address = socket.gethostbyname("www.acme.com")
```
- Programs convert an IP name to an IP address with this call.

> ⚠️ **Slide quirks:** Java's real method is **`getByName`** (capital B), and it returns an **`InetAddress` object**. Python's `gethostbyname` returns a **string** like `"192.168.210.5"`, not an InetAddress.

### 12. DNS hierarchy ★
- **DNS is hierarchical** (tree, e.g. `com → foobar → candy → peanut/almond/walnut`, `foobar → soap`).
- Each server must know **all servers directly below it**.
- Each server must also know a **root server** to ask when it doesn't know a name.
- Each server links to name servers **up and down** the hierarchy.
- **Autonomy:** each organization assigns/changes its own names **without telling a central authority**, by running DNS for **its part of the tree** (A&T runs the server for `*.ncat.edu`).

### 13. Zones & replication
| Concept | Meaning |
|---|---|
| **Zone** | A domain can be split into **multiple zones**; each zone has a DNS **responsible for all names in that zone**; a zone **may have multiple DNS** |
| **Replication** | Multiple **physical copies** of one DNS server. Used for heavily loaded servers like **root servers** (top-level domain info). Admins must keep all copies **coordinated** to give **identical** answers. A&T has several DNS servers for **redundancy** |

### 14. DNS request types ★
| Type | Response | Used by |
|---|---|---|
| **Recursive** | The **answer**, or an **error** if unknown. Server does the legwork | **Client → local DNS** (what `gethostbyname` sends) |
| **Iterative** | The answer **or the name of another DNS** that might know | **DNS ↔ DNS** |

- **Search path:** a DNS replies directly if it has the info; otherwise it **follows the hierarchy tree** until found.
- The **primary DNS** for a domain knows the names and addresses of **all computers in its domain**.

### 15. DNS search example ★ (me.ncat.edu → www.acme.com, cold cache)
| # | From → To | Message | Type |
|---|---|---|---|
| 1 | me.ncat.edu → DNS.ncat.edu | App calls `gethostbyname("www.acme.com")` | **Recursive** |
| 2 | DNS.ncat.edu → ROOT | "Address of www.acme.com?" | Iterative |
| 3 | ROOT → DNS.ncat.edu | "Ask the **.com DNS**" | referral |
| 4 | DNS.ncat.edu → COM DNS | "Address of www.acme.com?" | Iterative |
| 5 | COM DNS → DNS.ncat.edu | "Ask the **acme.com DNS**" | referral |
| 6 | DNS.ncat.edu → DNS.ACME.COM | "Address of www.acme.com?" | Iterative |
| 7 | DNS.ACME.COM → DNS.ncat.edu | **IP of www.acme.com** (it's the **primary DNS** for acme.com) | answer |
| 8 | DNS.ncat.edu → me.ncat.edu | IP returned to the calling app | answer |

**8 messages.** The client sends **one** request; the **local DNS does all the walking**.

### 16. Caching & the second search ★
- A DNS **saves what it finds**: **name↔address pairs** *and* **IP addresses of other DNS servers**.

**Second example — me.ncat.edu → ftp.acme.com (right after the first search)**
| # | From → To | Message |
|---|---|---|
| 1 | me.ncat.edu → DNS.ncat.edu | `gethostbyname("ftp.acme.com")` |
| 2 | DNS.ncat.edu → DNS.ACME.COM | Request sent **directly, using the cached address** |
| 3 | DNS.ACME.COM → DNS.ncat.edu | IP of ftp.acme.com |
| 4 | DNS.ncat.edu → me.ncat.edu | IP returned to app |

**4 messages — root and .com are skipped** because acme.com's DNS address is cached.
(Ask for www.acme.com again → **2 messages**: the local DNS answers straight from cache.)

### 17. DNS server entries & resource records ★
- DNS info comes from a **database maintained by the domain administrator**.
- Clients query the DNS over **UDP** *(port 53 — not on slides)*.

| Record | Name | Purpose |
|---|---|---|
| **SOA** | Start of Authority | Names the **primary DNS** and **time limits** |
| **A** | Address | Host name → **IPv4 address** |
| **CNAME** | Canonical Name | **Alias** host names |
| **MX** | Mail Exchanger | Domain's **mail servers** |
| **NS** | Name Server | Domain's **name servers** |

*(Not on slides: IPv6 addresses use an **AAAA** record.)*

**RR example (acme.com zone)**
```
acme.com.   IN SOA  dns.acme.com. dnsowner.acme.com. (
                    20010313   ; serial # (date format)
                       10800   ; refresh (3 hours)
                        3600   ; retry (1 hour)
                      604800   ; expire (1 week)
                       86400)  ; TTL (1 day)
acme.com.     IN NS     dns.acme.com.
              IN NS     ns1.isp.net.
acme.com.     IN MX 20  mail.acme.com.
              IN MX 40  mail.isp.com.
dns.acme.com. IN A      192.168.210.2
mail          IN A      192.168.210.4
www.acme.com. IN A      192.168.210.5
ftp.acme.com. IN CNAME  www.acme.com.
pc            IN A      192.168.210.6
```
**Reading it:**
- **SOA:** primary DNS = `dns.acme.com`; admin email = `dnsowner@acme.com` (first `.` → `@`). Serial `20010313` = Mar 13, 2001 (bump it on every edit so secondaries know to refresh). Time values are **seconds**: 10800 = 3 h, 3600 = 1 h, 604800 = 7 days, 86400 = 1 day.
- **NS:** two name servers, one in-house and one at the ISP (redundancy, §13).
- **MX:** **lower number = higher priority**. Mail goes to `mail.acme.com` (20) first, `mail.isp.com` (40) as backup.
- **Trailing dot** = fully qualified name. **No dot** (`mail`, `pc`) = relative, so the domain is appended → `mail.acme.com`, `pc.acme.com`.
- **CNAME:** `ftp.acme.com` is an alias for `www.acme.com` → resolves to **192.168.210.5**. Same machine. Ties to the §16 example.
- Blank owner field = **same name as the line above**.
- 192.168.x.x is a **private** range; fine for a teaching example.

---

### 18. Course logistics
| Date | Topic | Reading |
|---|---|---|
| Mon Oct 5 | Subnets & review | — |
| **Wed Oct 7** | **Exam 2** | — |
| Mon Oct 12 | **Fall Break** (no classes) | — |
| Wed Oct 14 | Transport layer | 7.1 |
| Mon Oct 19 | Sockets | 5.2 – 5.4 |
| Wed Oct 21 | Online APIs | 6.5 |

- **Spring/Summer advisement** began Mon Oct 5, 2026. **Registration: Nov 2 – 23.** Make an appointment with your advisor and come prepared.

### 19. Quick formula & fact sheet
| Concept | Formula / value |
|---|---|
| Local or router? | (dest AND mask) == (own IP AND mask) → ARP; else → router |
| Subnet bits for N subnets | ⌈log₂N⌉ |
| New mask | /(M + ⌈log₂N⌉) |
| Block size (partial octet) | 256 − mask octet |
| Usable hosts | 2^(32 − mask) − 2 |
| /16 → 4 subnets | /18 = 255.255.192.0, blocks of 64, 16,382 hosts each |
| /18 → 12 subnets | /22 = 255.255.252.0, 16 subnets, 1022 hosts each |
| Outside view of a subnetted net | Original NetID only |
| DNS gives | Name → IP (**never MAC**) |
| Recursive vs. iterative | Client→local DNS: answer/error · DNS↔DNS: answer or referral |
| Cold lookup (example) | 8 messages: client, local, root, .com, acme, back |
| Cached DNS address | Skip root & TLD → 4 messages |
| Records | SOA primary+timers · A name→IP · CNAME alias · MX mail (low # first) · NS name servers |
| DNS transport | UDP (port 53) |
