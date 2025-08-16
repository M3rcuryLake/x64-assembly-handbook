# x86-64 Assembly Language

This guide was written by me while taking the OpenSecurityTraining2 [Architecture 1001: x86-64 Assembly](https://apps.p.ost2.fyi/learning/course/course-v1:OpenSecurityTraining2+Arch1001_x86-64_Asm+2021_v1/home) class. This markdown file covers the basics of x86-64 assembly language programming, including a useful subset of the available instructions and assembler directives.

This document assumes you have prior knowledge on the C programmming language, Number System (hex, binary and decimals) and are more or less aquainted with computer architecture.

### General-Purpose Registers and Their Sizes

Each register in x86-64 has several aliases depending on the size of the value being accessed:

```nasm
| Register | 8-bit (Byte) | 16-bit (Word) | 32-bit (Double Word) | 64-bit (Quad Word) |
|----------|--------------|---------------|-----------------------|--------------------|
| RAX      | AL           | AX            | EAX                   | RAX                |
| RBX      | BL           | BX            | EBX                   | RBX                |
| RCX      | CL           | CX            | ECX                   | RCX                |
| RDX      | DL           | DX            | EDX                   | RDX                |
| RSP      | SPL          | SP            | ESP                   | RSP                |
| RSI      | SIL          | SI            | ESI                   | RSI                |
| RDI      | DIL          | DI            | EDI                   | RDI                |
| RBP      | BPL          | BP            | EBP                   | RBP                |
| R8       | R8B          | R8W           | R8D                   | R8                 |
| R9       | R9B          | R9W           | R9D                   | R9                 |
| R10      | R10B         | R10W          | R10D                  | R10                |
| R11      | R11B         | R11W          | R11D                  | R11                |
| R12      | R12B         | R12W          | R12D                  | R12                |
| R13      | R13B         | R13W          | R13D                  | R13                |
| R14      | R14B         | R14W          | R14D                  | R14                |
| R15      | R15B         | R15W          | R15D                  | R15                |

```

> Note: In 16-bit registers like AX, the higher and lower 8 bits can be accessed separately:
> 
> - `AH` = High 8 bits of AX
> - `AL` = Low 8 bits of AX

---

### Register Purpose and Usage

- **RCX** – Used as a **counter** in loops and string operations.
- **RSP** – Stack Pointer; used by `PUSH`, `POP`, `CALL`, `RET`.
- **RSI / RDI** – Source and Destination Index; used in **string manipulation instructions**.
- **RBP** – Stack Frame Base Pointer; points to the **base of the current stack frame**.
- **RIP** – Instruction Pointer; holds the address of the **next instruction**.

---

### `NOP` Instruction — First Steps.

- **Opcode:** `0x90`
- Used for:
    - Padding / aligning instructions in memory
    - Delaying execution (timing adjustments)
- **Mnemonic meaning:**`NOP` is an alias for the instruction `XCHG RAX, RAX` — which effectively **does nothing**.

## What is `r/mX`?

Before we start, we need to know what is `r/mX`. Well it is just a shorthand term (credit: *OpenSecurityTraining*) for `r/m8`, `r/m16`, `r/m32`, or `r/m64` in the Intel manuals. Here `r` stands for register and `m` for memory. Another FYI, In Intel syntax, Square brackets `[]` treat the value inside as a **memory address**, and fetch the value stored **at that address** (similar to pointer dereferencing in C). We would need these later to understand the documentation.

An `r/mX` can take **four main forms**:

- **Register** : `rbx`
- **Memory (Base Only)** : `[rbx]`
- **Memory (Base + Index × Scale)** : `[rbx + rcx * x]` ; x can be 1, 2, 4, or 8
- **Memory (Base + Index × Scale + Displacement)** : `[rbx + rcx * x + y]`
    
    ```
    - 'y' can be:
    - 0–2⁸ (1 byte displacement)
    - 0–2³² (4 byte displacement)
    - This is **useful for multi-dimensional array indexing, arrays of structs, etc.
    
    ```
    

# The Stack

A **stack** is a **Last-In, First-Out (LIFO)** data structure where:

- Data is **pushed** (added) to the top of the stack.
- Data is **popped** (removed) from the top of the stack.
- It is a **conceptual area** of main memory (RAM).

### Memory Growth Convention

- **By convention**, in most architectures (including x86-64), the stack **grows towards lower memory addresses**.
- Adding (pushing) something to the stack means:
    - The **top of the stack** moves **downward** in memory.
    - The **stack pointer (RSP)** is **decremented**.
- When you pop something, the SP moves **up**.

### Typical Process Memory Layout

```
   		   High Addresses

| -------------- | ------------ |
| Process Memory |              |
| Stack ↓        | ← grows down |
| Heap  ↑        | ← grows up   |
| Code Segment   |              |
|----------------|--------------|

   		   Low Addresses
```

> As stack grows down and heap grows up, if you use too much of these, it will result in some sort of error.
> 

RSP points to the top of the stack, the lowest address which is being used. While data will still exist beyond the top of the stack, it is considered undefined. It’s kind of backwards compared to how we talk about "top" in real life. Here the top is the lower memory address.

- **Push**: Decrements `RSP`, stores data at new top.
- **Pop**: Reads data from top, increments `RSP`.
- **RSP** always points to the **current top** of the stack.

---

## push Instruction

- In **64-bit mode**, the `PUSH` instruction **automatically decrements** the Stack Pointer (`RSP`) by **8 bytes**.
- The operand can be:
    1. The value in a 64-bit register
    2. A 64-bit value from memory, described in the **`r/mX`** form

### `PUSH` Examples

```nasm
push rbx                     ; Push value in register
push [rbx]                   ; Push value at memory address in RBX
push [rbx + rcx * 4]         ; Push value using base + index * scale
push [rbx + rcx * 8 + 0x20]  ; Push value with base + index * scale + disp dvbv3w
```

---

## pop Instruction

- The **opposite** of `PUSH`.
- Automatically **increments** `RSP` by **8 bytes**.
- The pop instruction removes the 8-byte data element from the top of the hardware-supported stack into the specified operand (i.e. register or memory location). It first moves the 8 bytes located at memory location [SP] into the specified register or memory location, and then increments SP by 8.
*Examples*

```
pop rdi — pop the top element of the stack into RDI.
pop [rbx] — pop the top element of the stack into memory at the four bytes starting at location RBX.
```

# AT&T Syntax vs Intel Syntax

## Key Differences

| Feature | **Intel Syntax** | **AT&T Syntax** |
| --- | --- | --- |
| Operand Order | `destination ← source` | `source → destination` |
| Register Prefix | No prefix | `%` before register names |
| Immediate Prefix | No prefix | `$` before immediate values |
| Example Move | `mov rbp, rsp` | `mov %rsp, %rbp` |
| Example Add | `add rsp, 0x14` | `add $0x14, %rsp` |

### Intel Syntax

```nasm
mov rbp, rsp        ; RBP = RSP
add rsp, 0x14       ; RSP = RSP + 0x14

```

- More common in reverse engineering tools (IDA, Ghidra default).
- Matches how most x86-64 documentation is written.

### AT&T Syntax

```nasm
mov %rsp, %rbp      # RBP = RSP
add $0x14, %rsp     # RSP = RSP + 0x14
```

- Default for GNU `as` assembler (GAS).
- Common in Unix/Linux documentation.

### suffixes in intel and at&t.

the `b`/`w`/`d`/`q` suffix pattern is a **general operand size hint**, and it applies to a lot of  instructions, like in `movb` and `movw` .

**Intel syntax** (NASM, MASM) usually doesn’t require suffixes — operand size is determined from the operands:

```nasm
mov al, [rsi]   ; byte
mov ax, [rsi]   ; word
mov eax, [rsi]  ; dword
mov rax, [rsi]  ; qword
```

Suffixes (`b/w/d/q`) mainly show up in **GNU assembler (GAS) AT&T syntax**, not Intel syntax. **Not every instruction uses these** — some opcodes are inherently fixed-size:

- `pushfq` always pushes 8 bytes in 64-bit mode, no suffix needed.
- `cpuid` always uses `EAX`/`EBX`/`ECX`/`EDX`, so no size ambiguity

Most instructions that operate on registers/memory and can be encoded in multiple operand sizes use it:

- **Data movement**: `movb`, `movw`, `movl` (`l` is used in AT&T syntax for 4 bytes), `movq`
- **Arithmetic**: `addb`, `addw`, `addl`, `addq`
- **Logic**: `andb`, `orw`, `xorl`, `xorq`
- **String ops**: `stosb`, `stosw`, `stosd`, `stosq`, `movsb`, etc.

# *Basic Instructions…*

### call — Function calls

The `call` instruction first pushes the current code location onto the hardware supported stack in memory (see the push instruction for details), and then changes the Instruction Pointer (`RIP`) to the address specified by the operand. This is an **unconditional jump** to the subroutine. Unlike the simple jump instructions, the call instruction saves the location to return to when the subroutine completes. The pushed value is the **return address.**

*Syntax*

```nasm
call <label>
```

*Working*

```nasm
RSP = RSP - 8        ; Allocate space for return address
[ RSP ] = RIP        ; Push next instruction address onto stack
RIP = <label>        ; Jump to subroutine
```

### ret — Return Instruction

The `ret` instruction implements a subroutine return mechanism. This instruction first the **return address** from the stack into `RIP` and restores where to return from the stack. It then performs an unconditional jump to the retrieved code location.

There are **two forms** of `RET`:

1. **Simple Return** :`ret`, Pops the top of the stack into `RIP`.
2. **Return and Clean Up Stack** : `ret 0x8`, `ret 0x20`, Pops the return address into `RIP`. This **adds** a constant number of bytes to `RSP` (used to clean up parameters in some calling conventions).

*Working*

```
RIP = [ RSP ]        ; Pop return address
RSP = RSP + 8        ; Move stack pointer past return address
(For `ret X`) RSP = RSP + X  ; Clean up parameters
```

### mov  — move operation :

The mov instruction copies the data item referred to by its second operand (i.e. register contents, memory contents, or a constant value) into the location referred to by its first operand (i.e. a register or memory). While register-to-register moves are possible, direct memory-to-memory moves are not. In cases where memory transfers are desired, the source memory contents must first be loaded into a register, then can be stored to the destination memory address.
*Examples*

- immediate to memory : `mov [rbx], 0x14`
- immediate to register : `mov rbx, 0x14`
- register to memory : `mov [rbx], rax`
- memory to register : `mov rax, [rbx]`

### movzx & movsx — extensions :

`movsx` and `movzx` are x86/x86-64 instructions used for moving data into a larger register while extending its size.

- `movsx` (**move with sign-extension**) takes a smaller signed value (like 8-bit or 16-bit) and copies it into a larger register (like 16, 32, or 64 bits), filling the extra high bits with the most significant bit (so negatives stay negative).
- `movzx` (**move with zero-extension**) does the same size increase but treats the value as unsigned, filling the high bits with zeros.
- For example, if `al` = `0xFF` (-1 in signed 8-bit), `movsx rax, al` gives `0xFFFFFFFFFFFFFFFF` (-1), but `movzx rax, al` gives `0x00000000000000FF` (255).

### movsxd :

`movsx` technically only sign extends from 8 or 16 bit values. If you want to sign extend a 32 bit value to 64 bits, you need to use `movsxd`. Also, there is no "movzxd". it's always just `movzx`*example*

```nasm
mov eax, 0xFFFFFFFF   ; eax = -1 in signed 32-bit
movsxd rax, eax       ; rax = 0xFFFFFFFFFFFFFFFF (-1 in signed 64-bit)

```

### add and sub :

The add instruction adds together its two operands, storing the result in its first operand. The sub instruction stores in the value of its first operand the result of subtracting the value of its second operand from the value of its first operand
*Examples*

- `add rsp, 8`
- `sub rax, [rbx*2]` -> rax = rax - memoryPointedToBy(rbx*2)

### imul — **signed** Integer Multiplication :

The imul instruction has three basic formats: one-operand, two-operand and three-operand,

- The one-operand form multiplies the value in **RAX/EAX/AX/AL** by the operand, i.e. `imul r/mX` which multiplies RAX/EAX/AX/AL by r/m64
*Example*:
    
    ```nasm
    mov rax, 5
    imul qword ptr [var]    ; RDX:RAX = RAX × [var]
    ```
    
- The two-operand form multiplies its two operands together and stores the result in the first operand. The result (i.e. first) operand must be a register.
*Example:* `imul rX, r/mX`
- The three operand form multiplies its second and third operands together and stores the result in its first operand. Again, the result operand must be a register. Furthermore, the third operand is restricted to being a constant value.
*Example:* `imul rX, r/mX, immX` (RAX → RBX * immediate)

### div — Unsigned Divide :

The `div` instruction divides the contents by three forms;

- Dividing the contents of the register `al` by `r/m8`, where `al` = quotient and `ah` = remainder.
- Dividing the contents of the 64 bit integer `EDX:EAX`(constructed by viewing `EDX` as the most significant four bytes and `EAX` as the least significant four bytes) by the specified `r/m32` value. The quotient result of the division is stored into `EAX`, while the remainder is placed in `EDX`.
- Dividing the contents of the 128 bit integer `RDX:RAX`(constructed by viewing `RDX` as the most significant eight bytes and `RAX` as the least significant eight bytes) by the specified `r/m64` value. The quotient result of the division is stored into `RAX`, while the remainder is placed in `RDX`.

```nasm
mov     rax, 20       ; Dividend
mov     rbx, 3        ; Divisor
xor     rdx, rdx      ; Clear remainder
div     rbx           ; Unsigned division → RAX=6, RDX=2
```

### idiv — Signed Divide :

for signed division (`idiv`), we treats numbers as signed (two’s complement). In **`div`**, negative values don’t make sense (will cause a fault if given). So we use **`idiv`** for negative dividends or divisors work correctly.

```nasm
mov     rax, -20
cqo                   ; Sign-extend RAX into RDX:RAX
mov     rbx, 3
idiv    rbx           ; Signed division → RAX=-6, RDX=-2
```

### lea — Load Effective Address :

`lea` in x86-64 is **Load Effective Address,** it takes the address that a memory operand would refer to, calculates it, and puts that *calculated value* into a register. It uses the ‘mX’ form but is thhe exception to the rule that the square brackets [ ] means dereference.

Frequently used with pointer arithmatic or just with arithmatic in general.

*examples:*

```nasm
lea rax, [var]
lea rbx, [rax+rcx*4+16] ; pointer arithmatic
lea rdx, [rax*2+rax] ; doing arithmatic without using flags like imul
```

The magic is that `lea` treats the bracket expression as math, not just as a dereference.

### *Simple Operation Example* :

*Decompilation of a simple ‘adding two numbers’ C code:*

```nasm
main:
        push    rbp                     ; Save old base pointer on the stack
        mov     rbp, rsp                ; Set up new stack frame (RBP now points to top of stack)

        ; This is the standard function prologue in most compiler outputs (especially with no optimizations).
		; It saves the old base pointer (`RBP`) and then sets `RBP` to the current stack pointer (`RSP`) so local variables can be referenced with fixed offsets.

		; DWORD PTR is just an operand size specifier in Intel syntax, in this case it is a double word (4 bytes = 32 bits).
		; It basically uses DWORD PTR to treat [rbp-4] as a 4-byte location
		; This matches 'int' size in most 32-bit/64-bit C ABIs for local variables

        mov     DWORD PTR [rbp-4], 12   ; Store integer 12 at local variable (offset -4 from RBP)
        mov     DWORD PTR [rbp-8], 18   ; Store integer 18 at local variable (offset -8 from RBP)

        mov     edx, DWORD PTR [rbp-4]  ; Load value from local var at [RBP-4] into EDX (edx = 12)
        mov     eax, DWORD PTR [rbp-8]  ; Load value from local var at [RBP-8] into EAX (eax = 18)

        add     eax, edx                ; EAX = EAX + EDX → (18 + 12 = 30)

        mov     DWORD PTR [rbp-12], eax ; Store result (30) into local var at [RBP-12]

        mov     eax, 0                  ; Set return value of `main` to 0 (success)
        pop     rbp                     ; Restore old base pointer
        ret                             ; Return to caller (exit main)

```

- In intel syntax for r/mX memory descriptions, it will ue things like `qword ptr`, `dword ptr` or `word ptr`, to indicate the size of the data being operated on. (8, 4 an 2 bytes respectively)
- In `mov DWORD PTR [rbp-4], 12`, We subtract 4 form `rbp` because in memory, the stack grows downward toward lower addresses.

# *Bitwise Operators* :

### **and, or, xor** — Bitwise logical and, or and exclusive or :

These instructions perform the specified logical operation (logical bitwise and, or, and exclusive or, respectively) on their operands, placing the result in the first operand location.

> FYI, `xor`s is commonly used to zero a register, by xor-ing it with itself, because it is faster than a `mov` .
> 

*examples:*

```nasm
and dest, src
or dest, src
xor dest, src
```

### **not — Bitwise Logical Not**

Logically negates the operand contents (that is, flips all bit values in the operand).

*example:*

```nasm
not BYTE PTR [var] ; negate all bits in the byte at the memory location var.
NOT dest
```

### **neg** — Negate

Performs the two's complement negation of the operand contents.

```nasm
neg eax ; EAX → - EAX
```

### inc/dec — Increment / Decrement

The inc instruction increments the contents of its operand by one. The dec instruction decrements the contents of its operand by one.

When optimised, compilers will tend to favor not using `inc` / `dec` , as directed by the intel optimisation guide. So their presense may be indicative of hand-written, or un-optimised code.

Modifies `OF`, `SF`, `ZF`, `AF`, `PF`, and `CF` flags.

*example:*

```nasm
xor rax, rax ; rax -> rax ^ rax, which is 0x0 
inc rax ; rax -> 0x1
mov rbx, 0
dec rbx ; rbx -> 0xffffffff'ffffffff (integer-underflow)
```

### test - Logical Compare

Computes the bitwise `and` of the first operand (source1 op) and the second operand (source2 op) and sets the `sf`, `zf`, `pf` flags according to the result. Like `cmp`, which subtracts the first from the second, `test` adds the first and second then -sets flags, and throws away the result.

```nasm
test eax,ecx ; perform eax & ecx and set flags
```

We can see this in decompiled C code where we do not store result anywhere, like in :

```c
int main(){
    int a = 0x01; int b = 0x02;
    if(a & b) return 0;}
```

### shl,  sal — Shift Logical Left / Shift Arithmatic Left

First operand (source and destination) is an r/mX, the second operand is either `cl` or an one byte immediate. The 2nd operand is the number of places to shift.

Bits shifted off the left hand side are shifted into (set) the `cf` .

Shift the bits by n bits left, which leads to new value that is multiplied by 2 n types, n is the 2nd operand which is 1 byte long. *The LSB's after shifting n bits left are filled by 0's, used for multiplication in optimised code.*

```nasm
eax = 0x12deff99
shl eax,2
eax = 0x4B7BFE64
```

### shr, sar — Shift Logical Right / Shift Arithmatic Right

Shift the bits by n bits right, which leads to new value that is divided by 2 n types, n is the 2nd operand which is 1 byte long. *The MSB's after shifting n bits right are filled by 0's, used for division in optimised code*

Bits shifted off the right hand side are shifted into (set) the `cf` .

```nasm
eax = 0x4B7BFE64
shr eax,2
eax = 0x12deff99
```

the only difference between `SHR` and `SAR` is that the MSB's that are moved are filled with the original MSB value, if MSB was 1 all the shifted bit would be filled as 1 else 0.

*used when there is division of signed int.*

> `shl` / `shr` are used to optimise C code if the variable is multiplied or divided by a number in form of `2^n` . So when we are using mul/div in powers of 2^n, it would prefer `shr` and `shl`
> 

# *Control flow instructions.*

The x86-64 processor maintains an instruction pointer (IP) register indicating the location in memory where the current instruction starts. Normally, it increments to point to the next instruction in memory begins after execution an instruction. The IP register cannot be manipulated directly, but is updated implicitly by provided control flow instructions.

### rflags :

In 64 bits mode, `eflags` is extended to 64 bits and called `rflags` . The upper 32 bits of `rflags` register is reserved. The lower 32 bits of `rflags` is the same as `eflags`. RFLAGS register holds many single bit flags.

you can think of **`RFLAGS`** in x86-64 as *functionally similar* to **`CPSR`** (Current Program Status Register) in ARM, but there are architectural differences.

| Bit | Name | Meaning |
| --- | --- | --- |
| 0 | **CF** (Carry) | Set if an unsigned operation overflows. |
| 2 | **PF** (Parity) | Set if the low byte of the result has an even number of 1s. |
| 4 | **AF** (Adjust) | Used for BCD math (rarely relevant today). |
| 6 | **ZF** (Zero) | Set if result == 0. |
| 7 | **SF** (Sign) | Copy of result’s most significant bit (sign bit). |
| 8 | **TF** (Trap) | Single-step debug mode. |
| 9 | **IF** (Interrupt) | Enables/disables maskable interrupts. |
| 10 | **DF** (Direction) | Affects string instructions (`movs`, `stos`): 0 = increment, 1 = decrement. |
| 11 | **OF** (Overflow) | Set if signed operation overflows. |

*example :*

```nasm
mov eax, 5
cmp eax, 10     ; subtracts 10 from eax internally
; ZF = 0 (result ≠ 0)
; SF = 1 (result negative)
; CF = 1 (borrow in unsigned)
; OF = 0 (no signed overflow)
```

### CMP — Compare Two Operands :

`cmp` in x86-64 is basically a subtract-without-storing instruction. It subtracts the second operand from the first, throws away the result, and just updates the `RFLAGS` so the next conditional jump (or set/test instruction) can use them.

***What gets affected :***

- **ZF (Zero Flag)** – set if `dest == src` (temp = 0).
- **SF (Sign Flag)** – set if result’s most significant bit is 1 (negative in signed).
- **CF (Carry Flag)** – set if an *unsigned* borrow occurred (`dest < src` in unsigned).
- **OF (Overflow Flag)** – set if *signed* overflow occurred.
- **PF** and **AF** also updated, but usually ignored in normal integer compares.

*Example :*

```nasm
mov eax, 5
cmp eax, 10     ; 5 - 10 = -5
; ZF = 0  (not equal)
; SF = 1  (negative result)
; CF = 1  (unsigned borrow happened)
; OF = 0  (no signed overflow here)
jg greater      ; checks ZF=0 and SF=OF → won’t jump
jl less         ; checks SF≠OF → jumps
```

### JMP  —  jumping unconditionally :

In x86-64, `jmp` is the instruction for an **unconditional jump,** it tells the CPU to set the instruction pointer (`RIP`) to a new location, and execution continues from there instead of the next sequential instruction. It doesn’t care about flags or conditions; it *always* transfers control.

*example :*

```nasm
_start:
    jmp skip
    mov eax, 1       ; this never executes
skip:
    mov eax, 42
```

*Ways to specify the address :* 

- Short, relative (`rip` = `rip` of next instruction + 1 byte, sign extended to 64 bits displacement)
- Short jumps are used in small loops, some dissassemblers will indicate this with a mnemonic writing it as `jmp short`
- `jmp -2` is and infinite loop for short relative jmp.
- Far jumps are covered later.
- Near, relative (`rip` = `rip` of the next instruction + 4 bytes, sign extended to 64 bits displacement. This means you can jump 4 billion bytes forward or backward.)

### **Conditional Jumps**

Only jump if a specific **status flag** matches a condition.

- Based on **ZF, SF, CF, OF, PF** after a compare (`cmp`) or test (`test`).

### **Equality / Inequality**

- `je` / `jz` — Jump if equal / zero (`ZF=1`)
- `jne` / `jnz` — Jump if not equal / not zero (`ZF=0`)

---

### **Signed Comparisons**

(These use **SF** and **OF** together)

- `jg` / `jnle` — Greater than (`ZF=0` and `SF=OF`)
- `jge` / `jnl` — Greater or equal (`SF=OF`)
- `jl` / `jnge` — Less than (`SF≠OF`)
- `jle` / `jng` — Less or equal (`ZF=1` or `SF≠OF`)

---

### **Unsigned Comparisons**

(These use **CF** and **ZF**)

- `ja` / `jnbe` — Above (`CF=0` and `ZF=0`)
- `jae` / `jnb` / `jnc` — Above or equal (`CF=0`)
- `jb` / `jnae` / `jc` — Below (`CF=1`)
- `jbe` / `jna` — Below or equal (`CF=1` or `ZF=1`)

---

### **Special Flag Checks**

- `js` / `jns` — Sign set / sign not set (`SF=1` / `SF=0`)
- `jo` / `jno` — Overflow / no overflow (`OF=1` / `OF=0`)
- `jp` / `jpe` — Parity even (`PF=1`)
- `jnp` / `jpo` — Parity odd (`PF=0`)

For an example, the decompiled output of the following simple C code, which checks if variable `a` is equal to `b` and returns `1` would look like :

```c
int main(){
    int a = 1; int b = 2;
    if ( a == b ) return 1;
}
```

And the decompiled output (I used [GodBolt](https://godbolt.org/)),

```nasm
main:
    push    rbp                     ; init sequence
    mov     rbp, rsp                 

    mov     DWORD PTR [rbp-4], 1     ; local var1 = 1   (stored at rbp-4)
    mov     DWORD PTR [rbp-8], 2     ; local var2 = 2   (stored at rbp-8)

    mov     eax, DWORD PTR [rbp-4]   ; eax = var1
    cmp     eax, DWORD PTR [rbp-8]   ; compare eax (var1) with var2
    jne     .L2                      ; if var1 != var2, jump to .L2
    mov     eax, 1                   ; (if equal) set return value = 1
    jmp     .L3                      ; skip over .L2 block

.L2:
    mov     eax, 0                   ; (if not equal) set return value = 0
.L3:
    pop     rbp                      
    ret                              ; return sequence
```

## Loops in x86-64

These are special single-instruction loops that use the `RCX` register as a counter. It is to be noted that `LOOP` instructions are much slower than the `DEC ECX / JNZ` counterpart. This is intended as `LOOP` should nowadays only be used for delay calibration loops used for hardware-drivers and the like.

### loop

Here `RCX` is decremented each time. and if `RCX` ≠ 0, it causes the program to jump to the specified label. Else if `RCX` becomes zero, the program continues to the next instruction. This is unsigned logic **:** doesn’t care about flags like `ZF` or `SF`.

*example :*

```nasm
mov rcx, 5        ; loop counter = 5
start_loop:
    ; do stuff
    loop start_loop
```

### loope, loopz — **loop until mismatch**

`LOOPE` decrements `rcx` and checks that `rcx` is not zero *and* ZF is set (1) - if these conditions are met, it jumps at label, otherwise falls through. It keeps looping while the compared values are equal (ZF=1) and it hasn’t exhausted the counter. Often in search loops where you stop if mismatch happens before count runs out.

```nasm
_start:
    mov rcx, 5        ; loop counter
    mov rax, 10       ; number to decrement

check_loop:
    sub rax, 2        ; subtract 2
    cmp rax, 0        ; set ZF if eax == 0
    loope check_loop  ; loop while ecx != 0 AND ZF == 1
```

### loopne / loopnz — loop until match

`LOOPNE` is same as `LOOPE` except that it requires `ZF` to be not set (i.e be 0) to do the jump. It keeps looping while compared values are not equal (ZF=0) and counter is not exhausted. Typically used to search until a match.

```nasm
_start:
    mov rcx, 5        ; loop counter
    mov rax, 10       ; number to decrement

check_loop2:
    sub rax, 3        ; subtract 3
    cmp rax, 1        ; ZF set only if eax == 1
    loopne check_loop2 ; loop while ecx != 0 AND ZF == 0
```

### **Using cmp + Conditional Jumps**

This is more flexible than `LOOP`.

Example (countdown loop):

```nasm
mov rax, 5        ; counter
start:
    ; do stuff here
    sub rax, 1
    cmp rax, 0
    jne start      ; jump if not zero
    ; Works like `while (rax != 0)` in C.
```

### **Using test + Conditional Jumps**

`TEST` does a bitwise AND but doesn’t store the result, just sets flags. Often used to check if a value is zero without changing it.

*Example*:

```nasm
mov rax, 8
start:
    ; do stuff
    dec rax
    test rax, rax
    jnz start      ; jump if rax != 0
```

# Arrays

In assembly or any other language, an *array* is just a block of consecutive memory locations. But there is no specific "array type", it’s just data stored sequentially, and you access it by computing offsets from its starting address.

For example:

```nasm
arr:   10   20   30   40
       ^    ^    ^    ^
    addr0 addr1 addr2 addr3
```

If each element is 4 bytes (like a `DWORD`),

`arr[0]` is at `arr + 0`,

`arr[1]` is at `arr + 4`,

`arr[2]` is at `arr + 8`, etc.

### **Declaring Arrays :**

```nasm
section .data
    int_array dd 10, 20, 30, 40   ; 4 integers (32-bit each)
    char_array db "hello", 0   ; null-terminated string
    ; dd = define doubleword
    ; db = define byte
```

## **Accessing Array Elements**

For example, to access the element at index `2` in `myArray` (where each element is 1 byte), the address would be `myArray + (2 * 1)`.

```nasm
section .data
    myString db "AssemblyRocks!", 0   ; null-terminated string

section .text
    global _start

_start:
    ; Let's say we want the 4th character ('e')
    mov rsi, myString      ; rsi points to start of string
    mov ebx, 2                ; Load the desired index (2) into EBX
		mov al, [esi + ebx]     ; use al as the char is a 1-byte 
```

To access a element (index i) of an array in assembly, the offset from the base address depends on the size of each element:

- For a **`dw`** (word) array named `my_words`, use:
    
    ```nasm
    mov ax, [my_words + i * 2]    ; 2-byte elements × index 
    ```
    
- For a **`dd`** (doubleword) array named `my_dwords`, use:
    
    ```nasm
    mov eax, [my_dwords + i * 4]  ; 4-byte elements × index 
    ```
    
- For a **`dq`** (quadword) array named `my_qwords`, use:
    
    ```nasm
    mov rax, [my_qwords + i * 8]  ; 8-byte elements × index
    ```
    

In general, the memory address for element *i* is calculated as:

**`base_address + (i * element_size)`** 

Use the appropriate register size (e.g., `ax` for words, `eax` for doublewords, `rax` for quadwords) to load or store the value at that memory address

### what is equ

In assembly language, `EQU` (short for "Equate") is an assembler directive used to define a constant value or a symbolic name for a value. The syntax goes like `CONSTANT_NAME EQU expression` 

### The `$ -` symbol :

Length in bytes from the start of `symbol` to current location `$`. NASM evaluates `equ` and `$ - label` **at assembly time**, so these don’t exist at runtime, they’re just constants in the assembled code.

```nasm
section .data
    my_dwords dd 10, 20, 30, 40

    ; Number of bytes in array:
    array_size_bytes equ $ - my_dwords

    ; Number of elements:
    array_length equ (array_size_bytes / 4) ; each dd = 4 bytes

section .text
    global _start
_start:
    mov eax, array_size_bytes    ; 16
    mov ebx, array_length        ; 4
```

# rep — Repeating String Operations.

`rep` in x86-64 is a prefix that tells the CPU to repeat the following string instruction as long as `RCX` (or `ECX`/`CX` depending on operand size) is not zero. You load the iteration count into `RCX`, and after each execution of the instruction, the CPU automatically decrements `RCX`. Once it reaches zero, the repetition stops. `rep` only works with string instructions, which are special instructions that implicitly use registers like `RSI`, `RDI`, or `RAX/AL`.

`rep` only works with **string instructions** (those that implicitly use RSI/RDI or RAX/AL):

- `movs` — copy from `[RSI]` to `[RDI]`
- `stos` — store AL/AX/EAX/RAX into `[RDI]`
- `lods` — load from `[RSI]` into AL/AX/EAX/RAX
- `scas` — scan `[RDI]` for a value in AL/AX/EAX/RAX
- `cmps` — compare `[RSI]` with `[RDI]`

## **Direction Flag (DF) Affects It**

The **DF** in `RFLAGS` decides whether RSI/RDI moves **forward** or **backward**:

- **`cld`** → DF = 0 → RSI/RDI increment after each iteration / clear direction flag (forward)
- **`std`** → DF = 1 → RSI/RDI decrement after each iteration (backward)

There are also conditional versions:

- `repe` / `repz` — repeat while **RCX != 0** *and* ZF = 1
- `repne` / `repnz` — repeat while **RCX != 0** *and* ZF = 0

## rep stos — Repeat Store String

`STOS` (store string) is on one the number of instructions that can have a `rep` prefix added to it, which repeat a single instruction multiple times.

All rep operations use a `*CX` register as a counter to determine how many times to loop through the instruction. It takes the value in `AL`, `AX`, `EAX`, or `RAX` (depending on operand size) and stores it into memory at the address pointed to by `RDI` (or `EDI` in 32-bit mode). Each time it executes. After storing, it automatically increments or decrements `RDI` based on the **`DF`** (direction flag) in `RFLAGS`:

- **`DF = 0`**→ increment RDI (forward memory direction)
- **`DF = 1`**→ decrement RDI (backward memory direction)

Either stores 1, 2, 4 or 8 bytes at a time.

There are three setup steps which must happen before the actual `rep stos` occurs :

- set `*di` to the start destination,
- set `*ax/al` to the value to store,
- set `*cx` to the number of times to store.

```nasm
section .bss
    buffer resb 8       ; Reserve 8 bytes

section .text
    global _start

_start:
    mov al, 'A'         ; The byte to store
    mov rdi, buffer     ; Destination address
    mov rcx, 8          ; How many times to store
    cld                 ; Increment direction
    rep stosb           ; Fill RCX bytes at [RDI] with AL
```

- `stosb` → store `AL` (1 byte)
- `stosw` → store `AX` (2 bytes)
- `stosd` → store `EAX` (4 bytes)
- `stosq` → store `RAX` (8 bytes)

## rep movs — Repeat Move Data

`movs` is the **“move string”** instruction — it copies data from the address pointed to by **RSI** (source) to the address pointed to by **RDI** (destination). Without a repeat prefix, `movs` only copies **one element**.

Before running, you load **RCX** with the number of elements to copy. As `rep` will decrement RCX after each copy and stop when RCX reaches **0**.

we need to set the DF to 0 or 1 as required as it is responsible for forward copy and backwared copy

- **`CLD`** → `DF` = 0 → increment RSI/RDI after each copy (forward)
- **`STD`** → `DF` = 1 → decrement RSI/RDI after each copy (backward)

***Example: Copy 16 bytes forward***

```nasm
section .data
    src     db 'HELLO WORLD', 0        ; Source string
    dst     times 20 db 0              ; Empty buffer
section .text
    global _start

_start:
    mov rsi, src        ; RSI → source address
    mov rdi, dst        ; RDI → destination address
    mov rcx, 12         ; Number of bytes to copy
    cld                 ; Clear DF → increment pointers
    rep movsb           ; Copy RCX bytes from [RSI] to [RDI]

```

## resb — Reserve Bytes (misc)

`resb` in NASM means **"reserve bytes"** — it allocates uninitialized space in the **BSS section** of your program. So **`resb N`** means reserve N bytes.

- `resw` → reserve words (2 bytes each)
- `resd` → reserve doublewords (4 bytes each)
- `resq` → reserve quadwords (8 bytes each)

BSS section doesn’t store actual data in the executable — it’s just a promise to the OS: “I’ll need this much memory at runtime.” The OS will zero it for you when the program loads.

# Misc flag Instructions.

The following are some miscellaneous instructions used without any operands which we may have use to modify flags, which may give us better control over the program. 

| Instruction | Meaning | Effect |
| --- | --- | --- |
| **CLD** | Clear Direction Flag | DF = 0 → string ops increment index registers (forward) |
| **STD** | Set Direction Flag | DF = 1 → string ops decrement index registers (backward) |
| **CLI** | Clear Interrupt Flag | IF = 0 → disables maskable hardware interrupts |
| **STI** | Set Interrupt Flag | IF = 1 → enables maskable hardware interrupts |
| **CMC** | Complement Carry Flag | CF = !CF |
| **CLC** | Clear Carry Flag | CF = 0 |
| **STC** | Set Carry Flag | CF = 1 |
| **LAHF** | Load AH from Flags | Loads low 8 bits of FLAGS into `AH` |
| **SAHF** | Store AH into Flags | Stores `AH` into low 8 bits of FLAGS |
| **PUSHF / PUSHFD / PUSHFQ** | Push FLAGS/32-bit EFLAGS/64-bit RFLAGS onto the stack |  |
| **POPF / POPFD / POPFQ** | Pop FLAGS from the stack |  |
| **HLT** | Halt CPU until next interrupt |  |

# **Calling convention**

A **calling convention** in assembly is basically the *“rulebook”* for how functions talk to each other, specifically, how arguments are passed, how return values are given back, and who is responsible for cleaning up the stack. We’ll take the **System V AMD64 ABI** (x86-64 Linux) example; since that’s what Ghidra, objdump, and most RE on Linux deal with.

*For example for the following C code,*

```c
int sum(int a, int b, int c, int d, int e, int f, int g);
sum(1, 2, 3, 4, 5, 6, 7);
```

Arguments look like the following:

```nasm
; Registers
RDI = 1   (arg1)
RSI = 2   (arg2)
RDX = 3   (arg3)
RCX = 4   (arg4)
R8  = 5   (arg5)
R9  = 6   (arg6)

; Extra args (7th onwards) → Stack
[ RSP+  0 ]  return address  (set by CALL)
[ RSP+  8 ]  arg7 (value 7)
[ RSP+ 16 ]  arg8 if any...
```

The stack has to be 16-byte aligned *before* the `call` instruction. The caller is responsible for setting that alignment. When the callee runs, it usually saves **RBP** as a frame pointer (with `push rbp` / `mov rbp, rsp`) and may push other registers it wants to keep.

Registers are classified as **caller-saved** (RAX, RCX, RDX, RSI, RDI, R8–R11 — the caller must save them if it needs them after the call) and **callee-saved** (RBX, RBP, R12–R15 — the callee must restore them before returning).

When the function finishes, it puts the result in RAX and executes `ret`, which pops the return address from the stack and jumps back to the caller.

### *The prologue and the epilogue*

1. CPU pushes the *return address* on the stack (`RSP` decreases by 8).
2. Jumps to `sum`.

Inside `sum`:

- **Prologue** (set up stack frame):

```nasm
push rbp        ; save old base pointer
mov  rbp, rsp   ; set new base pointer
sub  rsp, 32    ; reserve local space (must keep 16-byte align)
; Now `rbp` is the anchor point for locals & saved registers.
```

- **Epilogue** (restore stack frame):

```nasm
mov rsp, rbp
pop rbp
ret ; `ret` pops the return address from stack → execution continues in caller.
```

**Example in x86-64 Linux:**

```nasm
section .text
global _start

; long add(long a, long b, long c)
; args: a=RDI, b=RSI, c=RDX
; ret:  RAX = a + b + c
add:
    push rbp
    mov  rbp, rsp
    
    mov  rax, rdi
    add  rax, rsi
    add  rax, rdx
    
    pop  rbp
    ret

_start:
    ; a = 5, b = 10
    
    push rbp
    mov  rbp, rsp
    sub  rsp, 24   ; 8 for local 'c' + 16 padding → RSP%16=8 before call
    
    mov  rdi, 5
    mov  rsi, 10
    mov  qword [rbp-8], 20

    mov  rdx, [rbp-8]
    call add                  ; RAX = a+b+c
    
    leave                     ; mov rsp, rbp / pop rbp

    ; exit(0)
    mov  rax, 60
    xor  rdi, rdi
    syscall

```

### *Why did we subtract `24` from `rsp`?*

The SysV ABI requires before a `call` instruction, `rsp % 16 == 8`. This is because the CPU will push the return address (8 bytes) onto the stack inside the call instruction, so the callee will see `rsp % 16 == 0`(16-byte aligned).

At `_start`:

- Linux places `rsp` aligned to 16 bytes **before** `_start` runs.
- After `push rbp`, alignment becomes **off by 8** (because we subtracted 8).

So we need **8 bytes** for `c` (`[rbp-8]`) and **16 bytes** extra padding so that when we hit the `call` instruction, `rsp % 16 == 8`.  Thats 24 bytes total.

# Console Output

## **System calls**

System calls are the gateway between user-space programs and the kernel, allowing applications to request services like file I/O or process creation from the operating system.

In 32-bit x86 Linux, system calls are made using the `int 0x80` software interrupt instruction. The system call number is placed in the `EAX` register. 64-bit Linux uses the more efficient `syscall` instruction instead of `int 0x80`. The system call number is put in the `RAX` register, and arguments are passed in registers such as `*DI`, `*SI`, and `*DX`,  The `syscall` instruction directly enters kernel mode, bypassing the interrupt vector used by `int 0x80` which switches the CPU to kernel mode, where the kernel uses the value in `EAX` to find the correct system call handler. The return value is placed in `*AX` by the kernel.

refer to [https://x86.syscall.sh/](https://x86.syscall.sh/) manual for more information on x86 and x64 syscalls.

```nasm
sys_write equ 4h
sys_read equ 3h
stdout equ 1h ; file descriptor 1 (standard output).
stdin equ 0h ; file descriptor 0 (standard input).

section .data
	welcomeString db "hello world", 0x0A
	.lengthof equ $-welcomeString
	
section .text
	global _start
_start :
	mov eax, sys_write
	mov ebx,stdout
	mov ecx, welcomeString
	mov edx, welcomeString.lengthof
	int 80h ; poke the kernel	
	
	nop
	mov eax, 1
	mov ebx, 0
	int 80h
```

For the 32-bit version, the read and write syscalls are different from 64-bit as it takes 3 an 4 respectively instead of 0 and 1.

| rax | System call | rdi | rsi | rdx |
| --- | --- | --- | --- | --- |
| 3 | sys_read | unsigned int fd | char *buf | size_t count |
| 4 | sys_write | unsigned int fd | const char *buf | size_t count |

### **Modern Direct Syscall Example** (x86-64)

If you want to *stay low-level* but use the modern convention:

```nasm
section .data
    msg db "Hello, world!", 0x0A
    len equ $ - msg

section .text
    global _start

_start:
    ; write(fd=1, buf=msg, count=len)
    mov rax, 1          ; syscall number for sys_write
    mov rdi, 1          ; fd = 1 = stdout
    mov rsi, msg        ; pointer to string
    mov rdx, len        ; length of string
    syscall

    ; exit(0)
    mov rax, 60         ; syscall number for exit
    xor rdi, rdi        ; status = 0
    syscall
```

### Why this is “better” now

- **`int 0x80`** is only guaranteed for 32-bit Linux, and on 64-bit it runs in compatibility mode (slower).
- **`syscall`** is the 64-bit native way and follows the System V AMD64 ABI. This avoids the deprecated stuff and matches what modern Linux kernels expect.

| rax | System call | rdi | rsi | rdx |
| --- | --- | --- | --- | --- |
| 0 | sys_read | unsigned int fd | char *buf | size_t count |
| 1 | sys_write | unsigned int fd | const char *buf | size_t count |

the file descriptor integers that go inside the `rdi` register are

- **`0`** → `stdin` (standard input)
- **`1`** → `stdout` (standard output)
- **`2`** → `stderr` (standard error)

## `printf()` via Linux userspace assembly :

This is specially useful for minor debugging as we don’t have to fire up gdb every time. Also you can actually call **any** C library function from assembly as long as you follow the calling convention 

**calling conventions**

- **x86 (cdecl)**: Push args on stack **right to left**, then `call` function, then clean up stack.
- **x86-64 (SysV ABI)**: First six integer args go in `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`; floats in `xmm0+`.

```nasm

extern printf

section .data
    format_string db "The value is: %d", 0x0A ; Format string with newline

section .text
    global _start

_start:
    mov rdi, format_string ; First argument: format string
    mov rsi, 123     ; Second argument: integer value (moved to esi)
    xor rax, rax           ; Set al to 0 as no floating-point arguments

    call printf ; Call printf

    ; Exit the program
    mov rax, 60            ; syscall number for exit
    xor rdi, rdi           ; exit code 0
    syscall
```

- **Compiling it** : First we need to use `-lc` , this tells `ld` to link with libc (GNU C Library). (`printf` is inside `libc.so.6` (shared library), so `-lc` ensures it’s included.) And then `-I /lib64/ld-linux-x86-64.so.2` to dynamically load shared libraries ( like `libc.so.6`, `libm.so.6`, etc.)

```bash
nasm -f elf64 print_asm.asm -o print_asm.o
ld print_asm.o -lc -o print_asm -I /lib64/ld-linux-x86-64.so.2
```

# Console Input

Example of reading a string from the screen and echo-ing it back.

```nasm
section .bss
    buffer resb 64    ; reserve 64 bytes for input

section .text
    global _start

_start:
    ; read(fd=0, buf=buffer, count=64)
    mov rax, 0        ; syscall number for read
    mov rdi, 0        ; fd = stdin
    mov rsi, buffer   ; pointer to buffer
    mov rdx, 64       ; max bytes to read
    syscall

    ; write(fd=1, buf=buffer, count=rax)
    mov rdx, rax      ; number of bytes read
    mov rax, 1        ; syscall number for write
    mov rdi, 1        ; fd = stdout
    mov rsi, buffer   ; pointer to buffer
    syscall

    ; exit(0)
    mov rax, 60
    xor rdi, rdi
    syscall
```

For reading integer values we can use two methods, firstly we can treat the integer input as an string input, then convert the string input to an integer, and store it in a register. This requires basic understanding of strings in assembly. In another way we can just use the `scanf` function from the linux userspace.

### Integer Input via String Operations :

```nasm
section .bss
    input resb 20            ; reserve 20 bytes for the input string

section .text
    global _start

_start:
    ; sys_read(stdin=0, buf=input, count=20)
    mov     rax, 0           ; sys_read
    mov     rdi, 0           ; stdin
    mov     rsi, input       ; buffer address
    mov     rdx, 20          ; max bytes
    syscall

    ; ASCII to integer conversion
    mov     rsi, input       ; point to buffer
    xor     rbx, rbx         ; store integer here (result)
.convert_loop:
    mov     al, [rsi]        ; load next char
    cmp     al, 10           ; newline?
    je      .done_convert
    sub     al, '0'          ; convert ASCII to digit
    imul    rbx, rbx, 10     ; multiply previous value by 10
    add     rbx, rax         ; add digit
    inc     rsi
    jmp     .convert_loop

.done_convert:
    ; rbx now contains the integer value

    ; exit(0)
    mov     rax, 60
    xor     rdi, rdi
    syscall
```

### Integer Input via `scanf` :

```nasm
extern scanf
section .data
    fmt db "%d", 0x0A     ; scanf format string for integer

section .bss
    number resq 1         ; reserve space for integer

section .text
    global _start

_start:
    ; call scanf("%d", &number)
    mov     rdi, fmt      ; 1st arg: format string
    lea     rsi, [number] ; 2nd arg: address to store the int
    xor     eax, eax      ; scanf needs AL=0 for varargs
    call    scanf

    ; exit(0)
    mov     eax, 60
    xor     edi, edi
    syscall
```
