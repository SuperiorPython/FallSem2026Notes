# Unit 2: Assembly Language – Basic Constructs

**Course:** ECEN 375 – Computer Architecture and Organization
**Instructor:** Dr. Daniel Limbrick, ECE Dept., NC A&T
**Reference:** *Digital Design and Computer Architecture: ARM Edition* (Harris & Harris), Ch. 6

---

## 1. Operands: Registers

- Registers are written with an **R** before the number, all capitals.
  - Example: `R0`, spoken as "register zero" or "register R0"
- Registers are used for specific purposes:
  - **Saved registers:** `R4`–`R11` hold variables
  - **Temporary registers:** `R0`–`R3` and `R12` hold intermediate values
  - (Other special-purpose registers discussed later, e.g., SP, LR, PC)

### Instructions with Registers

Revisit the `ADD` instruction:

```
; R0 = a, R1 = b, R2 = c
ADD R0, R1, R2      ; a = b + c
```

---

## 2. Operands: Constants / Immediates

- Many instructions (e.g., `ADD`, `SUB`) can use constants, called **immediate** operands.
- The value is *immediately* available from the instruction itself (no memory or register lookup needed).

```
; R0 = a, R1 = b
ADD R0, R0, #4       ; a = a + 4;
SUB R1, R0, #12      ; b = a - 12;
```

### Generating Constants

**Small constants** can be generated with `MOV`:

```
; R0 = a, R1 = b
MOV R0, #23          ; int a = 23;
MOV R1, #0x45        ; int b = 0x45;
```

- **Constant must have < 8 bits of precision** for a single `MOV`.
- **Note:** `MOV` can also use two registers: `MOV R7, R9` (copies R9 into R7).

**Larger constants** (more than 8 bits) are built using `MOV` + `ORR`, one byte (or chunk) at a time:

```
# R0 = a; int a = 0x7EDC8765;
MOV R0, #0x7E000000
ORR R0, R0, #0xDC0000
ORR R0, R0, #0x8700
ORR R0, R0, #0x65
```

---

## 3. Operands: Memory

- There are too many variables to fit in only 16 registers.
- Additional data is stored in **memory**.
- Memory is large but slow — commonly used variables are still kept in registers.

### Byte-Addressable Memory

- Each data **byte** has a unique address.
- A 32-bit **word** = 4 bytes, so word addresses increment by 4 (0, 4, 8, 12, …).

| Byte Address | Word Address |
|---|---|
| 0x00000000–0x00000003 | Word 0 |
| 0x00000004–0x00000007 | Word 1 |
| 0x00000008–0x0000000B | Word 2 |
| 0x0000000C–0x0000000F | Word 3 |

### Reading Memory — `LDR`

- Memory read = **load**. Mnemonic: `LDR` (load register).
- Format: `LDR R0, [R1, #12]`
  - **Address calculation:** address = base register (R1) + offset (12)
  - **Result:** R0 holds the data found at that memory address
  - Any register may be used as the base address.

**Example:** Read a word at memory address 8 into R3:

```
MOV R2, #0
LDR R3, [R2, #8]     ; address = (R2 + 8) = 8; R3 = data at address 8
```

### Writing Memory — `STR`

- Memory write = **store**. Mnemonic: `STR` (store register).

**Example:** Store the value in R7 into memory word 21:

```
MOV R5, #0
STR R7, [R5, #0x54]  ; memory address = 4 x 21 = 84 = 0x54
```

- The offset can be written in decimal or hexadecimal.

### Recap: Accessing Memory

- The **address of a memory word** must be multiplied by 4 (word size in bytes).
- Examples:
  - Address of memory word 2 = 2 × 4 = 8
  - Address of memory word 10 = 10 × 4 = 40

### Big-Endian vs. Little-Endian Memory

How to number bytes within a word:

- **Little-endian:** byte numbering starts at the **little** (least significant) end.
- **Big-endian:** byte numbering starts at the **big** (most significant) end.

> Reference: Jonathan Swift's *Gulliver's Travels* — the Little-Endians broke their eggs on the little end, the Big-Endians on the big end. It doesn't matter which convention a system uses, **except** when two systems share data.

**Example:** Suppose R2 = 8 and R5 = `0x23456789`.

```
STR  R5, [R2, #0]
LDRB R7, [R2, #1]
```

- On a **big-endian** system: R7 = `0x00000045`
- On a **little-endian** system: R7 = `0x00000067`

(Because in big-endian, byte address 9 holds `45`; in little-endian, byte address 9 holds `67`.)

---

## 4. Programming Background

- **High-level languages** (e.g., C, Java, Python) are written at a higher level of abstraction than assembly.
- **Ada Lovelace (1815–1852):** British mathematician; wrote the first computer program (calculated Bernoulli numbers on Charles Babbage's Analytical Engine); daughter of the poet Lord Byron.

### Programming Building Blocks

1. Data-processing instructions
2. Conditional execution
3. Branches
4. High-level constructs: `if`/`else`, `for` loops, `while` loops, arrays, function calls

---

## 5. Data-Processing Instructions

Three main categories:
- Add / Subtract
- Logical operations
- Shifts / rotate

### Logical Instructions

| Instruction | Operation |
|---|---|
| `AND` | Bitwise AND |
| `ORR` | Bitwise OR |
| `EOR` | Bitwise XOR |
| `BIC` | Bit Clear (AND NOT) |
| `MVN` | Move and NOT |

**Example** (R1 = `0100 0110`, R2 = `1111 1111` for the first column):

```
AND R3, R1, R2   ; R3 = R1 AND R2
ORR R4, R1, R2   ; R4 = R1 OR  R2
EOR R5, R1, R2   ; R5 = R1 XOR R2
BIC R6, R1, R2   ; R6 = R1 AND (NOT R2)
MVN R7, R2       ; R7 = NOT R2
```

### Logical Instruction Uses

**`AND` or `BIC`: masking bits**

```
0xF234012F AND 0x000000FF = 0x0000002F   ; keep only LSB
0xF234012F BIC 0xFFFFFF00 = 0x0000002F   ; clear all but LSB
```

**`ORR`: combining bit fields**

```
0xF2340000 ORR 0x000012BC = 0xF23412BC
```

### Shift Instructions

| Instruction | Meaning |
|---|---|
| `LSL` | Logical shift left |
| `LSR` | Logical shift right |
| `ASR` | Arithmetic shift right (sign-extending) |
| `ROR` | Rotate right |

**Examples:**

```
LSL R0, R7, #5     ; R0 = R7 << 5
LSR R3, R2, #31    ; R3 = R2 >> 31
ASR R9, R11, R4    ; R9 = R11 >>> R4[7:0]  (arithmetic, sign-extended)
ROR R8, R1, #3     ; R8 = R1 ROR 3
```

**Immediate shift amount:** 5-bit immediate, range 0–31.

**Register shift amount:** uses the low 8 bits of a register, range 0–255.

```
LSL R4, R8, R6     ; shift amount taken from low byte of R6
ROR R5, R8, R6
```

---

## 6. Conditional Execution

Programs don't always execute sequentially. Examples that need conditional behavior:
- `if`/`else` statements, `while` loops — only execute code *if* a condition is true
- Branching — jump to another portion of code *if* a condition is true

ARM includes **condition flags** that can be:
- Set by an instruction
- Used to conditionally execute a later instruction

### ARM Condition Flags (NZCV)

| Flag | Name | Description |
|---|---|---|
| N | Negative | Instruction result is negative |
| Z | Zero | Instruction result is zero |
| C | Carry | Instruction causes an unsigned carry out |
| V | oVerflow | Instruction causes a (signed) overflow |

- Set by the ALU; held in the **Current Program Status Register (CPSR)**.

### Setting the Condition Flags

**Method 1: `CMP` (compare) instruction**

```
CMP R5, R6      ; performs R5 - R6
```
- Does **not** save the result — only sets the flags.
- If result is 0 → Z=1; if negative → N=1; if unsigned carry out → C=1; if signed overflow → V=1.

**Method 2: Append `S` to an instruction mnemonic**

```
ADDS R1, R2, R3  ; performs R2 + R3, sets flags, AND saves result in R1
```

### Condition Mnemonics

Instructions may be conditionally executed based on the flags, using a **condition mnemonic** appended to the instruction mnemonic.

**Example:**
```
CMP   R1, R2
SUBNE R3, R5, R8   ; SUB only executes if R1 != R2 (i.e., Z = 0)
```

| cond | Mnemonic | Name | Condition (CondEx) |
|---|---|---|---|
| 0000 | EQ | Equal | Z |
| 0001 | NE | Not equal | !Z |
| 0010 | CS/HS | Carry set / unsigned higher or same | C |
| 0011 | CC/LO | Carry clear / unsigned lower | !C |
| 0100 | MI | Minus / negative | N |
| 0101 | PL | Plus / positive or zero | !N |
| 0110 | VS | Overflow set | V |
| 0111 | VC | Overflow clear | !V |
| 1000 | HI | Unsigned higher | !Z & C |
| 1001 | LS | Unsigned lower or same | Z or !C |
| 1010 | GE | Signed greater than or equal | N == V |
| 1011 | LT | Signed less than | N != V |
| 1100 | GT | Signed greater than | !Z & (N == V) |
| 1101 | LE | Signed less than or equal | Z or (N != V) |
| 1110 | AL (or none) | Always / unconditional | ignored |

**Worked example:** Suppose R5 = 17, R9 = 23:

```
CMP   R5, R9        ; performs 17 - 23 = -6 -> N=1, Z=0, C=0, V=0
SUBEQ R1, R2, R3     ; does NOT execute (not equal, Z=0)
ORRMI R4, R0, R9     ; executes (result was negative, N=1)
```

---

## 7. Branching

- Branches enable **out-of-sequence** instruction execution.
- Types of branches:
  - **Branch (`B`)** — branches to another instruction
  - **Branch and link (`BL`)** — used for function calls (discussed later)
- Both `B` and `BL` can be **conditional** or **unconditional**.

### The Stored Program

Assembly instructions are encoded into machine code and stored sequentially in memory, referenced by the **Program Counter (PC)**.

| Address | Instructions (machine code) |
|---|---|
| 0x00008000 | `0xE3A01064` ← PC (MOV R1, #100) |
| 0x00008004 | `0xE3A02045` (MOV R2, #69) |
| 0x00008008 | `0xE1510002` (CMP R1, R2) |
| 0x0000800C | `0x25813024` (STRHS R3, [R1, #0x24]) |

### Unconditional Branching (`B`)

```
MOV R2, #17          ; R2 = 17
B   TARGET            ; branch to target
ORR R1, R1, #0x4      ; NOT executed (skipped by branch)

TARGET
SUB R1, R1, #78       ; R1 = R1 + 78
```

- **Labels** (like `TARGET`) mark an instruction's location in code.
- Labels cannot be reserved words (e.g., `ADD`, `ORR`, etc.).

### The Branch Not Taken

```
MOV R0, #4            ; R0 = 4
ADD R1, R0, R0         ; R1 = R0 + R0 = 8
CMP R0, R1              ; sets flags based on R0 - R1
BEQ THERE                ; branch NOT taken (Z=0, since R0 != R1)
ORR R1, R1, #1            ; R1 = R1 OR 1 = 9   (this DOES execute)
THERE
ADD R1, R1, 78             ; R1 = R1 + 78 = 87
```

- Since `R0 (4) != R1 (8)`, the `Z` flag is 0, so `BEQ` does not branch — execution falls through to the `ORR` instruction, then continues to `THERE`.

---

## Quick Reference Summary

| Category | Instructions |
|---|---|
| Move/Constants | `MOV`, `ORR` (for building large constants) |
| Logical | `AND`, `ORR`, `EOR`, `BIC`, `MVN` |
| Shifts | `LSL`, `LSR`, `ASR`, `ROR` |
| Memory | `LDR` (load), `STR` (store), `LDRB` (load byte) |
| Compare/Flags | `CMP`, instruction + `S` suffix (e.g., `ADDS`) |
| Branch | `B`, `BL`, plus conditional suffixes (e.g., `BEQ`, `BNE`, `BMI`) |

**Key formula:** Word address × 4 = byte address (e.g., word 10 → byte address 40)
