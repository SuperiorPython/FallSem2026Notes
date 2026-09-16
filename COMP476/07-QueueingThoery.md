# COMP476 — Networked Computer Systems
## Lecture 7 — Queuing Theory (slides: "Queuing Theory," Wed Sept 9, 2026)

### 0. Admin notes from this lecture
- **HW due** this lecture, **new HW assigned** this lecture (Canvas assignment, due **5:00pm Monday Sept 14**) — asks several numerical questions on topics from this semester.
- **Friday Sept 11, 2026** is the last day to apply for December 2026 graduation.
- **Monday Sept 14**: Telephones & review (read 2.5), HW due.
- **Exam 1: Wednesday Sept 16, 2026.**

### 1. What is queuing theory?
- Queuing theory = the **mathematics of waiting lines**. Useful for predicting/evaluating system performance.
- Originated in **operations research**. Classic framing: customers visiting a store ↔ requests arriving at a device.
- **Long-term averages only**: queuing theory gives long-run average values — it does **not** predict when the next specific event happens. Input data should be measured over an extended period, and arrival/service times are assumed random.

### 2. Assumptions (not always true, but still give good results)
- Independent arrivals
- Exponential distributions
- Customers do not leave or change queues (no reneging/jockeying)
- Large queues do not discourage new customers (no balking)

### 3. The Queuing Model
A queue (buffer) feeds a server:

```
λ → [ queue ] → (S) →
```

- **λ (lambda)** = arrival rate into the queue
- **Q** = spans queue + server = number in the *system* (waiting + being served)
- **W** = spans just the queue = number *waiting*
- **Tq** = time spent in the *system* (waiting + service)
- **Tw** = time spent *waiting* in the queue only

### 4. Interesting values (definitions)
| Symbol | Meaning |
|---|---|
| Arrival rate (λ) | average **rate** customers arrive |
| Service time (s) | average **time** to service one customer |
| Number waiting (W) | average **number** of customers waiting (queue only) |
| Number in system (Q) | average total **number** of customers in system (waiting + being served) |
| Time in system (Tq) | average **time** each customer is in the system, waiting + service |
| Time waiting (Tw) | average **time** each customer waits in queue only |

**Relationship:** `Tq = Tw + s`

### 5. Arrival rate & inter-arrival time
- Arrival rate λ: measured in arrivals/time period (e.g. packets/second).
- Inter-arrival time **a**: average time *between* arrivals, measured in time/customer (e.g. seconds/packet).
  - `a = 1 / λ`

### 6. Random values & the exponential distribution
- Real-world event times (request arrival, service time, user request time) are assumed random, but we may know their **average value and distribution**.
- Many of these random values are **exponentially distributed**: `Frequency of Occurrence = e^(-t)` — many small values, a few large ones. Inter-arrival time of customers is naturally exponentially distributed.

### 7. Poisson arrival rate
If customers arrive at exponentially distributed rate λ, the probability of **exactly k** customers arriving during time *t* is:

```
Pk(t) = (λt)^k / k! * e^(-λt)
```

Math notes used with this formula: `0! = 1! = 1`, `x^0 = 1`, `x^1 = x`.

**Worked example — printer maintenance:**
A networked printer usually gets 15 print jobs/hour. It's turned off for 10 min maintenance. Probability nobody wants to use it during that time?
- Arrival rate: 15/60 = **0.25 jobs/min**
- `P0(10) = (0.25*10)^0 / 0! * e^(-0.25*10) = 0.082`

**Worked example — 2+ requests in a second:**
On average, 4 requests/sec arrive at a network. Probability that **2 or more** arrive in one second?
- Sum of all probabilities = 1.0 (100%), so `P(≥2) = 1.0 − (P0 + P1)`
- `P(k>1)(1) = 1.0 − (0.0183 + 0.082) = 90.8%`

### 8. Expected number of arrivals
If customers arrive at exponentially distributed rate λ, expected arrivals in time *t*:

```
Expected = λ * t
```
Example: λ = 4 requests/sec → in 10 seconds, expect **40** network requests.

### 9. Queuing models (Kendall notation)
Queuing systems are described as three values separated by slashes:

```
Arrival distribution / Service distribution / # of servers
```
where:
- **M** = Markovian (exponentially distributed)
- **D** = Deterministic (constant)
- **G** = General (binomial distribution)

**Common models:**
- **M/M/1** — simplest model; both arrival and service time exponentially distributed
- **M/D/1** — arrivals exponentially distributed, but **fixed/constant** service time
- **M/M/n** — multiple (n) servers

### 10. Why queuing happens
Arrivals come at random times — sometimes far apart, sometimes clustered. When more customers arrive in a short window than can be serviced, queues form. **If arrivals weren't random, queues wouldn't form.**

### 11. Utilization (ρ)
- Utilization ρ (Greek "rho") = fraction of time the server is **busy**.
- Always: `0 ≤ ρ ≤ 1`
- Example: bank teller busy 6 of 8 hours → ρ = 6/8 = 0.75

**Calculating ρ:**
```
ρ = λ * s
```
Units of λ and s must match (convert to common units if needed).

### 12. M/M/1 Formulas
| Formula | Meaning |
|---|---|
| `Tq = s / (1 − ρ)` | avg time in system |
| `Tw = sρ / (1 − ρ)` | avg time waiting |
| `Q = ρ / (1 − ρ)` | avg number in system |
| `W = ρ² / (1 − ρ)` | avg number waiting |

**Tq grows explosively as ρ → 1** (graph shown: near-flat until ~ρ=0.7, then shoots up toward infinity as ρ approaches 1). Key takeaway: pushing utilization too close to 100% causes wait times to blow up.

### 13. Little's Formula
Number in the system = arrival rate × avg time in system:
```
Q = λ * Tq
```
Also true for just the queue:
```
W = λ * Tw
```
**Derivation check:** multiplying `Tq = s/(1−ρ)` by λ gives `λTq = λs/(1−ρ) = ρ/(1−ρ) = Q` ✓ (consistent with the M/M/1 formula for Q).

### 14. Solution process (8 steps, use for every queuing problem)
1. Determine what quantities you need to know
2. Identify the server
3. Identify the queued items
4. Identify the queuing model
5. Determine the service time
6. Determine the arrival rate
7. Calculate ρ
8. Calculate the desired values

**Worked example — network interface (M/M/1):**
Network interface completes an avg request in 10 ms (exponentially distributed). Over 30 min, 117,000 requests were made. Find avg completion time and avg # of queued requests.
- Server = network interface; queued items = network requests; model = **M/M/1**
- `s = 10 ms = 0.01 sec/request`
- `λ = 117,000 / (30 min * 60 sec/min) = 65 requests/sec`
- `ρ = λs = 0.01 * 65 = 0.65`
- `Tq = s/(1−ρ) = 0.01/(1−0.65) = 28.6 ms`
- `W = ρ²/(1−ρ) = 0.65²/(1−0.65) = 1.21`

### 15. Queue size probabilities
Probability of **exactly** N jobs in the system:
```
Prob[Q = N] = (1 − ρ) * ρ^N
```
Probability of **N or fewer**:
```
Prob[Q ≤ N] = Σ(i=0 to N) (1 − ρ) * ρ^i
```
Probability of **more than N** (complement — total probability is always 1.0):
```
Prob[Q > N] = 1 − Σ(i=0 to N) (1 − ρ) * ρ^i
```

**Worked example continued (from §14, but ρ = 0.375 here):** Probability a request does **not** get queued = probability of zero jobs already in system:
```
P[Q=0] = (1 − 0.375) * 0.375^0 = 0.625
```

### 16. Accuracy & significant digits
- A 10-digit calculator display does **not** mean your answer is accurate to 10 digits.
- Answer accuracy is capped by input data accuracy — 3 significant digits in → no more than 3 digits out.
- Keep full precision through intermediate steps; **round off only at the very end**.

### 17. M/D/1 — constant (deterministic) service time
- Used when service time is always the same constant (e.g., a network that always sends fixed-size packets).
- Less randomness in the system → **wait time will be less** than the equivalent M/M/1 case.

**M/D/1 Formulas:**
| Formula |
|---|
| `Tq = s(2 − ρ) / [2(1 − ρ)]` |
| `Q = ρ²/[2(1−ρ)] + ρ` |
| `Tw = sρ / [2(1 − ρ)]` |
| `W = ρ² / [2(1 − ρ)]` |

**Worked example — ATM network:**
53-byte packets over a 155 Mb/sec line, always 2.74 μs/packet (fixed). 145,000 packets sent/sec. How long does a packet wait?
- `s = 53 bytes * 8 bits/byte / 155×10^6 bits/sec = 2.74×10^-6 sec` (constant)
- `λ = 145,000 packets/sec`
- `ρ = λs = 145,000 * 2.74×10^-6 = 0.3973`
- `Tw = sρ / [2(1−ρ)] = (2.74×10^-6 * 0.3973) / [2(1−0.3973)] = 9.03×10^-7 sec`

### 18. Multiple servers (M/M/N)
- Customers arrive and join a **single** queue.
- Whenever any server is idle, it serves the first customer in that single queue.
- All servers must be **identical** — any customer can be served by any server.
- With N servers, the model is **M/M/N**.

**Multiple-server utilization** (average across all N servers):
```
ρ = λs / N
```

**Intermediate value K** (simplifies the math, has no intrinsic meaning on its own — always < 1 since the denominator sum is always the numerator sum plus one extra term):
```
K = [ Σ(i=0 to N−1) (λs)^i / i! ] / [ Σ(i=0 to N) (λs)^i / i! ]
```

**Probability all servers are busy (C)** — this is the probability a new customer must wait in queue at all:
```
C = (1 − K) / (1 − λsK/N)
```

**M/M/N formulas:**
| Formula |
|---|
| `Tq = Cs/[N(1−ρ)] + s` |
| `Tw = Cs/[N(1−ρ)]` |
| `Q = Cρ/(1−ρ) + λs` |
| `W = Cρ/(1−ρ)` |

(note: ρ = λs/N here)

### 19. Worked example series — slow printer problem
**Setup:** Printer prints an avg file in 2 min. A new file arrives every 2.5 min. How long until a user gets their output?

**(a) Baseline — one printer, M/M/1:**
- `s = 2 min`, `λ = 1/2.5 = 0.4`, `ρ = λs = 0.4*2 = 0.8`
- `Tq = s/(1−ρ) = 2/(1−0.8) = 10 min`

**(b) Add a second identical printer, single shared queue → M/M/2:**
- All values the same, but `ρ = λs/2 = 0.4` now
- Calculate K: `K = (1 + λs) / (1 + λs + (λs)²/2) = (1 + 0.8) / (1 + 0.8 + 0.64/2) = 0.849057`
- `C = (1 − K)/(1 − λsK/N) = 0.22857`
- `Tq = Cs/[N(1−ρ)] + s = 2.57 min`
- **Twice the printers → runs about 4× as fast** (10 min → 2.57 min)

**(c) Replace with one faster printer (1 min/file) instead of adding a second — M/M/1:**
- `λ = 0.4`, `s = 1.0`, `ρ = 0.4`
- `Tq = s/(1−ρ) = 1/(1−0.4) = 1.67 min`
- **A single fast printer beats two slow printers here — 6× better than the original slow printer.**

**(d) Two separate printers, each with its own queue (grocery-store model, split arrival stream):**
- Same as original M/M/1 but input rate is halved: `λ = 0.2`, `s = 2.0`, `ρ = 0.4`
- `Tq = s/(1−ρ) = 2/(1−0.4) = 3.33 min`

**Summary comparison table:**
| Scenario | Model | λ | s | Tq |
|---|---|---|---|---|
| Original single printer | M/M/1 | 0.4 | 2 | 10.0 |
| Two printers, one shared queue | M/M/2 | 0.4 | 2 | 2.57 |
| Two printers, two separate queues | M/M/1 | 0.2 | 2 | 3.33 |
| One faster printer | M/M/1 | 0.4 | 1 | 1.67 |

**Takeaway:** for this scenario, a single faster server beats multiple slower servers, and one shared queue beats splitting into separate queues (shared queue lets any idle server grab the next job instead of a job being stuck behind a slow queue).

### 20. Single vs. multiple queues
- **Bank teller model**: one shared queue feeding multiple servers (arrival rate λ into one line, whichever teller frees up next takes the front customer).
- **Grocery store model**: each server has its own separate queue, and arrivals split evenly between them (λ/2 into each queue if there are 2 checkout lines).

### 21. Merging & dividing arrival streams
- **Merging**: exponentially distributed arrival streams can be combined — total arrival rate = sum of individual rates: `λa + λb`
- **Dividing**: an exponential arrival stream can be split into sub-streams; the split rates must sum back to the original rate (e.g., splitting evenly: `λ/2` and `λ/2`).

### 22. Linking multiple queues (queues in series)
- The **exit rate** of one queuing system equals its arrival rate (what goes in must come out, in steady state).
- Output from one queuing system can feed directly into another as its arrival stream.
- **Total time through a linked system = sum of the time through each individual queuing component.**

**Worked example — computer → router → server:**
Computer sends a packet in 12 ms; router forwards it to the server in 7 ms. Computer generates 40 pkt/sec; router receives a total of 100 pkt/sec (from multiple sources). How long for a packet to reach the server?
- Two separate M/M/1 queues in series: computer-transmitter, then router
- `s_computer = 12 ms`, `λ_computer = 40 pkt/sec` → `ρ_computer = 40*0.012 = 0.48`
- `s_router = 7 ms`, `λ_router = 100 pkt/sec` → `ρ_router = 100*0.007 = 0.7`
- `Tq_computer = 0.012/(1−0.48) = 0.0231 sec`
- `Tq_router = 0.007/(1−0.7) = 0.0233 sec`
- **Total = 46.4 ms** (sum of both stages)

### 23. Reusing a server
- Example: a file server where requests travel over the **network** to reach the server, then use the **disk**, then travel over the network **again** to return.
- The network is used twice per request → **the load on the network is effectively doubled** (2λ through the network stage) even though the original external arrival/exit rate is still λ.
