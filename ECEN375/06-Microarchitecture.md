# ECEN 375 — Unit 6: Microarchitecture — Single-Cycle Processor: Datapath

> **Course:** ECEN 375 Computer Architecture & Organization — Dr. Daniel Limbrick
> **Textbook:** *Digital Design and Computer Architecture: ARM Edition* (Harris & Harris), Ch. 7.1–7.3 (review Ch. 4 for SystemVerilog)

---

## 1. Introduction to Microarchitecture

**Microarchitecture** = *how* an architecture is implemented in hardware. It sits between the **Architecture** layer (the ISA the programmer sees) and the **Logic** layer (gates, adders, muxes).

A processor is split into two parts:

| Part | What it is |
|---|---|
| **Datapath** | The functional blocks that hold and move data: memories, register file, ALU, muxes, PC register |
| **Control** | The logic that reads the current instruction and produces the **control signals** that steer the datapath |

### One architecture, many implementations

| Implementation | Idea |
|---|---|
| **Single-cycle** | Every instruction executes in **one** (long) clock cycle |
| **Multicycle** | Each instruction is broken into a series of **shorter steps**, one step per cycle |
| **Pipelined** | Each instruction is broken into steps **and** multiple instructions are in flight at once |

All three run the same ARM programs; they differ in cost, speed, and complexity.

---

## 2. Processor Performance

$$
\text{Execution Time} = (\#\text{instructions}) \times \left(\frac{\text{cycles}}{\text{instruction}}\right) \times \left(\frac{\text{seconds}}{\text{cycle}}\right)
$$

| Term | Meaning |
|---|---|
| **CPI** | Cycles per instruction |
| **Clock period ($T_c$)** | Seconds per cycle ($= 1/f$) |
| **IPC** | Instructions per cycle $= 1/\text{CPI}$ |

The design challenge is balancing **cost, power, and performance**.

**Worked example:** 100 billion instructions, CPI = 1 (single-cycle), $T_c$ = 840 ps
→ $100\times10^9 \times 1 \times 840\times10^{-12}\text{ s} = 84\text{ s}$

> Single-cycle CPI is always **1**, but $T_c$ must be long enough for the **slowest** instruction (usually `LDR`), which is the core weakness of this design.

---

## 3. The ARM Subset We Build

| Category | Instructions | Restrictions |
|---|---|---|
| Data-processing | `ADD`, `SUB`, `AND`, `ORR` | Register or immediate `Src2`, **no shifts** |
| Memory | `LDR`, `STR` | **Positive immediate offset** only |
| Branch | `B` | — |

---

## 4. Architectural State Elements

The **architectural state** determines everything about the processor:

- **16 registers** (R0–R15, where **R15 = PC**)
- **Status register** (the NZCV flags)
- **Memory**

### State elements in hardware

| Element | Ports | Width | Behavior |
|---|---|---|---|
| **PC** | PC′ → PC, CLK | 32 bits | Register; loads PC′ on the rising clock edge |
| **Instruction Memory** | A → RD | 32-bit addr / 32-bit instr | Read-only, combinational read |
| **Register File** | A1, A2, A3 (4 bits each); RD1, RD2 (32); WD3 (32); WE3; R15 input; CLK | 16 × 32 | **2 combinational read ports**, **1 clocked write port** (writes when WE3 = 1). R15 is fed in separately |
| **Data Memory** | A, WD (32); RD (32); WE; CLK | 32 bits | Combinational read, clocked write when WE = 1 |
| **Status** | 4-bit flags | 4 bits | Clocked register holding NZCV |

> **Key timing idea:** reads are combinational (output follows the address right away); writes only happen on the clock edge. This is what lets a whole instruction finish in one cycle.

---

## 5. Hardware Description Languages (SystemVerilog Intro)

An **HDL** describes *logic function only*; a CAD tool then **synthesizes** the optimized gates. Most commercial designs are built with HDLs.

| HDL | History |
|---|---|
| **SystemVerilog** | Gateway Design Automation (1984) → IEEE 1364 (1995) → extended as IEEE 1800-2009 (2005/2009) |
| **VHDL 2008** | Dept. of Defense (1981) → IEEE 1076 (1987) → updated IEEE 1076-2008 |

### HDL → gates

- **Simulation:** apply inputs to the described circuit and check the outputs. Catching bugs here saves millions compared with debugging real hardware.
- **Synthesis:** turns HDL code into a **netlist**, a list of gates and the wires connecting them.

> **IMPORTANT:** When writing HDL, think about the **hardware** the code should produce. It is not software.

### Module types

- **Behavioral:** describes **what** a module does.
- **Structural:** describes **how** it is built from simpler modules.

### Behavioral example

```systemverilog
module example(input  logic a, b, c,
               output logic y);
  assign y = ~a & ~b & ~c | a & ~b & ~c | a & ~b & c;
endmodule
```

| Piece | Meaning |
|---|---|
| `module` / `endmodule` | Required to begin and end a module |
| `example` | Module name |
| `input logic` / `output logic` | Port declarations |
| `assign` | Continuous assignment (combinational logic) |
| `~` / `&` / `\|` | NOT / AND / OR |

**Truth table** (what the simulation waveform shows):

| a | b | c | y |
|:-:|:-:|:-:|:-:|
| 0 | 0 | 0 | **1** |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | **1** |
| 1 | 0 | 1 | **1** |
| 1 | 1 | 0 | 0 |
| 1 | 1 | 1 | 0 |

**Synthesized result:** the tool simplifies it to $y = \bar{b}\bar{c} + a\bar{b}$, which is **two AND gates feeding one OR gate**. That matches the synthesis schematic on the slides. You wrote three product terms; the tool found the minimal form.

> Tool for practice: **EDAPlayground.com** (free online SystemVerilog simulator).

---

## 6. Single-Cycle Datapath: Built One Instruction at a Time

The method: start with `LDR`, then keep adding hardware (muxes and wires) until every instruction in the subset works.

### 6.1 `LDR Rd, [Rn, imm12]`, the base case

Example: `LDR R1, [R2, #5]` → machine code `0xE5921005`

**Memory format (review):**

```
 31:28  27:26  25  24  23  22  21  20  19:16  15:12  11:0
 cond    op=01  Ī   P   U   B   W   L    Rn     Rd    Src2
                └──────── funct ───────┘
Ī = 0 → Src2 = imm12          Ī = 1 → Src2 = shamt5 | sh | 0 | Rm
```

| Step | What happens | Hardware added |
|---|---|---|
| **1. Fetch** | `Instr = IMem[PC]` | PC → Instruction Memory A; RD = `Instr` |
| **2. Read source operand** | `RD1 = RF[Rn]` | `Instr[19:16]` → A1 |
| **3. Extend immediate** | `ExtImm = ZeroExt(imm12)` | `Instr[11:0]` → **Extend** unit |
| **4. Compute address** | `ALUResult = RD1 + ExtImm` | RD1 → SrcA, ExtImm → SrcB, **ALU** (ALUControl = add) |
| **5. Read memory, write back** | `ReadData = DMem[ALUResult]`; `RF[Rd] = ReadData` | ALUResult → DMem A; ReadData → WD3; `Instr[15:12]` → A3; **RegWrite** = 1 |
| **6. Next PC** | `PC′ = PC + 4` | **PCPlus4** adder |

### 6.2 Access to the PC (R15)

The PC can be both a **source** and a **destination** of an instruction.

- **Source:** R15 must be readable from the register file. **Reading R15 returns PC + 8** (a leftover from ARM's original 3-stage pipeline). A second adder makes **PCPlus8 = PCPlus4 + 4** and feeds it into the R15 input of the RF.
- **Destination:** a result must be able to go into the PC. A mux controlled by **PCSrc** picks the next PC:

| PCSrc | PC′ |
|:-:|---|
| 0 | PCPlus4 (normal) |
| 1 | Result (branch, or an instruction that writes R15) |

### 6.3 `STR Rd, [Rn, imm12]`

Example: `STR R1, [R2, #5]` → `0xE5821005`

Same address calculation as `LDR`, but now **`Rd` is a source**: its value must be written to memory.

- Read `Rd` on the **second** RF read port: `Instr[15:12]` → A2 → RD2
- RD2 → Data Memory **WD** (this wire is labeled **WriteData**)
- **MemWrite** = 1 (DMem write enable); **RegWrite** = 0 (nothing goes back to the RF)

### 6.4 Data-processing with **immediate** `Src2`

Example: `ADD Rd, Rn, imm8`, e.g. `ADD R1, R2, #5` → `0xE2821005`

```
 31:28  27:26  25  24:21  20  19:16  15:12  11:8  7:0
 cond   op=00  I=1  cmd    S    Rn     Rd    rot   imm8
```

- Read `Rn` (RD1) and **imm8**. **ImmSrc** now tells the Extend unit to zero-extend `Imm8` instead of `Imm12`.
- **ALUResult** (not ReadData) gets written back to `Rd`. That needs a new mux:

| MemtoReg | Result |
|:-:|---|
| 0 | ALUResult (data-processing) |
| 1 | ReadData (`LDR`) |

- ALUControl now has to choose the operation (`ADD`/`SUB`/`AND`/`ORR`), and the ALU outputs **ALUFlags** (NZCV) for the Status register.

> The `rot` field is **ignored** in this simplified processor. Only plain 8-bit immediates are supported.

### 6.5 Data-processing with **register** `Src2`

Example: `ADD Rd, Rn, Rm`, e.g. `ADD R1, R2, R3` → `0xE0821003`

```
 I = 0 → Src2[11:0] = shamt5 | sh | 0 | Rm     (shifts not supported → Rm only)
```

- Read `Rn` **and `Rm`**: `Instr[3:0]` → A2. A2 now has two possible sources (`Rm` for data-processing, `Rd` for `STR`), so we add a mux controlled by **RegSrc[1]**.
- SrcB can now be RD2 **or** ExtImm, so we add a mux controlled by **ALUSrc**.

| ALUSrc | SrcB |
|:-:|---|
| 0 | RD2 (register) |
| 1 | ExtImm (immediate) |

### 6.6 Branch: `B Label`

```
 31:28  27:26  25:24  23:0
 cond   op=10   1L    imm24
```

**Branch Target Address:**

$$
\text{BTA} = \text{ExtImm} + (\text{PC} + 8), \qquad \text{ExtImm} = \text{SignExt}(\text{imm24}) \ll 2
$$

To compute this with the existing ALU:

- **SrcA = PC + 8**: read R15 by sending the constant **15** into A1, using a mux controlled by **RegSrc[0]**
- **SrcB = ExtImm** (ALUSrc = 1, ImmSrc = 10)
- ALU adds them, and the result goes through MemtoReg = 0 → Result → **PCSrc = 1** → PC′

**Worked example:** a `B` at address `0x8000` that should jump to `0x8010`
→ PC + 8 = `0x8008`; offset = `0x8010 − 0x8008 = 8` bytes = **2** words → imm24 = 2 → `0xEA000002`

### 6.7 The Extend unit (`ImmSrc`)

| ImmSrc₁:₀ | ExtImm | Used by |
|:-:|---|---|
| `00` | `{24'b0, Instr[7:0]}` | Zero-extended **imm8**, data-processing |
| `01` | `{20'b0, Instr[11:0]}` | Zero-extended **imm12**, `LDR`/`STR` |
| `10` | `{{6{Instr[23]}}, Instr[23:0], 2'b00}` | Sign-extended **imm24**, `B` |

> ⚠️ **Flag:** the slide's table lists the branch case as `{6{Instr23}, Instr23:0}`, which is only **30 bits**. The textbook version appends `2'b00` (the `<< 2` from the BTA formula) to make 32 bits. Use the textbook form: 6 + 24 + 2 = 32.

```systemverilog
module extend(input  logic [23:0] Instr,
              input  logic [1:0]  ImmSrc,
              output logic [31:0] ExtImm);
  always_comb
    case (ImmSrc)
      2'b00:   ExtImm = {24'b0, Instr[7:0]};
      2'b01:   ExtImm = {20'b0, Instr[11:0]};
      2'b10:   ExtImm = {{6{Instr[23]}}, Instr[23:0], 2'b00};
      default: ExtImm = 32'bx;
    endcase
endmodule
```

---

## 7. Complete Datapath: Mux and Control Signal Reference

```
PCSrc ─┐                    RegSrc[0]          RegSrc[1]
       ▼                        ▼                   ▼
PC′ ◄─[mux]◄─ PCPlus4 / Result  A1 ◄─[Rn | 15]      A2 ◄─[Rm | Rd]
 │                                                  A3 ◄─ Rd
 ▼                                                  WD3 ◄─ Result
PC ─► IMem ─► Instr ─► Register File ─► RD1 ─► SrcA ─┐
 │                     (R15 ◄─ PCPlus8)  RD2 ─┬─► [ALUSrc mux] ─► SrcB ─► ALU ─► ALUResult ─┬─► DMem A
 └─► +4 ─► PCPlus4 ─► +4 ─► PCPlus8           │        ▲                    │              │
                                              │      ExtImm ◄─ Extend ◄─ ImmSrc            │
                                              └──► WriteData ─► DMem WD (MemWrite)          │
                                                                ReadData ─┐  ALUResult ──┐  │
                                                                          ▼              ▼
                                                                     [MemtoReg mux] ─► Result
```

| Signal | Width | Selects / enables |
|---|:-:|---|
| **PCSrc** | 1 | 0 = PCPlus4, 1 = Result → PC′ |
| **RegSrc[0]** | 1 | A1: 0 = `Instr[19:16]` (Rn), 1 = **15** (read PC+8 for branches) |
| **RegSrc[1]** | 1 | A2: 0 = `Instr[3:0]` (Rm), 1 = `Instr[15:12]` (Rd, for `STR`) |
| **RegWrite** | 1 | RF write enable (WE3) |
| **ImmSrc** | 2 | Extend mode (table in §6.7) |
| **ALUSrc** | 1 | SrcB: 0 = RD2, 1 = ExtImm |
| **ALUControl** | 2 | `00` ADD, `01` SUB, `10` AND, `11` ORR |
| **MemWrite** | 1 | Data memory write enable |
| **MemtoReg** | 1 | Result: 0 = ALUResult, 1 = ReadData |
| **ALUFlags** | 4 | ALU output (NZCV) sent to the control unit |

### Control values per instruction (preview of the Control unit, next unit)

| Instr | RegSrc | ImmSrc | ALUSrc | ALUControl | MemWrite | MemtoReg | RegWrite | PCSrc |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| DP register | `00` | `XX` | 0 | by cmd | 0 | 0 | 1 | 0* |
| DP immediate | `X0` | `00` | 1 | by cmd | 0 | 0 | 1 | 0* |
| `LDR` | `X0` | `01` | 1 | `00` (add) | 0 | 1 | 1 | 0* |
| `STR` | `10` | `01` | 1 | `00` (add) | 1 | X | 0 | 0 |
| `B` | `X1` | `10` | 1 | `00` (add) | 0 | 0 | 0 | 1 |

\* PCSrc = 1 if the destination register is R15. All writes are also gated by the condition field (`cond`) checked against the flags.

---

## 8. Quick Review / Self-Check

1. **Which instruction sets the single-cycle clock period, and why?**
   `LDR`. It uses every major block in series: IMem → RF → ALU → DMem → RF write.
2. **Why does R15 read as PC + 8?**
   It is a holdover from the original ARM 3-stage pipeline. The datapath generates PCPlus8 to feed the R15 port.
3. **What does `STR` need that `LDR` doesn't?**
   A path from RD2 to DMem WD, `Rd` routed to A2 (RegSrc[1] = 1), and MemWrite = 1 with RegWrite = 0.
4. **Which mux makes `ADD` write ALUResult instead of ReadData?**
   MemtoReg (= 0).
5. **Encode `SUB R1, R2, #5`.**
   cmd = `0010`, I = 1 → `0xE2421005`
6. **A `B` at `0x1000` with imm24 = `0xFFFFFE`. Where does it go?**
   SignExt(−2) << 2 = −8 → BTA = `0x1008 − 8` = **`0x1000`** (branch to itself, an infinite loop)

---

## Key Takeaways

- Datapath = **what** moves data; Control = **who** steers it.
- The single-cycle design has CPI = 1, but $T_c$ is set by the slowest instruction.
- Build the datapath incrementally: `LDR` → PC access → `STR` → DP-imm → DP-reg → `B`. Each step adds a mux or a wire.
- Five muxes to know cold: **PCSrc, RegSrc[0], RegSrc[1], ALUSrc, MemtoReg**, plus the **ImmSrc** Extend modes.
- HDL code describes **hardware**: simulation checks behavior, and synthesis produces the netlist.
