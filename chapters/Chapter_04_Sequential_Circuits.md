# 🔄 Chapter 4: Build Your Own Memory Circuits
## Flip-Flops থেকে FSM - তোমার Processor কে Memory দাও!

> **"Combinational circuits compute. Sequential circuits remember. Time to add memory!"**
>
> **"Combinational circuits হিসাব করে। Sequential circuits মনে রাখে। এবার memory যোগ করো!"**

---

## 🎯 এই Chapter এ তুমি বানাবে:

```
✅ SR Latch - তোমার প্রথম memory element
✅ D Latch - data storage
✅ D Flip-Flop - edge-triggered memory
✅ JK Flip-Flop - universal flip-flop
✅ T Flip-Flop - toggle flip-flop
✅ 4-bit Register - data storage
✅ Shift Register - data mover
✅ Counter - number generator
✅ Finite State Machine - তোমার processor এর control! 🎉
```

**Time Required:** 2 weeks (3-4 hours/day)  
**Tools Needed:** CircuitVerse, Paper, Timing Diagrams

---

## 🚀 Quick Win - 5 মিনিটে তোমার First Memory!

### এখনই বানাও - SR Latch:

**যাও CircuitVerse.org এ এবং:**

```
Components:
- 2 × NOR gates (বা 2 × NAND gates)
- 2 × Buttons (S, R)
- 2 × LEDs (Q, Q')

Circuit (Using NOR):
        S ──┐
            ├─[NOR]──┬── Q (output)
        ┌───┘        │
        │            │
        │            └──┐
        │               │
        └───[NOR]───────┤
            ├───────────┘
        R ──┘        
                     └── Q' (complement)

Test:
1. Press S → Q=1 (SET) ✓
2. Release S → Q stays 1 (MEMORY!) ✓✓
3. Press R → Q=0 (RESET) ✓
4. Release R → Q stays 0 (MEMORY!) ✓✓
```

🎉 **Congratulations! তুমি একটা memory element বানিয়েছো!**

**এটাই তোমার processor এর register এর foundation!**

---

## ৪.১ Sequential Circuits কী?

### Combinational vs Sequential:

```
COMBINATIONAL (Chapter 3):
Input → [Logic] → Output
        No Memory!
Output = f(Current Input)

SEQUENTIAL (This Chapter):
Input ──→ [Logic] ──→ Output
   ↑        │    ↓
   └───── [Memory] ──┘
         
Output = f(Current Input, Previous State)
```

### কেন Sequential?

```
Processor এ দরকার:
✅ Registers - data store করতে
✅ Program Counter - next instruction track করতে
✅ State machines - control logic এর জন্য
✅ Memory - data remember করতে

Without Sequential = No Processor!
```

### Clock Signal - The Heartbeat:

```
Clock (CLK):
     ┌───┐   ┌───┐   ┌───┐
     │   │   │   │   │   │
─────┘   └───┘   └───┘   └───

Everything happens at clock edges!

Rising Edge:  ─┐ (0→1)
              └
              
Falling Edge:   ┌─ (1→0)
              ──┘
```

---

## ৪.২ Build SR Latch - Foundation of Memory

### কিভাবে কাজ করে?

**SR Latch = Set-Reset Latch**

```
Inputs:
- S (Set): Makes Q = 1
- R (Reset): Makes Q = 0

Output:
- Q: Main output
- Q': Complement (always opposite of Q)
```

### Using NOR Gates:

**Circuit:**
```
    S ──┐
        ├─[NOR]──┬── Q
    ┌───┘        │
    │            │
    │         feedback
    │            │
    └───[NOR]────┤
        ├────────┘
    R ──┘        
                └── Q'
```

**Truth Table:**
```
┌───┬───┬────────┬─────────────────────┐
│ S │ R │   Q    │    Description      │
├───┼───┼────────┼─────────────────────┤
│ 0 │ 0 │  Hold  │  Remember previous  │ ← Memory!
│ 0 │ 1 │   0    │  Reset Q to 0       │
│ 1 │ 0 │   1    │  Set Q to 1         │
│ 1 │ 1 │  ???   │  Invalid! (avoid)   │
└───┴───┴────────┴─────────────────────┘

Invalid state: Both outputs try to be 0!
```

### Using NAND Gates:

**Circuit:**
```
    S ──┐
        ├─[NAND]──┬── Q
    ┌───┘         │
    │             │
    │             │
    │             │
    └───[NAND]────┤
        ├─────────┘
    R ──┘         
                 └── Q'

Note: NAND version - inputs are active LOW
```

### 🎯 Build & Test SR Latch:

**Build Both Versions:**
```
Task 1: NOR-based SR Latch
- Build circuit
- Test all 4 input combinations
- Observe memory behavior!

Task 2: NAND-based SR Latch  
- Build circuit
- Note: Active LOW inputs
- Compare with NOR version
```

**Test Sequence:**
```
1. S=0, R=0: Q = ? (check previous state)
2. S=1, R=0: Q = 1 (SET)
3. S=0, R=0: Q = 1 (HOLDS!) ✓✓
4. S=0, R=1: Q = 0 (RESET)
5. S=0, R=0: Q = 0 (HOLDS!) ✓✓
```

---

## ৪.৩ Build D Latch - Data Latch

### Problem with SR Latch:

```
❌ Two inputs (S, R) confusing
❌ Invalid state (S=1, R=1)
❌ Need simpler interface

Solution: D Latch!
✅ One data input (D)
✅ One enable input (E)
✅ No invalid states
```

### Circuit Design:

```
Data (D) ──┬────────[AND]───── S  ┐
           │            ↑         │
           │            E         │
           │                      ├─[SR Latch]── Q
           │                      │
           └─[NOT]─[AND]───── R  ┘
                       ↑
                       E

When E=1: Q follows D
When E=0: Q holds previous value
```

**Truth Table:**
```
┌───┬───┬────────┬─────────────────────┐
│ E │ D │   Q    │    Description      │
├───┼───┼────────┼─────────────────────┤
│ 0 │ X │  Hold  │  Ignore D, remember │
│ 1 │ 0 │   0    │  Q becomes 0        │
│ 1 │ 1 │   1    │  Q becomes 1        │
└───┴───┴────────┴─────────────────────┘

X = Don't care
```

### Timing Diagram:

```
     ┌───────┐       ┌───────
E ───┘       └───────┘

   ┌───┐       ┌─────┐
D ─┘   └───────┘     └────

       ┌───────────┐     ┌──
Q ─────┘           └─────┘

When E=1: Q follows D (transparent)
When E=0: Q freezes
```

### 🎯 Build D Latch:

**Step-by-step:**
```
1. Build SR Latch (from previous section)
2. Add NOT gate for D
3. Add 2 AND gates
4. Connect: D → AND → S, D'→ AND → R
5. Connect E to both ANDs
6. Test with timing sequence!
```

---

## ৪.৪ Build D Flip-Flop - Edge-Triggered Memory! 🎉

### Problem with D Latch:

```
D Latch (Level-triggered):
- When E=1: Q continuously follows D
- Problem: No precise control
- Data can change during E=1

Need: Edge-triggered!
- Only change at clock edge
- Precise timing control
```

### D Flip-Flop Solution:

```
Features:
✅ Changes only at clock edge (rising or falling)
✅ Ignores input at other times
✅ Perfect for registers!
✅ Synchronous operation
```

### Master-Slave D Flip-Flop:

**Circuit:**
```
           ┌──────────┐  ┌──────────┐
D ─────────┤  Master  ├──┤  Slave   ├──── Q
           │ D Latch  │  │ D Latch  │
CLK ───┬───┤ E        │  │ E        │
       │   └──────────┘  └──────────┘
       │        ↑             ↑
       │        │             │
       │       CLK           CLK'
       └─[NOT]──┘

Master active when CLK=1
Slave active when CLK=0
Output changes at falling edge!
```

**Operation:**
```
CLK high (1):
- Master latch transparent (captures D)
- Slave latch holds (outputs old value)

CLK low (0):  ← Falling Edge
- Master latch holds (captured value)
- Slave latch transparent (outputs to Q)

Result: Q changes at falling edge!
```

### Positive Edge-Triggered D Flip-Flop:

**Symbol:**
```
       ┌─────┐
D ─────┤     │
       │  D  ├──── Q
CLK ───┤>    │
       │     ├──── Q'
       └─────┘
       
> = Edge trigger
```

**Timing Diagram:**
```
CLK  ───┐   ┌───┐   ┌───┐
        └───┘   └───┘   └───
        ↑       ↑       ↑  (Trigger points)

D    ──┐   ┌───────┐
       └───┘       └───────

Q    ────┐       ┌─────────
         └───────┘
         ↑       ↑  (Changes at rising edges)
```

### 🎯 Build D Flip-Flop:

**Build Master-Slave:**
```
Components:
- 2 × D Latches (master & slave)
- 1 × NOT gate (for clock)
- 1 × Clock generator

Steps:
1. Build two D Latches
2. Connect CLK to master enable
3. Connect CLK' to slave enable
4. Connect master output to slave input
5. Test with clock pulses!
```

**Test Sequence:**
```
Time | CLK | D | Q | Notes
-----|-----|---|---|----------------------
  0  |  0  | 0 | X | Initial
  1  |  1  | 0 | X | Master captures 0
  2  |  0  | 0 | 0 | Slave outputs (edge!)
  3  |  1  | 1 | 0 | Master captures 1
  4  |  0  | 1 | 1 | Slave outputs (edge!)
  5  |  1  | 0 | 1 | Master captures 0
  6  |  0  | 0 | 0 | Slave outputs (edge!)
```

---

## ৪.৫ Build JK Flip-Flop - The Universal One!

### Why JK?

```
D Flip-Flop:
✅ Simple
❌ Can't toggle easily

JK Flip-Flop:
✅ Can SET
✅ Can RESET  
✅ Can TOGGLE
✅ Can HOLD
✅ Most versatile!
```

### Truth Table:

```
┌───┬───┬─────────┬──────────────────┐
│ J │ K │  Q(t+1) │   Operation      │
├───┼───┼─────────┼──────────────────┤
│ 0 │ 0 │   Q(t)  │  Hold (no change)│
│ 0 │ 1 │    0    │  Reset           │
│ 1 │ 0 │    1    │  Set             │
│ 1 │ 1 │  Q'(t)  │  Toggle          │ ← Special!
└───┴───┴─────────┴──────────────────┘

Q(t) = current state
Q(t+1) = next state after clock edge
```

### Circuit (from SR Flip-Flop):

```
J ──┬───[AND]───── S  ┐
    │      ↑          │
    │      │          │
    │    Q' (feedback)│
    │                 ├─[SR FF]── Q
    │                 │      ↑
    └──────────────────────Q'│
                        │     │
K ──┬───[AND]───── R  ┘     │
    │      ↑                 │
    │      │                 │
    │    Q (feedback) ←──────┘
    
CLK ──→ (to SR FF clock)
```

### Applications:

```
J=0, K=0: Hold → Memory storage
J=0, K=1: Reset → Clear register
J=1, K=0: Set → Load data
J=1, K=1: Toggle → Frequency divider, counters
```

### 🎯 Build JK Flip-Flop:

**Method 1: From SR FF**
```
1. Take SR flip-flop
2. Add feedback (Q to K's AND, Q' to J's AND)
3. Add AND gates for J and K inputs
4. Connect to SR inputs
5. Test all 4 modes!
```

**Test All Modes:**
```
Mode 1 - Hold:
J=0, K=0, Q=1 → After clock: Q=1 ✓

Mode 2 - Reset:
J=0, K=1, Q=1 → After clock: Q=0 ✓

Mode 3 - Set:
J=1, K=0, Q=0 → After clock: Q=1 ✓

Mode 4 - Toggle:
J=1, K=1, Q=0 → After clock: Q=1 ✓
J=1, K=1, Q=1 → After clock: Q=0 ✓
```

---

## ৪.৬ Build T Flip-Flop - The Toggle Master

### Simple Toggle!

```
T Flip-Flop = Toggle Flip-Flop
One input (T), one clock

T=0: Hold
T=1: Toggle (flip Q)
```

### Truth Table:

```
┌───┬─────────┬──────────────┐
│ T │  Q(t+1) │  Operation   │
├───┼─────────┼──────────────┤
│ 0 │   Q(t)  │  Hold        │
│ 1 │  Q'(t)  │  Toggle      │
└───┴─────────┴──────────────┘
```

### Build from JK:

```
       T ──┬── J  ┐
           │       │
           └── K  ├─[JK FF]── Q
                  │
CLK ───────────────┘

Simply connect J=K=T!
```

### Build from D:

```
       Q' ──┬──[XOR]── D  ┐
            │             │
       T ───┘             ├─[D FF]── Q
                          │
CLK ───────────────────────┘

D = T ⊕ Q
```

### Application - Frequency Divider:

```
Input:  ┌─┐ ┌─┐ ┌─┐ ┌─┐  (4 cycles)
CLK ────┘ └─┘ └─┘ └─┘ └─

T=1 (always toggle)

Output: ┌───────┐         (2 cycles)
Q ──────┘       └──────

Divides frequency by 2!
```

### 🎯 Build T Flip-Flop:

**Build Both Methods:**
```
1. From JK: Connect J=K=T
2. From D: Add XOR gate
3. Test both
4. Build frequency divider chain!
```

**Frequency Divider Chain:**
```
CLK ──[T-FF]──[T-FF]──[T-FF]──[T-FF]
       ÷2      ÷2      ÷2      ÷2
       
Total division: ÷16
```

---

## ৪.৭ Build Registers - Data Storage

### What's a Register?

```
Register = Collection of flip-flops
Stores multi-bit data

4-bit Register = 4 × D Flip-Flops
8-bit Register = 8 × D Flip-Flops
32-bit Register = 32 × D Flip-Flops (in your CPU!)
```

### 4-bit Register Design:

**Circuit:**
```
D3 ───[D FF]─── Q3  ┐
      CLK ↑         │
D2 ───[D FF]─── Q2  │
      CLK ↑         ├─ 4-bit Output
D1 ───[D FF]─── Q1  │
      CLK ↑         │
D0 ───[D FF]─── Q0  ┘
      CLK ↑
       │
Common CLK ────┘
```

**With Enable:**
```
        D3 ──┬──[MUX]──[D FF]── Q3
             │    ↑
         Q3 ─┘    │
                 EN
                 
When EN=1: Load new data
When EN=0: Hold current data

Apply to all 4 bits!
```

### Register with Parallel Load:

```
Features:
✅ Load all bits simultaneously
✅ Enable control
✅ Clear/Reset
```

**Complete 4-bit Register:**
```
             CLR (clear all)
              │
D3 ─[MUX]─[D FF]─ Q3
      ↑      ↑
D2 ─[MUX]─[D FF]─ Q2
      ↑      ↑
D1 ─[MUX]─[D FF]─ Q1
      ↑      ↑
D0 ─[MUX]─[D FF]─ Q0
      ↑      ↑
      │      │
     EN    CLK
```

### 🎯 Build 4-bit Register:

**Build Steps:**
```
1. Create 4 × D Flip-Flops
2. Connect common clock
3. Add enable logic (MUX for each bit)
4. Add clear functionality
5. Test load and hold!
```

**Test Cases:**
```
Test 1: Load 1010
EN=1, D=1010 → Clock → Q=1010 ✓

Test 2: Hold
EN=0, D=0101 → Clock → Q=1010 (unchanged) ✓

Test 3: Load new
EN=1, D=0101 → Clock → Q=0101 ✓

Test 4: Clear
CLR=1 → Q=0000 ✓
```

---

## ৪.৮ Build Shift Registers - Data Movers

### What's Shifting?

```
Shift Left:  1011 → 0110 (lose MSB, 0 enters LSB)
Shift Right: 1011 → 0101 (lose LSB, 0 enters MSB)

Used for:
✅ Serial communication (UART, SPI)
✅ Multiplication/Division by 2
✅ Data conversion
```

### Types of Shift Registers:

```
1. SISO: Serial In, Serial Out
2. SIPO: Serial In, Parallel Out
3. PISO: Parallel In, Serial Out
4. PIPO: Parallel In, Parallel Out (regular register)
```

---

### 4-bit SISO (Serial In, Serial Out):

**Circuit:**
```
Serial In ─[D FF]─[D FF]─[D FF]─[D FF]─ Serial Out
            Q0 →   Q1 →   Q2 →   Q3
             ↑      ↑      ↑      ↑
             └──────┴──────┴──────┘
                 Common CLK
```

**Operation:**
```
Initial: Q3 Q2 Q1 Q0 = 0000
Input: 1

Clock 1: 1000 (1 enters Q0)
Clock 2: 0100 (shifts right)
Clock 3: 0010
Clock 4: 0001
Clock 5: 0000 (1 exits at Serial Out)
```

---

### 4-bit SIPO (Serial In, Parallel Out):

**Circuit:**
```
Serial In ─[D FF]─[D FF]─[D FF]─[D FF]
            ↓      ↓      ↓      ↓
           Q0     Q1     Q2     Q3
            └──────┴──────┴──────┘
               Parallel Output
```

**Application:** Serial to Parallel converter (UART receiver)

---

### 4-bit PISO (Parallel In, Serial Out):

**Circuit:**
```
D3 ─[MUX]─[D FF]─[MUX]─[D FF]─[MUX]─[D FF]─[MUX]─[D FF]─ Serial Out
     ↑            ↑            ↑            ↑
     │            │            │            │
    LOAD/SHIFT control

LOAD=1: Parallel data loaded
LOAD=0: Shift right
```

**Application:** Parallel to Serial converter (UART transmitter)

---

### Universal Shift Register:

**Can do everything!**
```
Mode Control (2 bits):
00: Hold
01: Shift Left
10: Shift Right
11: Parallel Load
```

### 🎯 Build Shift Register Project:

**Build SIPO Register:**
```
1. Chain 4 D Flip-Flops
2. Common clock
3. Serial input to first FF
4. Parallel outputs from all FFs
5. Test serial data transmission!
```

**Test Serial Data:**
```
Send: 1101 (one bit per clock)

Clock 1: 1000
Clock 2: 0100  
Clock 3: 1010
Clock 4: 1101 ✓ (received!)
```

---

## ৪.৯ Build Counters - Number Generators

### What's a Counter?

```
Counter = Sequence generator
Generates binary sequences

Applications in processor:
✅ Program Counter (PC)
✅ Instruction cycles
✅ Loop counters
✅ Timers
```

### Types:

```
1. Asynchronous (Ripple) Counter
2. Synchronous Counter
3. Up Counter
4. Down Counter
5. Up/Down Counter
6. Modulo-N Counter
```

---

### 4-bit Asynchronous Up Counter:

**Circuit:**
```
         ┌─[T FF]─ Q0 (LSB)
         │   ↑
CLK ─────┘   │
             └─[T FF]─ Q1
                 ↑
                 └─[T FF]─ Q2
                     ↑
                     └─[T FF]─ Q3 (MSB)

T=1 for all (always toggle when triggered)
Each FF triggered by previous Q output
```

**Count Sequence:**
```
Clock | Q3 Q2 Q1 Q0 | Decimal
------|-------------|--------
  0   | 0  0  0  0  |   0
  1   | 0  0  0  1  |   1
  2   | 0  0  1  0  |   2
  3   | 0  0  1  1  |   3
  4   | 0  1  0  0  |   4
  ...
  15  | 1  1  1  1  |  15
  16  | 0  0  0  0  |   0 (repeats)
```

**Problem:** Ripple delay (not synchronous)

---

### 4-bit Synchronous Up Counter:

**Circuit:**
```
All FFs clocked simultaneously!

T0=1 (always)
T1 = Q0
T2 = Q0 · Q1
T3 = Q0 · Q1 · Q2

         [T FF]─ Q0
          T=1
          ↑
         CLK
         
Q0 ────→ T1
         [T FF]─ Q1
          ↑
         CLK
         
Q0·Q1 ──→ T2
         [T FF]─ Q2
          ↑
         CLK
         
Q0·Q1·Q2→ T3
         [T FF]─ Q3
          ↑
         CLK

All share same clock!
```

**Advantage:** No ripple delay, fast!

---

### BCD Counter (Decade Counter):

**Counts 0-9, then resets**

```
Uses synchronous counter with reset at 10

When Q3Q2Q1Q0 = 1010 (10):
→ Reset to 0000

Circuit: Add NAND gate
Q3 · Q1 → Reset
```

---

### Ring Counter:

**Only one bit high at a time**

```
State: 1000 → 0100 → 0010 → 0001 → 1000

Circuit: Shift register with feedback
Q3 → Serial In

Used in: State machines, sequencers
```

---

### 🎯 Build Counter Projects:

**Project 1: 4-bit Up Counter**
```
Build synchronous up counter
Count 0-15
Display on LEDs
Add reset button
```

**Project 2: BCD Counter**
```
Build decade counter (0-9)
Reset at 10
Connect to 7-segment display
Show decimal digits!
```

**Project 3: Stopwatch**
```
Multiple BCD counters
Count seconds, minutes
Display on 4 × 7-segment displays
Add start/stop/reset buttons
```

---

## ৪.১০ Build Finite State Machines - Control Logic! 🎉

### What's an FSM?

```
FSM = Finite State Machine
- Specific number of states
- Transitions based on inputs
- Outputs based on state

তোমার processor এর control unit = FSM!
```

### Types of FSM:

```
1. Moore Machine:
   Output = f(current state only)
   
2. Mealy Machine:
   Output = f(current state, input)
```

---

### Moore Machine Example: Traffic Light

**States:**
```
S0: Red (30 sec)
S1: Yellow (5 sec)
S2: Green (25 sec)
```

**State Diagram:**
```
       timer=30
     ┌──────────┐
     │          ↓
   [Red] ──→ [Yellow] ──→ [Green]
     ↑         timer=5      │
     │                      │
     └──────────────────────┘
            timer=25
```

**State Table:**
```
┌────────┬───────┬─────────────┬────────┐
│ State  │ Input │ Next State  │ Output │
├────────┼───────┼─────────────┼────────┤
│  Red   │Timer30│   Yellow    │  100   │
│ Yellow │Timer5 │    Green    │  010   │
│ Green  │Timer25│     Red     │  001   │
└────────┴───────┴─────────────┴────────┘

Output: R Y G (lights)
```

---

### FSM Design Steps:

```
1. Define states
2. Draw state diagram
3. Create state table
4. Assign binary codes to states
5. Derive next-state logic
6. Derive output logic
7. Build circuit
```

### Example: 2-bit Sequence Detector (101)

**Detects pattern "101" in serial input**

**States:**
```
S0: Initial (no pattern)
S1: Got "1"
S2: Got "10"
S3: Got "101" (detected!)
```

**State Diagram:**
```
        0/0        1/0
       ┌───┐      ┌───┐
       ↓   │      │   ↓
     ┌──────┐  ┌──────┐
  ───┤  S0  ├──┤  S1  │
     └──────┘  └──┬───┘
        ↑        │
        │     0/0│
        │        ↓
     ┌──────┐  ┌──────┐
     │  S3  │←─┤  S2  │
     └──┬───┘  └───┬──┘
        │          │
        └──────────┘
         1/1    1/0

Format: Input/Output
```

**State Table:**
```
┌────────┬───────┬─────────────┬────────┐
│Current │ Input │ Next State  │ Output │
├────────┼───────┼─────────────┼────────┤
│   S0   │   0   │     S0      │   0    │
│   S0   │   1   │     S1      │   0    │
│   S1   │   0   │     S2      │   0    │
│   S1   │   1   │     S1      │   0    │
│   S2   │   0   │     S0      │   0    │
│   S2   │   1   │     S3      │   1    │ ← Detected!
│   S3   │   0   │     S2      │   0    │
│   S3   │   1   │     S1      │   0    │
└────────┴───────┴─────────────┴────────┘
```

**State Encoding:**
```
S0 = 00
S1 = 01  
S2 = 10
S3 = 11
```

**Circuit:**
```
Input ──┬──[Next State Logic]──┬── D1  ┐
        │                      │       │
Q1 Q0 ──┘                      └── D0  ├─[2 D-FFs]─ Q1 Q0
                                       │     ↑
                                       │    CLK
                                       │
                              Output ──┴── [Output Logic]
```

### 🎯 Build FSM Project:

**Build Sequence Detector:**
```
1. Design state diagram (on paper)
2. Create state table
3. Derive logic equations:
   - Next state: D1 = f(Q1,Q0,Input)
   - Next state: D0 = f(Q1,Q0,Input)
   - Output: Y = f(Q1,Q0,Input)
4. Build with flip-flops and gates
5. Test with input sequence!
```

**Test:**
```
Input sequence: 1 1 0 1 0 1 1

Expected detections at:
Position 4: "101" ✓  (input positions 2-3-4, detection completes when the last 1 arrives)
Position 6: "101" ✓  (overlapping: input positions 4-5-6)
```

---

## ৪.১১ Your 2-Week Build Plan

### Week 1: Memory Elements

**Day 1-2: Latches**
```
□ Build SR Latch (NOR & NAND)
□ Build D Latch
□ Understand level-triggering
□ Test memory behavior
```

**Day 3-4: Flip-Flops**
```
□ Build D Flip-Flop (master-slave)
□ Build JK Flip-Flop
□ Build T Flip-Flop
□ Understand edge-triggering
```

**Day 5-6: Registers**
```
□ Build 4-bit Register
□ Add enable control
□ Add clear functionality
□ Test load and hold
```

**Day 7: Review**
```
□ Review all memory elements
□ Understand timing
□ Prepare timing diagrams
```

---

### Week 2: Sequential Systems

**Day 8-9: Shift Registers**
```
□ Build SISO shift register
□ Build SIPO shift register
□ Test serial communication
□ Understand applications
```

**Day 10-11: Counters**
```
□ Build 4-bit synchronous counter
□ Build BCD counter
□ Build ring counter
□ Connect to 7-segment display
```

**Day 12-14: FSM - The Ultimate!**
```
□ Design state diagram
□ Create state table
□ Derive logic equations
□ Build sequence detector FSM
□ Test pattern detection
□ Share your FSM! #BuildYourOwnProcessor
```

---

## ৪.১২ Timing Parameters - Critical!

### Setup Time (Tsu):

```
Data must be stable BEFORE clock edge

       ┌────────Data stable──────
D ─────┘
                ↑
              Tsu│
                ↓
CLK ────────┐ ←─┘
            └─

Tsu = Minimum setup time
```

### Hold Time (Th):

```
Data must be stable AFTER clock edge

       ┌────────Data stable──────
D ─────┘
                ↑
                │Th
                ↓
CLK ────────┐ ─→┘
            └─
```

### Propagation Delay (Tpd):

```
Time from clock edge to output change

CLK ────────┐
            └─
            ↑
            │Tpd
            ↓
Q ──────────────┐
                └─
```

### Maximum Clock Frequency:

```
Fmax = 1 / (Tpd + Tsu + Tclk-to-q)

Example:
Tpd = 10ns
Tsu = 5ns  
Tclk-to-q = 5ns

Fmax = 1/(20ns) = 50 MHz
```

---

## ৪.১৩ Pro Tips & Common Mistakes

### ✅ Do This:
```
✅ Draw timing diagrams first
✅ Check setup/hold times
✅ Use common clock (synchronous)
✅ Add reset to all flip-flops
✅ Test with slow clock first
✅ Label all states clearly
```

### ❌ Avoid This:
```
❌ Asynchronous designs (hard to debug)
❌ Ignoring timing parameters
❌ No reset mechanism
❌ Clock skew issues
❌ Mixing edge and level triggering
❌ Invalid states in FSM
```

### Common Bugs:
```
1. Setup/hold violations
   Fix: Add buffers, slower clock

2. Race conditions
   Fix: Synchronous design

3. Missing reset
   Fix: Add reset to all FFs

4. Clock skew
   Fix: Use clock distribution tree
```

---

## ৪.১৪ Chapter 4 Mission Complete!

### তুমি এখন পারো:

```
✅ Build memory elements (latches, flip-flops)
✅ Design registers with control
✅ Build shift registers
✅ Design counters (up, down, BCD)
✅ Create finite state machines
✅ Understand timing parameters
✅ তোমার processor এ memory এবং control logic যোগ করা!
```

### তুমি বানিয়েছো:
```
✅ SR Latch
✅ D Latch
✅ D Flip-Flop (master-slave)
✅ JK Flip-Flop
✅ T Flip-Flop
✅ 4-bit Register
✅ Shift Registers (SISO, SIPO, PISO)
✅ Counters (sync, BCD, ring)
✅ Finite State Machine! 🎉
```

### Stats:
```
Total circuits built: 15+
Total flip-flops used: 50+
Total state machines: 2+
Level: Sequential Master! 🏆
```

### Next Level Unlocked:
```
→ Chapter 5: Verilog Programming
   তুমি শিখবে: Hardware description language
   Build in code, not just circuits!
   
   From visual circuits → Code!
```

---

## 🎯 Final Project - Before Next Chapter

### Project: Digital Lock System

**Build a combination lock FSM:**
```
Requirements:
- 4-bit code: 1-0-1-1
- 4 states + locked/unlocked
- LED shows locked/unlocked
- Reset button
- Wrong code → stay locked

Bonus:
- Add alarm after 3 wrong attempts
- Add timeout
- Multiple correct codes
- Share your design!
```

---

## 🏆 Achievement Unlocked!

```
Level 4: ✅ COMPLETE - Sequential Logic Expert!
Progress: [████████████████████] 20%

XP Gained: +1500
Skills: Memory, Registers, FSM, Control Logic

Badges Earned:
🥉 Latch Builder
🥈 Flip-Flop Master
🥇 Register Designer
🏅 Counter Creator
🎖️ FSM Architect
🏆 Sequential Systems Expert

Next: Chapter 5 - Learn to Code Hardware!
      Verilog is calling! 💻
```

---

**[⬅️ Previous: Chapter 3](Chapter_03_Combinational_Circuits.md)** | **[➡️ Next: Chapter 5](Chapter_05_Verilog_Basics.md)**

---

<div align="center">

**"You just gave your processor memory and control. Next, you'll code it!"**

**"তুমি তোমার processor কে memory এবং control দিয়েছো। এবার code করবে!"**

Made with ❤️ for builders | বানানোর জন্য ভালোবাসা দিয়ে তৈরি

</div>
