# Unit 4: Machine Language — Basic Constructs

**Reading:** Textbook Ch. 6.4–6.5 | **HW:** 6.14, 6.26, 6.30, 6.32 | **Lab 3**

## Why Machine Language

Two design principles drive the encoding:

- **Regularity supports design simplicity** — all instructions are 32 bits (same as data), which would ideally mean one instruction format
- **Good design demands good compromises** — but different instructions need different operands (ADD/SUB use 3 registers; LDR/STR use 2 registers + a constant), so ARM keeps a **small number of formats** instead of forcing one

Computers only understand 1s and 0s. Every instruction is a 32-bit binary word, split into three formats:

| Format | `op` |
|---|---|
| Data-processing | `00` |
| Memory | `01` |
| Branch | `10` |

`op` (bits 27:26) is always the first thing you read — it tells you how to parse the rest of the instruction.

---

## 1. Data-Processing Format

```
31:28  27:26  25  24:21  20  19:16  15:12  11:0
cond    op    I   cmd    S    Rn     Rd    Src2
 4b     2b    1b   4b    1b   4b     4b    12b
```

- **cond** — condition for execution (see Conditional Execution below)
- **op** = `00`
- **I** — 1 if Src2 is an immediate, 0 if Src2 is a register (shifted or not)
- **cmd** — the specific operation (opcode within data-processing)
- **S** — 1 if the instruction sets condition flags (e.g. `ADDS`, `CMP`)
- **Rn** — first source register
- **Rd** — destination register
- **Src2** — second source operand; its own internal layout depends on `I`

### cmd values (examples)

| Instruction | cmd |
|---|---|
| EOR | `0001` |
| SUB | `0010` |
| ADD | `0100` |
| ORR | `1100` |
| shift/MOV-type (LSL/LSR/ASR/ROR) | `1101` |

### Src2 has three sub-encodings

```
Immediate:              rot(4b)         imm8(8b)     [11:8][7:0]
Register:               shamt5(5b)  sh(2b) 0  Rm(4b) [11:7][6:5][4][3:0]
Register-shifted reg:   Rs(4b)  0  sh(2b) 1  Rm(4b)  [11:8][7][6:5][4][3:0]
```

**Immediate** (I = 1): `imm8` is an 8-bit unsigned value, `rot` is a 4-bit rotation.
32-bit constant = **imm8 ROR (rot × 2)**. Useful trick: `ROR by X = ROL by (32 − X)`.

**Register** (I = 0, bit 4 = 0): `Rm` optionally shifted by a fixed amount `shamt5`, shift type given by `sh`.

**Register-shifted register** (I = 0, bit 4 = 1): `Rm` shifted by an amount stored in register `Rs` instead of a fixed constant.

### Shift type encoding (`sh`)

| Shift | sh |
|---|---|
| LSL | `00` |
| LSR | `01` |
| ASR | `10` |
| ROR | `11` |

### Worked example — immediate Src2

`ADD R0, R1, #42`
cond=`1110`(AL) · op=`00` · cmd=`0100`(ADD) · I=1 · Rn=1 · Rd=0 · imm8=42, rot=0
→ **0xE281002A**

`SUB R2, R3, #0xFF0`
imm8 must ROR into `0xFF0`. Rotating right by 28 (= ROL by 4) puts `0xFF` at the top: imm8=`0xFF`, rot=14.
cond=`1110` · op=`00` · cmd=`0010`(SUB) · I=1 · Rn=3 · Rd=2 · rot=14, imm8=255
→ **0xE2432EFF**

### Worked example — register Src2 (unshifted)

`ADD R5, R6, R7` → I=0, Rn=6, Rd=5, shamt5=0, sh=00, Rm=7
→ **0xE0865007**

### Worked example — register Src2 (immediate-shifted)

`ORR R9, R5, R3, LSR #2` → R9 = R5 OR (R3 >> 2)
cmd=`1100`(ORR), Rn=5, Rd=9, shamt5=2, sh=`01`(LSR), Rm=3
→ **0xE1859123**

### Worked example — register-shifted register Src2

`EOR R8, R9, R10, ROR R12` → R8 = R9 XOR (R10 ROR R12)
cmd=`0001`(EOR), Rn=9, Rd=8, Rs=12, sh=`11`(ROR), Rm=10
→ **0xE0298C7A**

### Shift instructions are data-processing instructions

`cmd = 1101` for all of LSL/LSR/ASR/ROR (and MOV). `Rn` is unused/ignored.

`ROR R1, R2, #23` (immediate shamt) → uses the immediate-shifted-register Src2 encoding, I=0
→ **0xE1A01BE2**

`ASR R5, R6, R10` (register shamt, `R5 = R6 >>> R10[7:0]`) → uses register-shifted-register Src2 encoding
→ **0xE1A05A56**

---

## 2. Memory Format

Encodes `LDR`, `STR`, `LDRB`, `STRB`.

```
31:28  27:26  25  24  23  22  21  20  19:16  15:12  11:0
cond    op    T̄   P   U   B   W   L    Rn     Rd    Src2
 4b     2b    1b  1b  1b  1b  1b  1b   4b     4b    12b
```

- **op** = `01`
- **Rn** — base register
- **Rd** — destination (load) or source (store)
- **Src2** — the offset (immediate or optionally-shifted register)
- **funct** bits: **T̄** (immediate-bar — 0 = immediate offset, 1 = register offset), **P** (preindex), **U** (add/subtract offset from base), **B** (byte access), **W** (writeback), **L** (load/store)

### Src2 layout for Memory

```
Immediate (T̄=0):   imm12 (12 bits)                          [11:0]
Register  (T̄=1):   shamt5(5b)  sh(2b) 0  Rm(4b)             [11:7][6:5][4][3:0]
```
(No register-shifted-register option for memory offsets — only plain or shifted register.)

### funct bit meanings

| L | B | Instruction |
|---|---|---|
| 0 | 0 | STR |
| 0 | 1 | STRB |
| 1 | 0 | LDR |
| 1 | 1 | LDRB |

| P | W | Indexing Mode |
|---|---|---|
| 0 | 1 | *not supported* |
| 0 | 0 | Postindex |
| 1 | 0 | Offset (no writeback) |
| 1 | 1 | Preindex |

| T̄ | Meaning |
|---|---|
| 0 | imm12 is the offset |
| 1 | register (optionally shifted) is the offset |

**U**: 1 = add offset to base, 0 = subtract offset from base.

### Indexing modes (recall)

| Mode | Address used | Base register after |
|---|---|---|
| Offset | base ± offset | unchanged |
| Preindex | base ± offset | base ± offset (writeback) |
| Postindex | base | base ± offset (writeback) |

```
Offset:    LDR R1, [R2, #4]      ; R1 = mem[R2+4]
Preindex:  LDR R3, [R5, #16]!    ; R3 = mem[R5+16]; R5 = R5+16
Postindex: LDR R8, [R1], #8      ; R8 = mem[R1];    R1 = R1+8
```

### Worked example — immediate offset, postindex

`STR R11, [R5], #-26` → mem[R5] ← R11; R5 = R5 − 26
T̄=0 (immediate), P=0 (postindex), U=0 (subtract), B=0 (word), W=0, L=0 (store)
Rd=11, Rn=5, imm12=26
→ **0xE405B01A**

### Worked example — register offset

`LDR R3, [R4, R5]` → R3 = mem[R4 + R5]
T̄=1 (register), P=1 (offset mode), U=1 (add), B=0 (word), W=0, L=1 (load); shamt5=0, sh=00
Rd=3, Rn=4, Rm=5
→ **0xE7943005**

### Worked example — scaled register offset

`STR R9, [R1, R3, LSL #2]` → mem[R1 + (R3 << 2)] ← R9
T̄=1, P=1, U=1, B=0, W=0, L=0 (store); shamt5=2, sh=`00`(LSL)
Rd=9, Rn=1, Rm=3
→ **0xE7819103**

---

## 3. Branch Format

Encodes `B` and `BL`.

```
31:28  27:26  25:24  23:0
cond    op     1L    imm24
 4b     2b     2b     24b
```

- **op** = `10`
- **funct** = `1L` — L=1 for `BL`, L=0 for `B`
- **imm24** — signed 24-bit immediate

### Branch Target Address (BTA)

- BTA is relative to **current PC + 8** (pipeline effect)
- `imm24` = number of **words** (not bytes) the BTA is away from PC+8
- To decode: **PC_next = (PC+8) + SignExtend(imm24) × 4**

### Worked example — forward branch (positive offset)

```
0xA0        BLT THERE     <- PC
0xA4        ADD R0, R1, R2
0xA8        SUB R0, R0, R9   <- PC+8
0xAC        ADD SP, SP, #8
0xB0        MOV PC, LR
0xB4  THERE SUB R0, R0, #1   <- BTA
0xB8        BL  TEST
```
PC=0xA0, PC+8=0xA8. `THERE` is 3 instructions past PC+8 → imm24 = 3
cond = `1011` (LT), op=`10`, funct=`10`(B) → **0xBA000003**

### Worked example — backward branch (negative offset)

```
0x8040 TEST LDRB R5, [R0, R3]  <- BTA
0x8044      STRB R5, [R1, R3]
0x8048      ADD  R3, R3, #1
0x804C      MOV  PC, LR
0x8050      BL   TEST          <- PC
0x8054      LDR  R3, [R1], #4  <- PC+8
0x8058      SUB  R4, R3, #9
```
PC=0x8050, PC+8=0x8058. `TEST` is 6 instructions **before** PC+8 → imm24 = **−6** (two's complement, 24-bit: `1111 1111 1111 1111 1111 1010`)
cond=`1110`(AL), op=`10`, funct=`11`(BL) → **0xEBFFFFFA**

---

## 4. Interpreting Machine Code (Decoding Strategy)

1. Start with **op** (bits 27:26) — tells you which format to parse.
   - `00` → data-processing
   - `01` → memory
   - `10` → branch
2. **Data-processing:** check **I** (bit 25). If I=0, check bit 4 to tell register (0) vs register-shifted-register (1) Src2. Then read `cmd` for the operation.
3. **Memory:** examine the funct bits (T̄, P, U, B, W, L) for indexing mode, instruction type, and add/subtract.
4. **Branch:** funct bit L tells B vs BL; imm24 gives the target offset.

### Decode example 1

`0xE0475001`
- op = `00` → data-processing
- I = 0 → Src2 is a register
- bit 4 = 0 → plain (optionally shifted) register, not register-shifted-register
- cmd = `0010` → SUB
- Rn=7, Rd=5, shamt5=0, sh=00, Rm=1

→ **SUB R5, R7, R1**

### Decode example 2

`0xE5949010`
- op = `01` → memory
- funct: B=0, L=1 → LDR; P=1, W=0 → offset indexing (no writeback); T̄=0 → immediate offset; U=1 → add
- Rn=4, Rd=9, imm12=16

→ **LDR R9, [R4, #16]**

---

## 5. Conditional Execution

Every instruction's top 4 bits (`cond`) gate whether it executes, based on the NZCV flags.

```
ANDEQ R1, R2, R3   ; cond = 0000
ORRMI R4, R5, #0xF ; cond = 0100
SUBLT R9, R3, R8    ; cond = 1011
```

| cond | Mnemonic | Meaning | Condition (flags) |
|---|---|---|---|
| 0000 | EQ | Equal | Z |
| 0001 | NE | Not equal | Z̄ |
| 0010 | CS/HS | Carry set / unsigned ≥ | C |
| 0011 | CC/LO | Carry clear / unsigned < | C̄ |
| 0100 | MI | Minus/negative | N |
| 0101 | PL | Plus/positive or zero | N̄ |
| 0110 | VS | Overflow set | V |
| 0111 | VC | No overflow | V̄ |
| 1000 | HI | Unsigned higher | Z̄·C |
| 1001 | LS | Unsigned lower or same | Z + C̄ |
| 1010 | GE | Signed ≥ | N⊕V (equal) |
| 1011 | LT | Signed < | N⊕V (differ) |
| 1100 | GT | Signed > | Z̄·(N⊕V) |
| 1101 | LE | Signed ≤ | Z + (N⊕V) |
| 1110 | AL | Always (unconditional) | ignored |

---

## 6. Addressing Modes (Summary)

How operands are located, across all three formats:

| Mode | Used by | Submodes |
|---|---|---|
| **Register** | Data-processing | register-only, immediate-shifted register, register-shifted register |
| **Immediate** | Data-processing | `imm8 ROR (rot × 2)` |
| **Base** | Memory | base ± (immediate12 \| register \| immediate-shifted register) |
| **PC-relative** | Branch | `PC_next = (PC+8) + SignExtend(imm24) × 4` |

### Register addressing examples

```
Register-only:            ADD R0, R2, R7
Immediate-shifted reg:    ORR R5, R1, R3, LSL #1
Register-shifted reg:     SUB R12, R9, R0, ASR R1
```

### Base addressing examples

```
Immediate offset:               LDR R0, [R8, #-11]     ; R0 = mem[R8-11]
Register offset:                LDR R1, [R7, R9]       ; R1 = mem[R7+R9]
Immediate-shifted reg offset:   STR R5, [R3, R2, LSL #4]  ; mem[R3+(R2<<4)] = R5
```

### More offset examples (base register always fixed)

| ARM Assembly | Memory Address |
|---|---|
| `LDR R0, [R3, #4]` | R3 + 4 |
| `LDR R0, [R5, #-16]` | R5 − 16 |
| `LDR R1, [R6, R7]` | R6 + R7 |
| `LDR R2, [R8, -R9]` | R8 − R9 |
| `LDR R3, [R10, R11, LSL #2]` | R10 + (R11 << 2) |
| `LDR R4, [R1, -R12, ASR #4]` | R1 − (R12 >>> 4) |
| `LDR R0, [R9]` | R9 |

---

## 7. The Stored Program

- Both **instructions and data** are 32-bit words stored in the **same memory**
- The only difference between two programs is the sequence of instructions stored — no rewiring needed to run a new program
- **Program Counter (PC)** tracks the address of the current instruction
- Execution cycle: processor **fetches** the instruction at PC, then **executes** it

```
Assembly Code         Machine Code
MOV R1, #100          0xE3A01064
MOV R2, #69           0xE3A02045
ADD R3, R1, R2         0xE2813002
STR R3, [R1]           0xE5913000
```

```
Address       Instructions
0x0000000C    E5913000
0x00000008    E2813002
0x00000004    E3A02045
0x00000000    E3A01064   <- PC
```

**Up next:** Microarchitecture — how to implement the ARM ISA in hardware.
