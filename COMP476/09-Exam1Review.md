# COMP476 — Networked Computer Systems
## Exam 1 Review (slides: "Review for Exam 1," Fall 2026 — reviewed Mon Sept 14, exam Wed Sept 16, 2026)

### 0. Admin notes from this lecture
- **Exam format**: one and only one 8.5"×11" page of your own handwritten notes allowed — don't copy a friend's page.
- **Scope disclaimer**: "the exam may contain questions from any of the material covered in class since the beginning of the semester," but the two chapter ranges below are what this review session focused on.
- **Textbook chapters on the test**: 1.5 & 1.6 (Protocol layers), 2.1 to 2.6 (Physical layer). Topics not covered in class won't be on the exam.
- **Professor's "likely exam questions" list**: Nyquist & Shannon formulas · calculating transmission time · interpreting modulation waves · Tq for an M/M/1 system — all "very similar to the quiz."

### 1. Where this material already lives (study map)
This review re-covers ground already in the repo — use this as an index, not a replacement:

| Review topic | Full notes |
|---|---|
| Protocol layers (OSI vs Internet) | `02-ProtocolLayers.md` |
| Media considerations, fiber optics, RS-232 framing | `04-RadioMedia&Transmissions.md` |
| Nyquist/Shannon, Fourier/harmonics, AM/FM/PSK/QAM, multiplexing (FDM/WDM/TDM/CDM) | `05-Modulation.md` |
| Switching methods, packet structure, packet transmission time | `06-PacketTransmissions.md` |
| Queuing theory, M/M/1, M/D/1, Little's formula | `07-QueueingThoery.md` |
| A/D voice conversion, cells, handoff, MTSO | `08-TelephoneSystems.md` |

The rest of this file is what's **new** in the review: the worked practice problems, plus a condensed formula sheet.

### 2. Formula quick-reference
| Formula | Use |
|---|---|
| `max data rate = 2 * B * log2(V)` | Nyquist — max rate, no noise |
| `max data rate = B * log2(S/N + 1)` | Shannon — max rate, with noise |
| `maxrate = (B * dB) / 3.01` | Shannon, dB form |
| `V = sqrt(S/N + 1)` or `V = 10^(dB/20)` | max detectable states given noise |
| `time = dataBits / bitRate + distance / propagationSpeed` | simple point-to-point transmission time (propagation speed ≈ 2×10⁸ m/sec in cable) |
| `time = ⌈dataBits / PktSize⌉ * (PktSize + headerSize) / transmissionRate` | transmission time over a packet-switched link (accounts for header overhead per packet) |
| `Tq = s / (1 − ρ)`, `ρ = λ*s` | M/M/1 average time in system |
| parity bit = XOR of all data bits | error detection — mismatch on receive = error |
| RS-232 async byte = start(1) + data(8) + parity(1) + stop(1) = **11 bits/byte** | framing overhead for serial transmission time problems |

### 3. Worked practice problems

**Problem 1 — simple link transmission time**
*How long to send a 52 KB file to San Francisco (4,500 km away) over a 1.54 Mbps line?*
- Setup: `time = dataBits/bitRate + distance/propagationSpeed`
- Transmission: `52×10³ bytes * 8 bits/byte / 1.54×10⁶ bits/sec = 0.270 sec`
- Propagation: `4.5×10⁶ m / 2×10⁸ m/sec = 0.0225 sec`
- Total: `0.270 + 0.0225 = 0.293 sec = 293 msec`
- Verify: matches answer choice D (293 msec) on the slide.

**Problem 2 — RS-232C async transmission with framing overhead**
*How long to send a 256-byte message over a 31.2 Kbps RS-232C line, even parity, short distance (no propagation term)?*
- Setup: async RS-232 = 11 bits/byte (1 start + 8 data + 1 parity + 1 stop)
- `time = 256 bytes * 11 bits/byte / 31.2×10³ bits/sec`
- `= 2,816 bits / 31,200 bits/sec = 0.0903 sec = 90.3 msec`
- Verify: matches answer choice C (90.3 msec).
- **Contrast with Problem 1**: no propagation term here (short line) and framing overhead (11 vs 8 bits/byte) is what's added instead — know which term a problem is asking for.

**Problem 3 — ATM packet transmission time (header overhead)**
*How long to send 24 KB over a 56 Mbps ATM network, where each packet carries 48 bytes of data + 6 bytes of header?*
- Setup: `time = ⌈dataBits/PktSize⌉ * (PktSize+headerSize)/transmissionRate`
- Packets needed: `24×10³ bytes / 48 bytes/pkt = 500 packets` (exact, no rounding up needed)
- Bits per packet on the wire: `8 bits/byte * (48+6) bytes = 432 bits`
- `time = 500 * 432 bits / 56×10⁶ bits/sec = 216,000 / 56×10⁶ = 3.86×10⁻³ sec = 3.86 msec`
- Verify: matches answer choice D (3.86 msec).

**Problem 4 — reading bits off a QAM/PSK constellation + waveform**
*Given a constellation diagram (8 points: 000, 001, 010, 011, 100, 101, 110, 111 by phase/amplitude) and a signal graph with one wavelength per symbol, what bits are being sent?*
- Setup: read the phase shift and amplitude of each wavelength segment against the constellation diagram, one symbol at a time.
- Segment 1: matches **001** (90° shift). Segment 2: matches **100** (unshifted, low amplitude). Segment 3: matches **111** (270° shift, low amplitude).
- Concatenated: `001 100 111` → **answer A (001100111)**.
- Technique to reuse: line up the "signal" curve against the "base" reference curve at each wavelength — the shift tells you the phase bits, the height tells you the amplitude bit.

**Problem 5 — M/M/1 queuing (Little's/utilization)**
*A doorman takes an average of 1.5 min to approve someone. 30 people/hr arrive on average. What's the average waiting line length (W)?*
- Identify: server = doorman, queued items = people, model = M/M/1
- `s = 1.5 min`
- `λ = 30 person/hr ÷ 60 min/hr = 0.5 person/min`
- `ρ = s*λ = 1.5 * 0.5 = 0.75`
- `W = ρ²/(1−ρ) = 0.75²/(1−0.75) = 0.5625/0.25 = 2.25`
- Verify: matches answer choice B (2.25). Sanity check: ρ = 0.75 is high utilization, so a queue of ~2 people waiting (on top of whoever's being served) is plausible, not runaway — consistent with `Tq` only blowing up as ρ approaches 1.

### 4. Error detection — quick review
- **Parity bit**: XOR of all data bits in a byte. Receiver recomputes parity and compares — mismatch = error detected. Types: even, odd, mark, space, none.
- **Packet-level error detection functions**: sum the data bytes, XOR the data bytes, or **CRC** (Cyclic Redundancy Check) — result of polynomial division, the most commonly used method.

### 5. Framing techniques — quick review
How a receiver knows where one frame ends and the next begins:
- **Byte count**: frame starts with a count of how many bytes follow.
- **Flag bytes with byte stuffing**: a special byte marks a new frame; escaping prevents data from faking that byte.
- **Flag bits with bit stuffing**: same idea at the bit level.
- **Physical layer coding violations**: a special signal pattern that's illegal as data marks a new frame.

### 6. Multiplexing — quick review
- **FDM** (Frequency-Division): each channel gets its own frequency band + guard bands; electrical systems.
- **WDM** (Wavelength-Division): FDM's optical equivalent — different wavelengths (colors) of light.
- **TDM** (Time-Division): round-robin time slots; synchronous (fixed slot per device) vs asynchronous/statistical (slots assigned on demand, needs a header per slot).
- **CDM** (Code-Division): every sender spreads each bit across many "chips" (usually 64 or 128 chips/bit) using a unique chip code. Chip codes must be **orthogonal** — dot product `A·B = Σ aᵢbᵢ = 0`. Chip values are ±1. Common in 3G cellular.

### 7. Cellular telephony — quick review
- World divided into **cells**, each with its own antenna; cell size ranges from ~20 m (indoor) to ~50 km (rural).
- **Handoff**: as you move, once a neighboring cell's MTSO signal becomes stronger than your current one, the call is handed off — the new MTSO takes over and establishes its own connection over the land lines while the old one drops.
- Voice A/D conversion: 8,000 samples/sec (every 125 μsec), 8 bits/sample → this is the basis of the 64 kbps digital voice channel (see `08-TelephoneSystems.md` for the full derivation).
