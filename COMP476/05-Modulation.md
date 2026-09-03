# COMP476 — Networked Computer Systems
## Lecture 5 — Modulation & Multiplexing (slides: "20260831 Modulation")

### 0. Admin notes from this lecture
- **Metric quiz deadline** extended to **midnight, Monday Aug 31, 2026**. Score = percent correct − (seconds over 60 it took to answer all 20 questions).
- **Transmission Teams assignment** (send your partner a one-word message over the shared media, design your own error detection) is **due noon, Wednesday Sept 9, 2026**. (Full protocol notes already in `Transmission-Teams-Notes.md`.)

### 1. Goals of the lecture
- Understand the different ways to modulate a signal
- Know why we need modems

### 2. Nyquist & Shannon review (carried over from prior lecture, needed for this one)
| Formula | Use |
|---|---|
| `max data rate (bits/sec) = 2 * B * log2(V)` | Nyquist — max rate with **no noise**. B = bandwidth, V = number of distinguishable states/values that can be sent |
| `max data rate (bits/sec) = B * log2(S/N + 1)` | Shannon — max rate **with noise** |
| `maxrate = (B * dB) / 3.01` | Shannon, alternate form using dB |
| `dB = 10 * log10(S/N)` and `S/N = 10^(dB/10)` | Converting between dB and signal-to-noise ratio |
| `V = sqrt(S/N + 1)` or `V = 10^(dB/20)` | Max number of states possible given noise level |

**Key idea carried forward: "Waves are States."** The Nyquist formula's `V` is just "the number of different states a signal can take" — a state could be a voltage level, a wave amplitude/frequency/phase, or even a color. This is the bridge into modulation: modulating a wave = giving it more distinguishable states so more bits can be encoded per symbol.

### 3. Analog vs. digital signals
- **Analog** signal: continuous mathematical function — moves through all intermediate values when changing.
- **Digital** signal: fixed set of valid levels — changes are instantaneous jumps between levels.
- Signals are also classified **periodic** (repeats) vs **aperiodic/nonperiodic** (doesn't).
- RS-232-C transmits digital data as square waves (+15V / −15V) with start/stop/parity bits framing each character.

### 4. Sine waves — the building block
- Data comms analysis relies on sinusoidal functions, especially **sine** (`sin`), because natural phenomena (like EM radiation) produce sine waves.
- Four characteristics of a sine wave, all modifiable ("modulatable"):
  - **Frequency** — oscillations per second (Hz)
  - **Wavelength** — length of one cycle as the signal propagates (depends on propagation speed)
  - **Amplitude** — difference between max and min signal height
  - **Phase** — how far the wave's start is shifted from a reference time
- A sine wave can also be pictured as the **Y-position of a point moving counterclockwise around a unit circle** — this circle model is exactly what's used later for phase-shift/QAM constellation diagrams (0°, 90°, 180°, 270°).

### 5. Fourier: composite signals & harmonics
- **Simple signal**: a single sine wave, can't be decomposed further.
- **Composite signal**: most real signals — the sum of multiple simple sine waves.
- **Jean Baptiste Joseph Fourier** (1768–1830) proved any periodic composite wave = infinite sum of sines and cosines (**Fourier Series**):
  `Σ (aᵢ·sin(t) + bᵢ·cos(t))` for i = 0 → ∞
  - A pure sine wave: a₀ = 1, all other aᵢ, bᵢ (i > 0) = 0.
- Each component sine wave in the sum is a **harmonic**. This is why the same musical note sounds different on a piano vs. a flute — same fundamental frequency, different harmonic intensities.
- **Frequency-domain representation**: instead of plotting amplitude vs. time, plot amplitude vs. frequency — each harmonic shows up as a vertical bar/spike.
- **Bandwidth of an analog signal** = highest frequency − lowest frequency among its constituent harmonics (from Fourier analysis).

### 6. Why square waves are a problem
- A perfect square wave needs **infinite harmonics** (an instantaneous voltage change in zero time = infinite energy — physically impossible).
- Demo progression in the slides: square wave approximated with 4 → 8 → 16 → 64 harmonics — visually gets closer to "square" but still has ringing/overshoot at the edges even at 64.
- A **bandwidth-limited channel** (e.g., a phone line) filters out the higher-frequency harmonics needed to keep the wave square.
- Losing those harmonics **distorts the wave** so it no longer looks square — this makes it hard for the receiver to reliably tell what the original bit value was.
- **Fix: don't send square waves.** Send **sine waves** instead — a sine wave is the sum of exactly *one* harmonic, so it passes through a bandwidth-limited channel unchanged (no higher frequencies need to survive).

### 7. Modulation & modems
- Since sine waves survive band-limited channels intact, we can vary (**modulate**) a sine wave's properties — amplitude, frequency, or phase — to represent different data values.
- A device that modulates (transmit side) and demodulates (receive side) a sine wave = **mo**dulator-**dem**odulator = **modem**.
- Convention used in the example diagrams: each wave segment represents transmitting one value **during one wavelength**; example bit sequence used throughout: `01100`. Red = unmodulated carrier reference, blue = actual modulated signal sent.

### 8. The modulation techniques
| Technique | What varies | 0 bit | 1 bit |
|---|---|---|---|
| **Amplitude Modulation (AM)** | wave height/energy | half-height wave | full-height wave |
| **Frequency Modulation (FM)** | frequency/pitch | low frequency | high frequency |
| **Phase Shift Modulation (PSK)** | starting position/shift of the wave | unshifted (0°) | shifted 180° (mirror image) |

**Phase shift modulation scales up nicely — more phase points = more bits per symbol:**
- **2-value PSK**: points at 0° and 180° on the unit circle → 1 bit/symbol (0 = 0°, 1 = 180°)
- **4-value PSK**: points at 0°, 90°, 180°, 270° → 2 bits/symbol (`00`=0°, `01`=90°, `10`=180°, `11`=270°)
- **8-value PSK**: 8 evenly spaced points around the circle (000, 001, 010, 011, 100, 101, 110, 111) → 3 bits/symbol

**Quadrature Amplitude Modulation (QAM)** = combines amplitude **and** phase shift to pack in even more unique states:
- Example shown: 4 phases × 2 amplitudes = 8 distinct points (3 bits/symbol), points closer to center = low amplitude, outer points = high amplitude.
- Real modems use much denser **constellation diagrams** (the V.32bis example shows a large diamond-shaped grid of points) to pack many bits into each symbol.

### 9. Modem history (illustrates modulation getting more sophisticated over time)
| Connection | Modulation | Speed | Year |
|---|---|---|---|
| Bell 101 (110 baud) | FSK | 0.1 kbit/s | 1958 |
| Bell 103 / V.21 (300 baud) | FSK | 0.3 kbit/s | 1962 |
| Bell 202 (1200 baud) | FSK | 1.2 kbit/s | 1976 |
| V.22bis (600 baud) | QAM | 2.4 kbit/s | 1984 |
| V.32 (2400 baud) | trellis | 9.6 kbit/s | 1984 |
| V.32bis (2400 baud) | trellis | 14.4 kbit/s | 1991 |
| V.32terbo (2400 baud) | trellis | 19.2 kbit/s | 1993 |
| V.90 (8000/3429 baud) | digital | 56/33.6 kbit/s | 1998 |

(Trend: same or similar baud rate, but ever more bits encoded per symbol via richer modulation — this is the Nyquist "V = number of states" idea in action historically.)

### 10. Multiplexing — sharing one physical medium
Multiplexing = techniques that let multiple signals share a single data link simultaneously (avoids needing millions of separate wires). Four types:

1. **Frequency-Division Multiplexing (FDM)**
   - Each logical channel gets its own separate frequency band (with **guard bands** between channels to avoid crosstalk).
   - Sender side: each input is amplitude-modulated onto a different carrier frequency, then the modulated signals are summed onto one link.
   - Receiver side: **filters** split the combined signal back into per-channel bands, then each is demodulated back to data.
   - Used for TV and radio broadcast (many channels over the same media).

2. **Wavelength-Division Multiplexing (WDM)**
   - Theoretically identical to FDM, but used in **optical** systems (FDM is for electrical systems).
   - Like sending different signals as different colors of light — prisms combine/split the wavelengths (λ₁, λ₂, …, λₖ) at each end of the fiber.
   - Requires more spacing between channels than FDM.

3. **Time-Division Multiplexing (TDM)** — a "round robin" use of the channel; interleaves portions of each transmission in time instead of frequency.
   - **Synchronous TDM**: a *frame* = one complete cycle of time slots; # slots per frame = # of input devices. Each device always gets the same slot **every frame**, whether or not it has data to send (can waste slots). Receiver knows who's who purely by slot position — no header needed.
   - **Asynchronous TDM** (statistical TDM): slots in a frame are **not** dedicated to a specific device — # slots ≠ # of inputs necessarily, and a busy device can get more than one slot per frame. Maximizes link utilization (good for multiplexing several lower-speed lines onto one higher-speed line), but **needs a header on every slot** to say which device it belongs to — that header eats into transmission time/capacity.
   - Historical note: TDM was invented in the 1870s by **Émile Baudot** in France (also gave us the Baudot code, ASCII's predecessor, and the "baud" unit is named after him). A 1922 electromechanical telegraph multiplexer used spinning "rings" to interleave multiple telegraph transmitters/receivers onto one line.

4. **Code-Division Multiplexing (CDM)** — listed as the 4th multiplexing type (not detailed with its own slides in this lecture; expect more coverage later, likely tied to spread-spectrum/cellular).

### 11. Quick review (as given at end of slides)
- All waves are the sum of sine waves.
- Modulation is required to minimize harmonics (so signals survive band-limited channels).
- **FM** varies frequency/pitch; **AM** varies signal strength/loudness; **PSK** varies phase/starting point; **QAM** combines amplitude + phase.
- Multiplexing types: FDM, WDM, TDM (sync/async), CDM.
