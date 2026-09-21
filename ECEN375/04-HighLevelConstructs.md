# ECEN 375 — Unit 3: Assembly — High-Level Constructs

Source: Dr. Daniel Limbrick, Unit 3 slides (Spring 2022). Textbook: *Digital Design and Computer Architecture: ARM Edition* (Harris & Harris), Ch. 6.3–6.4.

## Overview

High-level C constructs don't exist natively in ARM assembly — they're built out of the basic building blocks (data-processing instructions, conditional execution, branches). This unit covers translating:

- `if`/`else` statements
- `while` loops
- `for` loops
- arrays
- function calls (and the stack)

---

## if Statement

**C code:**
```c
if (i == j)
    f = g + h;
f = f - i;
```

**ARM translation** (`R0=f, R1=g, R2=h, R3=i, R4=j`):
```asm
        CMP  R3, R4     ; set flags with R3-R4
        BNE  L1         ; if i!=j, skip if block
        ADD  R0, R1, R2 ; f = g + h
L1
        SUB  R0, R0, R3 ; f = f - i
```

**Key idea:** assembly tests the *opposite* case of the high-level condition. The C code checks `i == j`; the assembly branches away (`BNE`) when the condition is *false*, so the `if` body only executes when the branch is skipped.

### Alternate form — conditional execution

For short `if` blocks, use a condition suffix instead of a branch:
```asm
        CMP    R3, R4       ; set flags with R3-R4
        ADDEQ  R0, R1, R2   ; if (i==j) f = g + h
        SUB    R0, R0, R3   ; f = f - i
```
`ADDEQ` only executes if the `EQ` condition (Z=1, i.e. `i==j`) holds — no branch needed at all. Useful for **short** conditional blocks; avoids the pipeline cost of a branch.

---

## if/else Statement

**C code:**
```c
if (i == j)
    f = g + h;
else
    f = f - i;
```

**ARM translation (branch form):**
```asm
        CMP  R3, R4     ; set flags with R3-R4
        BNE  L1         ; if i!=j, skip if block
        ADD  R0, R1, R2 ; f = g + h
        B    L2         ; branch past else block
L1
        SUB  R0, R0, R3 ; else f = f - i
L2
```
Note the extra unconditional `B L2` after the `if` body — without it, execution would fall through into the `else` block.

**Alternate form — conditional execution:**
```asm
        CMP    R3, R4       ; set flags with R3-R4
        ADDEQ  R0, R1, R2   ; if (i==j) f = g + h
        SUBNE  R0, R0, R3   ; else f = f - i
```
`ADDEQ` runs when `i==j`; `SUBNE` runs when `i!=j` — complementary conditions replace both branches entirely.

---

## while Loops

**C code:**
```c
// determines the power of x such that 2^x = 128
int pow = 1;
int x = 0;
while (pow != 128) {
    pow = pow * 2;
    x = x + 1;
}
```

**ARM translation** (`R0 = pow, R1 = x`):
```asm
        MOV  R0, #1     ; pow = 1
        MOV  R1, #0     ; x = 0
WHILE
        CMP  R0, #128   ; R0-128
        BEQ  DONE       ; if (pow==128) exit loop
        LSL  R0, R0, #1 ; pow = pow*2
        ADD  R1, R1, #1 ; x = x+1
        B    WHILE      ; repeat loop
DONE
```

**Key idea:** same opposite-case pattern as `if` — the loop guard checks `pow == 128` (the exit condition) even though the C code's condition is `pow != 128`.

---

## for Loops

General form:
```c
for (initialization; condition; loop operation)
    statement
```
- **initialization** — executes once, before the loop begins
- **condition** — tested at the *beginning* of each iteration
- **loop operation** — executes at the *end* of each iteration
- **statement** — executes each time the condition holds

**C code:**
```c
// adds numbers from 1-9
int sum = 0;
for (i=1; i!=10; i=i+1)
    sum = sum + i;
```

**ARM translation** (`R0 = i, R1 = sum`):
```asm
        MOV  R0, #1     ; i = 1
        MOV  R1, #0     ; sum = 0
FOR
        CMP  R0, #10    ; R0-10
        BEQ  DONE       ; if (i==10) exit loop
        ADD  R1, R1, R0 ; sum = sum + i
        ADD  R0, R0, #1 ; i = i + 1
        B    FOR        ; repeat loop
DONE
```

### Decremented loops (optimization)

**In ARM, counting a loop *down* to zero is more efficient than counting up**, because `SUBS` sets the flags as a side effect of decrementing — no separate `CMP` instruction is needed.

**C code (same result, decremented form):**
```c
int sum = 0;
for (i=9; i!=0; i=i-1)
    sum = sum + i;
```

**ARM translation:**
```asm
        MOV   R0, #9     ; i = 9
        MOV   R1, #0     ; sum = 0
FOR
        ADD   R1, R1, R0 ; sum = sum + i
        SUBS  R0, R0, #1 ; i = i - 1  (and set flags)
        BNE   FOR        ; if (i!=0) repeat loop
```

**Why it saves instructions:**
- `SUBS R0, R0, #1` decrements *and* sets flags in one instruction — no separate compare.
- Only **1 branch** per iteration instead of 2 (no `BEQ DONE` exit check needed inside the loop body — the loop just falls through when `SUBS` produces zero and `BNE` fails).
- **Net savings: 2 instructions per iteration** compared to the ascending version.

---

## Arrays

An array is used to access large amounts of similar data.

- **Index** — provides access to each element
- **Size** — the number of elements

**Layout in memory:** a 5-element array `scores` has a **base address** = the address of its first element, `scores[0]`. Every other element is accessed **relative to the base address**, at increasing addresses (word-aligned, so each `int` element is 4 bytes apart).

```
Address      Data
1400031C     scores[199]
14000318     scores[198]
  ...          ...
14000004     scores[1]
14000000     scores[0]   ← base address
```

### Accessing array elements

**C code:**
```c
int array[5];
array[0] = array[0] * 8;
array[1] = array[1] * 8;
```

**ARM translation** (`R0` = array base address):
```asm
        MOV  R0, #0x60000000  ; R0 = base address

        LDR  R1, [R0]         ; R1 = array[0]
        LSL  R1, R1, 3        ; R1 = R1 << 3 = R1*8
        STR  R1, [R0]         ; array[0] = R1

        LDR  R1, [R0, #4]     ; R1 = array[1]
        LSL  R1, R1, 3        ; R1 = R1 << 3 = R1*8
        STR  R1, [R0, #4]     ; array[1] = R1
```
Note the `#4` offset for `array[1]` — each `int` element is 4 bytes, so element `i` sits at `base + 4*i`.

### Arrays using for loops

**C code:**
```c
int array[200];
int i;
for (i=199; i >= 0; i = i - 1)
    array[i] = array[i] * 8;
```

**ARM translation** (`R0` = array base address, `R1 = i`):
```asm
        MOV  R0, #0x60000000    ; R0 = base address
        MOV  R1, #199           ; i = 199

FOR
        LDR  R2, [R0, R1, LSL #2] ; R2 = array(i)
        LSL  R2, R2, #3          ; R2 = R2<<3 = R2*8
        STR  R2, [R0, R1, LSL #2]; array(i) = R2
        SUBS R0, R0, #1          ; i = i - 1  (and set flags)
        BPL  FOR                 ; if (i>=0) repeat loop
```
`[R0, R1, LSL #2]` computes the effective address as `base + (i << 2)` — shifting the index left by 2 multiplies it by 4 (word size), all in the addressing mode itself, with no separate instruction. `BPL` ("branch if plus/positive") replaces the `>= 0` compare.

---

## Function Calls

- **Caller** — the calling function (e.g. `main`)
- **Callee** — the called function (e.g. `sum`)

**C code:**
```c
void main()
{
    int y;
    y = sum(42, 7);
    ...
}

int sum(int a, int b)
{
    return (a + b);
}
```

### Function call conventions

**Caller responsibilities:**
- passes **arguments** to the callee
- jumps to the callee

**Callee responsibilities:**
- performs the function
- returns the result to the caller
- returns to the point of call
- **must not overwrite** any registers or memory needed by the caller

### ARM-specific conventions

| Role | Convention |
|---|---|
| Call a function | `BL <label>` (branch and link) |
| Return from a function | `MOV PC, LR` |
| Arguments | `R0`–`R3` |
| Return value | `R0` |

### BL and the link register

**C code:**
```c
int main() {
    simple();
    a = b + c;
}
void simple() {
    return;
}
```

**ARM translation:**
```asm
0x00000200  MAIN    BL   SIMPLE
0x00000204          ADD  R4, R5, R6
...
0x00401020  SIMPLE  MOV  PC, LR
```
- `BL SIMPLE` branches to `SIMPLE` **and** sets `LR = PC + 4` — i.e. `LR` is loaded with the address of the instruction right after the `BL` (`0x00000204`), so the callee knows exactly where to return.
- `MOV PC, LR` inside `SIMPLE` copies that saved address back into `PC`, resuming execution at `0x00000204`.
- `void simple()` means `simple` doesn't return a value — no result is placed in `R0`.

### Arguments and return value

**C code:**
```c
int main()
{
    int y;
    ...
    y = diffofsums(2, 3, 4, 5);  // 4 arguments
    ...
}

int diffofsums(int f, int g, int h, int i)
{
    int result;
    result = (f + g) - (h + i);
    return result;               // return value
}
```

**ARM translation** (`R4 = y` in `main`, `R4 = result` in `diffofsums`):
```asm
; MAIN
        MOV  R0, #2       ; argument 0 = 2
        MOV  R1, #3       ; argument 1 = 3
        MOV  R2, #4       ; argument 2 = 4
        MOV  R3, #5       ; argument 3 = 5
        BL   DIFFOFSUMS   ; call function
        MOV  R4, R0       ; y = returned value
        ...

; DIFFOFSUMS
        ADD  R8, R0, R1   ; R8 = f + g
        ADD  R9, R2, R3   ; R9 = h + i
        SUB  R4, R8, R9   ; result = (f + g) - (h + i)
        MOV  R0, R4       ; put return value in R0
        MOV  PC, LR       ; return to caller
```
**Problem:** `diffofsums` overwrote **three registers** — `R4`, `R8`, `R9` — that the caller might still need. `diffofsums` can use the **stack** to temporarily store (and later restore) registers it needs to use.

---

## The Stack

- Memory used to **temporarily save variables**.
- Behaves like a stack of dishes — **last-in, first-out (LIFO)**.
- **Expands**: uses more memory when more space is needed.
- **Contracts**: uses less memory when the space is no longer needed.
- **Grows down** — from higher addresses toward lower addresses.
- **Stack pointer (`SP` / `R13`)** always points to the current top of the stack.

```
Address      Data              Address      Data
BEFFFAE8     AB000001  ← SP    BEFFFAE8     AB000001
BEFFFAE4                       BEFFFAE4     12345678
BEFFFAE0                       BEFFFAE0     FFEEDDCC  ← SP
BEFFFADC                       BEFFFADC
   ...                            ...
                       Stack expands by 2 words →
```

### Storing register values on the stack

**ARM translation** (`diffofsums`, saving `R4`, `R8`, `R9`):
```asm
DIFFOFSUMS
        SUB  SP, SP, #12    ; make space on stack for 3 registers
        STR  R4, [SP, #8]   ; save R4 on stack
        STR  R8, [SP, #4]   ; save R8 on stack
        STR  R9, [SP]       ; save R9 on stack

        ADD  R8, R0, R1     ; R8 = f + g
        ADD  R9, R2, R3     ; R9 = h + i
        SUB  R4, R8, R9     ; result = (f + g) - (h + i)
        MOV  R0, R4         ; put return value in R0

        LDR  R9, [SP]       ; restore R9 from stack
        LDR  R8, [SP, #4]   ; restore R8 from stack
        LDR  R4, [SP, #8]   ; restore R4 from stack
        ADD  SP, SP, #12    ; deallocate stack space
        MOV  PC, LR         ; return to caller
```

### Preserved vs. nonpreserved registers

| Preserved (callee-saved) | Nonpreserved (caller-saved) |
|---|---|
| `R4`–`R11` | `R12` |
| `R14` (`LR`) | `R0`–`R3` |
| `R13` (`SP`) | `CPSR` |
| stack **above** `SP` | stack **below** `SP` |

Only registers in the **preserved** column need to be saved/restored by a function if it plans to modify them — the caller can't rely on nonpreserved registers surviving a call, so it must save any it still needs *before* calling.

### Storing only what's needed, with pre/post-indexed addressing

If a function only needs to save one register, the stack pointer update can be folded into the load/store itself:
```asm
DIFFOFSUMS
        STR  R4, [SP, #-4]!   ; save R4 on stack (pre-indexed: SP -= 4, then store)
        ADD  R8, R0, R1       ; R8 = f + g
        ADD  R9, R2, R3       ; R9 = h + i
        SUB  R4, R8, R9       ; result = (f + g) - (h + i)
        MOV  R0, R4           ; put return value in R0
        LDR  R4, [SP], #4     ; restore R4 from stack (post-indexed: load, then SP += 4)
        MOV  PC, LR           ; return to caller
```
The `!` and post-indexed `, #4` forms combine the address offset with the stack pointer update, so there's no separate `SUB SP, SP, #n` / `ADD SP, SP, #n` pair — a code-size optimization for the common expand/contract-by-one-register case.

---

## Nonleaf Functions

A **nonleaf function** is one that calls *another* function during its own execution — meaning it must save `LR` before making that call, or its own return address gets overwritten.

```asm
        STR  LR, [SP, #-4]!  ; store LR on stack
        BL   PROC2           ; call another function
        ...
        LDR  LR, [SP], #4    ; restore LR from stack
        ...                  ; return to caller (e.g. MOV PC, LR)
```
Without saving `LR` first, the nested `BL PROC2` would overwrite `LR` with `PROC2`'s return address, destroying the original caller's return address.

### Nonleaf function example

**C code:**
```c
int f1(int a, int b) {
    int i, x;
    x = (a + b) * (a - b);
    for (i=0; i<a; i++)
        x = x + f2(b+i);
    return x;
}

int f2(int p) {
    int r;
    r = p + 5;
    return r + p;
}
```

**ARM translation** (`R0=a, R1=b, R4=i, R5=x` in `f1`; `R0=p, R4=r` in `f2`):
```asm
; F1
        PUSH  {R4, R5, LR}    ; save regs
        ADD   R5, R0, R1      ; x = (a+b)
        SUB   R12, R0, R1     ; temp = (a-b)
        MUL   R5, R5, R12     ; x = x*temp
        MOV   R4, #0          ; i = 0
FOR
        CMP   R4, R0          ; i < a?
        BGE   RETURN          ; no: exit loop
        PUSH  {R0, R1}        ; save regs
        ADD   R0, R1, R4      ; arg is b+i
        BL    F2              ; call f2(b+i)
        ADD   R5, R5, R0      ; x = x+f2(b+i)
        POP   {R0, R1}        ; restore regs
        ADD   R4, R4, #1      ; i++
        B     FOR             ; repeat loop
RETURN
        MOV   R0, R5          ; return x
        POP   {R4, R5, LR}    ; restore regs
        MOV   PC, LR          ; return

; F2
        PUSH  {R4}            ; save regs
        ADD   R4, R0, 5       ; r = p+5
        ADD   R0, R4, R0      ; return r+p
        POP   {R4}            ; restore regs
        MOV   PC, LR          ; return
```
`f1` is nonleaf (it calls `f2`), so it saves `LR` at entry (bundled into `PUSH {R4, R5, LR}`) and restores it before returning. It also saves `R0` and `R1` around the `BL F2` call, since `f2` is free to clobber the nonpreserved argument registers.

### Stack during a nonleaf function call

Each function call pushes its own **stack frame** on top of the previous one:

```
Before f1 call:        Just before calling f2:      After calling f2:
                        f1's frame: LR, R5, R4,
                                    R1, R0
                        (f2's frame not yet pushed)   f2's frame: R4
                        SP → top of f1's frame         SP → top of f2's frame
```
`SP` always tracks the current top of the combined stack — each nested call's frame sits below (at a lower address than) its caller's frame.

---

## Recursive Function Call

A recursive function is simply a nonleaf function that calls **itself**.

**C code:**
```c
int factorial(int n) {
    if (n <= 1)
        return 1;
    else
        return (n * factorial(n-1));
}
```

**ARM translation:**
```asm
0x94  FACTORIAL  STR  R0, [SP, #-4]!   ; store R0 on stack
0x98             STR  LR, [SP, #-4]!   ; store LR on stack
0x9C             CMP  R0, #2           ; set flags with R0-2
0xA0             BHS  ELSE             ; if (r0>=2) branch to else
0xA4             MOV  R0, #1           ; otherwise return 1
0xA8             ADD  SP, SP, #8       ; restore SP
0xAC             MOV  PC, LR           ; return
0xB0  ELSE       SUB  R0, R0, #1       ; n = n - 1
0xB4             BL   FACTORIAL        ; recursive call
0xB8             LDR  LR, [SP], #4     ; restore LR
0xBC             LDR  R1, [SP], #4     ; restore R0 (n) into R1
0xC0             MUL  R0, R1, R0       ; R0 = n*factorial(n-1)
0xC4             MOV  PC, LR           ; return
```
Every recursive call pushes a **new stack frame** holding that call's own copy of `n` (`R0`) and its own return address (`LR`) — this is what lets each level of recursion keep its own state distinct from the others, even though every call runs the exact same code.

### Stack during a recursive call (factorial, n=3)

Each nested call pushes its own `R0` and `LR` onto the stack. As the base case (`n=1`) is reached and calls start returning, the frames are popped off in reverse order (LIFO) and each level multiplies its restored `n` by the return value from the level below — unwinding to the final result `R0 = 6`.

---

## Function Call Summary

**Caller:**
- puts arguments in `R0`–`R3`
- saves any registers it still needs (`LR`, maybe `R0`–`R3`, `R8`–`R12`)
- calls the function: `BL CALLEE`
- restores its saved registers
- looks for the result in `R0`

**Callee:**
- saves registers that might be disturbed (`R4`–`R7` if used)
- performs the function
- puts the result in `R0`
- restores the registers it saved
- returns: `MOV PC, LR`
