# ECEN 375 – Unit 1: Introduction

**Course:** Computer Architecture and Organization
**Instructor:** Dr. Daniel Limbrick, Dept. of ECE, NC A&T

---

## Computer Architecture vs. Organization

- **Architecture** — the design of the system *visible to the assembly-level programmer*
  - What instructions exist
  - How many registers
  - Memory addressing scheme
- **Organization** — *how* the architecture is implemented in hardware
  - How much cache memory
  - Microcode vs. direct hardware control
  - Implementation technology

### Same Architecture, Different Organization
- Almost any program that runs on an original Pentium (or 8088) can run on an Intel i7 — all Pentium-series computers share the same architecture, but each version has a different organization/implementation.
- The IBM 360 was released in several models. All shared the same architecture (a program compiled on one model would run on all), but each model had a different implementation, speed, and price.

---

## Basic Computer Components

- **CPU** — contains the control logic that initiates most computer activity
  - **ALU (Arithmetic Logic Unit)** — performs math and logic calculations
  - **Registers** — hold temporary data values
  - **Program Counter (PC)** — holds the address of the next instruction to execute
- **Cache** — fast memory sitting close to the CPU
- **Memory (RAM)** — main storage for data and instructions
- **I/O Controller / I/O Device**
- **Bus** — connects CPU, memory, and I/O controllers

### Bus
- A set of parallel wires connecting CPU, memory, and I/O controllers
- Has logic (the **chipset**) that determines who can use the bus at any instant
- Bus **width** determines the maximum memory configuration

### I/O Controllers
- Direct the flow of data to/from I/O devices
- CPU sends a request to the I/O controller to initiate I/O
- I/O controllers run **independently and in parallel** with the CPU
- May **interrupt** the CPU upon completion of a request or on error

### Registers
- Temporarily hold data being acted on by the CPU
- Different architectures have different numbers of registers
- Some are directly usable by user programs; some used indirectly (e.g., PC); some reserved for the OS (e.g., program status register)

### Memory
- Internal memory = **RAM (Random Access Memory)**
- Both data and program instructions are stored in RAM
- Instructions **must** be in RAM to be executed

### Memory Hierarchy (fastest/smallest → slowest/largest)
1. Registers
2. Cache
3. RAM
4. Disk
5. Removable Media

---

## Instruction Cycle

**Simple version:**
1. Fetch the instruction from memory
2. Execute the instruction

**Detailed version:**
1. Fetch the instruction from the memory address in the Program Counter register
2. Increment the Program Counter
3. Decode the type of instruction
4. Fetch the operands
5. Execute the instruction
6. Store the results

### Simple Model of Execution
- Instruction sequence is determined by a simple conceptual control point
- Each instruction completes before the next one starts
- One instruction executes at a time

---

## Layers of Abstraction

From highest to lowest level:
- Applications
- Middleware
- High-level languages
- **Machine Language**
- **Microcode**
- **Logic circuits**
- Gates
- Transistors
- Silicon structures

---

## Number Systems

### Binary Values
- Two discrete values: 1's and 0's (TRUE/FALSE, HIGH/LOW)
- Digital circuits represent 1 and 0 using **voltage** levels
- **Bit** = **B**inary dig**it**

### Decimal vs. Binary Place Value
- Decimal: 5374₁₀ = 5×10³ + 3×10² + 7×10¹ + 4×10⁰
- Binary: 1101₂ = 1×2³ + 1×2² + 0×2¹ + 1×2⁰ = 13₁₀

### Powers of Two (memorize up to 2⁹)
| Power | Value | Power | Value |
|---|---|---|---|
| 2⁰ | 1 | 2⁸ | 256 |
| 2¹ | 2 | 2⁹ | 512 |
| 2² | 4 | 2¹⁰ | 1024 |
| 2³ | 8 | 2¹¹ | 2048 |
| 2⁴ | 16 | 2¹² | 4096 |
| 2⁵ | 32 | 2¹³ | 8192 |
| 2⁶ | 64 | 2¹⁴ | 16384 |
| 2⁷ | 128 | 2¹⁵ | 32768 |

### Binary Values and Ranges
- **N-digit decimal number:** 10ᴺ possible values, range [0, 10ᴺ − 1]
  - Example: 3 digits → 10³ = 1000 values, range [0, 999]
- **N-bit binary number:** 2ᴺ possible values, range [0, 2ᴺ − 1]
  - Example: 3 bits → 2³ = 8 values, range [0, 7] = [000₂, 111₂]

### Hexadecimal Numbers
- Base 16 — shorthand for binary
- Digits 0–9, then A–F for 10–15

| Hex | Dec | Bin | Hex | Dec | Bin |
|---|---|---|---|---|---|
| 0 | 0 | 0000 | 8 | 8 | 1000 |
| 1 | 1 | 0001 | 9 | 9 | 1001 |
| 2 | 2 | 0010 | A | 10 | 1010 |
| 3 | 3 | 0011 | B | 11 | 1011 |
| 4 | 4 | 0100 | C | 12 | 1100 |
| 5 | 5 | 0101 | D | 13 | 1101 |
| 6 | 6 | 0110 | E | 14 | 1110 |
| 7 | 7 | 0111 | F | 15 | 1111 |

**Hex → Binary:** each hex digit maps to exactly 4 binary bits.
- Example: 4AF₁₆ (0x4AF) = 0100 1010 1111₂

**Hex → Decimal:** multiply each digit by its place value (16ⁿ) and sum.
- Example: 4AF₁₆ = 16²×4 + 16¹×10 + 16⁰×15 = 1199₁₀

### Bits, Bytes, Nibbles
- **Bit**: single binary digit (MSB = most significant bit, LSB = least significant bit)
- **Nibble**: 4 bits (half a byte)
- **Byte**: 8 bits
- Multi-byte values have a **most significant byte** and **least significant byte**

### Large Powers of Two
- 2¹⁰ = 1 kilo ≈ 1000 (actually 1024)
- 2²⁰ = 1 mega ≈ 1 million (1,048,576)
- 2³⁰ = 1 giga ≈ 1 billion (1,073,741,824)

**Estimating large powers of two:**
- 2²⁴ = 2⁴ × 2²⁰ ≈ 16 million
- A 32-bit variable can represent 2² × 2³⁰ ≈ 4 billion values

---

## Binary Arithmetic

### Addition
- Same process as decimal addition, but carries happen at value 2 instead of 10
- Example: 1011 + 0011 = 1110 (with carries)
- Example: 1011 + 0110 = 10001 → **5 bits needed**, but only 4 bits available → **Overflow!**

### Overflow
- Digital systems operate on a **fixed number of bits**
- **Overflow** occurs when the result is too big to fit in the available number of bits

---

## Signed Binary Numbers

Two common representations:
1. **Sign/Magnitude Numbers**
2. **Two's Complement Numbers**

### Sign/Magnitude Numbers — Problems
- **Addition doesn't work correctly.** Example: −6 + 6:
  ```
    1110
  + 0110
  ------
   10100  (wrong! should be 0)
  ```
- **Two representations of zero** (±0): `1000` and `0000`

### Two's Complement Numbers
- Solves both sign/magnitude problems:
  - **Addition works correctly**
  - **Single representation for 0**
- The **most significant bit (MSB)** has a value of **−2ᴺ⁻¹** (instead of positive)
  - Formula: A = a_(N−1)(−2^(N−1)) + Σ(i=0 to N−2) a_i·2^i
- MSB still indicates sign: **1 = negative, 0 = positive**
- For a 4-bit two's complement number:
  - Most positive value: `0111` = +7
  - Most negative value: `1000` = −8
- **Range of an N-bit two's complement number:** [−2^(N−1), 2^(N−1) − 1]

---

## Architecture and Microarchitecture

- **Architecture** = programmer's view of the computer, defined by instructions and operand locations
- **Microarchitecture** = how to implement an architecture in hardware (covered later, Ch. 7)

### Instructions
- **Assembly language** — human-readable format of instructions
- **Machine language** — computer-readable format (1's and 0's)

---

## ARM Architecture

- Developed in the 1980s by **Advanced RISC Machines** (now ARM Holdings)
- Nearly 10 billion ARM processors sold per year
- Almost all cell phones and tablets contain multiple ARM processors
- Over 75% of humans use a product containing an ARM processor
- Used in servers, cameras, robots, cars, pinball machines, etc.
- Once you've learned one architecture, it's easier to learn others

### Architecture Design Principles (Hennessy & Patterson)
1. **Regularity supports design simplicity**
2. **Make the common case fast**
3. **Smaller is faster**
4. **Good design demands good compromises**

---

## ARM Instructions

### ADD
```
C Code:            a = b + c;
ARM Assembly:       ADD a, b, c
```
- `ADD` — mnemonic indicating the operation
- `b, c` — source operands
- `a` — destination operand

### SUB
```
C Code:            a = b - c;
ARM Assembly:       SUB a, b, c
```
- Same operand structure as ADD, just a different mnemonic

### Design Principle 1 — Regularity Supports Design Simplicity
- Consistent instruction format
- Same number of operands (two sources, one destination)
- Eases encoding and hardware handling

### Multiple Instructions for Complex Expressions
More complex C code is broken into multiple simple ARM instructions:
```
C Code:  a = b + c - d;

ARM:     ADD t, b, c   ; t = b + c
         SUB a, t, d   ; a = t - d
```

### Design Principle 2 — Make the Common Case Fast
- ARM includes only simple, commonly used instructions
- Hardware to decode/execute instructions stays simple, small, and fast
- Less-common complex operations are built from multiple simple instructions
- **ARM = RISC** (Reduced Instruction Set Computer) — small number of simple instructions
- **Intel x86 = CISC** (Complex Instruction Set Computer)

---

## Operand Locations

Operands can physically reside in:
- **Registers**
- **Constants** (also called *immediates*)
- **Memory**

### Design Principle 3 — Smaller Is Faster
- ARM includes only a small number of registers

### ARM Registers
- ARM has **16 registers**, each **32 bits** wide
- Registers are faster than memory
- ARM is called a **"32-bit architecture"** because it operates on 32-bit data
- Register naming: "R" + number, all caps (e.g., "R0" or "register zero")

| Name | Use |
|---|---|
| R0 | Argument / return value / temporary variable |
| R1–R3 | Argument / temporary variables |
| R4–R11 | Saved variables |
| R12 | Temporary variable |
| R13 (SP) | Stack Pointer |
| R14 (LR) | Link Register |
| R15 (PC) | Program Counter |

- **Saved registers** (R4–R11): hold variables across calls
- **Temporary registers** (R0–R3, R12): hold intermediate values

### Instructions Using Registers
```
C Code:   a = b + c
          ; R0 = a, R1 = b, R2 = c
ARM:      ADD R0, R1, R2
```

### Operands: Constants / Immediates
- Many instructions (e.g., `ADD`, `SUB`) can use a constant/immediate operand
- "Immediate" = the value is immediately available from the instruction itself (no register/memory lookup needed)

```
C Code:
a = a + 4;
b = a - 12;

ARM Assembly:
; R0 = a, R1 = b
ADD R0, R0, #4
SUB R1, R0, #12
```

### Generating Constants with MOV
- `MOV` loads a small constant into a register
- Constant must have **< 8 bits of precision**
- `MOV` can also move a value between two registers: `MOV R7, R9`

```
C Code:
int a = 23;
int b = 0x45;

ARM Assembly:
; R0 = a, R1 = b
MOV R0, #23
MOV R1, #0x45
```

### Generating Larger Constants with MOV + ORR
- Larger (>8-bit) constants are built up using `MOV` for the first chunk, then `ORR` (bitwise OR) to add in additional bits

```
C Code:
int a = 0x7EDC8765;

ARM Assembly:
# R0 = a
MOV R0, #0x7E000000
ORR R0, R0, #0xDC0000
ORR R0, R0, #0x8700
ORR R0, R0, #0x65
```

---

## Key Terms
- **Architecture** vs **Organization** vs **Microarchitecture**
- **CPU, ALU, PC, Cache, Bus, RAM**
- **Instruction cycle**: fetch → decode → fetch operands → execute → store
- **Bit, Nibble, Byte, MSB, LSB**
- **Two's complement** — standard signed representation; range [−2^(N−1), 2^(N−1)−1]
- **Overflow** — result doesn't fit in the fixed number of bits
- **RISC (ARM)** vs **CISC (x86)**
- **Register, Immediate/Constant, Memory** — the three operand locations
- ARM: 16 registers, 32-bit architecture, mnemonics `ADD`, `SUB`, `MOV`, `ORR`
