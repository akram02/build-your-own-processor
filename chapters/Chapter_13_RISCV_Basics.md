# 🚀 Chapter 13: Build Your Own RISC-V Processor - ISA Fundamentals
## From Toy CPU to Real Architecture - RISC-V RV32I!

> **"8-bit was learning. RISC-V is professional. Time to build REAL processors!"**
>
> **"8-bit ছিল শেখা। RISC-V professional। এবার REAL processor বানাও!"**

---

## 🎯 এই Chapter এ তুমি শিখবে:

```
✅ RISC-V Overview - open ISA
✅ RV32I Base Integer - 32-bit core
✅ Instruction Formats - 6 types
✅ Register Set - 32 registers
✅ Instruction Set - 47 instructions
✅ Addressing Modes - memory access
✅ Calling Convention - function calls
✅ তোমার RISC-V processor এর specification! 🎉
```

**Time Required:** 1 week (5-6 hours/day)  
**Prerequisites:** Chapter 12 complete

---

## 🚀 Quick Understanding - 10 মিনিটে RISC-V!

### What is RISC-V?

```
RISC-V = UC Berkeley-র পঞ্চম (5th) RISC ISA (RISC-I, RISC-II, SOAR, SPUR, তারপর RISC-V)

Key facts:
✅ Open source (free to use!)
✅ Modular design (pick what you need)
✅ Clean, simple ISA
✅ Growing rapidly
✅ Industry adoption (Google, NVIDIA, etc.)

Pronunciation: "Risk Five"

Why RISC-V?
- No licensing fees (unlike ARM, x86)
- Educational friendly
- Research friendly
- Production ready
- Future-proof

Perfect for learning!
```

### RISC-V Modularity:

```
Base ISA (required):
RV32I  - 32-bit integer
RV64I  - 64-bit integer
RV128I - 128-bit integer

Extensions (optional):
M - Multiply/Divide
A - Atomic operations
F - Single-precision floating point
D - Double-precision floating point
C - Compressed instructions
V - Vector operations

We'll implement: RV32I
(Base 32-bit, enough for complete processor!)
```

### RV32I at a Glance:

```
Data width: 32-bit
Address width: 32-bit (4GB memory)
Registers: 32 × 32-bit
Instructions: 47 base instructions
Instruction size: 32-bit (fixed)

Instruction types:
- Arithmetic (ADD, SUB, etc.)
- Logical (AND, OR, XOR)
- Shifts (SLL, SRL, SRA)
- Branches (BEQ, BNE, etc.)
- Loads (LW, LH, LB)
- Stores (SW, SH, SB)
- Jumps (JAL, JALR)
- System (ECALL, EBREAK)
```

🎉 **Now you know what RISC-V is!**

---

## ১৩.১ RISC-V Philosophy

### Design Principles:

```
1. Simplicity
   - Clean, orthogonal ISA
   - Easy to understand
   - Easy to implement

2. Modularity
   - Base + Extensions
   - Only pay for what you use
   - Scalable

3. Stability
   - Base never changes
   - Extensions frozen when ratified
   - Software compatibility

4. Openness
   - Free to use
   - Open specification
   - Community driven

5. Extensibility
   - Custom extensions allowed
   - Standard extensions defined
   - Room for innovation
```

### RISC-V vs Others:

```
┌──────────┬────────┬────────┬─────────┐
│ Feature  │ RISC-V │  ARM   │   x86   │
├──────────┼────────┼────────┼─────────┤
│ License  │ Free   │ $$$    │ $$$     │
│ ISA      │ Open   │ Closed │ Closed  │
│ Simplicity│ High  │ Medium │ Low     │
│ Modular  │ Yes    │ No     │ No      │
│ Education│ Best   │ Good   │ Complex │
│ Industry │ Growing│ Mature │ Mature  │
│ Custom   │ Easy   │ Hard   │ No      │
└──────────┴────────┴────────┴─────────┘

RISC-V perfect for:
✅ Learning
✅ Research
✅ Custom processors
✅ IoT/Embedded
✅ Future projects
```

---

## ১৩.২ Register Set - 32 Registers

### Register Overview:

```
32 registers: x0 to x31
Each: 32 bits wide
Total: 1024 bits storage

x0: Always zero (hardwired)
x1-x31: General purpose

ABI Names:
x0  = zero  (always 0)
x1  = ra    (return address)
x2  = sp    (stack pointer)
x3  = gp    (global pointer)
x4  = tp    (thread pointer)
x5-x7 = t0-t2 (temporaries)
x8  = s0/fp (saved/frame pointer)
x9  = s1    (saved)
x10-x11 = a0-a1 (arguments/return)
x12-x17 = a2-a7 (arguments)
x18-x27 = s2-s11 (saved)
x28-x31 = t3-t6 (temporaries)
```

### Register Table:

```
┌──────┬───────┬─────────────────────────────┐
│ Reg  │ ABI   │ Description                 │
├──────┼───────┼─────────────────────────────┤
│ x0   │ zero  │ Hardwired zero             │
│ x1   │ ra    │ Return address             │
│ x2   │ sp    │ Stack pointer              │
│ x3   │ gp    │ Global pointer             │
│ x4   │ tp    │ Thread pointer             │
│ x5-7 │ t0-2  │ Temporaries                │
│ x8   │ s0/fp │ Saved register/Frame ptr   │
│ x9   │ s1    │ Saved register             │
│ x10-11│ a0-1 │ Function args/return values│
│ x12-17│ a2-7 │ Function arguments         │
│ x18-27│ s2-11│ Saved registers            │
│ x28-31│ t3-6 │ Temporaries                │
└──────┴───────┴─────────────────────────────┘

Caller-saved: t0-t6, a0-a7
Callee-saved: s0-s11
```

### x0 (zero register):

```
Special property:
- Always reads as 0
- Writes are discarded

Uses:
1. Generate constant 0
   addi x5, x0, 0  # x5 = 0

2. Discard results
   add x0, x5, x6  # Compute but throw away

3. Comparisons
   beq x5, x0, label  # Branch if x5 == 0

4. NOP instruction
   addi x0, x0, 0  # Do nothing

Hardware: Just wire to 0!
```

---

## ১৩.৩ Instruction Formats - 6 Types

### Format Overview:

```
All instructions: 32 bits
6 formats: R, I, S, B, U, J

Common fields:
- opcode: [6:0] (7 bits) - Operation type
- rd: [11:7] (5 bits) - Destination register
- rs1: [19:15] (5 bits) - Source register 1
- rs2: [24:20] (5 bits) - Source register 2
- funct3: [14:12] (3 bits) - Function modifier
- funct7: [31:25] (7 bits) - Function modifier
```

### R-Type (Register):

```
Used for: Register-to-register operations

Format:
31        25 24    20 19    15 14  12 11     7 6      0
┌───────────┬────────┬────────┬──────┬────────┬────────┐
│  funct7   │  rs2   │  rs1   │funct3│   rd   │ opcode │
└───────────┴────────┴────────┴──────┴────────┴────────┘
  7 bits     5 bits   5 bits  3 bits  5 bits   7 bits

Examples:
- ADD, SUB, AND, OR, XOR
- SLL, SRL, SRA
- SLT, SLTU

Example: ADD x5, x6, x7
- funct7 = 0000000
- rs2 = x7 (00111)
- rs1 = x6 (00110)
- funct3 = 000
- rd = x5 (00101)
- opcode = 0110011
```

### I-Type (Immediate):

```
Used for: Register-immediate operations, loads

Format:
31                    20 19    15 14  12 11     7 6      0
┌───────────────────────┬────────┬──────┬────────┬────────┐
│    imm[11:0]          │  rs1   │funct3│   rd   │ opcode │
└───────────────────────┴────────┴──────┴────────┴────────┘
     12 bits             5 bits  3 bits  5 bits   7 bits

Examples:
- ADDI, ANDI, ORI, XORI
- SLTI, SLTIU
- LW, LH, LB, LHU, LBU
- JALR

Immediate: Sign-extended to 32 bits

Example: ADDI x5, x6, 10
- imm = 000000001010
- rs1 = x6
- funct3 = 000
- rd = x5
- opcode = 0010011
```

### S-Type (Store):

```
Used for: Store instructions

Format:
31        25 24    20 19    15 14  12 11     7 6      0
┌───────────┬────────┬────────┬──────┬────────┬────────┐
│ imm[11:5] │  rs2   │  rs1   │funct3│imm[4:0]│ opcode │
└───────────┴────────┴────────┴──────┴────────┴────────┘
  7 bits     5 bits   5 bits  3 bits  5 bits   7 bits

Immediate split: [11:5] and [4:0]
Why? Keep rs1, rs2, funct3 in same position

Examples:
- SW, SH, SB

Example: SW x7, 8(x6)
- imm[11:5] = 0000000
- rs2 = x7 (data)
- rs1 = x6 (base)
- funct3 = 010
- imm[4:0] = 01000
- opcode = 0100011
```

### B-Type (Branch):

```
Used for: Conditional branches

Format:
31   30        25 24    20 19    15 14  12 11   8 7   6      0
┌─────┬───────────┬────────┬────────┬──────┬──────┬─┬────────┐
│imm12│ imm[10:5] │  rs2   │  rs1   │funct3│imm   │0│ opcode │
│     │           │        │        │      │[4:1] │ │        │
└─────┴───────────┴────────┴────────┴──────┴──────┴─┴────────┘
 1bit   6 bits     5 bits   5 bits  3 bits  4bits 1  7 bits

Immediate: PC-relative byte offset; imm[0] সবসময় 0 (2-byte aligned)
Range: ±4KB

Examples:
- BEQ, BNE, BLT, BGE, BLTU, BGEU

Note: imm[0] always 0 (halfword aligned)
      Reconstructed: {imm[12], imm[10:5], imm[4:1], 0}
```

### U-Type (Upper immediate):

```
Used for: Large immediate values

Format:
31                                    12 11     7 6      0
┌──────────────────────────────────────┬────────┬────────┐
│            imm[31:12]                │   rd   │ opcode │
└──────────────────────────────────────┴────────┴────────┘
           20 bits                      5 bits   7 bits

Immediate placed in upper 20 bits
Lower 12 bits = 0

Examples:
- LUI (Load Upper Immediate)
- AUIPC (Add Upper Immediate to PC)

Example: LUI x5, 0x12345
Result: x5 = 0x12345000
```

### J-Type (Jump):

```
Used for: Unconditional jumps

Format:
31   30      21 20  19        12 11     7 6      0
┌─────┬──────────┬──┬───────────┬────────┬────────┐
│imm20│imm[10:1] │11│ imm[19:12]│   rd   │ opcode │
└─────┴──────────┴──┴───────────┴────────┴────────┘
 1bit   10 bits   1    8 bits     5 bits   7 bits

Immediate: PC-relative byte offset; imm[0] সবসময় 0 (2-byte aligned)
Range: ±1MB

Example:
- JAL (Jump and Link)

Note: Stores PC+4 in rd
      Immediate encoding scrambled (for efficient layout)
```

---

## ১৩.৪ RV32I Instruction Set - Complete

### Arithmetic Instructions:

```
ADD   rd, rs1, rs2    # rd = rs1 + rs2
ADDI  rd, rs1, imm    # rd = rs1 + imm
SUB   rd, rs1, rs2    # rd = rs1 - rs2

Note: No SUBI (use ADDI with negative immediate)

Examples:
ADD  x5, x6, x7       # x5 = x6 + x7
ADDI x5, x6, 10       # x5 = x6 + 10
SUB  x5, x6, x7       # x5 = x6 - x7
ADDI x5, x5, -5       # x5 = x5 - 5
```

### Logical Instructions:

```
AND   rd, rs1, rs2    # rd = rs1 & rs2
ANDI  rd, rs1, imm    # rd = rs1 & imm
OR    rd, rs1, rs2    # rd = rs1 | rs2
ORI   rd, rs1, imm    # rd = rs1 | imm
XOR   rd, rs1, rs2    # rd = rs1 ^ rs2
XORI  rd, rs1, imm    # rd = rs1 ^ imm

Examples:
AND  x5, x6, x7       # Bitwise AND
ORI  x5, x6, 0xFF     # Set lower 8 bits
XOR  x5, x5, x5       # Clear register (x5 = 0)
```

### Shift Instructions:

```
SLL   rd, rs1, rs2    # rd = rs1 << rs2[4:0]
SLLI  rd, rs1, shamt  # rd = rs1 << shamt
SRL   rd, rs1, rs2    # rd = rs1 >> rs2[4:0] (logical)
SRLI  rd, rs1, shamt  # rd = rs1 >> shamt (logical)
SRA   rd, rs1, rs2    # rd = rs1 >> rs2[4:0] (arithmetic)
SRAI  rd, rs1, shamt  # rd = rs1 >> shamt (arithmetic)

Shift amount: 5 bits (0-31)

Examples:
SLLI x5, x6, 2        # x5 = x6 << 2 (multiply by 4)
SRLI x5, x6, 3        # x5 = x6 >> 3 (divide by 8)
SRAI x5, x6, 4        # Arithmetic right shift (sign extend)
```

### Compare Instructions:

```
SLT   rd, rs1, rs2    # rd = (rs1 < rs2) ? 1 : 0 (signed)
SLTI  rd, rs1, imm    # rd = (rs1 < imm) ? 1 : 0 (signed)
SLTU  rd, rs1, rs2    # rd = (rs1 < rs2) ? 1 : 0 (unsigned)
SLTIU rd, rs1, imm    # rd = (rs1 < imm) ? 1 : 0 (unsigned)

Set if Less Than

Examples:
SLT  x5, x6, x7       # x5 = 1 if x6 < x7
SLTI x5, x6, 10       # x5 = 1 if x6 < 10
```

### Branch Instructions:

```
BEQ  rs1, rs2, offset # Branch if rs1 == rs2
BNE  rs1, rs2, offset # Branch if rs1 != rs2
BLT  rs1, rs2, offset # Branch if rs1 < rs2 (signed)
BGE  rs1, rs2, offset # Branch if rs1 >= rs2 (signed)
BLTU rs1, rs2, offset # Branch if rs1 < rs2 (unsigned)
BGEU rs1, rs2, offset # Branch if rs1 >= rs2 (unsigned)

PC-relative addressing
Range: ±4KB

Examples:
BEQ  x5, x6, loop     # if (x5 == x6) goto loop
BNE  x5, x0, skip     # if (x5 != 0) goto skip
BLT  x5, x6, less     # if (x5 < x6) goto less
```

### Jump Instructions:

```
JAL  rd, offset       # rd = PC+4; PC = PC + offset
JALR rd, rs1, offset  # rd = PC+4; PC = rs1 + offset

JAL: Jump and Link (function calls)
JALR: Jump and Link Register (return, indirect)

Examples:
JAL  x1, function     # Call function, save return in x1(ra)
JALR x0, x1, 0        # Return (jump to address in x1)
JAL  x0, label        # Unconditional jump (discard return)
```

### Load Instructions:

```
LW   rd, offset(rs1)  # Load Word (32-bit)
LH   rd, offset(rs1)  # Load Halfword (16-bit, sign extend)
LHU  rd, offset(rs1)  # Load Halfword Unsigned
LB   rd, offset(rs1)  # Load Byte (8-bit, sign extend)
LBU  rd, offset(rs1)  # Load Byte Unsigned

Address = rs1 + offset
offset: 12-bit signed

Examples:
LW   x5, 0(x6)        # x5 = memory[x6]
LW   x5, 8(x6)        # x5 = memory[x6 + 8]
LH   x5, 4(x6)        # Load 16-bit, sign extend
LBU  x5, 0(x6)        # Load byte, zero extend
```

### Store Instructions:

```
SW  rs2, offset(rs1)  # Store Word
SH  rs2, offset(rs1)  # Store Halfword
SB  rs2, offset(rs1)  # Store Byte

Address = rs1 + offset
Data from rs2

Examples:
SW  x7, 0(x6)         # memory[x6] = x7
SW  x7, 12(x6)        # memory[x6 + 12] = x7
SH  x7, 2(x6)         # Store 16-bit
SB  x7, 0(x6)         # Store byte
```

### Upper Immediate:

```
LUI   rd, imm         # rd = imm << 12
AUIPC rd, imm         # rd = PC + (imm << 12)

LUI: Load Upper Immediate
AUIPC: Add Upper Immediate to PC

Examples:
LUI   x5, 0x12345     # x5 = 0x12345000
AUIPC x5, 0x1000      # x5 = PC + 0x01000000

Used for:
- Loading large constants (LUI + ADDI)
- PC-relative addressing (AUIPC + offset)
```

### System Instructions:

```
ECALL                 # Environment call (syscall)
EBREAK                # Environment break (debugger)

ECALL: Transfer to OS
EBREAK: Transfer to debugger

Examples:
ECALL                 # Make system call
EBREAK                # Breakpoint
```

### Pseudo-Instructions:

```
Not real instructions, but assembler shortcuts:

NOP              → ADDI x0, x0, 0
MV   rd, rs1     → ADDI rd, rs1, 0
NOT  rd, rs1     → XORI rd, rs1, -1
NEG  rd, rs1     → SUB  rd, x0, rs1
LI   rd, imm     → LUI + ADDI (for large imm)
LA   rd, label   → AUIPC + ADDI
J    offset      → JAL x0, offset
JR   rs1         → JALR x0, rs1, 0
RET              → JALR x0, x1, 0
CALL offset      → JAL x1, offset
```

---

## ১৩.৫ Addressing Modes

### 1. Register Direct:

```
Operation uses register contents directly

ADD x5, x6, x7    # x5 = x6 + x7
```

### 2. Immediate:

```
Operation uses constant value

ADDI x5, x6, 10   # x5 = x6 + 10
```

### 3. Base + Displacement:

```
Memory address = register + offset

LW x5, 8(x6)      # x5 = memory[x6 + 8]
SW x7, 12(x6)     # memory[x6 + 12] = x7
```

### 4. PC-Relative:

```
Target address = PC + offset

BEQ x5, x6, label # if equal, goto PC + offset
JAL x1, function  # call PC + offset
```

### 5. Register Indirect:

```
Target address = register + offset

JALR x0, x1, 0    # goto address in x1
```

---

## ১৩.৬ Calling Convention

### Function Call Example:

```assembly
# Caller
main:
    addi sp, sp, -4      # Allocate stack
    sw   x1, 0(sp)       # Save return address
    
    li   a0, 5           # First argument
    li   a1, 3           # Second argument
    jal  x1, add_func    # Call function
    
    # Result in a0
    
    lw   x1, 0(sp)       # Restore return address
    addi sp, sp, 4       # Deallocate stack
    ret                  # Return

# Callee
add_func:
    add  a0, a0, a1      # a0 = a0 + a1
    ret                  # Return (result in a0)
```

### Register Usage:

```
Arguments: a0-a7 (x10-x17)
Return values: a0-a1 (x10-x11)
Return address: ra (x1)
Stack pointer: sp (x2)
Frame pointer: fp/s0 (x8)

Temporaries: t0-t6 (caller-saved)
Saved: s0-s11 (callee-saved)
```

---

## ১৩.৭ Your 1-Week Build Plan

### Day 1: ISA Study
```
□ Read RISC-V spec (relevant parts)
□ Understand all instruction formats
□ Study each instruction
□ Create reference sheet
```

### Day 2: Instruction Decoder
```
□ Design decoder logic
□ Extract fields from instruction
□ Generate control signals
□ Test with examples
```

### Day 3: Register File
```
□ 32-register implementation
□ x0 = 0 enforcement
□ Dual-port read
□ Single-port write
```

### Day 4: ALU Design
```
□ All arithmetic operations
□ All logical operations
□ Shifts (SLL, SRL, SRA)
□ Comparisons
```

### Day 5: Branch Logic
```
□ Branch comparators
□ PC calculation
□ Target address generation
□ Test branches
```

### Day 6: Memory Interface
```
□ Load unit (LW, LH, LB, LHU, LBU)
□ Store unit (SW, SH, SB)
□ Address calculation
□ Byte/halfword alignment
```

### Day 7: Integration
```
□ Connect all components
□ Control unit
□ Test simple programs
□ Debug issues
```

---

## ১৩.৮ Chapter 13 Mission Complete!

### তুমি এখন জানো:

```
✅ RISC-V philosophy and design
✅ RV32I complete instruction set
✅ 6 instruction formats
✅ 32 register conventions
✅ All 47 base instructions
✅ Addressing modes
✅ Calling conventions
✅ Ready to implement RISC-V! 🎉
```

### তুমি শিখেছো:
```
✅ Industry-standard ISA
✅ Professional architecture
✅ Complete specification
✅ Real-world design
✅ Production-ready knowledge
✅ RISC-V expertise! 🏆
```

### Stats:
```
Instructions learned: 47
Formats mastered: 6
Registers: 32
ISA: Industry standard
Level: RISC-V Architect! 🚀
```

### Next Level Unlocked:
```
→ Chapter 14: Single-Cycle RISC-V Processor
   তুমি বানাবে: Complete RV32I CPU!
   1 cycle per instruction!
   
   From ISA → Real processor!
```

---

## 🎯 Final Project

### Project: RISC-V Assembly Programs

**Write and test:**
```
✅ Factorial function (recursion)
✅ Fibonacci sequence
✅ String copy
✅ Bubble sort
✅ Matrix multiplication

Use:
- All instruction types
- Function calls
- Stack operations
- Loops and branches
```

---

## 🏆 Achievement Unlocked!

```
Level 13: ✅ COMPLETE - RISC-V Expert!
Progress: [███████████████████████████████] 65%

XP Gained: +4000
Skills: RISC-V ISA, Professional Architecture

Badges Earned:
🥉 Instruction Format Master
🥈 ISA Expert
🥇 RISC-V Programmer
🏅 Architecture Specialist
🎖️ Industry Standard
🏆 Professional CPU Architect

Next: Chapter 14 - Build Complete RISC-V CPU!
      Single-cycle processor! 💻
```

---

**[⬅️ Previous: Chapter 12](Chapter_12_Processor_Architecture.md)** | **[➡️ Next: Chapter 14](Chapter_14_Single_Cycle_CPU.md)**

---

<div align="center">

**"You know RISC-V. Now BUILD the processor!"**

**"তুমি RISC-V জানো। এবার processor বানাও!"**

Made with ❤️ for builders | বানানোর জন্য ভালোবাসা দিয়ে তৈরি

</div>
