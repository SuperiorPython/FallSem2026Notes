# ECEN 375 — Unit 7: Single-Cycle ARM Processor — Control

> **Course:** ECEN 375 Computer Architecture & Organization — Dr. Daniel Limbrick
> **Textbook:** *Digital Design and Computer Architecture: ARM Edition* (Harris & Harris), Ch. 7.1–7.3 (review Ch. 4)

---

## 1. Where Control Fits

Unit 6 built the **datapath**. This unit builds the **Control Unit**, the logic that reads the current instruction and produces every control signal the datapath needs.

### Control Unit inputs

| Input | Instruction bits | What it is |
|---|---|---|
| **Cond₃:₀** | `Instr[31:28]` | Condition field |
| **Op₁:₀** | `Instr[27:26]` | Instruction class (data-processing / memory / branch) |
| **Funct₅:₀** | `Instr[25:20]` | `I`, `cmd`, `S` (DP) or the memory-format bits (`I`, `P`, `U`, `B`, `W`, `L`) |
| **Rd₃:₀** | `Instr[15:12]` | Destination register (needed to detect writes to R15 = PC) |
| **ALUFlags₃:₀** | from the ALU | NZCV produced by the current instruction |

### Control Unit outputs (to the datapath)

`PCSrc`, `RegWrite`, `MemWrite`, `MemtoReg`, `ALUSrc`, `ImmSrc₁:₀`, `RegSrc₁:₀`, `ALUControl₁:₀`

---

## 2. Control Unit Structure

The Control Unit has **two blocks**:

| Block | Job |
|---|---|
| **Decoder** | Turns `Op`, `Funct`, `Rd` into the *raw* control signals |
| **Conditional Logic** | Checks the condition and flags, then decides whether the instruction is allowed to change state |

**Decoder outputs:** `FlagW₁:₀`, `PCS`, `RegW`, `MemW`, `MemtoReg`, `ALUSrc`, `ImmSrc₁:₀`, `RegSrc₁:₀`, `ALUControl₁:₀`

### Two routes to the datapath

| Route | Signals | Why |
|---|---|---|
| **Sent directly to datapath** | `MemtoReg`, `ALUSrc`, `ImmSrc`, `RegSrc`, `ALUControl` | These only steer muxes and the ALU; they change no state |
| **Sent through Conditional Logic first** | `PCSrc`, `RegWrite`, `MemWrite` | These **change state** (PC, register file, memory), so if the instruction shouldn't execute they are **forced to 0** |

> **Key idea:** a skipped (condition-false) instruction still flows through the datapath, but it must not be allowed to *write* anything. Gating the three state-changing signals is what makes that true.

### Decoder submodules

| Submodule | Produces |
|---|---|
| **Main Decoder** | `Branch`, `ALUOp`, `RegW`, `MemW`, `MemtoReg`, `ALUSrc`, `ImmSrc`, `RegSrc` |
| **ALU Decoder** | `ALUControl`, `FlagW` (uses `ALUOp` + `Funct₄:₀`) |
| **PC Logic** | `PCS` (uses `Rd`, `RegW`, `Branch`) |

---

## 3. Main Decoder

Inputs: `Op₁:₀`, `Funct₅`, `Funct₀`. Outputs: everything below.

| Op | Funct₅ | Funct₀ | Type | Branch | MemtoReg | MemW | ALUSrc | ImmSrc | RegW | RegSrc | ALUOp |
|:-:|:-:|:-:|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| `00` | 0 | X | DP Reg | 0 | 0 | 0 | 0 | `XX` | 1 | `00` | 1 |
| `00` | 1 | X | DP Imm | 0 | 0 | 0 | 1 | `00` | 1 | `X0` | 1 |
| `01` | X | 0 | `STR` | 0 | X | 1 | 1 | `01` | 0 | `10` | 0 |
| `01` | X | 1 | `LDR` | 0 | 1 | 0 | 1 | `01` | 1 | `X0` | 0 |
| `10`* | X | X | `B` | 1 | 0 | 0 | 1 | `10` | 0 | `X1` | 0 |

\* ⚠️ **Flag:** the slide's table lists `B` with Op = `11`. Branch is **Op = `10`** (bits 27:25 = `101`), which matches the branch encoding in Unit 5/6 and the textbook. Use `10`.

### How to read the table

- **Funct₅ = the `I` bit** for data-processing: 0 = register `Src2`, 1 = immediate `Src2`.
- **Funct₀ = `S` (DP) or `L` (memory):** for memory instructions, `L = 0` is `STR`, `L = 1` is `LDR`.
- **`ALUOp`** is an internal signal to the ALU Decoder: **1** = data-processing (look at `cmd`), **0** = not DP (ALU just **adds**, for addresses and branch targets).
- **X** = don't care.

### Why each value makes sense

| Signal | Reasoning |
|---|---|
| `MemtoReg` | Only `LDR` writes **ReadData** back; DP writes ALUResult. `STR` writes nothing, so X |
| `ALUSrc` | 1 whenever SrcB is an immediate (DP Imm, `LDR`, `STR`, `B`); 0 for DP Reg |
| `ImmSrc` | `00` imm8 (DP), `01` imm12 (mem), `10` imm24 (branch); `XX` for DP Reg (no immediate) |
| `RegW` | 1 for DP and `LDR` (they write a register); 0 for `STR` and `B` |
| `RegSrc` | `RegSrc₁` = 1 only for `STR` (read Rd on A2); `RegSrc₀` = 1 only for `B` (read R15 on A1) |
| `MemW` | 1 only for `STR` |

---

## 4. ALU Decoder

### Review: the ALU

| ALUControl₁:₀ | Function |
|:-:|---|
| `00` | Add |
| `01` | Subtract |
| `10` | AND |
| `11` | OR |

The ALU also outputs **ALUFlags = {N, Z, C, V}** (Negative, Zero, Carry, oVerflow).

### ALU Decoder table

| ALUOp | Funct₄:₁ (`cmd`) | Funct₀ (`S`) | Type | ALUControl₁:₀ | FlagW₁:₀ |
|:-:|:-:|:-:|---|:-:|:-:|
| 0 | X | X | Not DP | `00` | `00` |
| 1 | `0100` | 0 | `ADD` | `00` | `00` |
| 1 | `0100` | 1 | `ADDS` | `00` | `11` |
| 1 | `0010` | 0 | `SUB` | `01` | `00` |
| 1 | `0010` | 1 | `SUBS` | `01` | `11` |
| 1 | `0000` | 0 | `AND` | `10` | `00` |
| 1 | `0000` | 1 | `ANDS` | `10` | `10` |
| 1 | `1100` | 0 | `ORR` | `11` | `00` |
| 1 | `1100` | 1 | `ORRS` | `11` | `10` |

### What `FlagW₁:₀` means

`FlagW` says **which flags should be saved** if the instruction runs and has `S = 1`.

| Bit | When 1 | Flags saved |
|---|---|---|
| **FlagW₁** | **NZ** saved | `ALUFlags₃:₂` |
| **FlagW₀** | **CV** saved | `ALUFlags₁:₀` |

- `ADD`/`SUB` update **all four** flags (NZCV) → `FlagW = 11`
- `AND`/`ORR` update **only NZ** → `FlagW = 10`
- `S = 0` → `FlagW = 00` (no flags change)

> **Textbook form of the same logic:** `FlagW₁ = S` and `FlagW₀ = S & (ALUControl == 00 or 01)`. That is just "S set, and the operation is an add or subtract."

---

## 5. PC Logic

`PCS = 1` means **the PC is being written by this instruction** (either a normal write to R15 or a branch):

$$
PCS = \big((Rd == 15)\ \&\ RegW\big)\ |\ Branch
$$

| Case | Example |
|---|---|
| Writes R15 through the register file | `ADD PC, R1, R2` or `LDR PC, [R1]` (`Rd == 15` and `RegW = 1`) |
| Branch | `B Label` (`Branch = 1`) |

Then the Conditional Logic applies the condition:

- **If the instruction is executed:** `PCSrc = PCS`
- **Else:** `PCSrc = 0` (so `PC = PC + 4`)

---

## 6. Conditional Logic

### Function

1. **Check if the instruction should execute.** If not, force `PCSrc`, `RegWrite`, and `MemWrite` to 0.
2. **Possibly update the Status Register** (`Flags₃:₀` = NZCV).

### Condition mnemonics (`Cond₃:₀` → `CondEx`)

| Cond₃:₀ | Mnemonic | Name | CondEx |
|:-:|:-:|---|---|
| `0000` | EQ | Equal | $Z$ |
| `0001` | NE | Not equal | $\bar{Z}$ |
| `0010` | CS / HS | Carry set / unsigned higher or same | $C$ |
| `0011` | CC / LO | Carry clear / unsigned lower | $\bar{C}$ |
| `0100` | MI | Minus / negative | $N$ |
| `0101` | PL | Plus / positive or zero | $\bar{N}$ |
| `0110` | VS | Overflow / overflow set | $V$ |
| `0111` | VC | No overflow / overflow clear | $\bar{V}$ |
| `1000` | HI | Unsigned higher | $\bar{Z}C$ |
| `1001` | LS | Unsigned lower or same | $Z \text{ OR } \bar{C}$ |
| `1010` | GE | Signed greater than or equal | $\overline{N \oplus V}$ |
| `1011` | LT | Signed less than | $N \oplus V$ |
| `1100` | GT | Signed greater than | $\bar{Z}\,\overline{(N \oplus V)}$ |
| `1101` | LE | Signed less than or equal | $Z \text{ OR } (N \oplus V)$ |
| `1110` | AL (or none) | Always / unconditional | ignored (CondEx = 1) |

The **Condition Check** block takes `Cond₃:₀` and the saved flags `Flags₃:₀` and outputs **CondEx** (1 = execute).

> **`Flags₃:₀` is the Status Register.** It is **separate** from `ALUFlags`: `ALUFlags` is the ALU's output for *this* instruction, while `Flags` holds the saved result of a *previous* flag-setting instruction. The condition check reads `Flags`.

### Gating the state-changing signals

$$
PCSrc = PCS \cdot CondEx \qquad RegWrite = RegW \cdot CondEx \qquad MemWrite = MemW \cdot CondEx
$$

### Updating the flags

`Flags₃:₀` is **updated with `ALUFlags₃:₀`** only if:

- **`FlagW` is 1** (the instruction's S-bit is 1) **AND**
- **`CondEx` is 1** (the instruction should be executed)

$$
FlagWrite_{1:0} = FlagW_{1:0} \cdot CondEx
$$

The two flag registers are written independently:

| Register | Holds | Clocked write when |
|---|---|---|
| `Flags₃:₂` | N, Z | `FlagWrite₁ = 1` |
| `Flags₁:₀` | C, V | `FlagWrite₀ = 1` |

### Examples

| Instruction | Cond | CondEx | FlagW | FlagWrite | Result |
|---|---|---|---|---|---|
| `AND R1, R2, R3` | `1110` (AL) | 1 | `00` | `00` | Executes, no flags change |
| `EOREQ R5, R6, R7` | `0000` (EQ) | 1 **if Z = 1** | | | Executes only when the Z flag is set |
| `SUBS R5, R6, R7` | `1110` (AL) | 1 | `11` | `11` | **All** flags (NZCV) updated |
| `ANDS R7, R1, R3` | `1110` (AL) | 1 | `10` | `10` | **Only NZ** (`Flags₃:₂`) updated |

> ⚠️ **Flag:** the slide for `EOREQ` writes "if `Flags₃:₂` = `0100` ⇒ CondEx = 1." `Flags₃:₂` is only **2 bits** (N, Z). `0100` is the full 4-bit **NZCV** value with Z = 1. The condition is simply **Z = 1**. Also, `EOR` (`cmd = 0001`) isn't in the ALU Decoder table above, so the slide uses it only to illustrate the EQ condition.

---

## 7. Worked Example: `ORR` (Data-Processing, Register)

`ORR R1, R2, R3` (assume Cond = AL, S = 0)

| Decoder step | Value |
|---|---|
| Op | `00` → data-processing |
| Funct₅ (`I`) | 0 → **DP Reg** row |
| Main Decoder | Branch = 0, MemtoReg = 0, MemW = 0, ALUSrc = 0, ImmSrc = `XX`, RegW = 1, RegSrc = `00`, ALUOp = 1 |
| ALU Decoder (`cmd = 1100`) | ALUControl = `11` (OR), FlagW = `00` |
| PC Logic | `Rd ≠ 15`, Branch = 0 → PCS = 0 |
| Conditional Logic | CondEx = 1 → RegWrite = 1, PCSrc = 0, MemWrite = 0 |

**Data flow:** `Rn` → RD1 → SrcA; `Rm` → RD2 → SrcB (ALUSrc = 0); ALU ORs them; ALUResult → Result (MemtoReg = 0) → WD3, written to `Rd` since RegWrite = 1; `PC′ = PC + 4`.

---

## 8. Extended Functionality

### 8.1 `CMP` (compare)

`CMP` subtracts and **sets flags**, but **does not write a result** to any register.

**No change to the datapath.** Only the controller changes:

- Decoder gains a new output: **`NoWrite`**
- `NoWrite` is routed to the Conditional Logic and **blocks `RegWrite`**

$$
RegWrite = RegW \cdot \overline{NoWrite} \cdot CondEx
$$

New row in the ALU Decoder:

| ALUOp | Funct₄:₁ (`cmd`) | Funct₀ (`S`) | Type | ALUControl₁:₀ | FlagW₁:₀ | **NoWrite** |
|:-:|:-:|:-:|---|:-:|:-:|:-:|
| 1 | `1010` | 1 | `CMP` | `01` (subtract) | `11` | **1** |

- All existing rows get `NoWrite = 0`.
- `CMP` always has `S = 1` (setting flags is its whole purpose).

### 8.2 Shifted register `Src2`

**No change to the controller.** The datapath only gains a **Shift** unit between RD2 and the SrcB mux.

Example: `ADD R7, R2, R12, LSR #5`

| Field | cond | op | I | cmd | S | rn | rd | shamt5 | sh | bit 4 | rm |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Bits | 31:28 | 27:26 | 25 | 24:21 | 20 | 19:16 | 15:12 | 11:7 | 6:5 | 4 | 3:0 |
| Value | 14 | 0 | 0 | 4 | 0 | 2 | 7 | 5 | `01` | 0 | 12 |

(`sh = 01` is `LSR`.) Assembled, this is `0xE08272AC`.

> The Shift unit reads `Instr[11:5]` (shamt5 and sh) directly. The Main Decoder and ALU Decoder see the same `Op`/`Funct` as an ordinary DP-register instruction, which is why the controller is unchanged.

---

## 9. Full Control Signal Summary

| Instr | Branch | ALUOp | RegW | MemW | MemtoReg | ALUSrc | ImmSrc | RegSrc | ALUControl | PCS |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| DP Reg | 0 | 1 | 1 | 0 | 0 | 0 | `XX` | `00` | by `cmd` | 1 if `Rd = 15` |
| DP Imm | 0 | 1 | 1 | 0 | 0 | 1 | `00` | `X0` | by `cmd` | 1 if `Rd = 15` |
| `STR` | 0 | 0 | 0 | 1 | X | 1 | `01` | `10` | `00` | 0 |
| `LDR` | 0 | 0 | 1 | 0 | 1 | 1 | `01` | `X0` | `00` | 1 if `Rd = 15` |
| `B` | 1 | 0 | 0 | 0 | 0 | 1 | `10` | `X1` | `00` | 1 |

All writes (`RegWrite`, `MemWrite`, `PCSrc`) are additionally ANDed with **CondEx**.

---

## 10. Quick Review / Self-Check

1. **Which control signals pass through the Conditional Logic, and why?**
   `PCSrc`, `RegWrite`, `MemWrite`. They change state (PC, RF, memory), so they must be forced to 0 when the instruction's condition fails.
2. **What does `FlagW = 10` mean, and which instructions produce it?**
   Only NZ (`Flags₃:₂`) get saved. It is produced by `ANDS` and `ORRS`. `ADDS`/`SUBS` give `11`.
3. **`SUBS R5, R6, R7` (Cond = AL). Give ALUOp, ALUControl, FlagW, FlagWrite.**
   ALUOp = 1, ALUControl = `01`, FlagW = `11`, FlagWrite = `11` (CondEx = 1).
4. **`ADDEQ R1, R2, R3` when Z = 0. What happens?**
   CondEx = 0, so RegWrite = 0, MemWrite = 0, PCSrc = 0. The instruction does nothing and PC advances by 4.
5. **Control values for `STR R1, [R2, #5]`?**
   Branch = 0, MemtoReg = X, MemW = 1, ALUSrc = 1, ImmSrc = `01`, RegW = 0, RegSrc = `10`, ALUOp = 0 → ALUControl = `00`, PCS = 0.
6. **When is `PCS = 1` for a non-branch instruction?**
   When it writes R15: `Rd = 15` and `RegW = 1` (e.g. `LDR PC, [R1]`).
7. **What changes in the hardware to add `CMP`?**
   Only the controller: a `NoWrite` signal from the ALU Decoder that blocks `RegWrite`. No datapath change.
8. **What changes to add shifted-register `Src2`?**
   Only the datapath (a Shift unit before SrcB). The controller is unchanged.

---

## Key Takeaways

- **Control = Decoder + Conditional Logic.** The decoder produces raw signals from `Op`/`Funct`/`Rd`; the Conditional Logic decides whether state may change.
- Only **`PCSrc`, `RegWrite`, `MemWrite`** are gated by `CondEx`. The other five signals go straight to the datapath.
- The **Main Decoder** is a lookup on `Op`, `Funct₅`, `Funct₀`; its internal `ALUOp` tells the **ALU Decoder** whether to look at `cmd`.
- **`FlagW`**: `ADD`/`SUB` → `11` (NZCV), `AND`/`ORR` → `10` (NZ only), `S = 0` → `00`.
- **`PCS = ((Rd == 15) & RegW) | Branch`**, then `PCSrc = PCS & CondEx`.
- `ALUFlags` (this instruction's result) ≠ `Flags` (saved status register). Conditions read `Flags`.
- **`CMP`** = controller change only (`NoWrite`). **Shifted register** = datapath change only (Shift unit).

---

## Slide Corrections Flagged in These Notes

| Slide | Issue | Used in notes |
|---|---|---|
| 15 (Main Decoder) | `B` listed with Op = `11` | Op = **`10`** |
| 32 (`EOREQ`) | "`Flags₃:₂` = `0100`" (2-bit field shown with 4 bits) | Condition is just **Z = 1** |
| 30 (Condition table) | "PL: Plus / Positive **of** zero" | "positive **or** zero" |
