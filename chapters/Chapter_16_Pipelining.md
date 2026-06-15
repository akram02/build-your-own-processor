# 🚀 Chapter 16: Build Your Own Pipelined Processor
## From Sequential to Parallel - 5× Performance Boost!

> **"Sequential was slow. Parallel is FAST. Time to unleash true performance!"**
>
> **"Sequential ছিল slow। Parallel FAST। এবার true performance unleash করো!"**

---

## 🎯 এই Chapter এ তুমি বানাবে:

```
✅ 5-Stage Pipeline - parallel execution
✅ Pipeline Registers - isolate stages
✅ Hazard Detection - identify problems
✅ Ideal Pipeline - maximum throughput
✅ Performance Analysis - 5× speedup!
✅ Pipeline Diagrams - visual execution
✅ Real Pipelined CPU - working design!
✅ তোমার high-performance processor! 🎉
```

**Time Required:** 2 weeks (6-8 hours/day)  
**Prerequisites:** Chapters 14-15 complete

---

## 🚀 Quick Understanding - Pipeline Magic!

### What is Pipelining?

```
Like an assembly line:

Without Pipeline (Sequential):
Car 1: [Build] → [Paint] → [Test] → Done (3 hours)
Car 2:                              [Build] → [Paint] → [Test] → Done
Total: 6 hours for 2 cars

With Pipeline (Parallel):
Time 0: Car 1 [Build]
Time 1: Car 1 [Paint], Car 2 [Build]
Time 2: Car 1 [Test],  Car 2 [Paint], Car 3 [Build]
Time 3: Car 1 Done,    Car 2 [Test],  Car 3 [Paint], Car 4 [Build]

Throughput: 1 car/hour (vs 1 car/3 hours)
3× speedup!

CPU Pipeline = Same idea!
Execute multiple instructions simultaneously!
```

### CPU Pipeline Stages:

```
5-Stage RISC-V Pipeline:

1. IF (Instruction Fetch)
   - Fetch instruction from memory
   - Update PC

2. ID (Instruction Decode)
   - Decode instruction
   - Read registers
   - Generate immediate

3. EX (Execute)
   - ALU operation
   - Branch decision
   - Address calculation

4. MEM (Memory)
   - Load/Store data
   - Access memory

5. WB (Write Back)
   - Write result to register
```

### Pipeline Visualization:

```
Time →
Cycle: 1    2    3    4    5    6    7    8
Inst1: IF   ID   EX   MEM  WB
Inst2:      IF   ID   EX   MEM  WB
Inst3:           IF   ID   EX   MEM  WB
Inst4:                IF   ID   EX   MEM  WB
Inst5:                     IF   ID   EX   MEM  WB

After 5 cycles:
- 1 instruction completes per cycle!
- 5 instructions in flight simultaneously!
- 5× throughput (ideal case)!
```

🎉 **This is how modern processors achieve speed!**

---

## ১৬.১ Pipeline Benefits

### Performance Gain:

```
Non-Pipelined:
Time per instruction: 5 cycles
100 instructions: 500 cycles

Pipelined (Ideal):
Time per instruction: Still 5 cycles (latency)
But: 1 instruction completes per cycle (throughput)
100 instructions: 5 + 99 = 104 cycles

Speedup: 500/104 ≈ 4.8× !

General formula:
Speedup = (Pipeline depth) × (Efficiency)
Ideal 5-stage: 5×
```

### Why Not Just Increase Clock?

```
Problem with faster clock:
❌ Critical path limits speed
❌ Can't make combinational logic infinitely fast
❌ Power increases dramatically

Pipeline solution:
✅ Break into smaller stages
✅ Each stage faster
✅ Overall throughput increases
✅ More efficient

Balance point:
5-7 stages: Good for embedded/low-power
10-20 stages: Desktop processors
20-31 stages: Intel Pentium 4 (Northwood 20, Prescott 31 — too deep!)

We'll use 5 stages: Classic RISC!
```

---

## ১৬.২ Pipeline Registers

### Between Each Stage:

```
Pipeline registers:
- Isolate stages
- Hold values for next stage
- Enable parallel execution
- Store control signals

IF/ID Register:
┌──────────────┐
│ Instruction  │ (32 bits)
│ PC + 4       │ (32 bits)
└──────────────┘

ID/EX Register:
┌──────────────┐
│ PC + 4       │
│ rs1_data     │
│ rs2_data     │
│ Immediate    │
│ rd_addr      │
│ Control sigs │
└──────────────┘

EX/MEM Register:
┌──────────────┐
│ ALU result   │
│ rs2_data     │ (for store)
│ rd_addr      │
│ Control sigs │
└──────────────┘

MEM/WB Register:
┌──────────────┐
│ ALU result   │
│ Memory data  │
│ rd_addr      │
│ Control sigs │
└──────────────┘
```

### Pipeline Register Implementation:

```verilog
// IF/ID Pipeline Register
module if_id_register(
    input wire clk,
    input wire reset,
    input wire stall,
    input wire flush,
    input wire [31:0] instruction_in,
    input wire [31:0] pc_plus_4_in,
    output reg [31:0] instruction_out,
    output reg [31:0] pc_plus_4_out
);
    always @(posedge clk or posedge reset) begin
        if (reset || flush) begin
            instruction_out <= 32'h00000013;  // NOP
            pc_plus_4_out <= 32'h00000000;
        end else if (!stall) begin
            instruction_out <= instruction_in;
            pc_plus_4_out <= pc_plus_4_in;
        end
    end
endmodule

// ID/EX Pipeline Register
module id_ex_register(
    input wire clk,
    input wire reset,
    input wire flush,
    // Inputs
    input wire [31:0] pc_plus_4_in,
    input wire [31:0] rs1_data_in,
    input wire [31:0] rs2_data_in,
    input wire [31:0] immediate_in,
    input wire [4:0] rd_addr_in,
    input wire [4:0] rs1_addr_in,
    input wire [4:0] rs2_addr_in,
    // Control signals in
    input wire reg_write_in,
    input wire mem_read_in,
    input wire mem_write_in,
    input wire mem_to_reg_in,
    input wire [3:0] alu_control_in,
    input wire alu_src_in,
    input wire branch_in,
    input wire jump_in,
    input wire [2:0] funct3_in,
    // Outputs
    output reg [31:0] pc_plus_4_out,
    output reg [31:0] rs1_data_out,
    output reg [31:0] rs2_data_out,
    output reg [31:0] immediate_out,
    output reg [4:0] rd_addr_out,
    output reg [4:0] rs1_addr_out,
    output reg [4:0] rs2_addr_out,
    // Control signals out
    output reg reg_write_out,
    output reg mem_read_out,
    output reg mem_write_out,
    output reg mem_to_reg_out,
    output reg [3:0] alu_control_out,
    output reg alu_src_out,
    output reg branch_out,
    output reg jump_out,
    output reg [2:0] funct3_out
);
    always @(posedge clk or posedge reset) begin
        if (reset || flush) begin
            pc_plus_4_out <= 32'h00000000;
            rs1_data_out <= 32'h00000000;
            rs2_data_out <= 32'h00000000;
            immediate_out <= 32'h00000000;
            rd_addr_out <= 5'b00000;
            rs1_addr_out <= 5'b00000;
            rs2_addr_out <= 5'b00000;
            reg_write_out <= 0;
            mem_read_out <= 0;
            mem_write_out <= 0;
            mem_to_reg_out <= 0;
            alu_control_out <= 4'b0000;
            alu_src_out <= 0;
            branch_out <= 0;
            jump_out <= 0;
            funct3_out <= 3'b000;
        end else begin
            pc_plus_4_out <= pc_plus_4_in;
            rs1_data_out <= rs1_data_in;
            rs2_data_out <= rs2_data_in;
            immediate_out <= immediate_in;
            rd_addr_out <= rd_addr_in;
            rs1_addr_out <= rs1_addr_in;
            rs2_addr_out <= rs2_addr_in;
            reg_write_out <= reg_write_in;
            mem_read_out <= mem_read_in;
            mem_write_out <= mem_write_in;
            mem_to_reg_out <= mem_to_reg_in;
            alu_control_out <= alu_control_in;
            alu_src_out <= alu_src_in;
            branch_out <= branch_in;
            jump_out <= jump_in;
            funct3_out <= funct3_in;
        end
    end
endmodule

// Similar for EX/MEM and MEM/WB
```

---

## ১৬.৩ Pipelined Datapath

### Complete Pipeline Stages:

```
Stage 1: IF (Instruction Fetch)
┌─────────┐
│   PC    │──→ Instruction Memory ──→ IF/ID
└─────────┘

Stage 2: ID (Instruction Decode)
IF/ID ──→ Control Unit
      ──→ Register File ──→ ID/EX
      ──→ Immediate Gen

Stage 3: EX (Execute)
ID/EX ──→ ALU ──→ EX/MEM
      ──→ Branch Unit

Stage 4: MEM (Memory Access)
EX/MEM ──→ Data Memory ──→ MEM/WB

Stage 5: WB (Write Back)
MEM/WB ──→ Register File
```

### IF Stage (Instruction Fetch):

```verilog
module if_stage(
    input wire clk,
    input wire reset,
    input wire stall,
    input wire branch_taken,
    input wire [31:0] branch_target,
    output reg [31:0] pc,
    output wire [31:0] instruction,
    output wire [31:0] pc_plus_4
);
    wire [31:0] pc_next;
    
    // PC update
    always @(posedge clk or posedge reset) begin
        if (reset)
            pc <= 32'h00000000;
        else if (!stall)
            pc <= pc_next;
    end
    
    // PC source mux
    assign pc_next = branch_taken ? branch_target : (pc + 4);
    assign pc_plus_4 = pc + 4;
    
    // Instruction memory
    instruction_memory imem(
        .address(pc),
        .instruction(instruction)
    );
endmodule
```

### ID Stage (Instruction Decode):

```verilog
module id_stage(
    input wire clk,
    input wire reset,
    input wire [31:0] instruction,
    input wire [31:0] pc_plus_4,
    // From WB stage (write back)
    input wire [4:0] wb_rd_addr,
    input wire [31:0] wb_rd_data,
    input wire wb_reg_write,
    // Outputs
    output wire [31:0] rs1_data,
    output wire [31:0] rs2_data,
    output wire [31:0] immediate,
    output wire [4:0] rd_addr,
    output wire [4:0] rs1_addr,
    output wire [4:0] rs2_addr,
    // Control signals
    output wire reg_write,
    output wire mem_read,
    output wire mem_write,
    output wire mem_to_reg,
    output wire [3:0] alu_control,
    output wire alu_src,
    output wire branch,
    output wire jump
);
    // Extract instruction fields
    wire [6:0] opcode = instruction[6:0];
    assign rd_addr = instruction[11:7];
    wire [2:0] funct3 = instruction[14:12];
    assign rs1_addr = instruction[19:15];
    assign rs2_addr = instruction[24:20];
    wire [6:0] funct7 = instruction[31:25];
    
    // Register File
    register_file rf(
        .clk(clk),
        .reset(reset),
        .rs1_addr(rs1_addr),
        .rs2_addr(rs2_addr),
        .rs1_data(rs1_data),
        .rs2_data(rs2_data),
        .rd_addr(wb_rd_addr),
        .rd_data(wb_rd_data),
        .reg_write(wb_reg_write)
    );
    
    // Immediate Generator
    imm_gen imm_gen_inst(
        .instruction(instruction),
        .immediate(immediate)
    );
    
    // Control Unit
    control_unit ctrl(
        .opcode(opcode),
        .funct3(funct3),
        .funct7(funct7),
        .reg_write(reg_write),
        .mem_read(mem_read),
        .mem_write(mem_write),
        .mem_to_reg(mem_to_reg),
        .alu_control(alu_control),
        .alu_src(alu_src),
        .branch(branch),
        .jump(jump)
    );
endmodule
```

### EX Stage (Execute):

```verilog
module ex_stage(
    input wire [31:0] pc_plus_4,
    input wire [31:0] rs1_data,
    input wire [31:0] rs2_data,
    input wire [31:0] immediate,
    input wire [3:0] alu_control,
    input wire alu_src,
    input wire branch,
    input wire [2:0] funct3,
    output wire [31:0] alu_result,
    output wire branch_taken,
    output wire [31:0] branch_target
);
    wire [31:0] alu_b;
    wire zero;
    
    // ALU source mux
    assign alu_b = alu_src ? immediate : rs2_data;
    
    // ALU
    alu alu_inst(
        .a(rs1_data),
        .b(alu_b),
        .alu_control(alu_control),
        .result(alu_result),
        .zero(zero),
        .negative()
    );
    
    // Branch comparator (drives a separate net, then AND with branch —
    // driving branch_taken from both the port and an assign is illegal)
    wire comp_taken;
    branch_comparator branch_comp(
        .rs1_data(rs1_data),
        .rs2_data(rs2_data),
        .funct3(funct3),
        .branch_taken(comp_taken)
    );
    
    assign branch_taken = branch & comp_taken;
    assign branch_target = pc_plus_4 + immediate;
endmodule
```

### MEM Stage (Memory Access):

```verilog
module mem_stage(
    input wire clk,
    input wire [31:0] alu_result,
    input wire [31:0] rs2_data,
    input wire mem_read,
    input wire mem_write,
    input wire [2:0] funct3,
    output wire [31:0] mem_data
);
    data_memory dmem(
        .clk(clk),
        .address(alu_result),
        .write_data(rs2_data),
        .mem_write(mem_write),
        .mem_read(mem_read),
        .mem_size(funct3),
        .read_data(mem_data)
    );
endmodule
```

### WB Stage (Write Back):

```verilog
module wb_stage(
    input wire [31:0] alu_result,
    input wire [31:0] mem_data,
    input wire [31:0] pc_plus_4,
    input wire mem_to_reg,
    input wire jump,
    output wire [31:0] wb_data
);
    assign wb_data = jump ? pc_plus_4 :
                     mem_to_reg ? mem_data :
                     alu_result;
endmodule
```

---

## ১৬.৪ Complete Pipelined Processor

```verilog
module riscv_pipelined(
    input wire clk,
    input wire reset,
    // Debug
    output wire [31:0] pc_debug
);
    // IF stage signals
    wire [31:0] if_pc, if_instruction, if_pc_plus_4;
    wire if_stall;
    
    // IF/ID pipeline register
    wire [31:0] id_instruction, id_pc_plus_4;
    wire if_id_flush;
    
    // ID stage signals
    wire [31:0] id_rs1_data, id_rs2_data, id_immediate;
    wire [4:0] id_rd_addr, id_rs1_addr, id_rs2_addr;
    wire id_reg_write, id_mem_read, id_mem_write;
    wire id_mem_to_reg, id_alu_src, id_branch, id_jump;
    wire [3:0] id_alu_control;
    
    // ID/EX pipeline register
    wire [31:0] ex_pc_plus_4, ex_rs1_data, ex_rs2_data, ex_immediate;
    wire [4:0] ex_rd_addr, ex_rs1_addr, ex_rs2_addr;
    wire ex_reg_write, ex_mem_read, ex_mem_write;
    wire ex_mem_to_reg, ex_alu_src, ex_branch, ex_jump;
    wire [3:0] ex_alu_control;
    wire [2:0] ex_funct3;
    wire id_ex_flush;
    
    // EX stage signals
    wire [31:0] ex_alu_result, ex_branch_target;
    wire ex_branch_taken;
    
    // EX/MEM pipeline register
    wire [31:0] mem_alu_result, mem_rs2_data, mem_pc_plus_4;
    wire [4:0] mem_rd_addr;
    wire mem_reg_write, mem_mem_read, mem_mem_write;
    wire mem_mem_to_reg, mem_jump;
    wire [2:0] mem_funct3;
    
    // MEM stage signals
    wire [31:0] mem_data;
    
    // MEM/WB pipeline register
    wire [31:0] wb_alu_result, wb_mem_data, wb_pc_plus_4;
    wire [4:0] wb_rd_addr;
    wire wb_reg_write, wb_mem_to_reg, wb_jump;
    
    // WB stage signals
    wire [31:0] wb_data;
    
    // IF Stage
    if_stage if_stage_inst(
        .clk(clk),
        .reset(reset),
        .stall(if_stall),
        .branch_taken(ex_branch_taken),
        .branch_target(ex_branch_target),
        .pc(if_pc),
        .instruction(if_instruction),
        .pc_plus_4(if_pc_plus_4)
    );
    
    // IF/ID Pipeline Register
    if_id_register if_id_reg(
        .clk(clk),
        .reset(reset),
        .stall(if_stall),
        .flush(if_id_flush),
        .instruction_in(if_instruction),
        .pc_plus_4_in(if_pc_plus_4),
        .instruction_out(id_instruction),
        .pc_plus_4_out(id_pc_plus_4)
    );
    
    // ID Stage
    id_stage id_stage_inst(
        .clk(clk),
        .reset(reset),
        .instruction(id_instruction),
        .pc_plus_4(id_pc_plus_4),
        .wb_rd_addr(wb_rd_addr),
        .wb_rd_data(wb_data),
        .wb_reg_write(wb_reg_write),
        .rs1_data(id_rs1_data),
        .rs2_data(id_rs2_data),
        .immediate(id_immediate),
        .rd_addr(id_rd_addr),
        .rs1_addr(id_rs1_addr),
        .rs2_addr(id_rs2_addr),
        .reg_write(id_reg_write),
        .mem_read(id_mem_read),
        .mem_write(id_mem_write),
        .mem_to_reg(id_mem_to_reg),
        .alu_control(id_alu_control),
        .alu_src(id_alu_src),
        .branch(id_branch),
        .jump(id_jump)
    );
    
    // ID/EX Pipeline Register
    id_ex_register id_ex_reg(
        .clk(clk),
        .reset(reset),
        .flush(id_ex_flush),
        .pc_plus_4_in(id_pc_plus_4),
        .rs1_data_in(id_rs1_data),
        .rs2_data_in(id_rs2_data),
        .immediate_in(id_immediate),
        .rd_addr_in(id_rd_addr),
        .rs1_addr_in(id_rs1_addr),
        .rs2_addr_in(id_rs2_addr),
        .reg_write_in(id_reg_write),
        .mem_read_in(id_mem_read),
        .mem_write_in(id_mem_write),
        .mem_to_reg_in(id_mem_to_reg),
        .alu_control_in(id_alu_control),
        .alu_src_in(id_alu_src),
        .branch_in(id_branch),
        .jump_in(id_jump),
        .funct3_in(id_instruction[14:12]),
        .pc_plus_4_out(ex_pc_plus_4),
        .rs1_data_out(ex_rs1_data),
        .rs2_data_out(ex_rs2_data),
        .immediate_out(ex_immediate),
        .rd_addr_out(ex_rd_addr),
        .rs1_addr_out(ex_rs1_addr),
        .rs2_addr_out(ex_rs2_addr),
        .reg_write_out(ex_reg_write),
        .mem_read_out(ex_mem_read),
        .mem_write_out(ex_mem_write),
        .mem_to_reg_out(ex_mem_to_reg),
        .alu_control_out(ex_alu_control),
        .alu_src_out(ex_alu_src),
        .branch_out(ex_branch),
        .jump_out(ex_jump),
        .funct3_out(ex_funct3)
    );
    
    // EX Stage
    ex_stage ex_stage_inst(
        .pc_plus_4(ex_pc_plus_4),
        .rs1_data(ex_rs1_data),
        .rs2_data(ex_rs2_data),
        .immediate(ex_immediate),
        .alu_control(ex_alu_control),
        .alu_src(ex_alu_src),
        .branch(ex_branch),
        .funct3(ex_funct3),
        .alu_result(ex_alu_result),
        .branch_taken(ex_branch_taken),
        .branch_target(ex_branch_target)
    );
    
    // EX/MEM Pipeline Register
    ex_mem_register ex_mem_reg(
        .clk(clk),
        .reset(reset),
        .alu_result_in(ex_alu_result),
        .rs2_data_in(ex_rs2_data),
        .pc_plus_4_in(ex_pc_plus_4),
        .rd_addr_in(ex_rd_addr),
        .reg_write_in(ex_reg_write),
        .mem_read_in(ex_mem_read),
        .mem_write_in(ex_mem_write),
        .mem_to_reg_in(ex_mem_to_reg),
        .jump_in(ex_jump),
        .funct3_in(ex_funct3),
        .alu_result_out(mem_alu_result),
        .rs2_data_out(mem_rs2_data),
        .pc_plus_4_out(mem_pc_plus_4),
        .rd_addr_out(mem_rd_addr),
        .reg_write_out(mem_reg_write),
        .mem_read_out(mem_mem_read),
        .mem_write_out(mem_mem_write),
        .mem_to_reg_out(mem_mem_to_reg),
        .jump_out(mem_jump),
        .funct3_out(mem_funct3)
    );
    
    // MEM Stage
    mem_stage mem_stage_inst(
        .clk(clk),
        .alu_result(mem_alu_result),
        .rs2_data(mem_rs2_data),
        .mem_read(mem_mem_read),
        .mem_write(mem_mem_write),
        .funct3(mem_funct3),
        .mem_data(mem_data)
    );
    
    // MEM/WB Pipeline Register
    mem_wb_register mem_wb_reg(
        .clk(clk),
        .reset(reset),
        .alu_result_in(mem_alu_result),
        .mem_data_in(mem_data),
        .pc_plus_4_in(mem_pc_plus_4),
        .rd_addr_in(mem_rd_addr),
        .reg_write_in(mem_reg_write),
        .mem_to_reg_in(mem_mem_to_reg),
        .jump_in(mem_jump),
        .alu_result_out(wb_alu_result),
        .mem_data_out(wb_mem_data),
        .pc_plus_4_out(wb_pc_plus_4),
        .rd_addr_out(wb_rd_addr),
        .reg_write_out(wb_reg_write),
        .mem_to_reg_out(wb_mem_to_reg),
        .jump_out(wb_jump)
    );
    
    // WB Stage
    wb_stage wb_stage_inst(
        .alu_result(wb_alu_result),
        .mem_data(wb_mem_data),
        .pc_plus_4(wb_pc_plus_4),
        .mem_to_reg(wb_mem_to_reg),
        .jump(wb_jump),
        .wb_data(wb_data)
    );
    
    // Control hazard detection (for flush signals)
    assign if_id_flush = ex_branch_taken;
    assign id_ex_flush = ex_branch_taken;
    
    // Debug
    assign pc_debug = if_pc;
endmodule
```

---

## ১৬.৫ Pipeline Performance

### Ideal Case:

```
100 instructions:

Non-pipelined: 100 × 5 = 500 cycles

Pipelined:
Fill time: 5 cycles (first instruction)
Steady state: 99 cycles (one per cycle)
Total: 5 + 99 = 104 cycles

Speedup: 500 / 104 = 4.8×!

CPI: 104 / 100 = 1.04
(Almost 1!)
```

### Real Performance:

```
With hazards (Chapter 17):
- Data hazards: +stalls
- Control hazards: +flushes
- Structural hazards: rare

Real CPI: 1.2 - 1.5

Still 3-4× better than non-pipelined!
```

---

## ১৬.৬ Pipeline Diagrams

### Space-Time Diagram:

```
Time (cycles) →
0   1   2   3   4   5   6   7   8

ADD  IF  ID  EX  MEM WB
SUB      IF  ID  EX  MEM WB
AND          IF  ID  EX  MEM WB
OR               IF  ID  EX  MEM WB
XOR                  IF  ID  EX  MEM WB

Stage usage:
IF:  ADD SUB AND OR  XOR
ID:      ADD SUB AND OR  XOR
EX:          ADD SUB AND OR  XOR
MEM:             ADD SUB AND OR
WB:                  ADD SUB AND

All stages busy!
Maximum utilization!
```

---

## ১৬.৭ Your 2-Week Build Plan

### Week 1: Pipeline Structure

**Day 1-2: Pipeline Registers**
```
□ IF/ID register
□ ID/EX register
□ EX/MEM register
□ MEM/WB register
```

**Day 3-4: Stage Modules**
```
□ IF stage
□ ID stage
□ EX stage
□ MEM stage
□ WB stage
```

**Day 5-7: Integration**
```
□ Connect all stages
□ Wire pipeline registers
□ Control signal propagation
□ Initial testing
```

### Week 2: Testing & Analysis

**Day 8-10: Testing**
```
□ Run simple programs
□ Analyze pipeline behavior
□ Visualize execution
□ Verify correctness
```

**Day 11-12: Performance**
```
□ Measure CPI
□ Calculate speedup
□ Compare with single-cycle
□ Optimization
```

**Day 13-14: Documentation**
```
□ Pipeline diagrams
□ Performance reports
□ Final testing
□ Prepare for hazards
```

---

## ১৬.৮ Chapter 16 Mission Complete!

### তুমি এখন পারো:

```
✅ Design pipelined processors
✅ Implement pipeline registers
✅ Create 5-stage pipeline
✅ Analyze performance
✅ Calculate speedup
✅ Draw pipeline diagrams
✅ Understand throughput
✅ Build high-performance CPUs! 🎉
```

### তুমি বানিয়েছো:
```
✅ 5-stage pipelined RISC-V
✅ 4 pipeline registers
✅ Parallel execution engine
✅ Near 5× speedup
✅ Modern processor design
✅ High-performance CPU! 🚀
```

### Stats:
```
Pipeline stages: 5
Throughput: 1 inst/cycle (ideal)
Speedup: 4.8× (ideal)
Parallelism: 5 instructions
CPI: 1.04 (ideal)
Level: High-Performance Architect! 🏆
```

### Next Level Unlocked:
```
→ Chapter 17: Hazards & Forwarding
   তুমি শিখবে: Handle dependencies
   Data forwarding, stalling, flushing!
   
   From ideal → Real pipeline!
```

---

## 🎯 Final Project

### Project: Performance Analysis

**Compare all three:**
```
✅ Single-cycle CPU
✅ Multi-cycle CPU
✅ Pipelined CPU

Metrics:
- Clock period
- CPI
- Total execution time
- Hardware cost
- Power consumption (estimate)

Programs:
- Matrix multiply
- Fibonacci
- Sorting
- Real benchmark!

Create comprehensive report!
```

---

## 🏆 Achievement Unlocked!

```
Level 16: ✅ COMPLETE - Performance Engineer!
Progress: [███████████████████████████████████] 80%

XP Gained: +5000 🎉
Skills: Pipelining, Parallel Execution, Performance

Badges Earned:
🥉 Pipeline Designer
🥈 Parallel Execution Master
🥇 Throughput Optimizer
🏅 Performance Analyst
🎖️ Modern CPU Architect
🏆 High-Performance Expert
⭐ 5× SPEEDUP ACHIEVED! ⭐

YOU BUILT A FAST PROCESSOR! 🚀⚡

Next: Chapter 17 - Real Pipeline!
      Handle hazards! Fix problems! 🛠️
```

---

**[⬅️ Previous: Chapter 15](Chapter_15_Multi_Cycle_CPU.md)** | **[➡️ Next: Chapter 17](Chapter_17_Hazards_Forwarding.md)**

---

<div align="center">

**"You pipelined your CPU! 5× faster! Now handle the HAZARDS!"**

**"তুমি pipeline করেছো! 5× দ্রুত! এবার HAZARDS handle করো!"**

Made with ❤️ for builders | বানানোর জন্য ভালোবাসা দিয়ে তৈরি

</div>
