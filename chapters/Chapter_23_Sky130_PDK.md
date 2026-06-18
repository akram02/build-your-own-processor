# 🔬 Chapter 23: Sky130 PDK - Your Fabrication Technology
## Understanding the Open Source Chip Manufacturing Process!

> **"Know your technology. Know your transistors. Build better chips!"**
>
> **"তোমার technology চেনো। তোমার transistor চেনো। Better chips বানাও!"**

---

## 🎯 এই Chapter এ তুমি শিখবে:

```
✅ Sky130 PDK - Google's open process
✅ Process Technology - 130nm details
✅ Standard Cells - pre-built blocks
✅ Transistor Models - SPICE simulation
✅ Design Rules - manufacturing limits
✅ IP Blocks - Memory, IO pads
✅ Characterization - cell timing
✅ তোমার technology expert! 🎉
```

**Time Required:** 1 week (deep understanding)  
**Prerequisites:** Chapters 21-22 complete

---

## 🌟 মুখস্থ করার দরকার নেই

এই অধ্যায়ে অনেক সংখ্যা আর নিয়ম আসবে (metal width, design rule, cell নাম…)।
**এগুলো মুখস্থ করতে হবে না** — এটা একটা reference। ঠিক যেমন অভিধান মুখস্থ করো
না, দরকারে দেখো। কেউ অভিধানের প্রতিটা শব্দ মাথায় নিয়ে ঘোরে না; কিন্তু একটা
শব্দ লাগলে কোথায় খুঁজবে, সেটা জানে। এই chapter-ও ঠিক তেমন — তোমার কাজ হলো
**কোন তথ্য কোথায় থাকে আর কেন থাকে** সেটা বোঝা, সংখ্যাগুলো গিলে ফেলা নয়।

আর সত্যি বলতে, এই নিয়মগুলোর বেশিরভাগই tool নিজে নিজে দেখে নেয়। তুমি যখন
OpenLane চালাও বা Magic দিয়ে DRC করো, তখন এই PDK থেকেই নিয়ম পড়ে নিয়ে তোমার
design যাচাই করে। তোমাকে শুধু জানতে হবে — **এই নিয়মগুলো আসলে কী জিনিস, আর
কেন এগুলো আছে।** সেটাই হলো একজন junior designer আর একজন engineer-এর তফাত:
junior tool-এর error দেখে ভয় পায়, engineer বুঝে ফেলে error-টা আসলে কী বলছে।

এখন একবার চোখ বুলিয়ে নাও যাতে জানো কী কী আছে; পরে নিজের chip বানানোর সময়
নির্দিষ্ট নিয়মটা খুঁজে নেবে। চলো, তোমার fabrication technology-র সাথে পরিচিত
হই! 🔬

---

## 🚀 What is Sky130?

আগের দুই chapter-এ তুমি RTL থেকে GDSII পর্যন্ত পুরো flow দেখেছো, OpenLane
দিয়ে নিজের processor-এর layout বানিয়েছো। কিন্তু একটা প্রশ্ন হয়তো মাথায়
ঘুরছিল: OpenLane জানল **কীভাবে** যে একটা AND gate দেখতে কেমন, একটা তার কত
সরু হতে পারে, একটা transistor কত জোরে চলে? এসব তথ্য OpenLane-এর ভেতরে লেখা
নেই। ও এগুলো পায় **PDK** থেকে। PDK হলো foundry আর designer-এর মাঝখানের
চুক্তিপত্র।

### PDK = Process Design Kit

একটা উপমা দিয়ে শুরু করি। ধরো তুমি একটা বাড়ি বানাবে, কিন্তু নিজে ইট পোড়াও
না, সিমেন্ট বানাও না — একটা নির্দিষ্ট construction company-র সাথে কাজ করবে।
সেই company তোমাকে একটা মোটা manual ধরিয়ে দেয়: "আমাদের ইট এই মাপের, আমাদের
দেয়াল এত পাতলা করা যাবে না, দুটো জানালার মাঝে এত ফাঁকা রাখতেই হবে, আর এই
হলো আমাদের রেডিমেড দরজা-জানালার catalogue।" সেই manual অনুযায়ী নকশা করলে
তবেই তারা বাড়িটা বানিয়ে দিতে পারবে।

**PDK ঠিক সেই manual + catalogue।** Foundry (যেখানে chip বানানো হয়) তাদের
process-এর সব নিয়ম, সব model, আর সব রেডিমেড block একসাথে গুছিয়ে designer-কে
দেয়। এটা ছাড়া তুমি যত সুন্দর Verilog-ই লেখো, foundry সেটা silicon-এ আনতে
পারবে না — কারণ তুমি তাদের "ভাষায়" কথা বলোনি।

PDK-র ভেতরে কী কী থাকে, সেটা এক নজরে দেখে নাও:

| উপাদান | কী জিনিস | কেন দরকার |
|---|---|---|
| **Design rules (DRC)** | তার কত সরু, কত কাছাকাছি হতে পারে — geometry-র নিয়ম | নিয়ম ভাঙলে chip ফেইল করবে |
| **Device models (SPICE)** | transistor কীভাবে আচরণ করে তার গাণিতিক বর্ণনা | circuit simulate করে আগেই দেখার জন্য |
| **Standard cell library** | রেডিমেড gate, flip-flop ইত্যাদির catalogue | শূন্য থেকে gate না বানিয়ে কাজ এগোনো |
| **IO pad library** | chip-এর বাইরে সংযোগের জন্য pad | core logic-কে বাইরের জগতের সাথে জোড়া |
| **Memory compilers** | চাহিদামতো SRAM তৈরির tool | chip-এ memory যোগ করা |
| **Verification rules (LVS)** | layout আর schematic একই কিনা মেলানোর নিয়ম | "যা আঁকলে তা-ই কি বানালে" — যাচাই |
| **Documentation** | সবকিছুর ব্যাখ্যা | প্রয়োজনে খুঁজে নেওয়া |

আর Sky130 PDK নিজে কী, সেটা সংক্ষেপে:

| বৈশিষ্ট্য | মান |
|---|---|
| তৈরি করেছে | SkyWater Technology |
| Open source করেছে | Google (২০২০) |
| Process node | 130nm |
| অবস্থা | Production-ready! |
| খরচ | একদম FREE |

এখানে একটা ব্যাপার থামিয়ে ভাবার মতো। এত দিন chip বানানোর প্রতিটা নিয়ম,
প্রতিটা model ছিল foundry-র **গোপন সম্পদ** — NDA সই না করে কেউ ছুঁতেও পারত
না, খরচও আকাশছোঁয়া। ২০২০ সালে Google আর SkyWater মিলে প্রথমবারের মতো একটা
পুরো, production-ready PDK পুরো দুনিয়ার জন্য **খুলে দিল**। এর মানে — তুমি,
বাংলাদেশের একটা ছোট্ট ঘরে বসে, ঠিক সেই same নিয়ম-model দিয়ে chip design করতে
পারো যা দিয়ে professional-রা করে। এটাই Sky130-কে এত বিশেষ করে তোলে। 🎉

> 💡 **চটজলদি মনে রাখার সূত্র:** **PDK = "foundry-র সাথে designer-এর
> চুক্তি।"** এই চুক্তি মানলে তোমার design silicon হবে; না মানলে হবে না।
> তুমি নিয়ম বানাও না, নিয়ম *মেনে* design করো।

---

## ২৩.১ Sky130 Process Technology

### 130nm CMOS Process:

```
Transistor size: 130nm minimum length
That's 0.13 micrometers!

Comparison:
- Intel 4004 (1971): 10,000nm
- Intel Pentium (1993): 800nm
- Sky130 (2020): 130nm
- Modern (2024): 3-5nm

130nm is:
→ Old by today's standards
→ But perfect for learning!
→ Still used in automotive, IoT
→ Cheaper to manufacture
→ More forgiving design rules
```

### Available Layers:

```
Metal Layers: 5 routing layers (Metal1–Metal5)
- li1: separate local-interconnect layer (sits below Metal1)
- Metal1-Metal2 (fine routing, 140nm)
- Metal3-Metal4 (thicker routing, 300nm)
- Metal5 (thick — power, long routes, 1600nm)

Device Layers:
- N-diffusion (NMOS transistors)
- P-diffusion (PMOS transistors)  
- Polysilicon (transistor gates)
- N-well (isolation)

Special Layers:
- Deep N-well
- High-voltage devices
- Varactors (voltage variable caps)
- Resistors (poly, diffusion)
```

---

## ২৩.২ Standard Cell Library

### What's in the Library?

```
sky130_fd_sc_hd (High Density):
- 500+ standard cells
- Optimized for density
- Most common choice

Variants:
- sky130_fd_sc_hd: High density
- sky130_fd_sc_hdll: High density low leakage
- sky130_fd_sc_hs: High speed
- sky130_fd_sc_ms: Medium speed
- sky130_fd_sc_ls: Low speed
- sky130_fd_sc_lp: Low power
```

### Common Cells:

```
Logic Gates:
✅ AND2, AND3, AND4
✅ OR2, OR3, OR4
✅ NAND2, NAND3, NAND4
✅ NOR2, NOR3, NOR4
✅ XOR2, XNOR2
✅ INV (inverter)
✅ BUF (buffer)

Flip-Flops:
✅ DFF (D flip-flop)
✅ DFFR (with reset)
✅ DFFS (with set)
✅ DFFSR (with set and reset)
✅ LATCH

Multiplexers:
✅ MUX2, MUX4
✅ MUX2I (with invert)

Adders:
✅ FA (full adder)
✅ HA (half adder)

Total: 500+ cells!
```

### Cell Naming Convention:

```
Example: sky130_fd_sc_hd__and2_1

sky130     - Process
fd         - Foundry (SkyWater)
sc         - Standard cell
hd         - High density
__         - Separator
and2       - Function (2-input AND)
_1         - Drive strength

Drive strengths: _1, _2, _4, _8
(Higher = stronger, faster, bigger, more power)
```

---

## ২৩.৩ Transistor Models

### SPICE Models:

```
For circuit simulation:
- NMOS: nfet_01v8
- PMOS: pfet_01v8
- High voltage NMOS: nfet_g5v0d10v5
- High voltage PMOS: pfet_g5v0d10v5

Parameters include:
- Threshold voltage (Vth)
- Mobility
- Channel length modulation
- Gate oxide thickness
- Junction capacitances

Used in: Ngspice, Xyce, HSPICE
```

### Example SPICE Netlist:

```spice
* Simple inverter
.include "sky130_fd_pr/models/sky130.lib.spice tt"

Xm1 out in gnd gnd nfet_01v8 w=1 l=0.15
Xm2 out in vdd vdd pfet_01v8 w=2 l=0.15

Vdd vdd 0 1.8
Vin in 0 pulse(0 1.8 0 10p 10p 100n 200n)

.tran 1p 400n
.end
```

---

## ২৩.৪ Design Rules

### Minimum Dimensions:

```
Metal1:
- Min width: 140nm
- Min spacing: 140nm
- Min area: 0.083 µm²

Metal2:
- Min width: 140nm
- Min spacing: 140nm

Metal3-4:
- Min width: 300nm
- Min spacing: 300nm

Metal5:
- Min width: 1600nm (1.6µm)
- Min spacing: 1600nm

Polysilicon:
- Min width: 150nm
- Min spacing: 210nm

Via:
- Size: 150nm × 150nm
- Spacing: 170nm
```

### Why Rules Matter:

```
Violate rules → Chip fails!

Example failures:
❌ Too narrow wire → Opens (breaks)
❌ Too close wires → Shorts (touch)
❌ Missing via → No connection
❌ Wrong enclosure → Reliability issues

Magic/KLayout checks these automatically!
```

---

## ২৩.৫ IP Blocks

### SRAM Compiler:

```
OpenRAM - Open source memory compiler

Generates SRAMs:
- Size: 32 bytes to 16 KB
- Ports: Single or dual port
- Word width: 8, 16, 32, 64 bits

Usage:
python3 openram.py myconfig.py

Outputs:
- GDSII layout
- Verilog model  
- SPICE netlist
- Timing library

Add memory to your chip! 🧠
```

### IO Pads:

```
For chip-to-outside connection:

Available pads:
✅ Digital input
✅ Digital output
✅ Bidirectional
✅ Analog
✅ Power (VDD, VSS)
✅ ESD protection

Pad ring:
Surrounds your core logic
Connects to package pins

Size: ~100µm × 100µm per pad
```

---

## ২৩.৬ Cell Characterization

### Timing Parameters:

```
For each standard cell:

Delays:
- Input to output delay (tpd)
- Rise time (tr)
- Fall time (tf)

Constraints:
- Setup time (tsu)
- Hold time (th)
- Clock-to-Q (tcq)

Varies with:
- Input slew
- Output load
- Voltage
- Temperature
```

### Liberty Format (.lib):

```
cell (sky130_fd_sc_hd__and2_1) {
    area : 3.5;
    
    pin(A) {
        direction : input;
        capacitance : 0.0015;
    }
    
    pin(Y) {
        direction : output;
        function : "(A&B)";
        timing() {
            related_pin : "A";
            cell_rise(delay_template) {
                index_1 ("0.01, 0.1, 1");
                index_2 ("0.01, 0.1, 1");
                values (
                    "0.1, 0.15, 0.3",
                    "0.12, 0.17, 0.32",
                    "0.2, 0.25, 0.4"
                );
            }
        }
    }
}
```

---

## ২৩.৭ Process Corners

### What are Corners?

```
Manufacturing variations:
- Transistors not exactly 130nm
- Vary ±10% or more!

Process corners:
✅ TT (Typical-Typical) - Nominal
✅ FF (Fast-Fast) - Best case
✅ SS (Slow-Slow) - Worst case
✅ FS (Fast-Slow) - Skewed
✅ SF (Slow-Fast) - Skewed

Must verify all corners!
```

### Corner Testing:

```
Your design must work at:
- SS corner: Slowest
- FF corner: Fastest  
- TT corner: Typical
- All temperatures: -40°C to 125°C
- All voltages: ±10%

If works at all corners → Robust design! ✅
```

---

## ২৩.৮ Analog Capabilities

### Analog Devices:

```
Sky130 includes:
✅ BJT transistors
✅ Varactors
✅ Resistors (multiple types)
✅ Capacitors (MiM, MoM)
✅ Photodiodes
✅ ESD devices

Can build:
→ Analog-to-Digital Converters (ADC)
→ Digital-to-Analog Converters (DAC)
→ Voltage references
→ Amplifiers
→ PLLs
→ Complete mixed-signal chips!
```

---

## ২৩.৯ Documentation

### Where to Learn More:

```
Official docs:
→ sky130-pdk.readthedocs.io
→ Design rules manual
→ Device specifications
→ Standard cell docs

GitHub:
→ github.com/google/skywater-pdk
→ All source files
→ Examples
→ Issues/discussions

Very well documented! 📚
```

---

## ২৩.১০ Using Sky130 in Your Design

### Best Practices:

```
1. Start with standard cells
   - Don't design custom gates first time
   - Use library cells
   - Proven, characterized

2. Follow design rules
   - Use DRC-clean layouts only
   - Magic checks this
   - Fix ALL violations

3. Simulate extensively
   - Test all corners
   - Verify timing
   - Check power

4. Use reference designs
   - Learn from examples
   - Caravel SoC
   - TinyTapeout projects

5. Community support
   - Ask questions!
   - Share learnings
   - Help others
```

---

## ২৩.১১ Chapter 23 Mission Complete!

### তুমি এখন জানো:

```
✅ Sky130 PDK complete overview
✅ Process technology details
✅ Standard cell library
✅ Transistor models
✅ Design rules
✅ IP blocks (SRAM, IO)
✅ Cell characterization
✅ Process corners
✅ Ready for real chip! 🎉
```

### Knowledge Level:

```
You now understand:
→ How chips are manufactured
→ What's in a PDK
→ How to use standard cells
→ Design rule constraints
→ Timing characterization

Professional knowledge! 💼
```

---

## 🎯 Chapter Exercise

### Project: Characterize a Standard Cell

```
Task: Analyze sky130_fd_sc_hd__and2_1

1. Extract from GDSII
   magic -T sky130A.tech
   gds read sky130_fd_sc_hd.gds
   load sky130_fd_sc_hd__and2_1

2. View layout
   - Count transistors
   - Measure dimensions
   - Identify layers

3. Simulate in SPICE
   - Extract netlist
   - Run transient analysis
   - Measure delay

4. Compare with .lib
   - Check documented delay
   - Verify functionality
   - Understand variations
```

---

## 🏆 Achievement Unlocked!

```
Level 23: ✅ COMPLETE - PDK Expert!
Progress: [████████████████████████████████████] 92%

XP Gained: +1500
Skills: PDK Knowledge, Process Technology

Badges Earned:
🥉 Sky130 Expert
🥈 Standard Cell Master
🥇 Design Rule Pro
🏅 Process Corner Tester
🎖️ Transistor Guru

NEXT: Submit to TinyTapeout! 📤
```

---

**[⬅️ Previous: Chapter 22](Chapter_22_OpenLane_Physical_Design.md)** | **[➡️ Next: Chapter 24](Chapter_24_TinyTapeout.md)**

---

<div align="center">

**"You know the technology. Time to submit your chip!"**

**"তুমি technology জানো। এবার chip submit করো!"**

Made with ❤️ for IC designers | IC designers দের জন্য ভালোবাসা দিয়ে তৈরি

</div>
