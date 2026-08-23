# ECEN 375 – Unit 2: Logic Gates

**Course:** Computer Architecture and Organization
**Instructor:** Dr. Daniel Limbrick, Dept. of ECE, NC A&T

---

## Overview

- **Logic gates** are the basic building blocks of digital circuits — small circuits that implement Boolean functions
- Each gate takes one or more binary inputs and produces a single binary output
- A gate's behavior can be described three equivalent ways:
  1. **Schematic symbol** (the drawn shape)
  2. **Boolean equation** (algebraic expression)
  3. **Truth table** (every input combination and its output)
- Convention: inputs are usually named **A**, **B** (etc.), output is named **Y**

---

## NOT Gate (Inverter)

- **Symbol:** triangle with a small bubble (circle) on the output
- **Equation:** Y = Ā (read "Y equals NOT A" or "A bar")
- Inverts a single input: 0 becomes 1, and 1 becomes 0
- The output **bubble** is what indicates inversion — this same bubble convention shows up on NAND and NOR later

| A | Y |
|---|---|
| 0 | 1 |
| 1 | 0 |

---

## AND Gate

- **Symbol:** flat-backed shape with a rounded (D-shaped) front, two inputs on the flat side
- **Equation:** Y = AB (multiplication notation means AND)
- **Output is 1 only when *all* inputs are 1**
- Think of it as "both A **and** B must be true"

| A | B | Y |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

---

## OR Gate

- **Symbol:** curved back, pointed front
- **Equation:** Y = A + B (plus notation means OR)
- **Output is 1 if *at least one* input is 1**
- Think of it as "either A **or** B (or both) is true"

| A | B | Y |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

---

## XOR Gate (Exclusive OR)

- **Symbol:** like an OR gate, but with an extra curved line just before the inputs
- **Equation:** Y = A ⊕ B
- **Output is 1 when the inputs are *different* (exactly one is 1)**
- Output is 0 when the inputs match (both 0 or both 1)
- Useful for comparing bits / detecting differences; also used in binary addition (sum bit)

| A | B | Y |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

---

## NAND Gate (NOT AND)

- **Symbol:** same D-shape as AND, but with a bubble on the output
- **Equation:** Y = 𝐴𝐵̄ (AND, then inverted)
- **Output is the opposite of AND** — 0 only when *all* inputs are 1, otherwise 1
- The bubble = invert, same idea as the NOT gate's bubble

| A | B | Y |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

---

## NOR Gate (NOT OR)

- **Symbol:** same curved shape as OR, but with a bubble on the output
- **Equation:** Y = 𝐴+𝐵̄ (OR, then inverted)
- **Output is the opposite of OR** — 1 only when *all* inputs are 0, otherwise 0

| A | B | Y |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 0 |

---

## Quick Comparison

| Gate | Equation | Output is 1 when... |
|---|---|---|
| NOT | Y = Ā | the single input is 0 |
| AND | Y = AB | **all** inputs are 1 |
| OR | Y = A + B | **at least one** input is 1 |
| XOR | Y = A ⊕ B | inputs **differ** |
| NAND | Y = ‾AB | **not all** inputs are 1 (opposite of AND) |
| NOR | Y = ‾(A+B) | **all** inputs are 0 (opposite of OR) |

---

## Key Terms
- **Gate** — smallest digital circuit implementing a single Boolean function
- **Truth table** — lists the output for every possible combination of inputs
- **Bubble (on a gate symbol)** — indicates logical inversion/NOT
- **AND vs. NAND**, **OR vs. NOR** — inverted pairs (bubble added to output)
- **XOR** — outputs 1 only when inputs differ; central to binary addition (sum bit) and parity checking
