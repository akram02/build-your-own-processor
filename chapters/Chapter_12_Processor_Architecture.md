# 🖥️ Chapter 12: Build Your Own Processor - Architecture Basics
## From Circuits to CPU - Understanding How Processors Work!

> **"Gates were building blocks. Now build the COMPUTER. Time to create intelligence!"**
>
> **"Gates ছিল building blocks। এখন COMPUTER বানাও। Intelligence তৈরি করো!"**

---

## 🎯 এই Chapter এ তুমি শিখবে:

```
✅ What is a Processor? - CPU fundamentals
✅ Instruction Set Architecture (ISA) - the interface
✅ CPU Components - datapath & control
✅ Instruction Execution - fetch-decode-execute
✅ Simple CPU Design - 8-bit processor
✅ Register File - fast storage
✅ ALU Integration - arithmetic unit
✅ তোমার নিজের processor এর architecture! 🎉
```

**Time Required:** 1 week (5-6 hours/day)  
**Prerequisites:** Chapters 1-11 complete

---

## 🚀 Quick Understanding - ৫ মিনিটে Processor Basics!

### Processor আসলে কী?

এতদিন তুমি gate, adder, register, FSM — এগুলো আলাদা আলাদা বানিয়েছো। একটা **processor** হলো এই সবকিছুকে এমনভাবে জুড়ে দেওয়া যেন তারা একসাথে মিলে **memory-তে রাখা instruction পড়ে, একটার পর একটা চালিয়ে যায়**। অর্থাৎ processor কোনো জাদুর বাক্স নয় — তুমি এতদিন যা শিখেছো, তারই একটা সাজানো-গোছানো রূপ।

একটা CPU কে মূলত তিন ভাগে ভাবলে সবচেয়ে সহজ হয়:

- **Datapath** — যেখানে আসল কাজটা হয়। এখানে থাকে register (data রাখার জায়গা), ALU (যোগ-বিয়োগ-AND-OR করার unit), আর multiplexer (কোন data কোথায় যাবে সেই রাস্তা ঠিক করার switch)। Datapath হলো processor-এর হাত-পা।
- **Control Unit** — যে ঠিক করে দেয় কখন কী হবে। সে instruction পড়ে decode করে, তারপর datapath-এর প্রতিটা অংশকে control signal পাঠায়: "এখন ALU যোগ করবে", "এখন এই register-এ লিখবে"। Control unit হলো processor-এর মস্তিষ্ক।
- **Memory Interface** — বাইরের memory-র সাথে কথা বলার জানালা। এর মধ্য দিয়ে instruction আসে, আবার data load/store হয়।

একটা **কারখানার** সাথে মিলিয়ে দেখো, পুরো ছবিটা পরিষ্কার হয়ে যাবে:

| কারখানা | Processor | কাজ |
|---|---|---|
| শ্রমিক ও মেশিন | Datapath (ALU, register) | আসল উৎপাদন |
| ম্যানেজার ও রুটিন | Control Unit | কে কখন কী করবে ঠিক করা |
| গুদাম | Memory | কাঁচামাল ও পণ্য জমা রাখা |

ম্যানেজার নিজে হাতে কিছু বানায় না, কিন্তু তার নির্দেশ ছাড়া শ্রমিকেরাও জানে না কী করতে হবে — control unit আর datapath-এর সম্পর্কটা ঠিক এমনই।

### Instruction Execution Cycle:

Processor কখনো থামে না। সে একটা ছোট্ট চক্র (cycle) বারবার ঘোরাতে থাকে — প্রতিটা চক্রে একটা করে instruction-কে জন্ম থেকে মৃত্যু পর্যন্ত নিয়ে যায়। চারটে মূল ধাপ:

```mermaid
flowchart LR
    A["1️⃣ FETCH<br/>memory থেকে<br/>instruction আনা<br/>(PC → Memory → IR)"]
    B["2️⃣ DECODE<br/>instruction বোঝা<br/>(IR → Control Unit → Signals)"]
    C["3️⃣ EXECUTE<br/>কাজটা করা<br/>(Signals → Datapath → Result)"]
    D["4️⃣ WRITEBACK<br/>result রাখা<br/>(Result → Register/Memory)"]
    A --> B --> C --> D
    D -- "আবার শুরু" --> A
```

লক্ষ করো — শেষ ধাপের পর তীরটা আবার প্রথম ধাপে ফিরে গেছে। এই ঘুরতে থাকা চক্রই হলো একটা জীবন্ত processor-এর হৃৎস্পন্দন। তোমার ল্যাপটপ, ফোন, এমনকি মাইক্রোওয়েভ ওভেনের ভেতরের chip — সবাই এই একই চক্র সেকেন্ডে কোটি কোটি বার ঘোরাচ্ছে। ছোট হোক বা বড়, প্রতিটা CPU-র গল্প এখান থেকেই শুরু।

🎉 **ব্যস, এটুকুতেই তুমি processor-এর মূল ধারণাটা ধরে ফেলেছো!**

---

## ১২.১ Processor Architecture Overview

Processor বানানোর আগে একটা বড় সিদ্ধান্ত নিতে হয়: **instruction আর data একই memory-তে রাখবে, নাকি আলাদা?** শুনতে সামান্য মনে হলেও এই একটা সিদ্ধান্তের উপরেই পুরো computer-এর গঠন দাঁড়িয়ে আছে। দুটো ঐতিহাসিক উত্তর আছে — Von Neumann আর Harvard। চলো দুটোই বুঝে নিই।

### Von Neumann Architecture:

১৯৪৫ সালে গণিতবিদ John von Neumann একটা যুগান্তকারী ধারণা দেন: program (অর্থাৎ instruction) আর data — দুটোকেই **একই memory-তে, একসাথে** রাখা যায়। এর আগে computer-কে নতুন কাজ করাতে হলে তার ভেতরের তার (wire) খুলে নতুন করে জুড়তে হতো। von Neumann বললেন, "তারের বদলে memory-তে নতুন instruction লিখে দাও — হার্ডওয়্যার একই থাকবে, শুধু program পাল্টে গেলেই computer নতুন কাজ করবে।" এই **stored-program** ধারণাটাই আধুনিক computer-এর ভিত্তি।

```mermaid
flowchart TD
    subgraph CPU["CPU"]
        direction LR
        CU["Control Unit"] -->|control signals| DP["Datapath<br/>(ALU + Registers)"]
    end
    MEM["একটিই Memory<br/>(Instructions + Data একসাথে)"]
    CPU <-->|"একটিই bus<br/>(instruction ও data<br/>এই পথেই যাওয়া-আসা করে)"| MEM
```

এর মূল ভাবনাগুলো:

- **Stored program** — program memory-তেই থাকে, তাই software পাল্টে computer-কে নতুন কাজ করানো যায়।
- **একই memory-তে code আর data** — সরল ডিজাইন, কম তার, কম খরচ।
- **Sequential execution** — instruction গুলো একটার পর একটা চলে, আমাদের চেনা fetch-decode-execute চক্র মেনে।

তবে একটা দুর্বলতাও আছে: যেহেতু instruction আর data একই রাস্তা (bus) দিয়ে যাতায়াত করে, processor একই সময়ে instruction **আর** data — দুটো আনতে পারে না। একটা আনতে গেলে অন্যটাকে দাঁড়িয়ে থাকতে হয়। এই সীমাবদ্ধতাটার নাম **"von Neumann bottleneck"**, আর এখান থেকেই পরের ধারণাটা জন্ম নেয়।

### Harvard Architecture:

Harvard architecture এই bottleneck-টাকেই সমাধান করে — খুব সোজা উপায়ে: **instruction আর data-র জন্য আলাদা আলাদা memory আর আলাদা bus** বানিয়ে দাও। তাহলে processor এক হাতে instruction আনতে পারে, একই মুহূর্তে অন্য হাতে data — কোনো ঠেলাঠেলি নেই।

```mermaid
flowchart TD
    CPU["CPU"]
    IMEM["Instruction Memory"]
    DMEM["Data Memory"]
    CPU <-->|instruction bus| IMEM
    CPU <-->|data bus| DMEM
```

এর সুবিধাগুলো:

- **একই সময়ে instruction আর data আনা যায়** — দুটো আলাদা পথ থাকায় কোনো conflict নেই।
- **বেশি bandwidth** — প্রতি cycle-এ বেশি কাজ এগোয়।
- তাই DSP (Digital Signal Processor) আর অনেক microcontroller-এ Harvard ব্যবহার হয়, যেখানে গতিটা খুব জরুরি।

আমরা এই বইয়ে একটা মাঝামাঝি পথ — **Modified Harvard** — বেছে নেব, যেটা আসলে দুই দুনিয়ার সেরাটা একসাথে দেয়:

- প্রোগ্রামারের কাছে একটাই memory address space (von Neumann-এর মতো সরল)।
- কিন্তু ভেতরে instruction আর data-র জন্য আলাদা cache (Harvard-এর মতো দ্রুত)।

বাস্তবে তোমার ফোন আর ল্যাপটপের প্রায় সব আধুনিক processor ঠিক এই Modified Harvard পথেই চলে — তাই তুমি যা বানাবে, তা মোটেও খেলনা নয়, একদম শিল্পজগতের পথ।

---

## ১২.২ Instruction Set Architecture (ISA)

### ISA জিনিসটা আসলে কী?

ধরো তুমি একটা restaurant-এ গেছো। তুমি রান্নাঘরে গিয়ে নিজে রান্না করো না, আবার রাঁধুনিও তোমার টেবিলে এসে জিজ্ঞেস করে না কীভাবে তরকারি কাটতে হবে। মাঝখানে থাকে একটা **menu** — তুমি menu দেখে অর্ডার দাও, রাঁধুনি সেই অর্ডার পেয়ে রান্না করে। তুমি জানো না রান্নাঘরে কী হচ্ছে, রাঁধুনি জানে না তুমি কে — দুজনকে শুধু menu-টা মেলাতে হয়।

**ISA (Instruction Set Architecture)** হলো হার্ডওয়্যার আর সফটওয়্যারের মাঝখানের ঠিক সেই menu — একটা **চুক্তি (contract)**। সফটওয়্যার (compiler, programmer) জানে কোন কোন instruction পাঠানো যাবে; হার্ডওয়্যার জানে সেই instruction পেলে কী করতে হবে। ভেতরে হার্ডওয়্যার কীভাবে বানানো হয়েছে তা সফটওয়্যারকে জানতে হয় না — এটাই ISA-র সৌন্দর্য।

একটা ISA মূলত এই জিনিসগুলো ঠিক করে দেয়:

- **Instructions** — কোন কোন operation করা যাবে (যোগ, load, jump...)।
- **Registers** — কতগুলো register আছে, প্রতিটা কত bit।
- **Data types** — byte, word ইত্যাদি কত বড়।
- **Addressing modes** — memory থেকে data খুঁজে বের করার নিয়ম।
- **Memory model** — memory কীভাবে সাজানো ও আচরণ করে।

```mermaid
flowchart LR
    SW["Software<br/>(compiler, program)"] -->|"ISA (চুক্তি)"| HW["Hardware<br/>(তোমার processor)"]
```

এই চুক্তির আসল শক্তিটা হলো **স্বাধীনতা**: একই ISA মেনে Intel আর AMD আলাদা আলাদা chip বানায়, কিন্তু দুটোতেই একই Windows চলে। আবার তুমি আজ যে program লিখলে, সেটা ভবিষ্যতের আরও দ্রুত chip-এও চলবে — যতক্ষণ ISA এক থাকে। কিছু চেনা উদাহরণ:

- **x86/x64** — Intel, AMD-র desktop/laptop processor।
- **ARM** — প্রায় সব ফোন আর tablet।
- **RISC-V** — open-source, যে কেউ বিনামূল্যে ব্যবহার করতে পারে ← **আমরা এটাই বানাবো!**
- **MIPS** — পড়াশোনার জন্য জনপ্রিয়।

### RISC vs CISC:

ISA নিয়ে গত কয়েক দশকে একটা বড় দার্শনিক বিতর্ক চলেছে: instruction গুলো কেমন হওয়া উচিত? দুটো ঘরানা আছে।

**CISC (Complex Instruction Set Computer)** বলে — "একটা instruction দিয়েই অনেক কাজ করিয়ে নাও।" যেমন x86-এ এমন একটা instruction আছে যা একই সাথে memory থেকে data আনে, যোগ করে, আবার memory-তে ফেরত রাখে। সুবিধা: program ছোট হয়, compiler-এর কাজ সহজ। অসুবিধা: হার্ডওয়্যার জটিল, আর প্রতিটা instruction চালাতে কত সময় লাগবে তা আগে থেকে বলা কঠিন।

**RISC (Reduced Instruction Set Computer)** বলে উল্টোটা — "প্রতিটা instruction ছোট, সরল আর দ্রুত রাখো; বড় কাজ হলে কয়েকটা instruction জুড়ে করে নাও।" সুবিধা: হার্ডওয়্যার সরল, timing নিয়মিত, pipeline করা সহজ। এই কারণেই আজকের প্রায় সব নতুন ডিজাইন (ARM, RISC-V) RISC।

পাশাপাশি রাখলে পার্থক্যটা স্পষ্ট হয়:

| Feature | RISC | CISC |
|---|---|---|
| Philosophy | সরল instruction | জটিল instruction |
| Instructions | কম, সহজ | বেশি, বিচিত্র |
| Cycles/instruction | ১ (ideal) | একাধিক |
| Instruction size | Fixed (নির্দিষ্ট) | Variable (পরিবর্তনশীল) |
| Registers | বেশি (৩২টি) | কম (৮–১৬টি) |
| Addressing | সরল | জটিল |
| Compiler | জটিল | সরল |
| Examples | ARM, RISC-V | x86 |

আমরা **RISC** পথেই হাঁটব, আর কারণগুলো খুবই বাস্তব:

- **সরল হার্ডওয়্যার** — কম gate মানে তোমার পক্ষে নিজে বানানো সম্ভব।
- **Pipelining সহজ** — সব instruction একই আকারের হওয়ায় পরের chapter-গুলোতে pipeline বানানো সহজ হবে।
- **Predictable timing** — কোন instruction কত সময় নেবে তা আগে থেকেই জানা।
- **শেখার জন্য সেরা** — কম জটিলতা মানে তুমি গঠনটা পরিষ্কার দেখতে পাবে।

### আমাদের Simple ISA (শেখার জন্য):

বড় ISA সরাসরি বানাতে গেলে ঘাবড়ে যাবে। তাই এই chapter-এ আমরা নিজেরাই একটা **খেলনা ISA** ডিজাইন করব — যথেষ্ট ছোট যেন মাথায় পুরোটা ধরে, আবার যথেষ্ট সম্পূর্ণ যেন সত্যিকারের program চালানো যায়। পরের chapter-এ আমরা এই ভিত্তির উপরেই আসল RISC-V-তে যাব।

আমাদের processor-টা হবে:

- **8-bit data width** — একবারে ৮ bit data নিয়ে কাজ করে।
- **8-bit address** — মানে $2^8 = 256$ byte পর্যন্ত memory ঠিকানা দেওয়া যায়।
- **৪টি register** (R0–R3) — অল্প কিন্তু যথেষ্ট।
- **৯টি instruction** — মূল কাজগুলো ঢেকে ফেলার মতো।

প্রতিটা instruction ঠিক ৮ bit — fixed size, একদম RISC ঘরানার। ৮টা bit তিন ভাগে ভাগ করা: কোন কাজ (opcode), আর দুটো register কোনগুলো (Reg A, Reg B)।

```
 bit:   7   6   5   4   3   2   1   0
       ┌───────────────┬───────┬───────┐
       │    Opcode     │ Reg A │ Reg B │
       │    (4 bit)    │ (2 b) │ (2 b) │
       └───────────────┴───────┴───────┘
```

Opcode ৪ bit মানে $2^4 = 16$টা ভিন্ন কাজ লেখা সম্ভব, যার মধ্যে আমরা ৯টা ব্যবহার করছি। আর Reg A / Reg B ২ bit করে — $2^2 = 4$টা register-কে ঠিকঠাক চিনিয়ে দেওয়ার জন্য যথেষ্ট। নিচে আমাদের ৯টা instruction:

| Opcode | Instruction | কাজ |
|---|---|---|
| `0000` | NOP | কিছুই করে না (no operation) |
| `0001` | LOAD | memory থেকে register-এ data আনা |
| `0010` | STORE | register থেকে memory-তে data রাখা |
| `0011` | ADD | দুই register যোগ করা |
| `0100` | SUB | বিয়োগ করা |
| `0101` | AND | bit-wise AND |
| `0110` | OR | bit-wise OR |
| `0111` | JUMP | শর্তহীনভাবে অন্য জায়গায় লাফ দেওয়া |
| `1000` | BEQ | সমান হলে branch করা (branch if equal) |

ছোট, কিন্তু সম্পূর্ণ — যোগ-বিয়োগ আছে, logic আছে, memory আছে, আর loop বানানোর জন্য jump/branch-ও আছে। এটুকু দিয়েই সত্যিকারের program লেখা যায়, যা তুমি একটু পরেই দেখবে!

---

## ১২.৩ CPU Components

এবার আসল মজা শুরু। একটা processor আসলে কয়েকটা ছোট ছোট building block-এর সমষ্টি, আর তুমি এর প্রতিটাই এর আগের chapter-গুলোতে বানিয়েছো। চলো প্রতিটা block আলাদা করে বুঝে নিই — কে কী কাজ করে, কেন দরকার — তারপর একসাথে জুড়ে দেব। নিচের ছবিটা মাথায় রেখো, পুরো section জুড়ে আমরা এই অংশগুলোই একে একে বানাবো:

```mermaid
flowchart LR
    PC["Program Counter<br/>(কোন instruction এখন?)"] --> IR["Instruction Register<br/>(instruction ধরে রাখা)"]
    IR --> CU["Control Unit<br/>(decode + নির্দেশ)"]
    IR --> RF["Register File<br/>(দ্রুত storage)"]
    RF --> ALU["ALU<br/>(হিসাব-নিকাশ)"]
    CU -.->|control signals| ALU
    CU -.->|control signals| RF
    CU -.->|control signals| PC
    ALU --> RF
```

পাঁচটা অংশ: **PC, IR, Register File, ALU, Control Unit**। কঠিন তীরগুলো (data path) দেখাচ্ছে data কোন পথে বয়ে যায়, আর ফুটকি-তীরগুলো (control signals) দেখাচ্ছে control unit কীভাবে বাকিদের পরিচালনা করে। এই দুই ধরনের সংযোগের পার্থক্যটা মনে রাখা খুব জরুরি — data আর control আলাদা জিনিস।

### ১. Program Counter (PC):

প্রথম প্রশ্ন: processor কীভাবে জানে এখন **কোন** instruction চালাতে হবে? উত্তর হলো **Program Counter (PC)** — একটা ছোট্ট register যা memory-তে পরের instruction-এর ঠিকানা ধরে রাখে। তুমি একটা বই পড়ার সময় আঙুল দিয়ে যে লাইনটা পড়ছ তা দেখিয়ে রাখো না? PC ঠিক সেই আঙুল।

প্রতিবার একটা instruction শেষ হলে PC সাধারণত **এক বেড়ে যায়** (`PC = PC + 1`), যাতে পরের বার পরের ঠিকানার instruction আসে — অর্থাৎ লাইনগুলো একটার পর একটা পড়া হয়। কিন্তু jump বা branch হলে আঙুলটা লাফ দিয়ে অন্য লাইনে চলে যায় — তখন PC-তে সরাসরি গন্তব্যের ঠিকানা বসে যায়। এই "+1 না লাফ" সিদ্ধান্তটাই program-এর flow নিয়ন্ত্রণ করে। আর reset হলে PC `0`-তে ফিরে যায়, কারণ আমাদের program সবসময় address `0` থেকে শুরু:

```verilog
// Program Counter - tracks current instruction
module program_counter(
    input wire clk,
    input wire reset,
    input wire [7:0] pc_next,  // Next PC value
    input wire pc_write,        // Enable write
    output reg [7:0] pc         // Current PC
);
    always @(posedge clk or posedge reset) begin
        if (reset)
            pc <= 8'h00;  // Start at address 0
        else if (pc_write)
            pc <= pc_next;
    end
endmodule

// Normal operation: PC = PC + 1
// Branch/Jump: PC = target address
```

খেয়াল করো `pc_write` signal-টা — control unit যখন এটা `1` করে দেয় কেবল তখনই PC নতুন মান নেয়। মানে control unit ঠিক করে দেয় কখন আঙুলটা সামনে এগোবে আর কখন দাঁড়িয়ে থাকবে। এই ধরনের "enable" signal দিয়েই control unit গোটা datapath-কে তাল মিলিয়ে চালায়।

### ২. Instruction Register (IR):

PC শুধু ঠিকানা দেয়; কিন্তু সেই ঠিকানা থেকে memory যে instruction-টা ফেরত পাঠায়, সেটাকে কোথাও তো রাখতে হবে যাতে decode আর execute করার সময়টুকু হাতে পাওয়া যায়। সেই কাজটাই করে **Instruction Register (IR)** — এটা চলমান instruction-টাকে আঁকড়ে ধরে রাখে।

কেন আলাদা register লাগে? কারণ পরের cycle-এ PC বদলে যেতে পারে, memory অন্য ঠিকানার data দিতে শুরু করতে পারে — কিন্তু আমরা চাই এখনকার instruction-টা পুরো execution জুড়ে স্থির থাকুক। IR হলো সেই স্থির ছবি (snapshot): একবার ভেতরে নিয়ে নিলে, পরে memory যা-ই করুক, আমাদের instruction নিরাপদ।

```verilog
// Holds current instruction being executed
module instruction_register(
    input wire clk,
    input wire reset,
    input wire [7:0] instruction,
    input wire ir_write,
    output reg [7:0] ir
);
    always @(posedge clk or posedge reset) begin
        if (reset)
            ir <= 8'h00;
        else if (ir_write)
            ir <= instruction;
    end
endmodule

// Decode fields:
wire [3:0] opcode = ir[7:4];
wire [1:0] reg_a  = ir[3:2];
wire [1:0] reg_b  = ir[1:0];
```

লক্ষ করো instruction-টাকে আলাদা করে পড়তে কোনো জটিল হিসাব লাগছে না — শুধু সঠিক bit-গুলো কেটে নেওয়া হচ্ছে। উপরের bit-field ছবিটা মনে আছে? উপরের ৪ bit (`ir[7:4]`) opcode, পরের ২ bit (`ir[3:2]`) Reg A, শেষ ২ bit (`ir[1:0]`) Reg B। Fixed-size instruction-এর এটাই বড় সুবিধা — প্রতিটা ক্ষেত্র সবসময় একই জায়গায় থাকে, তাই decode করা প্রায় বিনামূল্যে। এই সরলতাই RISC-কে এত শক্তিশালী করে।

### ৩. Register File:

ALU যখন যোগ করবে, তখন সংখ্যা দুটো কোথা থেকে আসবে? প্রতিবার ধীরগতির memory-তে যাওয়া অপচয় হবে। তাই processor-এর ভেতরেই অল্প কিছু অতি-দ্রুত storage রাখা হয় — এদের বলে **register**, আর এদের একসাথে রাখা সংগ্রহটাকে বলে **Register File**। এগুলো processor-এর হাতের নাগালে থাকা ছোট্ট workbench: যে data নিয়ে এখন কাজ হচ্ছে, তা এখানেই রাখা থাকে।

একটা ব্যাপার খেয়াল করার মতো — এই register file-এ **দুটো read port আর একটা write port**। কেন দুটো read? কারণ `ADD R1, R2`-এর মতো instruction-এ ALU-র একই সময়ে **দুটো** operand লাগে, তাই দুটো register একসাথে পড়তে পারা চাই। আর read গুলো **combinational** (`assign` দিয়ে, সাথে সাথে মান পাওয়া যায়), কিন্তু write হয় **sequential** (`posedge clk`-এ, clock-এর ধারে)। এই পার্থক্যটা গুরুত্বপূর্ণ: তুমি যেকোনো মুহূর্তে register পড়তে পারো, কিন্তু নতুন মান লেখা হয় শুধু clock-এর টিক্-এ, আর তাও যখন `write_enable` চালু থাকে — অর্থাৎ আবারও control unit-ই ঠিক করে দেয় কখন লেখা হবে।

```verilog
// Register File - fast storage
module register_file(
    input wire clk,
    input wire reset,
    // Read ports
    input wire [1:0] read_addr1,
    input wire [1:0] read_addr2,
    output wire [7:0] read_data1,
    output wire [7:0] read_data2,
    // Write port
    input wire [1:0] write_addr,
    input wire [7:0] write_data,
    input wire write_enable
);
    // 4 registers: R0, R1, R2, R3
    reg [7:0] registers [0:3];
    
    // Combinational read
    assign read_data1 = registers[read_addr1];
    assign read_data2 = registers[read_addr2];
    
    // Sequential write
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            registers[0] <= 8'h00;
            registers[1] <= 8'h00;
            registers[2] <= 8'h00;
            registers[3] <= 8'h00;
        end else if (write_enable) begin
            registers[write_addr] <= write_data;
        end
    end
endmodule
```

### ৪. ALU (Arithmetic Logic Unit):

এসে গেছি processor-এর হৃদয়ে। **ALU (Arithmetic Logic Unit)** হলো সেই অংশ যেখানে আসল হিসাব-নিকাশ হয় — যোগ, বিয়োগ, AND, OR, shift, সবকিছু। তুমি Chapter 3-এ যে adder আর logic circuit বানিয়েছিলে, ALU হলো তাদেরই একটা দল, যাদের সামনে একটা switch বসানো — সেই switch দিয়ে ঠিক করা হয় এই মুহূর্তে কোন operation-টা চাই।

সেই switch-টাই হলো `alu_op` input। ভেতরে একটা `case` statement বসে আছে: `alu_op` যদি `OP_ADD` হয় তো ALU যোগ করে, `OP_SUB` হলে বিয়োগ — এভাবে। তুমি একটা ক্যালকুলেটরে যেমন আগে operation বোতাম চাপো তারপর সে হিসাব করে, control unit ঠিক তেমনি `alu_op` সেট করে দেয়, আর ALU সেই অনুযায়ী কাজ করে।

ALU শুধু ফলাফল দেয় না, সাথে দুটো ছোট্ট **flag**-ও দেয় যা পরে সিদ্ধান্ত নিতে কাজে লাগে:

- **`zero`** — ফলাফল কি ঠিক `0`? এটাই `BEQ` (branch if equal)-এর প্রাণ: দুটো সংখ্যা সমান কিনা জানতে হলে বিয়োগ করে দেখো ফল `0` কিনা।
- **`negative`** — ফলাফল কি ঋণাত্মক (সবচেয়ে উপরের bit `1`)? signed তুলনার সময় দরকার হয়।

এই flag-গুলোই হলো datapath থেকে control unit-এর কাছে ফিরে আসা খবর — এদের সাহায্যেই processor "যদি... তাহলে..." ধরনের সিদ্ধান্ত নিতে পারে।

```verilog
module alu(
    input wire [7:0] a,
    input wire [7:0] b,
    input wire [2:0] alu_op,
    output reg [7:0] result,
    output wire zero,
    output wire negative
);
    // ALU operations
    localparam OP_ADD = 3'b000;
    localparam OP_SUB = 3'b001;
    localparam OP_AND = 3'b010;
    localparam OP_OR  = 3'b011;
    localparam OP_XOR = 3'b100;
    localparam OP_SLL = 3'b101;  // Shift left
    localparam OP_SRL = 3'b110;  // Shift right
    localparam OP_PASS= 3'b111;  // Pass through
    
    always @(*) begin
        case (alu_op)
            OP_ADD:  result = a + b;
            OP_SUB:  result = a - b;
            OP_AND:  result = a & b;
            OP_OR:   result = a | b;
            OP_XOR:  result = a ^ b;
            OP_SLL:  result = a << b[2:0];
            OP_SRL:  result = a >> b[2:0];
            OP_PASS: result = a;
            default: result = 8'h00;
        endcase
    end
    
    // Flags
    assign zero = (result == 8'h00);
    assign negative = result[7];
endmodule
```

### ৫. Control Unit:

এতক্ষণ আমরা datapath-এর অংশগুলো বানালাম — কিন্তু এরা নিজে থেকে কিছু করে না। PC জানে না কখন এগোতে হবে, ALU জানে না কোন operation করতে হবে, register file জানে না কখন লিখতে হবে। এদের সবাইকে সঠিক সময়ে সঠিক নির্দেশ দেওয়াই **Control Unit**-এর কাজ। এ হলো সেই অর্কেস্ট্রার পরিচালক — নিজে কোনো বাদ্যযন্ত্র বাজায় না, কিন্তু তার ইশারাতেই গোটা দল একসুরে বাজে।

Control unit কাজটা করে একটা **state machine (FSM)** দিয়ে — তুমি Chapter 4-এ যা শিখেছিলে। আমাদের সেই চেনা cycle-টার প্রতিটা ধাপই এখানে একেকটা state: FETCH → DECODE → EXECUTE → MEMORY → WRITEBACK, তারপর আবার FETCH। প্রতিটা state-এ control unit আলাদা control signal-এর সমাহার চালু করে — যেমন FETCH state-এ সে memory পড়ার আর IR-এ লেখার signal দেয়, আর WRITEBACK state-এ register-এ লেখার আর PC এগোনোর signal দেয়।

```mermaid
stateDiagram-v2
    [*] --> FETCH
    FETCH --> DECODE
    DECODE --> EXECUTE
    EXECUTE --> MEMORY
    MEMORY --> WRITEBACK
    WRITEBACK --> FETCH
```

কোডে দুটো আলাদা `always` block দেখবে, আর এই ভাগটা ইচ্ছাকৃত। প্রথমটা **sequential** (`posedge clk`) — শুধু এক state থেকে পরের state-এ যাওয়াটা সামলায়, clock-এর তালে তালে। দ্বিতীয়টা **combinational** (`always @(*)`) — বর্তমান state দেখে ঠিক করে এখন কোন কোন signal চালু থাকবে। খেয়াল করো, এই দ্বিতীয় block-এর শুরুতেই সব signal default-এ `0` করে দেওয়া হয়েছে; এটা একটা চমৎকার অভ্যাস, কারণ এতে ভুলবশত আগের মান রয়ে গিয়ে latch তৈরি হওয়ার বিপদ থাকে না। এই "state বদলানো" আর "signal তৈরি করা"-কে আলাদা রাখাটাই পরিষ্কার FSM ডিজাইনের চাবিকাঠি।

```verilog
module control_unit(
    input wire clk,
    input wire reset,
    input wire [3:0] opcode,
    input wire zero_flag,
    // Control signals
    output reg pc_write,
    output reg ir_write,
    output reg reg_write,
    output reg mem_read,
    output reg mem_write,
    output reg [2:0] alu_op,
    output reg [1:0] alu_src,
    output reg pc_src
);
    // State machine
    localparam FETCH   = 3'b000;
    localparam DECODE  = 3'b001;
    localparam EXECUTE = 3'b010;
    localparam MEMORY  = 3'b011;
    localparam WRITEBACK = 3'b100;
    
    reg [2:0] state;
    
    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= FETCH;
        else begin
            case (state)
                FETCH:     state <= DECODE;
                DECODE:    state <= EXECUTE;
                EXECUTE:   state <= MEMORY;
                MEMORY:    state <= WRITEBACK;
                WRITEBACK: state <= FETCH;
                default:   state <= FETCH;
            endcase
        end
    end
    
    // Control signal generation
    always @(*) begin
        // Default values
        pc_write = 0;
        ir_write = 0;
        reg_write = 0;
        mem_read = 0;
        mem_write = 0;
        alu_op = 3'b000;
        alu_src = 2'b00;
        pc_src = 0;
        
        case (state)
            FETCH: begin
                mem_read = 1;
                ir_write = 1;
            end
            
            DECODE: begin
                // Decode instruction
            end
            
            EXECUTE: begin
                case (opcode)
                    4'b0011: alu_op = 3'b000; // ADD
                    4'b0100: alu_op = 3'b001; // SUB
                    4'b0101: alu_op = 3'b010; // AND
                    4'b0110: alu_op = 3'b011; // OR
                    default: alu_op = 3'b000;
                endcase
            end
            
            WRITEBACK: begin
                reg_write = 1;
                pc_write = 1;
            end
        endcase
    end
endmodule
```

---

## ১২.৪ Instruction Execution - বিস্তারিত

আমরা যন্ত্রাংশগুলো চিনেছি; এবার দেখি একটা instruction আসলে কীভাবে এই পাঁচটা ধাপের মধ্য দিয়ে জীবন কাটায়। ভাবো এটা যেন একটা উৎপাদন লাইন (assembly line) — প্রতিটা ধাপে instruction-টার উপর নির্দিষ্ট একটা কাজ হয়, তারপর সে পরের ধাপে এগিয়ে যায়। প্রতিটা ধাপ ঠিক এক clock cycle নেয়, তাই পাঁচ ধাপ মিলিয়ে আমাদের processor প্রতি instruction-এ পাঁচটা cycle খরচ করে। চলো একটা একটা করে দেখি।

### Fetch Stage (আনার ধাপ):

**উদ্দেশ্য:** memory থেকে পরের instruction-টা নিয়ে আসা।

এটা প্রতিটা instruction-এর প্রথম ধাপ — যেখানে processor বলে, "আমার পরের কাজটা কী, একটু এনে দাও।" PC যে ঠিকানাটা ধরে আছে, সেটা memory-কে দেওয়া হয়; memory সেই ঠিকানার instruction ফেরত পাঠায়; সেটাকে IR-এ তুলে রাখা হয়; আর সাথে সাথে PC এক বাড়িয়ে রাখা হয় যাতে পরের বার পরের instruction আসে। ধাপে ধাপে:

1. PC → Memory-র address-এ যায়।
2. Memory সেই ঠিকানা থেকে instruction পড়ে।
3. সেই instruction → IR-এ জমা হয়।
4. PC ← PC + 1 (পরের বারের জন্য তৈরি)।

ছবিটা যদি pseudocode-এ আঁকি:

```
address     = PC;            // PC-র ঠিকানা memory-কে দাও
instruction = memory[address]; // সেখান থেকে instruction পড়ো
IR          = instruction;   // IR-এ ধরে রাখো
PC          = PC + 1;        // আঙুল পরের লাইনে সরাও
```

পুরোটা হয় **এক clock cycle**-এ — একটা memory read আর একটা IR update। এই ধাপ শেষে আমাদের হাতে instruction আছে, কিন্তু আমরা এখনো জানি না সেটা কী বলছে — সেটা পরের ধাপের কাজ।

### Decode Stage (বোঝার ধাপ):

**উদ্দেশ্য:** হাতে আসা instruction-টা আসলে কী চাইছে, তা বুঝে নেওয়া।

Fetch করে আমরা ৮টা bit পেলাম — কিন্তু খালি bit তো অর্থহীন। Decode ধাপে control unit সেই bit-গুলো পড়ে বুঝে নেয়: কোন operation, কোন register-গুলো জড়িত। তারপর সে সেই অনুযায়ী control signal তৈরি করতে শুরু করে আর দরকারি register-গুলো পড়ে নেয়। ধাপে ধাপে:

1. IR থেকে opcode আলাদা করো।
2. operand (register-গুলো) আলাদা করো।
3. সেই অনুযায়ী control signal তৈরি করো।
4. দরকার হলে register-এর মান পড়ে নাও।

একটা উদাহরণ দিয়ে দেখা যাক। ধরো IR-এ আছে `00110110`, যার মানে `ADD R1, R2`। bit-গুলো এভাবে ভাগ হয়:

```
 bit:   7   6   5   4   3   2   1   0
       ┌───────────────┬───────┬───────┐
       │  0   0   1   1│  0   1│  1   0│
       └───────────────┴───────┴───────┘
        Opcode = 0011   RegA=01 RegB=10
        (ADD)           (R1)    (R2)
```

এই decode থেকে control unit বুঝে নেয় ঠিক কী করতে হবে:

- **ALU operation:** ADD
- **Register read:** R1 আর R2 (operand দুটো)
- **Register write:** R1 (ফল এখানেই ফিরবে)
- **Memory:** কোনো access লাগবে না

লক্ষ করো — এক ঝলকে instruction-টা পুরো "পড়া" হয়ে গেল, শুধু ঠিক bit-গুলো দেখে। এটাই fixed-format RISC instruction-এর সৌন্দর্য, যা আমরা আগেই বলেছিলাম।

### Execute Stage (কাজের ধাপ):

**উদ্দেশ্য:** আসল operation-টা চালানো।

Decode করে আমরা জেনে গেছি কী করতে হবে; Execute ধাপে সেটা সত্যিই করা হয়। বেশির ভাগ instruction-এর জন্য এর মানে ALU-কে দিয়ে হিসাব করানো। কিন্তু "হিসাব" শব্দটা একটু বড় অর্থে ভেবো — Execute ধাপে ALU তিন ধরনের কাজের যেকোনো একটা করতে পারে:

1. সরাসরি **গাণিতিক/যৌক্তিক হিসাব** (যোগ, AND ইত্যাদি)।
2. memory-র **address হিসাব করা** (LOAD/STORE-এর জন্য কোথায় যেতে হবে)।
3. branch/jump-এর **গন্তব্য ঠিকানা হিসাব করা**।

কয়েকটা উদাহরণে পরিষ্কার হবে:

```
ADD R1, R2     →  ALU_result = R1 + R2     (যোগফল বের করা)
LOAD R1, [R2]  →  Address    = R2          (কোন ঠিকানা পড়তে হবে)
JUMP target    →  PC_next    = target      (কোথায় লাফ দিতে হবে)
```

মজার ব্যাপারটা হলো — তিন ক্ষেত্রেই একই ALU ব্যবহার হচ্ছে, শুধু control unit তাকে আলাদা কাজে লাগাচ্ছে। একটা হার্ডওয়্যার, অনেক কাজে — এই পুনর্ব্যবহারই দক্ষ ডিজাইনের লক্ষণ। সময়: **এক clock cycle**, একটা ALU operation, একটা ফল তৈরি।

### Memory Stage (memory-র ধাপ):

**উদ্দেশ্য:** দরকার হলে memory-তে পড়া বা লেখা।

সব instruction-এর memory লাগে না — শুধু LOAD আর STORE-এর জন্যই এই ধাপটা আসল কিছু করে। Execute ধাপে যে address হিসাব হয়েছিল, এখানে সেটা ব্যবহার করা হয়:

1. **LOAD:** memory থেকে data পড়ে আনা → `data = memory[address]`
2. **STORE:** register-এর data memory-তে লিখে রাখা → `memory[address] = data`
3. **বাকি সব instruction:** এই ধাপে কিছুই করার নেই, পার হয়ে যায়।

তাই সময়ের হিসাবে: memory access থাকলে **এক clock cycle**, না থাকলে কার্যত শূন্য (ধাপটা শুধু পার হয়ে যায়)। ADD-এর মতো instruction এখানে এসে যেন একটা ফাঁকা স্টেশন পার করে এগিয়ে যায়।

### Writeback Stage (ফল রাখার ধাপ):

**উদ্দেশ্য:** এতক্ষণের পরিশ্রমের ফলটা জমা রাখা।

এ হলো শেষ ধাপ — যেখানে instruction তার ফলাফল রেখে বিদায় নেয়। হয় ALU-র ফল register-এ লেখা হয়, নয়তো memory থেকে আনা data register-এ লেখা হয়; আর সবশেষে PC update হয় যাতে পরের instruction-এর পালা শুরু হতে পারে। ধাপে ধাপে:

1. ALU-র ফল register-এ লেখো, **অথবা**
2. memory থেকে আনা data register-এ লেখো।
3. PC update করো।

দুটো উদাহরণ পাশাপাশি রাখলে পার্থক্যটা বোঝা যায় — শুধু register-এ কোন data ফিরছে সেটুকুই আলাদা:

```
ADD R1, R2     →  R1 ← ALU_result    ;  PC ← PC + 1
LOAD R1, [R2]  →  R1 ← memory_data   ;  PC ← PC + 1
```

সময়: **এক clock cycle** — একটা register write আর একটা PC update। এই ধাপ শেষ হওয়ামাত্র control unit আবার FETCH state-এ ফিরে যায়, আর পুরো চক্রটা নতুন instruction নিয়ে আবার শুরু হয়। এভাবেই, ধাপের পর ধাপ, cycle-এর পর cycle, তোমার processor একটা গোটা program চালিয়ে ফেলে।

---

## ১২.৫ Simple 8-bit Processor - সম্পূর্ণ Design

এতক্ষণ আমরা একেকটা যন্ত্রাংশ আলাদা আলাদা বানিয়েছি — PC, IR, register file, ALU, control unit। এবার সবচেয়ে তৃপ্তিদায়ক ধাপ: এদের একটা **top-level module**-এ জুড়ে দিয়ে একটা সত্যিকারের processor বানানো! এই top module নিজে নতুন কোনো হিসাব করে না — তার পুরো কাজ হলো ছোট module-গুলোকে instantiate করা আর তাদের মধ্যে ঠিকঠাক তার (wire) জুড়ে দেওয়া। ভাবো এটা যেন একটা বর্তনী-বোর্ড (circuit board), যার উপর তুমি chip-গুলো বসিয়ে তার দিয়ে সংযোগ করছ।

কোডটা পড়ার সময় তিনটে জিনিস খেয়াল রেখো, তাহলে পুরোটা জলের মতো পরিষ্কার লাগবে:

- **প্রতিটা module instantiate হচ্ছে** — `program_counter pc_inst(...)`-এর মতো করে আমাদের আগের প্রতিটা block এখানে বসছে।
- **`assign` দিয়ে তার জোড়া লাগছে** — যেমন `assign opcode = ir[7:4];` দিয়ে IR-এর bit থেকে opcode বের করে control unit-এ পাঠানো হচ্ছে, বা `assign pc_next = pc + 1;` দিয়ে PC বাড়ানোর তার বসানো হচ্ছে।
- **এক module-এর output পরের module-এর input হচ্ছে** — register file-এর `reg_read1` সোজা ALU-র `a`-তে যাচ্ছে, ALU-র `alu_result` ঘুরে register-এর write data হয়ে ফিরছে। এই সংযোগগুলোই আমাদের আগের block-diagram-টাকে জীবন্ত করে তোলে।

```verilog
module simple_processor(
    input wire clk,
    input wire reset,
    // Memory interface
    output wire [7:0] mem_addr,
    input wire [7:0] mem_read_data,
    output wire [7:0] mem_write_data,
    output wire mem_read,
    output wire mem_write,
    // Debug outputs
    output wire [7:0] pc_out,
    output wire [7:0] ir_out
);
    // Internal signals
    wire [7:0] pc, pc_next;
    wire [7:0] ir;
    wire [7:0] alu_a, alu_b, alu_result;
    wire [7:0] reg_read1, reg_read2, reg_write_data;
    wire [1:0] reg_addr1, reg_addr2, reg_write_addr;
    wire [2:0] alu_op;
    wire [3:0] opcode;
    wire zero, negative;
    wire pc_write, ir_write, reg_write;
    
    // Program Counter
    program_counter pc_inst(
        .clk(clk),
        .reset(reset),
        .pc_next(pc_next),
        .pc_write(pc_write),
        .pc(pc)
    );
    
    // Instruction Register
    instruction_register ir_inst(
        .clk(clk),
        .reset(reset),
        .instruction(mem_read_data),
        .ir_write(ir_write),
        .ir(ir)
    );
    
    // Decode instruction
    assign opcode = ir[7:4];
    assign reg_addr1 = ir[3:2];
    assign reg_addr2 = ir[1:0];
    assign reg_write_addr = ir[3:2];
    
    // Register File
    register_file rf_inst(
        .clk(clk),
        .reset(reset),
        .read_addr1(reg_addr1),
        .read_addr2(reg_addr2),
        .read_data1(reg_read1),
        .read_data2(reg_read2),
        .write_addr(reg_write_addr),
        .write_data(reg_write_data),
        .write_enable(reg_write)
    );
    
    // ALU
    assign alu_a = reg_read1;
    assign alu_b = reg_read2;
    
    alu alu_inst(
        .a(alu_a),
        .b(alu_b),
        .alu_op(alu_op),
        .result(alu_result),
        .zero(zero),
        .negative(negative)
    );
    
    // Control Unit
    control_unit cu_inst(
        .clk(clk),
        .reset(reset),
        .opcode(opcode),
        .zero_flag(zero),
        .pc_write(pc_write),
        .ir_write(ir_write),
        .reg_write(reg_write),
        .mem_read(mem_read),
        .mem_write(mem_write),
        .alu_op(alu_op),
        .alu_src(),
        .pc_src()
    );
    
    // PC increment
    assign pc_next = pc + 1;
    
    // Memory interface
    assign mem_addr = pc;
    assign mem_write_data = reg_read2;
    assign reg_write_data = alu_result;
    
    // Debug
    assign pc_out = pc;
    assign ir_out = ir;
endmodule
```

ব্যস — তুমি একটা সম্পূর্ণ processor বানিয়ে ফেললে! একটু থেমে এই অর্জনটা অনুভব করো: কয়েকটা ছোট, বোধগম্য module আর তাদের মাঝের কিছু তার — এটুকু দিয়েই এমন একটা যন্ত্র দাঁড়িয়ে গেল যা memory থেকে instruction পড়ে নিজে নিজে চালাতে পারে। শেষের দিকের `pc_out` আর `ir_out` debug output দুটোও খেয়াল করো — সিমুলেশনের সময় এগুলো দিয়ে তুমি ভেতরে উঁকি দিয়ে দেখতে পারবে কোন instruction-এ আছ আর PC কোথায়, যা debug করার সময় সোনার চেয়েও দামি।

---

## ১২.৬ Sample Programs

তত্ত্ব অনেক হলো — এবার দেখি তোমার processor সত্যিই কিছু **করতে** পারে কিনা! এখানে তিনটে ছোট program দেওয়া হলো, ক্রমশ কঠিন হতে হতে। প্রতিটা program আসলে memory-তে রাখা কিছু ৮-bit সংখ্যা — মনে রেখো, আমাদের instruction নিজেও তো শুধু bit, যা memory-তেই থাকে (এই তো von Neumann-এর stored-program ধারণা কাজে লাগছে!)। প্রতিটা লাইনের ডান পাশে human-readable assembly, বাঁ পাশে processor যে binary দেখবে তা।

### Program 1: দুটো সংখ্যা যোগ

সবচেয়ে সহজটা দিয়ে শুরু — দুটো register যোগ করে থেমে যাওয়া। R1-এ যদি 5 আর R2-তে 3 থাকে, চালানোর পর R1-এ হবে 8। মাত্র দুটো instruction: একটা যোগ, একটা NOP (যা কার্যত "এখানে থামো"-র কাজ করে)।

```assembly
; Add R1 and R2, store in R1
; R1 = 5, R2 = 3
; Expected: R1 = 8

Address | Instruction | Description
--------|-------------|-------------
0x00    | 0011_01_10  | ADD R1, R2
0x01    | 0000_00_00  | NOP (halt)

Binary:
0x00: 00110110  (ADD R1, R2)
0x01: 00000000  (NOP)
```

### Program 2: Load এবং Store

এবার memory-কে কাজে লাগাই। এই program memory থেকে একটা সংখ্যা পড়ে (LOAD), তার সাথে R2 যোগ করে, তারপর ফলটা একই জায়গায় ফেরত লিখে রাখে (STORE)। অর্থাৎ memory-তে রাখা একটা মানকে জায়গায় বসিয়েই বাড়িয়ে দেওয়া। এখানে তুমি প্রথমবার datapath-এর পুরো যাত্রা দেখছ — memory → register → ALU → memory।

```assembly
; Load value from memory, add, store back
Address | Instruction | Description
--------|-------------|-------------
0x00    | 0001_01_00  | LOAD R1, [R0]
0x01    | 0011_01_10  | ADD R1, R2
0x02    | 0010_01_00  | STORE R1, [R0]
0x03    | 0000_00_00  | NOP

; If memory[R0] = 10, R2 = 5
; Result: memory[R0] = 15
```

### Program 3: একটা সাধারণ Loop

এবার আসল মজা — একটা **loop**! এই program একটা counter-কে বারবার এক করে বাড়ায়, প্রতিবার পরীক্ষা করে সেটা লক্ষ্যে (10) পৌঁছেছে কিনা। এখানেই BEQ আর JUMP-এর শক্তি দেখা যায়: BEQ পরীক্ষা করে "R1 কি R0-র সমান?" — যদি হয়, loop থেকে বেরিয়ে যায়; না হলে JUMP আবার শুরুতে ফিরিয়ে নিয়ে যায়। এই "পরীক্ষা করো, না হলে আবার লাফ দাও" প্যাটার্নটাই পৃথিবীর সমস্ত loop-এর মূল কাঠামো — তোমার পরিচিত যেকোনো ভাষার `for` বা `while`-এর নিচেও ঠিক এটাই ঘটছে।

```assembly
; Count from 0 to 10
Address | Instruction | Description
--------|-------------|-------------
0x00    | 0001_01_11  | LOAD R1, [R3]  ; Load counter
0x01    | 0011_01_10  | ADD R1, R2     ; R2 = 1 (increment)
0x02    | 0010_01_11  | STORE R1, [R3] ; Save counter
0x03    | 1000_01_00  | BEQ R1, R0     ; If R1=R0, exit
0x04    | 0111_00_00  | JUMP 0x00      ; Loop back
0x05    | 0000_00_00  | NOP            ; Done

; R0 = 10 (target)
; R2 = 1  (increment)
; R3 = address of counter
```

---

## ১২.৭ Processor Performance

তোমার processor চলছে — কিন্তু কত **দ্রুত**? "দ্রুত" মাপার জন্য একটা পরিষ্কার পরিমাপ দরকার, আর সেখানেই আসে CPI আর MIPS-এর ধারণা। এই অংশটা বুঝলে তুমি বুঝবে কেন পরের chapter-গুলোতে আমরা এত কষ্ট করে pipeline বানাবো।

### Clock Cycles Per Instruction (CPI):

**CPI** মানে গড়ে একটা instruction চালাতে কতগুলো clock cycle লাগে — সংখ্যাটা যত ছোট, processor তত দক্ষ। একটা আদর্শ RISC processor-এর লক্ষ্য CPI = 1, অর্থাৎ প্রতি cycle-এ একটা করে instruction শেষ। কিন্তু আমাদের সরল processor প্রতিটা instruction-কে পাঁচটা আলাদা ধাপে নিয়ে যায়, প্রতিটা ধাপে এক cycle — তাই আমাদের CPI = 5:

| ধাপ | Cycle |
|---|---|
| Fetch | ১ |
| Decode | ১ |
| Execute | ১ |
| Memory (দরকার হলে) | ১ |
| Writeback | ১ |
| **মোট** | **৫** |

এখন স্বাভাবিক প্রশ্ন: আরও ভালো করা যায় না? **নিশ্চয়ই যায়!** খেয়াল করো, যখন একটা instruction Execute ধাপে আছে, তখন Fetch ইউনিটটা কিন্তু বসে আছে — অলস। যদি আমরা আগের instruction Execute করার সময়ই পরের instruction-টা Fetch করে ফেলতে পারতাম? এই ধারণাটার নামই **pipelining**, আর এটাই আমরা পরের chapter-গুলোতে বানাবো। আপাতত শুধু মনে রাখো — CPI = 5 আমাদের শুরুর জায়গা, শেষ নয়।

### Processor Speed (গতি):

CPI জানলে আসল গতি বের করা সহজ। প্রতি সেকেন্ডে কতগুলো instruction চলবে (IPS — Instructions Per Second) তা নির্ভর করে দুটো জিনিসের উপর: clock কত দ্রুত (frequency, $f$), আর প্রতি instruction-এ কত cycle লাগে (CPI)। সম্পর্কটা চমৎকার সরল:

$$\text{IPS} = \frac{f}{\text{CPI}}$$

চলো Tang Nano 9K board-এর বাস্তব সংখ্যা বসিয়ে দেখি, যেটা আমরা এই বইয়ে ব্যবহার করছি:

```
f   = 27 MHz  = 27,000,000 cycles/sec   (Tang Nano 9K)
CPI = 5

IPS = 27,000,000 / 5
    = 5,400,000 instructions/sec
    = 5.4 MIPS   (Million Instructions Per Second)
```

সেকেন্ডে ৫৪ লাখ instruction — তোমার নিজের হাতে গোড়া থেকে বানানো একটা সরল processor-এর জন্য মোটেও খারাপ নয়! আর মনে রেখো, CPI কমিয়ে (pipeline দিয়ে) বা clock বাড়িয়ে এই সংখ্যাটাকে আরও অনেক উপরে তোলা যায় — এটাই গোটা computer architecture বিদ্যার খেলা।

---

## ১২.৮ তোমার ১ সপ্তাহের Build Plan

পুরো processor একসাথে বানাতে গেলে পাহাড়ের মতো লাগতে পারে — তাই আমরা সেটাকে সাত দিনে, ছোট ছোট কামড়ে ভাগ করে নিচ্ছি। প্রতিদিন আগের দিনের উপর গড়ে উঠবে, ঠিক যেভাবে এই chapter-এ আমরা এক একটা block বানিয়েছি। প্রতিটা ধাপ শেষে নিজে testbench চালিয়ে নিশ্চিত হও — পরের ধাপে যাওয়ার আগে আগেরটা যেন সত্যিই কাজ করে। ধীরে চলো, কিন্তু থেমো না!

**Day 1 — PC ও IR:** processor-এর "এখন কোথায় আছি" অংশটা দিয়ে শুরু।
- [ ] Program Counter বানাও
- [ ] Instruction Register বানাও
- [ ] testbench দিয়ে test করো
- [ ] control flow-টা ভালো করে বুঝে নাও

**Day 2 — Register File:** processor-এর দ্রুত storage।
- [ ] register file ডিজাইন করো
- [ ] একাধিক read/write port বসাও
- [ ] ভালো করে test করো
- [ ] সমস্যা থাকলে debug করো

**Day 3 — ALU:** হিসাব-নিকাশের হৃদয়।
- [ ] পুরো ALU বানাও
- [ ] সব operation যোগ করো
- [ ] flag (zero, negative) তৈরি করো
- [ ] প্রতিটা operation আলাদা করে test করো

**Day 4 — Control Unit:** সবাইকে পরিচালনাকারী মস্তিষ্ক।
- [ ] state machine ডিজাইন করো
- [ ] control signal তৈরির অংশ লেখো
- [ ] instruction decode করাও
- [ ] FSM-টা test করো

**Day 5 — Integration:** সব অংশ এক জায়গায় জোড়া।
- [ ] সব component যুক্ত করো
- [ ] top-level module বানাও
- [ ] সব তার ঠিকঠাক জোড়া দাও
- [ ] প্রথমবার synthesis চালাও

**Day 6 — Memory Interface:** বাইরের memory-র সাথে সংযোগ।
- [ ] memory controller বানাও
- [ ] read/write timing ঠিক করো
- [ ] সবকিছুর সাথে integrate করো
- [ ] একটা সহজ program দিয়ে test করো

**Day 7 — Testing:** সব মিলিয়ে কাজ করছে কিনা যাচাই।
- [ ] sample program গুলো চালাও
- [ ] যেখানে আটকায় সেখানে debug করো
- [ ] waveform দেখে বিশ্লেষণ করো
- [ ] কী শিখলে লিখে রাখো

---

## ১২.৯ Chapter 12 Mission Complete!

একটু থেমে ভাবো তুমি ঠিক কী করলে। এই chapter শুরু করেছিলে শুধু কয়েকটা আলাদা circuit হাতে নিয়ে — gate, register, ALU। আর এখন? তুমি জানো এগুলো কীভাবে একসাথে জুড়ে একটা **জীবন্ত processor** হয়, যে memory থেকে instruction পড়ে, decode করে, চালায়, আর ফলাফল রাখে — বারবার, নিজে নিজে। "প্রসেসর কীভাবে কাজ করে?" — যে প্রশ্নটা দিয়ে এই বই শুরু হয়েছিল, তার উত্তরটা এখন আর রহস্য নয়, তোমার নিজের হাতে গড়া। এটা সত্যিকারের একটা মাইলফলক, নিজেকে একটা বাহবা দাও! 👏

### তুমি এখন জানো:

```
✅ Processor architecture fundamentals
✅ ISA concepts (RISC vs CISC)
✅ CPU components (PC, IR, RF, ALU, CU)
✅ Instruction execution cycle
✅ Control unit design
✅ Simple processor design
✅ Assembly programming basics
✅ তোমার processor architecture! 🎉
```

### তুমি ডিজাইন করেছো:
```
✅ Program Counter
✅ Instruction Register
✅ Register File (4 registers)
✅ ALU (8 operations)
✅ Control Unit (FSM)
✅ Complete 8-bit processor!
✅ Sample programs! 💻
```

### Stats:
```
Components designed: 5
Instructions: 9
Register file: 4 registers
Data width: 8-bit
CPI: 5 cycles
Level: Processor Architect! 🏆
```

### Next Level Unlocked:
```
→ Chapter 13: RISC-V Basics
   তুমি শিখবে: Real ISA!
   Industry-standard architecture!
   
   From toy → Professional CPU!
```

---

## 🎯 Final Project

পড়া আর বোঝা এক জিনিস, নিজে হাতে বাড়ানো আরেক জিনিস — আর আসল শেখা হয় দ্বিতীয়টাতেই। তাই এবার তোমার পালা: আমাদের সরল processor-টাকে নিয়ে আরও শক্তিশালী করে তোলো। নিচের feature-গুলো একটা একটা করে যোগ করার চেষ্টা করো; আটকে গেলেও চিন্তা নেই, আটকে যাওয়াটাই শেখার অংশ।

### Project: Enhanced 8-bit Processor

**যেসব feature যোগ করতে পারো:**
```
✅ More instructions (16 total)
✅ More registers (8 total)
✅ Immediate values
✅ Conditional branches
✅ Stack operations
✅ Interrupt support (basic)

Test with:
- Fibonacci sequence
- Sorting algorithm
- Calculator program
```

---

## 🏆 Achievement Unlocked!

```
Level 12: ✅ COMPLETE - Processor Architect!
Progress: [██████████████████████████████] 60%

XP Gained: +5000 🎉
Skills: CPU Design, ISA, Architecture

Badges Earned:
🥉 Datapath Designer
🥈 Control Unit Expert
🥇 ISA Architect
🏅 Processor Builder
🎖️ Assembly Programmer
🏆 CPU Architect

PROCESSOR PART STARTED! 🖥️

Next: Chapter 13 - RISC-V Architecture!
      Real-world ISA! 🚀
```

---

**[⬅️ Previous: Chapter 11](Chapter_11_FPGA_Projects.md)** | **[➡️ Next: Chapter 13](Chapter_13_RISCV_Basics.md)**

---

<div align="center">

**"You designed your first processor. Now make it REAL with RISC-V!"**

**"তুমি প্রথম processor design করেছো। এবার RISC-V দিয়ে REAL বানাও!"**

Made with ❤️ for builders | বানানোর জন্য ভালোবাসা দিয়ে তৈরি

</div>
