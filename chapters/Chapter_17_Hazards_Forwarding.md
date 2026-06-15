# 🛠️ Chapter 17: Pipeline Hazards & Forwarding
## From Ideal to Real - Handle Dependencies & Make It Work!

> **"Ideal pipeline was fast. But broken. Time to FIX it and make it REAL!"**
>
> **"Ideal pipeline ছিল fast। কিন্তু broken। এবার FIX করো আর REAL বানাও!"**

---

## 🎯 এই Chapter এ তুমি বানাবে:

```
✅ Data Hazard Detection - find dependencies
✅ Forwarding Unit - bypass data
✅ Stall Logic - wait when needed
✅ Control Hazard Handling - branches
✅ Branch Prediction - reduce penalty
✅ Hazard Resolution - complete solution
✅ Real Pipeline - working correctly!
✅ তোমার production-ready CPU! 🎉
```

**Time Required:** 2 weeks (7-8 hours/day)  
**Prerequisites:** Chapter 16 complete

---

## 🚀 Quick Understanding - Pipeline Problems!

### What Are Hazards?

```
Hazard = Problem that prevents next instruction
         from executing in next cycle

Three types:
1. Structural Hazards
   - Hardware resource conflict
   - Two instructions need same hardware
   - Rare in RISC-V (separate memories)

2. Data Hazards
   - Instruction needs result not yet ready
   - Read-After-Write (RAW) dependency
   - MOST COMMON!

3. Control Hazards
   - Don't know next instruction yet
   - Branches, jumps
   - CAUSES DELAYS!

Must handle ALL to make pipeline work!
```

### Example Data Hazard:

```assembly
ADD x1, x2, x3    # x1 = x2 + x3
SUB x4, x1, x5    # x4 = x1 - x5  ← needs x1!

Pipeline (without fix):
Cycle: 1   2   3   4   5   6
ADD:   IF  ID  EX  MEM WB
SUB:       IF  ID  EX  MEM WB
              ↑
           Reads x1 here (cycle 3)
           But ADD writes x1 in cycle 5!
           WRONG VALUE! 💥

Solution: FORWARDING!
Take value from EX/MEM register
Forward to SUB in cycle 3
CORRECT VALUE! ✅
```

🎉 **This chapter = Making pipeline actually work!**

---

## ১৭.১ Data Hazards - The Main Problem

### Types of Data Hazards:

```
1. EX Hazard (ALU-to-ALU)
   - Previous instruction in EX/MEM
   - Current instruction in EX
   - Forward from EX/MEM

2. MEM Hazard (ALU result, 2 instructions back)
   - Previous instruction in MEM/WB
   - Current instruction in EX
   - Forward from MEM/WB  (NOT a load-use case; see case 3)

3. Load-Use Hazard
   - Previous is LOAD (data not ready)
   - Current needs that data
   - MUST STALL! Cannot forward!
```

### Hazard Examples:

```assembly
# Example 1: EX Hazard
ADD x1, x2, x3    # x1 = x2 + x3
SUB x4, x1, x5    # Uses x1 (EX hazard)
AND x6, x7, x8    # No hazard

# Example 2: MEM Hazard
ADD x1, x2, x3    # x1 = x2 + x3
NOP               # (bubble)
SUB x4, x1, x5    # Uses x1 (MEM hazard)

# Example 3: Load-Use Hazard
LW  x1, 0(x2)     # Load x1 from memory
ADD x4, x1, x5    # Uses x1 IMMEDIATELY
                  # Must stall! Data not ready yet!
```

### Data Hazard Detection:

```verilog
module hazard_detection_unit(
    // ID stage (consumer — used for load-use stall detection)
    input wire [4:0] id_rs1,
    input wire [4:0] id_rs2,
    // EX stage: sources of the instruction in EX (used for forwarding),
    //           plus its rd / mem_read (to detect a load feeding the next instr)
    input wire [4:0] ex_rs1,
    input wire [4:0] ex_rs2,
    input wire [4:0] ex_rd,
    input wire ex_reg_write,
    input wire ex_mem_read,
    // MEM stage = EX/MEM register output (forward source 2'b10)
    input wire [4:0] mem_rd,
    input wire mem_reg_write,
    // WB stage = MEM/WB register output (forward source 2'b01)
    input wire [4:0] wb_rd,
    input wire wb_reg_write,
    // Outputs
    output wire stall,
    output wire [1:0] forward_a,
    output wire [1:0] forward_b
);
    // Load-Use Hazard: a load is in EX and the next instruction (in ID) needs
    // its result. Forwarding can't help (data arrives a cycle too late) -> stall.
    wire load_use_hazard = ex_mem_read &&
                          ((ex_rd == id_rs1) || (ex_rd == id_rs2)) &&
                          (ex_rd != 0);

    assign stall = load_use_hazard;

    // Forwarding feeds the instruction in EX. Compare ITS source registers
    // (ex_rs1/ex_rs2) against the destinations sitting in EX/MEM (mem_rd -> 2'b10)
    // and MEM/WB (wb_rd -> 2'b01). EX/MEM has priority (most recent value).
    assign forward_a =
        (mem_reg_write && (mem_rd != 0) && (mem_rd == ex_rs1)) ? 2'b10 :
        (wb_reg_write  && (wb_rd  != 0) && (wb_rd  == ex_rs1)) ? 2'b01 :
        2'b00;

    assign forward_b =
        (mem_reg_write && (mem_rd != 0) && (mem_rd == ex_rs2)) ? 2'b10 :
        (wb_reg_write  && (wb_rd  != 0) && (wb_rd  == ex_rs2)) ? 2'b01 :
        2'b00;
endmodule
```

---

## ১৭.২ Forwarding Unit

### Forwarding Logic:

```
Forward sources:
- 2'b00: No forwarding (use register file)
- 2'b01: Forward from MEM/WB (MEM stage)
- 2'b10: Forward from EX/MEM (EX stage)

Priority:
EX hazard > MEM hazard
(Most recent value wins!)
```

### Forwarding Multiplexers:

```verilog
module forwarding_mux(
    input wire [31:0] register_data,
    input wire [31:0] ex_mem_data,
    input wire [31:0] mem_wb_data,
    input wire [1:0] forward_select,
    output reg [31:0] forwarded_data
);
    always @(*) begin
        case (forward_select)
            2'b00: forwarded_data = register_data;  // From register file
            2'b01: forwarded_data = mem_wb_data;    // From MEM/WB
            2'b10: forwarded_data = ex_mem_data;    // From EX/MEM
            default: forwarded_data = register_data;
        endcase
    end
endmodule
```

### Updated EX Stage with Forwarding:

```verilog
module ex_stage_with_forwarding(
    input wire [31:0] pc_plus_4,
    input wire [31:0] rs1_data,
    input wire [31:0] rs2_data,
    input wire [31:0] immediate,
    // Forwarding inputs
    input wire [31:0] ex_mem_alu_result,
    input wire [31:0] mem_wb_write_data,
    input wire [1:0] forward_a,
    input wire [1:0] forward_b,
    // Control
    input wire [3:0] alu_control,
    input wire alu_src,
    input wire branch,
    input wire [2:0] funct3,
    // Outputs
    output wire [31:0] alu_result,
    output wire branch_taken,
    output wire [31:0] branch_target,
    output wire [31:0] store_data   // forwarded rs2 — the value a store writes
);
    wire [31:0] alu_a, alu_b;
    wire [31:0] forwarded_rs1, forwarded_rs2;
    wire zero;
    
    // Forwarding mux for rs1
    forwarding_mux fwd_a(
        .register_data(rs1_data),
        .ex_mem_data(ex_mem_alu_result),
        .mem_wb_data(mem_wb_write_data),
        .forward_select(forward_a),
        .forwarded_data(forwarded_rs1)
    );
    
    // Forwarding mux for rs2
    forwarding_mux fwd_b(
        .register_data(rs2_data),
        .ex_mem_data(ex_mem_alu_result),
        .mem_wb_data(mem_wb_write_data),
        .forward_select(forward_b),
        .forwarded_data(forwarded_rs2)
    );
    
    // ALU inputs
    assign alu_a = forwarded_rs1;
    assign alu_b = alu_src ? immediate : forwarded_rs2;
    
    // ALU
    alu alu_inst(
        .a(alu_a),
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
        .rs1_data(forwarded_rs1),
        .rs2_data(forwarded_rs2),
        .funct3(funct3),
        .branch_taken(comp_taken)
    );
    
    assign branch_taken = branch & comp_taken;
    // Target is relative to the branch's OWN PC; we carry pc_plus_4 (= branchPC+4)
    assign branch_target = (pc_plus_4 - 32'd4) + immediate;
    assign store_data = forwarded_rs2;   // stores must use the forwarded value
endmodule
```

---

## ১৭.৩ Pipeline Stalling

### Stall Logic:

```
When to stall:
- Load-Use hazard detected
- Data not available for forwarding

Stall effect:
- Keep IF/ID register unchanged
- Prevent PC update
- Insert NOP (bubble) into ID/EX

Duration:
- 1 cycle stall for load-use
- Then can forward
```

### Stall Implementation:

```verilog
module stall_controller(
    input wire stall_request,
    output reg pc_write,
    output reg if_id_write
);
    // On a load-use stall: freeze PC and IF/ID. The ID/EX bubble is inserted
    // by the ID/EX register's own flush input (id_ex_flush || stall), so this
    // controller must NOT also drive id_ex_flush (that was a double-driver).
    always @(*) begin
        if (stall_request) begin
            pc_write = 0;       // Don't update PC
            if_id_write = 0;    // Keep IF/ID same
        end else begin
            pc_write = 1;       // Normal operation
            if_id_write = 1;
        end
    end
endmodule
```

### Load-Use Example with Stall:

```assembly
LW  x1, 0(x2)     # Load x1
ADD x4, x1, x3    # Use x1

Pipeline with stall:
Cycle: 1   2   3   4   5   6   7
LW:    IF  ID  EX  MEM WB
ADD:       IF  ID  stall EX  MEM WB
                  ↑
              Insert bubble
              Wait for LW to reach MEM
              Then forward from MEM/WB

Total: 1 cycle penalty
```

---

## ১৭.৪ Control Hazards

### Branch Problem:

```
Branch decision made in EX stage (cycle 3)
But we fetch next instruction in cycle 2!

Options:
1. Always stall (2 cycle penalty) ❌
2. Predict not-taken (flush if wrong) ✅
3. Predict taken (complex)
4. Branch prediction (advanced)

We use: Predict not-taken
```

### Branch Handling:

```
Predict Not-Taken:
1. Always fetch PC+4
2. If branch NOT taken: Continue
3. If branch taken: Flush & jump

Branch taken (2 cycle penalty):
Cycle: 1   2   3   4   5   6
BEQ:   IF  ID  EX  MEM WB
+4:        IF  ID  XX  (flush)
+8:            IF  XX  (flush)
Target:            IF  ID  EX...

XX = flushed instructions (wasted work)
```

### Branch Flush Logic:

```verilog
module branch_controller(
    input wire branch_taken,
    output reg if_id_flush,
    output reg id_ex_flush
);
    // Branch resolves in EX, so only the two younger instructions (in IF/ID
    // and ID/EX) are on the wrong path. The instruction in EX/MEM is OLDER
    // than the branch and must complete — do NOT flush EX/MEM.
    always @(*) begin
        if (branch_taken) begin
            if_id_flush = 1;   // Flush IF/ID (branch+8, wrong path)
            id_ex_flush = 1;   // Flush ID/EX (branch+4, wrong path)
        end else begin
            if_id_flush = 0;
            id_ex_flush = 0;
        end
    end
endmodule
```

---

## ১৭.৫ Complete Pipeline with Hazard Handling

```verilog
module riscv_pipelined_with_hazards(
    input wire clk,
    input wire reset,
    // Debug
    output wire [31:0] pc_debug,
    output wire [31:0] cycles_debug,
    output wire [31:0] stalls_debug
);
    // Performance counters
    reg [31:0] cycle_count;
    reg [31:0] stall_count;
    
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            cycle_count <= 0;
            stall_count <= 0;
        end else begin
            cycle_count <= cycle_count + 1;
            if (stall)
                stall_count <= stall_count + 1;
        end
    end
    
    // IF stage signals
    wire [31:0] if_pc, if_instruction, if_pc_plus_4;
    wire pc_write, if_id_write;
    
    // IF/ID pipeline register
    wire [31:0] id_instruction, id_pc_plus_4;
    wire if_id_flush;
    
    // ID stage signals
    wire [31:0] id_rs1_data, id_rs2_data, id_immediate;
    wire [4:0] id_rd_addr, id_rs1_addr, id_rs2_addr;
    wire id_reg_write, id_mem_read, id_mem_write;
    wire id_mem_to_reg, id_alu_src, id_branch, id_jump;
    wire [3:0] id_alu_control;
    wire [2:0] id_funct3;
    
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
    wire [31:0] ex_store_data;   // forwarded rs2 for stores
    wire ex_branch_taken;
    wire [1:0] forward_a, forward_b;
    
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
    
    // Hazard detection
    wire stall;
    
    hazard_detection_unit hazard_unit(
        .id_rs1(id_rs1_addr),
        .id_rs2(id_rs2_addr),
        .ex_rs1(ex_rs1_addr),
        .ex_rs2(ex_rs2_addr),
        .ex_rd(ex_rd_addr),
        .ex_reg_write(ex_reg_write),
        .ex_mem_read(ex_mem_read),
        .mem_rd(mem_rd_addr),
        .mem_reg_write(mem_reg_write),
        .wb_rd(wb_rd_addr),
        .wb_reg_write(wb_reg_write),
        .stall(stall),
        .forward_a(forward_a),
        .forward_b(forward_b)
    );
    
    // Stall controller
    stall_controller stall_ctrl(
        .stall_request(stall),
        .pc_write(pc_write),
        .if_id_write(if_id_write)
    );
    
    // Branch controller
    branch_controller branch_ctrl(
        .branch_taken(ex_branch_taken),
        .if_id_flush(if_id_flush),
        .id_ex_flush(id_ex_flush)   // branch resolves in EX → flush IF/ID + ID/EX only
    );
    
    // IF Stage
    if_stage if_stage_inst(
        .clk(clk),
        .reset(reset),
        .stall(!pc_write),
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
        .stall(!if_id_write),
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
    
    assign id_funct3 = id_instruction[14:12];
    
    // ID/EX Pipeline Register
    id_ex_register id_ex_reg(
        .clk(clk),
        .reset(reset),
        .flush(id_ex_flush || stall),
        .pc_plus_4_in(id_pc_plus_4),
        .rs1_data_in(id_rs1_data),
        .rs2_data_in(id_rs2_data),
        .immediate_in(id_immediate),
        .rd_addr_in(id_rd_addr),
        .rs1_addr_in(id_rs1_addr),
        .rs2_addr_in(id_rs2_addr),
        .funct3_in(id_funct3),
        .reg_write_in(id_reg_write),
        .mem_read_in(id_mem_read),
        .mem_write_in(id_mem_write),
        .mem_to_reg_in(id_mem_to_reg),
        .alu_control_in(id_alu_control),
        .alu_src_in(id_alu_src),
        .branch_in(id_branch),
        .jump_in(id_jump),
        .pc_plus_4_out(ex_pc_plus_4),
        .rs1_data_out(ex_rs1_data),
        .rs2_data_out(ex_rs2_data),
        .immediate_out(ex_immediate),
        .rd_addr_out(ex_rd_addr),
        .rs1_addr_out(ex_rs1_addr),
        .rs2_addr_out(ex_rs2_addr),
        .funct3_out(ex_funct3),
        .reg_write_out(ex_reg_write),
        .mem_read_out(ex_mem_read),
        .mem_write_out(ex_mem_write),
        .mem_to_reg_out(ex_mem_to_reg),
        .alu_control_out(ex_alu_control),
        .alu_src_out(ex_alu_src),
        .branch_out(ex_branch),
        .jump_out(ex_jump)
    );
    
    // EX Stage with Forwarding
    ex_stage_with_forwarding ex_stage_inst(
        .pc_plus_4(ex_pc_plus_4),
        .rs1_data(ex_rs1_data),
        .rs2_data(ex_rs2_data),
        .immediate(ex_immediate),
        .ex_mem_alu_result(mem_alu_result),
        .mem_wb_write_data(wb_data),
        .forward_a(forward_a),
        .forward_b(forward_b),
        .alu_control(ex_alu_control),
        .alu_src(ex_alu_src),
        .branch(ex_branch),
        .funct3(ex_funct3),
        .alu_result(ex_alu_result),
        .branch_taken(ex_branch_taken),
        .branch_target(ex_branch_target),
        .store_data(ex_store_data)
    );
    
    // EX/MEM Pipeline Register
    ex_mem_register ex_mem_reg(
        .clk(clk),
        .reset(reset),
        .alu_result_in(ex_alu_result),
        .rs2_data_in(ex_store_data),
        .pc_plus_4_in(ex_pc_plus_4),
        .rd_addr_in(ex_rd_addr),
        .funct3_in(ex_funct3),
        .reg_write_in(ex_reg_write),
        .mem_read_in(ex_mem_read),
        .mem_write_in(ex_mem_write),
        .mem_to_reg_in(ex_mem_to_reg),
        .jump_in(ex_jump),
        .alu_result_out(mem_alu_result),
        .rs2_data_out(mem_rs2_data),
        .pc_plus_4_out(mem_pc_plus_4),
        .rd_addr_out(mem_rd_addr),
        .funct3_out(mem_funct3),
        .reg_write_out(mem_reg_write),
        .mem_read_out(mem_mem_read),
        .mem_write_out(mem_mem_write),
        .mem_to_reg_out(mem_mem_to_reg),
        .jump_out(mem_jump)
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
    
    // Debug outputs
    assign pc_debug = if_pc;
    assign cycles_debug = cycle_count;
    assign stalls_debug = stall_count;
endmodule
```

---

## ১৭.৬ Performance Analysis

### Real Pipeline Performance:

```
CPI calculation:
Base: 1.0 (ideal)
Data hazards: +0.1-0.3 (forwarding reduces!)
Load-use stalls: +0.2-0.5
Branch mispredictions: +0.5-1.0

Typical: CPI = 1.3-1.5

100 instructions:
Cycles: 5 + 99 + stalls
≈ 104 + 30 = 134 cycles

Still 3.7× faster than non-pipelined!
(500 / 134 = 3.7)

Real speedup: 3.5-4.0×
```

### Optimization Strategies:

```
1. Better Forwarding
   - Forward from more stages
   - Reduce stalls

2. Branch Prediction
   - Predict taken/not-taken
   - Branch target buffer
   - Reduce flush penalty

3. Code Scheduling
   - Compiler reorders instructions
   - Fill delay slots
   - Minimize hazards

4. Hardware Scheduling
   - Out-of-order execution
   - Superscalar (multiple issue)
   - Advanced topics!
```

---

## ১৭.৭ Advanced: Branch Prediction

### Simple 1-bit Predictor:

```verilog
module branch_predictor(
    input wire clk,
    input wire reset,
    input wire [31:0] pc,
    input wire branch_taken_actual,
    input wire update,
    output reg prediction
);
    // Simple prediction table (64 entries)
    reg [0:63] prediction_table;
    wire [5:0] index = pc[7:2];  // Use bits [7:2] as index
    
    // Initialize to not-taken
    integer i;
    initial begin
        for (i = 0; i < 64; i = i + 1)
            prediction_table[i] = 0;
    end
    
    // Prediction
    always @(*) begin
        prediction = prediction_table[index];
    end
    
    // Update on branch resolution
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            for (i = 0; i < 64; i = i + 1)
                prediction_table[i] <= 0;
        end else if (update) begin
            prediction_table[index] <= branch_taken_actual;
        end
    end
endmodule
```

---

## ১৭.৮ Testing & Debugging

### Comprehensive Testbench:

```verilog
module pipeline_hazards_tb;
    reg clk, reset;
    wire [31:0] pc_debug, cycles, stalls;
    
    // DUT
    riscv_pipelined_with_hazards dut(
        .clk(clk),
        .reset(reset),
        .pc_debug(pc_debug),
        .cycles_debug(cycles),
        .stalls_debug(stalls)
    );
    
    // Clock
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end
    
    // Test program with hazards
    initial begin
        $dumpfile("pipeline.vcd");
        $dumpvars(0, pipeline_hazards_tb);
        
        // Reset
        reset = 1;
        #20;
        reset = 0;
        
        // Run
        #2000;
        
        // Results
        $display("========================================");
        $display("Pipeline Performance Analysis");
        $display("========================================");
        $display("Total Cycles: %d", cycles);
        $display("Stall Cycles: %d", stalls);
        $display("CPI: %.2f", cycles / 100.0);
        $display("Stall Rate: %.1f%%", (stalls * 100.0) / cycles);
        $display("========================================");
        
        $finish;
    end
    
    // Monitor
    always @(posedge clk) begin
        if (!reset && pc_debug != 0) begin
            $display("Cycle %d: PC=%h, Stalls=%d", 
                    cycles, pc_debug, stalls);
        end
    end
endmodule
```

---

## ১৭.৯ Your 2-Week Build Plan

### Week 1: Hazard Detection

**Day 1-2: Data Hazard Detection**
```
□ Hazard detection unit
□ Compare register addresses
□ Generate forwarding signals
□ Test detection
```

**Day 3-4: Forwarding Unit**
```
□ Forwarding multiplexers
□ EX/MEM forwarding
□ MEM/WB forwarding
□ Test forwarding
```

**Day 5-7: Stalling Logic**
```
□ Stall controller
□ PC freeze logic
□ Bubble insertion
□ Load-use handling
```

### Week 2: Control Hazards & Integration

**Day 8-10: Branch Handling**
```
□ Branch flush logic
□ Predict not-taken
□ Flush mechanism
□ Test branches
```

**Day 11-12: Complete Integration**
```
□ All hazard logic
□ Full pipeline
□ Comprehensive testing
□ Debug issues
```

**Day 13-14: Optimization & Analysis**
```
□ Performance measurement
□ Branch prediction (optional)
□ Code optimization
□ Final testing
```

---

## ১৭.১০ Chapter 17 Mission Complete!

### তুমি এখন পারো:

```
✅ Detect data hazards
✅ Implement forwarding
✅ Handle stalls
✅ Resolve control hazards
✅ Analyze pipeline performance
✅ Optimize instruction sequences
✅ Build real pipelined processors
✅ Production-ready CPU design! 🎉
```

### তুমি বানিয়েছো:
```
✅ Complete hazard detection
✅ Forwarding unit
✅ Stall controller
✅ Branch handler
✅ Real working pipeline
✅ 3.5-4× speedup (real!)
✅ Production CPU! 💻
```

### Stats:
```
Hazard types handled: 3
Forwarding paths: 2
Stall cases: 1
Branch penalty: 2 cycles
Real CPI: 1.3-1.5
Real speedup: 3.5-4.0×
Level: Pipeline Expert! 🏆
```

### Next Level Unlocked:
```
→ Chapter 18: Memory Hierarchy
   তুমি শিখবে: Cache design
   10× faster memory access!
   
   From slow → FAST memory!
```

---

## 🎯 Final Project

### Project: Pipeline Optimizer

**Build & Optimize:**
```
✅ Complete pipeline with all hazards
✅ Performance profiling
✅ Branch predictor (1-bit or 2-bit)
✅ Code scheduler (software)
✅ Comparison analysis

Benchmarks:
- Fibonacci
- Matrix multiply
- Quicksort
- Real programs!

Report:
- CPI achieved
- Stall analysis
- Branch prediction accuracy
- Optimization impact
```

---

## 🏆 Achievement Unlocked!

```
Level 17: ✅ COMPLETE - Pipeline Master!
Progress: [████████████████████████████████████] 85%

XP Gained: +4000
Skills: Hazard Handling, Forwarding, Real Pipeline

Badges Earned:
🥉 Hazard Detective
🥈 Forwarding Master
🥇 Stall Handler
🏅 Branch Specialist
🎖️ Real Pipeline Designer
🏆 Production CPU Architect
⭐ WORKING PIPELINE! ⭐

YOUR CPU ACTUALLY WORKS NOW! 💻✅

Next: Chapter 18 - Memory Hierarchy!
      Cache design! 10× speedup! 🚀
```

---

**[⬅️ Previous: Chapter 16](Chapter_16_Pipelining.md)** | **[➡️ Next: Chapter 18](Chapter_18_Memory_Hierarchy.md)**

---

<div align="center">

**"Your pipeline WORKS! Data flows, hazards handled. Now add CACHE!"**

**"তোমার pipeline WORKS! Data flows, hazards handled। এবার CACHE যোগ করো!"**

Made with ❤️ for builders | বানানোর জন্য ভালোবাসা দিয়ে তৈরি

</div>
