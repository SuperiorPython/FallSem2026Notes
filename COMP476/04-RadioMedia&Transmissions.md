# COMP 476: Radio Media and Transmission Limitations

## Goals
- Review radio media
- Understand the RS-232 transmission format
- Be able to compute the parity of a byte
- Understand the difference between baud and bits per second
- Be able to calculate the maximum possible transmission rate (Nyquist)
- Be able to calculate the maximum transmission rate in the presence of noise (Shannon)

---

## Part 1: Radio Media

### Radio Wave Basics
- **Long wave (low frequency):** bends around objects — better at penetrating obstacles and reaching around corners/terrain
- **Short wave (high frequency):** more line-of-sight — blocked more easily by obstacles

### Radio Penetration
- Penetration depth **decreases** as frequency **increases**
- Long waves can pass through buildings
- A metal screen (e.g., a microwave oven door mesh) shields against radio waves if the holes in the screen are smaller than about **1/20th of the wavelength**
- Radio waves generally do **not** travel well through water

### Acoustic Communication
- Radio waves and lasers don't work well underwater, so some underwater systems use **acoustics** instead
- Used by Autonomous Underwater Vehicles (AUVs)
- Challenges: significant interference from natural and manmade noise; potential environmental concerns

### Microwave Communications
- Microwave signals are **line of sight**
- Antennas usually located on tall towers
- Often used to interconnect phone towers

### Spectrum Allocation
- Governments and international bodies like the **ITU** (International Telecommunication Union) allocate specific frequency bands for uses like broadcasting, mobile communications, aviation, etc.
- The radio spectrum is divided and licensed to prevent interference between different services

### Cell Phones
- Use multiple frequencies organized into **cells**
- A phone switches to a different antenna as the user moves between cells
- Maximum distance to an antenna tower: **~22 to 45 miles** in optimal conditions
- Typical distance to a tower: **1 to 3 miles**
- Cell phones operate across several frequency bands between **600 MHz and 39 GHz**

### Wi-Fi
- Operates at **2.4, 5, and 6 GHz**
- Relatively **low powered** compared to cell phones
- Speed has increased across generations; actual speed depends on **distance, noise, and obstacles**

| Gen. | IEEE Standard | Adopted | Link Rate (Mbit/s) |
|---|---|---|---|
| — | 802.11 | 1997 | 1–2 |
| — | 802.11b | 1999 | 1–11 |
| — | 802.11a | 1999 | 6–54 |
| — | 802.11g | 2003 | 6–54 |
| Wi-Fi 4 | 802.11n | 2009 | 6.5–600 |
| Wi-Fi 5 | 802.11ac | 2013 | 6.5–6,933 |
| Wi-Fi 6 / 6E | 802.11ax | 2021 | 0.4–9,608 |
| Wi-Fi 7 | 802.11be | 2024 | 0.4–23,059 |

### Other Radio Networks
- **RFID** (Radio Frequency Identification) — 13.56 MHz
- **NFC** (Near Field Communication) — 13.56 MHz
- **Zigbee** — IEEE 802.15.4, low data rate, long battery life, security-focused, 902–928 MHz
- **Bluetooth** — short-range radio connection, 2.4 GHz

### Satellites
- Geosynchronous satellites orbit **36,000 km** above Earth
- Round trip distance to a geosynchronous satellite: **72,000 km (72 Mm)**
- Radio propagates at **3 × 10⁸ m/s**
- This large round-trip distance is why geostationary satellite links have noticeable latency

**Satellite Altitude Classes:**

| Orbit | Altitude (km) | Uses |
|---|---|---|
| Low Earth Orbit (LEO) | 160 – 2,000 | Space Station, Starlink, reconnaissance |
| Medium Earth Orbit (MEO) | 2,000 – 35,786 | GPS |
| High Earth Orbit (HEO) | 35,786 | Geostationary, weather, TV |

**Starlink (example LEO system):**
- Over 7,600 mass-produced small satellites in LEO
- Satellites orbit at 525, 530, and 535 km altitude
- Provides internet connectivity to underserved areas
- 58–227 Mbps download in North Carolina
- $80 or $120/month
- A SpaceX company founded by Elon Musk

**Satellite Properties (general):**
- Very high initial (infrastructure) cost, but low cost per additional user
- High speed possible — over 100s of Mbps
- Both **space weather** (geomagnetic storms) and **terrestrial weather** can degrade signals
- Transmissions can be intercepted by adversaries (broadcast nature)
- Mobile — can be used across the globe
- Concern: Starlink and similar constellations may interfere with radio astronomy

### Power Line Communication (PLC)
- A modem converts the digital signal to a higher frequency and sends it over the existing power cable
- Can be used to control lighting, appliances, and other home devices

**Pros:**
- Cost-effective — uses existing wiring, allows fast deployment

**Cons / Limitations:**
- Signal interference from noise on the power lines
- Distance limitations, especially at higher speeds
- Data rates generally lower than Ethernet or Wi-Fi
- Signals might not pass through transformers
- Easy for someone in the same building to capture the signal
- Not mobile, but a device can easily be moved to another plug

### Moving a Storage Device ("Sneakernet" Transfer)
- Data can always be transferred by putting it on a thumb drive and carrying it to the destination
- Amazon built a truck trailer called **Snowmobile** with **100 Pbytes** of storage capacity
- High-speed fiber was used to load and unload the data at each end

### Media Comparison Table
(5 = good, 1 = bad)

| Property | Twisted Pair | Coax | Fiber | Infrared | Radio | Satellite | Power Line |
|---|---|---|---|---|---|---|---|
| Cost | 5 | 4 | 4 | 5 | 4 | 1 | 5 |
| Mobility | 1 | 1 | 1 | 4 | 5 | 5 | 2 |
| Installation & repair | 5 | 4 | 3/2 | 5 | 4 | 1 | 5 |
| Attenuation | 3 | 4 | 5 | 2 | 4 | 5 | 2 |
| Interference | 3 | 4 | 5 | 2 | 3 | 3 | 2 |
| Security | 3 | 4 | 5 | 1 | 2 | 1 | 3 |
| Can cross public land | 1 | 1 | 1 | 3 | 5 | 5 | 1 |

---

## Part 2: Transmission Limitations

### Purpose of a Network
- The purpose of a network is to send a stream of **bits** from one node to another
- Transmitted bits can represent text, data, digitized voice, or graphics
- The transmission media can be electrical, light, radio, etc.
- Groups of 8 bits are called **octets** or **bytes**

### Serial vs. Parallel Transmission
- Data is usually sent over a **single channel one bit at a time** (serial)
- A single wire transmits bits one after another
- Some short-distance systems (e.g., older printer cables) sent multiple bits at once over multiple parallel wires
- Over long distances, multiple parallel bits can get **out of sync**
- A single wire is cheaper than multiple wires — this is why serial transmission dominates over long distances

### Synchronous vs. Asynchronous Transmission
- **Asynchronous:** no fixed time relationship between transmissions
- **Synchronous:** once transmission starts, bits are sent at a regular interval; sender and receiver must have synchronized clocks
- Some systems are **bit-wise synchronous but byte-wise asynchronous** (e.g., RS-232) — bits within a byte are sent at a fixed rate, but bytes themselves can start at arbitrary times

### Sending Bits as Voltage
- Bits can be sent by varying voltage on a line
- Example convention: positive voltage = 0 bit, negative voltage = 1 bit
- **Totally asynchronous system:** each bit sent as an arbitrary-length voltage pulse — very few real systems use this because there's no way to tell where one bit ends and the next begins reliably
- **Improved asynchronous approach:** fixed-width pulses per bit — makes it easy for the receiver to know when the next bit arrives and when a bit's signal is complete; there is no single fixed transmission speed enforced across the whole system, but the sender maintains consistent bit timing

### The Problem With Leading 1 Bits
- If "idle" is represented the same way as a string of 1 bits, the receiver **cannot differentiate** between an idle line and leading 1 bits
- This ambiguity motivates the use of explicit **start** and **stop** bits

### RS-232-C Standard
- An Electronic Industry Association (EIA) standard for transmitting data over short distances
- Also known as **ITU V.24**
- Bit-wise synchronous but byte-wise asynchronous
- Voltage encoding:
  - **1 bit = −15 volts (MARK)**
  - **0 bit = +15 volts (SPACE)**
  - Idle line is held at **−15 volts** (same as a 1/MARK state)
  - If the voltage stays at **0 volts**, the line is considered **broken**
- RS-232 is used much less today — most computers now use USB

### Start and Stop Bits
- Each byte **starts** with a special 0 bit called the **start bit** (not a data bit)
- Each byte **ends** with a non-data 1 bit called the **stop bit**
- There may be **1, 1.5, or 2** stop bits
- This framing solves the "leading 1 bits vs. idle" ambiguity — the transition from idle (MARK/1) to the start bit (SPACE/0) tells the receiver exactly when a new byte begins

### Error Detection: Parity
- An extra parity bit is often sent with every byte to help detect errors
- The **parity bit** is the **XOR** of all data bits in the byte
- XOR can be thought of as "addition without carry"
- The receiver computes parity on the received data and compares it to the received parity bit — a mismatch indicates an error
- Parity types: **even, odd, mark, space,** or **none**
- **Even parity:** adds a bit so the total number of 1-bits (including the parity bit) is even

**Parity Example:**

| Data | Parity | Result |
|---|---|---|
| 01100100 | 1 | original |
| 01101100 | 0 | detected error |
| 01101110 | 1 | undetected error (two bits flipped — parity can't catch all errors) |

> **Key limitation:** simple parity only reliably detects an **odd number** of bit errors. If two bits flip, the parity can come out looking correct even though the data is wrong (an undetected error).

**Parity Example — ASCII "Z" (7-bit ASCII):**
Even parity for "Z" = 1 XOR 0 XOR 1 XOR 1 XOR 0 XOR 1 XOR 0 = **0**
(XOR all 7 data bits together to get the parity bit that would be appended.)

### Overhead
- **Overhead** = bits sent that are not part of the actual data
- RS-232 has **3 overhead bits** per **8 data bits**: start, stop, and parity
- Overhead fraction = 3/11 ≈ **27%**

### Bit Timing / Speed
- The time used to send each bit (including start and stop bits) in RS-232 is constant/uniform
- If each bit period is 1.0 millisecond → **1000 bits/second**
- Reducing the bit period increases the number of bits sent per second

---

## Part 3: More Data per Signal — Baud vs. Bits/Second

### Multi-Level Signaling
- RS-232 only sends one of **two** values (+15V and −15V)
- If the receiver can detect a **wider range of values** (e.g., +10V, +5V, −5V, −10V — four values), more data can be sent per signal
- If there are **V** possible values, each signal can represent **log2(V)** bits
- Example: 4 possible values → each transmitted signal carries **2 bits**

### Baud vs. Bits/Second
- **Baud rate** = the number of states/signals sent per second
- If V > 2 (more than two possible signal states), the **bit rate is greater than the baud rate**
- Systems may have a baud rate that is **greater or less than** the bits/second rate depending on the encoding scheme

### Manchester Encoding
- Used by early Ethernet to transmit bits
- The signal always changes in the **middle** of the bit period
- **0 bit** = transition high → low
- **1 bit** = transition low → high
- **Baud rate is twice the bit rate** (because each bit requires a mid-period transition in addition to any transition at the bit boundary)

### Limits on Number of States
- More states (4, 8, 16, 1024...) could theoretically improve throughput, but there's a **limit** to how many states can be reliably detected
- **Noise** makes it difficult to differentiate states that are close to each other in value

### Bandwidth
- **Bandwidth** = the range of frequencies that can be sent and received = (highest frequency used) minus (lowest frequency used)
- Example: human ear functions ~50 Hz to 20 kHz → bandwidth ~ **19,950 Hz**
- Example: traditional telephone system filters allow only 50 Hz–3050 Hz → bandwidth ~ **3 kHz**

---

## Part 4: Nyquist Formula (Noiseless Channels)

### The Formula
Derived by Harry Nyquist in 1924, relating bandwidth to maximum data rate:

**max data rate (bits/sec) = 2 x B x log2(V)**

Where:
- **B** = bandwidth (Hz)
- **V** = number of different signal values/states that can be sent

### Important Notes on Nyquist
- Applies to a **perfect, noiseless** channel
- Gives the **absolute maximum** speed possible under ideal conditions
- Any network advertising a speed greater than what Nyquist allows should be viewed with the same skepticism as a claimed perpetual motion machine

### Calculator Tip: Change of Base
Most calculators don't have a base-2 log function. Use the change of base formula:

**log_A(B) = log_X(B) / log_X(A)**

Useful constant: **log10(2) = 0.30103**

### Nyquist Example 1 — Solving for Rate
Telephone line, 3 kHz bandwidth, binary signals (V = 2):

max rate = 2 x 3000 x log2(2) = 2 x 3000 x 1 = **6000 bits/second**

### Nyquist Example 2 — Solving for Bandwidth
Need to transmit 128K bits/second, using 16 different states (V = 16). Find required bandwidth.

Rearranged formula: **B = transrate / (2 x log2V)**

B = 128,000 / (2 x log2(16)) = 128,000 / (2 x 4) = 128,000 / 8 = **16 kHz**

### Nyquist Example 3 — Solving for Number of States
A modem transmits 32K bits/sec over a 3 kHz landline. How many states (V) must it support?

Rearranged formula: **V = 2^(transrate / 2B)**

V = 2^(32,000 / 6,000) = 2^5.33 ~ **40 states**

---

## Part 5: Shannon Formula (Noisy Channels)

### Noise
- Real-world transmissions are subject to **noise and distortion**
- Example: audible static/noise on a weak radio channel
- Noise makes it difficult to distinguish between different signal states

### Signal-to-Noise Ratio
- Noise is represented as a **signal-to-noise ratio (S/N)** — the ratio of signal strength (energy) to noise strength (energy)
- Analogy: whispering (low signal strength) in a noisy room is hard to hear
- Noise **reduces** the achievable data transmission rate

### The Shannon Formula
Derived by Claude Shannon in 1948, for channels with random thermal noise:

**max data rate (bits/sec) = B x log2(S/N + 1)**

- This is a **physical law** applying to all communication systems (not just an engineering limit — a fundamental limit)
- Very weak signals (e.g., deep space probes) can only transmit a few bits per second because S/N is so low

### Decibels (dB)
Noise/signal levels are often measured in decibels:

**dB = 10 x log10(S/N)**
**S/N = 10^(dB/10)**

- The decibel scale is **exponential**
- A good telephone connection has a noise level of about **34–38 dB**

### Shannon Formula Simplified Using dB
Starting from the Shannon formula and substituting S/N = 10^(dB/10), and assuming S/N is large enough to ignore the "+1":

maxrate = B x log2(S/N) -> substitute -> simplify via change of base ->

**maxrate = B x dB / 3.01**

(Derivation collapses the log2 and the dB conversion constant, log10(2) ~ 0.30103, into the single divisor 3.01.)

### Shannon Example
Telephone line with a signal-to-noise ratio of **34 dB**, bandwidth 3000 Hz:

max rate = (B x dB) / 3.01 = (3000 x 34) / 3.01 = 102,000 / 3.01 ~ **33,890 bits/second (33.89 Kbps)**

**Why this matters:** This is why the fastest telephone modem **upload** speed tops out around **31.2 Kbps**, and why "56K" modems don't actually achieve 56,000 bits/second in practice — they are still bound by the Shannon limit for a typical phone line's S/N ratio.

### Maximum Possible States (Combining Nyquist and Shannon)
Setting the Nyquist formula equal to the Shannon formula lets us solve for the maximum number of usable states **V** given a specific noise level:

Starting point:
2B*log2V = B*log2(S/N + 1)

Simplify (divide by B, then work through the algebra):

2*log2V = log2(S/N + 1)
log2(V^2) = log2(S/N + 1)
**V^2 = S/N + 1**
**V = sqrt(S/N + 1)**

### Using Decibels for Maximum States
Assuming S/N is large enough that the "+1" can be ignored, and substituting S/N = 10^(dB/10):

V^2 = 10^(dB/10)
2*log10V = dB/10
log10V = dB/20

**V = 10^(dB/20)**

This gives the maximum number of distinguishable signal states supportable at a given decibel signal-to-noise ratio.

---

## Quick Reference: Key Formulas

| Concept | Formula |
|---|---|
| Bits per signal state | log2(V) |
| Nyquist max rate (noiseless) | 2 x B x log2(V) |
| Nyquist solved for B | B = rate / (2 x log2V) |
| Nyquist solved for V | V = 2^(rate / 2B) |
| Decibels | dB = 10 x log10(S/N) |
| S/N from dB | S/N = 10^(dB/10) |
| Shannon max rate (noisy) | B x log2(S/N + 1) |
| Shannon max rate (using dB, S/N large) | (B x dB) / 3.01 |
| Max usable states (exact) | V = sqrt(S/N + 1) |
| Max usable states (using dB, S/N large) | V = 10^(dB/20) |

---

## Reading
Read **section 2.6** of the textbook for Monday.
