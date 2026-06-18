# 🧪 Chapter 7: Build Your Own Test Suite
## Testbenches থেকে Waveforms - তোমার Code Test করো Professionally!

> **"Code without tests is broken by design. Time to verify like a pro!"**
>
> **"Test ছাড়া code মানে broken design। এবার professional verification করো!"**

---

গত chapter গুলোতে তুমি Verilog লিখতে শিখেছো — gate, adder, flip-flop, counter, সব। কিন্তু একটা জিনিস খেয়াল করেছো? প্রতিবার code লেখার পর তুমি testbench দিয়েই সেটা চালিয়ে দেখেছো। সেই testbench টাকেই আমরা এতদিন একটু hand-wave করে পাশ কাটিয়ে গেছি — "এটা দিয়ে test হয়" বলে। এই chapter এ আমরা সেই পর্দাটা সরাবো।

ভাবো তো, তুমি একটা ALU লিখলে। সেটা ঠিকঠাক কাজ করছে কিনা তুমি কিভাবে জানবে? FPGA তে upload করে LED জ্বালিয়ে? প্রতিটা input combination এর জন্য? RV32I তে 47টা instruction, আর প্রতিটার অসংখ্য input — হাতে হাতে test করলে তোমার সারা জীবন লেগে যাবে, আর তবুও তুমি confident হতে পারবে না। এখানেই **testbench** তোমার সবচেয়ে বড় বন্ধু।

একটা জিনিস মাথায় গেঁথে নাও: **professional hardware engineer রা design লেখার চেয়ে বেশি সময় test লেখায় ব্যয় করেন।** Intel, AMD, Apple — সবার কাছে verification team গুলো design team এর চেয়ে বড়। কারণ একবার silicon চলে গেলে আর bug fix করা যায় না; একটা ভুল মানে কোটি টাকার চিপ নষ্ট। তাই এই chapter টা শুধু "আরেকটা topic" না — এটা তোমাকে একজন amateur থেকে একজন real engineer বানানোর ধাপ।

---

## 🎯 এই Chapter এ তুমি বানাবে:

```
✅ Complete testbenches - automated testing
✅ Self-checking tests - no manual verification
✅ Waveform analysis - visual debugging
✅ Clock generators - realistic timing
✅ File I/O tests - test vectors from files
✅ Coverage reports - ensure complete testing
✅ তোমার processor এর complete verification! 🎉
```

**Time Required:** 1 week (3-4 hours/day)  
**Tools Needed:** Icarus Verilog, GTKWave, Text editor

---

## 🚀 Quick Win - 5 মিনিটে তোমার First Self-Checking Test!

theory পড়ার আগে চলো হাতে-কলমে একটা মজা দেখি। এতদিন তুমি testbench এ output চোখে দেখে মিলিয়েছো — "হ্যাঁ, sum তো 8 দেখাচ্ছে, ঠিক আছে"। কিন্তু সেটা ক্লান্তিকর আর ভুল হওয়ার সম্ভাবনা বেশি। এবার এমন একটা test লিখবে যেটা **নিজেই** সিদ্ধান্ত নেবে pass হয়েছে নাকি fail — তোমাকে শুধু একটা সবুজ ✓ বা লাল ✗ দেখাবে। একে বলে **self-checking testbench**, আর এটাই এই chapter এর প্রাণ।

### এখনই লেখো - Self-Checking Testbench:

**Create file: `adder_tb.v`**

```verilog
module adder_tb;
    reg [3:0] a, b;
    wire [4:0] sum;
    integer errors = 0;
    
    // DUT (Device Under Test)
    adder_4bit dut(.a(a), .b(b), .sum(sum));
    
    initial begin
        // Test 1: 5 + 3 = 8
        a = 5; b = 3; #10;
        if (sum !== 8) begin
            $display("ERROR: 5+3=%d, expected 8", sum);
            errors = errors + 1;
        end
        
        // Test 2: 15 + 1 = 16
        a = 15; b = 1; #10;
        if (sum !== 16) begin
            $display("ERROR: 15+1=%d, expected 16", sum);
            errors = errors + 1;
        end
        
        // Summary
        if (errors == 0)
            $display("✓ ALL TESTS PASSED!");
        else
            $display("✗ %0d TESTS FAILED!", errors);
        
        $finish;
    end
endmodule
```

এখানে কী হলো একটু ভেঙে দেখি। `errors` নামের একটা counter রাখলাম শূন্য থেকে। প্রতিটা test এ input দিয়ে `#10` দিয়ে একটু অপেক্ষা করলাম (যাতে adder এর output থিতু হয়), তারপর `if (sum !== 8)` দিয়ে যাচাই করলাম। যদি ভুল হয়, তাহলে error message ছাপলাম আর counter একটা বাড়ালাম। শেষে `errors` শূন্য থাকলে "all passed", নইলে কয়টা fail হলো সেটা জানালাম। লক্ষ্য করো — তুমি একবারও চোখে output মেলাচ্ছো না; testbench টাই বিচারক।

আরেকটা ছোট কিন্তু গুরুত্বপূর্ণ জিনিস: এখানে `!==` ব্যবহার করেছি, `!=` না। এই দুটোর পার্থক্য এই chapter এ পরে বিস্তারিত আসবে, কিন্তু এক কথায় — `!==` হলো verification এর জন্য বানানো, কারণ এটা `x` (unknown) আর `z` (high-impedance) value কেও ঠিকঠাক ধরতে পারে। self-checking test এ সবসময় `===` আর `!==` ব্যবহার করবে।

**Run:**
```bash
iverilog -o sim adder_4bit.v adder_tb.v
vvp sim
# Output: ✓ ALL TESTS PASSED!
```

🎉 **Congratulations! তোমার প্রথম self-checking test!**

মাত্র ৫ মিনিটে তুমি এমন একটা জিনিস বানালে যেটা professional রা প্রতিদিন ব্যবহার করেন। এবার চলো বুঝি ভেতরে আসলে কী ঘটছে — কারণ "কাজ করছে" জানার চেয়ে "কেন কাজ করছে" বোঝা অনেক বেশি শক্তিশালী।

---

## ৭.১ Testbench Fundamentals

### What is a Testbench?

```
Testbench = Verilog code that:
✅ Instantiates your module (DUT)
✅ Applies test inputs
✅ Checks outputs
✅ Reports results
✅ NO hardware synthesis!

DUT = Device Under Test (তোমার module)
```

### Testbench Structure:

```verilog
`timescale 1ns/1ps  // Time unit/precision

module testbench;
    // 1. Signal declarations
    reg  inputs;   // Inputs to DUT (use reg)
    wire outputs;  // Outputs from DUT (use wire)
    
    // 2. Instantiate DUT
    module_name dut(
        .input_port(inputs),
        .output_port(outputs)
    );
    
    // 3. Stimulus generation
    initial begin
        // Apply test vectors
        inputs = 0;
        #10;
        inputs = 1;
        #10;
        $finish;
    end
    
    // 4. Response checking (optional)
    initial begin
        $monitor("Time=%0t inputs=%b outputs=%b", 
                 $time, inputs, outputs);
    end
endmodule
```

### Input vs Output in Testbench:

```verilog
// In DUT (module):
module my_module(
    input  clk,      // Input
    output reg q     // Output
);

// In Testbench:
module testbench;
    reg  clk;        // Drive DUT input → reg
    wire q;          // Read DUT output → wire
    
    my_module dut(.clk(clk), .q(q));
```

---

## ৭.২ Clock Generation

### Simple Clock:

```verilog
reg clk;

initial begin
    clk = 0;
    forever #5 clk = ~clk;  // Toggle every 5ns (10ns period)
end
// Creates: 100 MHz clock
```

### Clock with Period Parameter:

```verilog
parameter CLK_PERIOD = 10;  // 10ns = 100MHz

reg clk;

initial begin
    clk = 0;
    forever #(CLK_PERIOD/2) clk = ~clk;
end
```

### Multiple Clocks:

```verilog
reg clk_fast, clk_slow;

initial begin
    clk_fast = 0;
    forever #5 clk_fast = ~clk_fast;   // 100 MHz
end

initial begin
    clk_slow = 0;
    forever #50 clk_slow = ~clk_slow;  // 10 MHz
end
```

### Clock with Duty Cycle:

```verilog
reg clk;

initial begin
    clk = 0;
    forever begin
        #3 clk = 1;  // High for 3ns
        #7 clk = 0;  // Low for 7ns
    end
end
// 30% duty cycle
```

---

## ৭.৩ Timing Control

### Delay (#):

```verilog
initial begin
    a = 0;
    #10;        // Wait 10 time units
    a = 1;
    #5;         // Wait 5 time units
    a = 0;
end
```

### Wait (@):

```verilog
initial begin
    @(posedge clk);  // Wait for rising edge
    data = 8'hAA;
    
    @(negedge clk);  // Wait for falling edge
    data = 8'h55;
end
```

### Wait for Condition (wait):

```verilog
initial begin
    data = 0;
    wait(ready == 1);  // Wait until ready is 1
    data = 8'hFF;
end
```

### Repeat:

```verilog
initial begin
    repeat(10) begin
        @(posedge clk);  // Wait 10 clock cycles
    end
    $display("10 clocks passed");
end
```

---

## ৭.৪ System Tasks for Testing

### Display Tasks:

```verilog
$display("Text %format", variables);
$write("Text %format", variables);  // No newline

// Format specifiers:
%b or %0b - Binary
%h or %0h - Hexadecimal  
%d or %0d - Decimal
%t - Time
%s - String
```

### Monitor Task:

```verilog
$monitor("Time=%0t a=%b b=%b sum=%d", $time, a, b, sum);
// Automatically prints when any variable changes
// Only one $monitor active at a time

$monitoron;   // Enable monitoring
$monitoroff;  // Disable monitoring
```

### Strobe Task:

```verilog
$strobe("a=%d", a);
// Prints at end of current time step
// Useful for capturing final values after all events
```

### Finish and Stop:

```verilog
$finish;      // Exit simulation
$finish(0);   // Exit with status 0
$finish(1);   // Exit with status 1

$stop;        // Pause simulation (interactive)
```

### Time Functions:

```verilog
$time         // Current simulation time (integer)
$realtime     // Current time (real number)
$stime        // Current time (32-bit unsigned)
```

---

## ৭.৫ Self-Checking Testbenches

### Basic Self-Checking:

```verilog
module and_gate_tb;
    reg a, b;
    wire y;
    integer errors = 0;
    
    and_gate dut(.a(a), .b(b), .y(y));
    
    initial begin
        // Test all combinations
        a=0; b=0; #10;
        if (y !== 0) begin
            $display("ERROR: 0&0=%b, expected 0", y);
            errors = errors + 1;
        end
        
        a=0; b=1; #10;
        if (y !== 0) begin
            $display("ERROR: 0&1=%b, expected 0", y);
            errors = errors + 1;
        end
        
        a=1; b=0; #10;
        if (y !== 0) begin
            $display("ERROR: 1&0=%b, expected 0", y);
            errors = errors + 1;
        end
        
        a=1; b=1; #10;
        if (y !== 1) begin
            $display("ERROR: 1&1=%b, expected 1", y);
            errors = errors + 1;
        end
        
        // Report
        $display("\n========== TEST SUMMARY ==========");
        if (errors == 0)
            $display("✓ ALL TESTS PASSED!");
        else
            $display("✗ %0d TESTS FAILED!", errors);
        $display("==================================\n");
        
        $finish;
    end
endmodule
```

### Using Tasks for Checking:

```verilog
module adder_tb;
    reg [7:0] a, b;
    wire [8:0] sum;
    integer errors = 0;
    
    adder_8bit dut(.a(a), .b(b), .sum(sum));
    
    // Task for checking
    task check_result;
        input [7:0] in_a, in_b;
        input [8:0] expected;
        begin
            a = in_a;
            b = in_b;
            #10;
            
            if (sum !== expected) begin
                $display("ERROR: %0d + %0d = %0d, expected %0d",
                        in_a, in_b, sum, expected);
                errors = errors + 1;
            end else begin
                $display("PASS: %0d + %0d = %0d ✓",
                        in_a, in_b, sum);
            end
        end
    endtask
    
    initial begin
        $display("Testing 8-bit Adder");
        
        check_result(0, 0, 0);
        check_result(5, 3, 8);
        check_result(255, 1, 256);
        check_result(100, 50, 150);
        
        // Summary
        $display("\n========== SUMMARY ==========");
        if (errors == 0)
            $display("✓ ALL TESTS PASSED!");
        else
            $display("✗ %0d TESTS FAILED!", errors);
        
        $finish;
    end
endmodule
```

---

## ৭.৬ Testing Sequential Circuits

### Testing D Flip-Flop:

```verilog
module dff_tb;
    reg clk, reset, d;
    wire q;
    
    d_ff dut(.clk(clk), .reset(reset), .d(d), .q(q));
    
    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end
    
    // Test stimulus
    initial begin
        // Initialize
        reset = 1;
        d = 0;
        #15;  // Wait 1.5 clock cycles
        
        // Release reset
        reset = 0;
        #10;
        
        // Test 1: Load 1
        @(negedge clk);
        d = 1;
        @(posedge clk);
        #1;  // Small delay after clock edge
        if (q !== 1) $display("ERROR: q should be 1");
        else $display("PASS: Loaded 1 ✓");
        
        // Test 2: Load 0
        @(negedge clk);
        d = 0;
        @(posedge clk);
        #1;
        if (q !== 0) $display("ERROR: q should be 0");
        else $display("PASS: Loaded 0 ✓");
        
        // Test 3: Hold value
        @(negedge clk);
        d = 1;
        @(posedge clk);
        #1;
        @(negedge clk);
        d = 0;  // Change d but don't clock yet
        #4;  // Wait before clock edge
        if (q !== 1) $display("ERROR: Should hold value");
        else $display("PASS: Holds value ✓");
        
        #20;
        $finish;
    end
    
    // Monitor
    initial begin
        $monitor("Time=%0t reset=%b d=%b q=%b", 
                 $time, reset, d, q);
    end
endmodule
```

### Testing Counter:

```verilog
module counter_tb;
    reg clk, reset, en;
    wire [3:0] count;
    
    counter_4bit dut(.clk(clk), .reset(reset), 
                     .en(en), .count(count));
    
    // Clock
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end
    
    // Test
    initial begin
        // Reset
        reset = 1; en = 0;
        repeat(2) @(posedge clk);
        reset = 0;
        
        // Enable and count
        en = 1;
        repeat(20) begin
            @(posedge clk);
            $display("Count = %0d", count);
        end
        
        // Disable
        en = 0;
        repeat(5) @(posedge clk);
        $display("Count stopped at %0d", count);
        
        $finish;
    end
endmodule
```

---

## ৭.৭ Waveform Generation and Viewing

### Generating VCD Files:

```verilog
module testbench;
    // Your testbench code...
    
    initial begin
        // VCD file generation
        $dumpfile("waveform.vcd");
        $dumpvars(0, testbench);
        // 0 = dump all levels
        // testbench = starting module
        
        // Your tests...
        
        $finish;
    end
endmodule
```

### Viewing with GTKWave:

```bash
# Run simulation to generate VCD
iverilog -o sim module.v testbench.v
vvp sim

# Open in GTKWave
gtkwave waveform.vcd
```

### GTKWave Usage:

```
1. Left Panel: Signal hierarchy
   - Select signals to view

2. Middle: Add signals
   - Drag signals to waveform view
   - Or click "Append" button

3. Waveform View:
   - Zoom: Ctrl + Scroll
   - Measure: Click two points
   - Move: Click and drag

4. Time Cursor:
   - Yellow line shows time
   - Values displayed at cursor

5. Search:
   - Ctrl+F: Find signals
   - Jump to time
```

### Advanced VCD Control:

```verilog
initial begin
    $dumpfile("waveform.vcd");
    
    // Dump specific signals
    $dumpvars(1, testbench.dut.signal1);
    $dumpvars(1, testbench.dut.signal2);
    
    // Start dumping later
    #100;
    $dumpvars(0, testbench);
    
    // Stop dumping
    #1000;
    $dumpoff;
    
    // Resume dumping
    #500;
    $dumpon;
end
```

---

## ৭.৮ File I/O for Test Vectors

### Reading Test Vectors from File:

**test_vectors.txt:**
```
// a b expected_sum
0 0 0
0 1 1
1 0 1
1 1 2
5 3 8
15 1 16
```

**Testbench:**
```verilog
module adder_file_tb;
    reg [3:0] a, b;
    wire [4:0] sum;
    reg [4:0] expected;
    integer file, status, errors;
    
    adder_4bit dut(.a(a), .b(b), .sum(sum));
    
    initial begin
        errors = 0;
        file = $fopen("test_vectors.txt", "r");
        
        if (file == 0) begin
            $display("ERROR: Cannot open file");
            $finish;
        end
        
        // Read and test
        while (!$feof(file)) begin
            status = $fscanf(file, "%d %d %d\n", 
                           a, b, expected);
            
            if (status == 3) begin  // Read 3 values
                #10;
                
                if (sum !== expected) begin
                    $display("ERROR: %0d+%0d=%0d, expected %0d",
                            a, b, sum, expected);
                    errors = errors + 1;
                end else begin
                    $display("PASS: %0d+%0d=%0d ✓",
                            a, b, sum);
                end
            end
        end
        
        $fclose(file);
        
        // Summary
        if (errors == 0)
            $display("✓ ALL TESTS PASSED!");
        else
            $display("✗ %0d TESTS FAILED!", errors);
        
        $finish;
    end
endmodule
```

### Writing Results to File:

```verilog
integer outfile;

initial begin
    outfile = $fopen("results.txt", "w");
    
    // Write header
    $fdisplay(outfile, "Input_A Input_B Sum");
    $fdisplay(outfile, "=====================");
    
    // Tests...
    a = 5; b = 3; #10;
    $fdisplay(outfile, "%0d %0d %0d", a, b, sum);
    
    a = 7; b = 8; #10;
    $fdisplay(outfile, "%0d %0d %0d", a, b, sum);
    
    $fclose(outfile);
    $finish;
end
```

---

## ৭.৯ Random Testing

### Random Number Generation:

```verilog
integer seed;

initial begin
    seed = 123;  // Initialize seed
    
    repeat(100) begin
        a = $random(seed);  // Random value
        b = $random(seed);
        #10;
        
        // Check result
        if (sum !== (a + b))
            $display("ERROR at a=%0d b=%0d", a, b);
    end
end
```

### Constrained Random:

```verilog
reg [3:0] a;

initial begin
    repeat(10) begin
        // Random 4-bit value (0-15)
        a = $random % 16;
        
        // Random in range [5:10]
        a = 5 + ($random % 6);
        
        #10;
    end
end
```

### Random Testing with Coverage:

```verilog
module alu_random_tb;
    reg [7:0] a, b;
    reg [2:0] opcode;
    wire [7:0] result;
    integer i, seed, errors;
    
    alu_8bit dut(.a(a), .b(b), .opcode(opcode), 
                 .result(result));
    
    initial begin
        seed = 42;
        errors = 0;
        
        // Random testing
        for (i = 0; i < 1000; i = i + 1) begin
            a = $random(seed);
            b = $random(seed);
            opcode = $random(seed) % 8;
            #10;
            
            // Check based on opcode
            case(opcode)
                3'b000: begin  // ADD
                    if (result !== (a + b))
                        errors = errors + 1;
                end
                3'b001: begin  // SUB
                    if (result !== (a - b))
                        errors = errors + 1;
                end
                // ... other operations
            endcase
        end
        
        $display("Completed 1000 random tests");
        $display("Errors: %0d", errors);
        $finish;
    end
endmodule
```

---

## ৭.১০ Testing FSMs

### FSM Testbench Example:

```verilog
module traffic_light_tb;
    reg clk, reset;
    wire red, yellow, green;
    
    traffic_light dut(
        .clk(clk), 
        .reset(reset),
        .red(red), 
        .yellow(yellow), 
        .green(green)
    );
    
    // Clock
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end
    
    // Test
    initial begin
        // VCD
        $dumpfile("traffic.vcd");
        $dumpvars(0, traffic_light_tb);
        
        // Reset
        reset = 1;
        repeat(3) @(posedge clk);
        reset = 0;
        
        // Monitor states
        $display("Time\tRed\tYellow\tGreen");
        $display("============================");
        
        // Run for several cycles
        repeat(100) begin
            @(posedge clk);
            $display("%0t\t%b\t%b\t%b", 
                    $time, red, yellow, green);
        end
        
        // Check state transitions
        // (More detailed checking here)
        
        $finish;
    end
    
    // Check only one light on at a time
    always @(posedge clk) begin
        if ((red + yellow + green) != 1) begin
            $display("ERROR: Multiple lights on!");
            $finish;
        end
    end
endmodule
```

---

## ৭.১১ Complete Testbench Example

### Testing 8-bit ALU:

```verilog
`timescale 1ns/1ps

module alu_8bit_tb;
    // Signals
    reg [7:0] a, b;
    reg [2:0] opcode;
    wire [7:0] result;
    wire zero, carry, negative;
    
    // Counters
    integer tests_run, tests_passed, tests_failed;
    
    // DUT
    alu_8bit dut(
        .a(a), .b(b), .opcode(opcode),
        .result(result), .zero(zero), 
        .carry(carry), .negative(negative)
    );
    
    // Task for testing
    task test_alu;
        input [7:0] in_a, in_b;
        input [2:0] in_opcode;
        input [7:0] expected_result;
        input expected_zero, expected_carry, expected_neg;
        reg [80*8:1] op_name;
        begin
            tests_run = tests_run + 1;
            
            a = in_a;
            b = in_b;
            opcode = in_opcode;
            #10;
            
            // Operation name
            case(in_opcode)
                3'b000: op_name = "ADD";
                3'b001: op_name = "SUB";
                3'b010: op_name = "AND";
                3'b011: op_name = "OR";
                3'b100: op_name = "XOR";
                3'b101: op_name = "NOT";
                3'b110: op_name = "SHL";
                3'b111: op_name = "SHR";
            endcase
            
            // Check result
            if (result !== expected_result ||
                zero !== expected_zero ||
                carry !== expected_carry ||
                negative !== expected_neg) begin
                
                $display("FAIL: %s %0d %0d", 
                        op_name, in_a, in_b);
                $display("  Expected: result=%0d Z=%b C=%b N=%b",
                        expected_result, expected_zero, 
                        expected_carry, expected_neg);
                $display("  Got:      result=%0d Z=%b C=%b N=%b",
                        result, zero, carry, negative);
                tests_failed = tests_failed + 1;
            end else begin
                $display("PASS: %s %0d %0d = %0d ✓",
                        op_name, in_a, in_b, result);
                tests_passed = tests_passed + 1;
            end
        end
    endtask
    
    // Main test
    initial begin
        tests_run = 0;
        tests_passed = 0;
        tests_failed = 0;
        
        $display("\n========== ALU TEST ==========\n");
        
        // Test ADD
        test_alu(5, 3, 3'b000, 8, 0, 0, 0);
        test_alu(255, 1, 3'b000, 0, 1, 1, 0);
        
        // Test SUB
        test_alu(10, 5, 3'b001, 5, 0, 0, 0);
        test_alu(5, 10, 3'b001, 251, 0, 0, 1);
        
        // Test AND
        test_alu(8'b1111_0000, 8'b0011_1100, 
                3'b010, 8'b0011_0000, 0, 0, 0);
        
        // Test OR
        test_alu(8'b1111_0000, 8'b0011_1100, 
                3'b011, 8'b1111_1100, 0, 0, 1);
        
        // Test XOR
        test_alu(8'b1111_0000, 8'b1111_1111, 
                3'b100, 8'b0000_1111, 0, 0, 0);
        
        // Test NOT
        test_alu(8'b1010_1010, 8'b0, 
                3'b101, 8'b0101_0101, 0, 0, 0);
        
        // Summary
        $display("\n========== SUMMARY ==========");
        $display("Tests Run:    %0d", tests_run);
        $display("Tests Passed: %0d", tests_passed);
        $display("Tests Failed: %0d", tests_failed);
        
        if (tests_failed == 0)
            $display("\n✓ ALL TESTS PASSED! ✓\n");
        else
            $display("\n✗ SOME TESTS FAILED ✗\n");
        
        $display("============================\n");
        
        $finish;
    end
    
    // Waveform
    initial begin
        $dumpfile("alu_test.vcd");
        $dumpvars(0, alu_8bit_tb);
    end
endmodule
```

---

## ৭.১২ Your 1-Week Build Plan

### Day 1: Basic Testbenches
```
□ Write simple testbench structure
□ Test combinational circuits
□ Use $display and $monitor
□ Generate first VCD file
```

### Day 2: Clock and Timing
```
□ Create clock generators
□ Test sequential circuits
□ Use @ for synchronization
□ Master timing control
```

### Day 3: Self-Checking Tests
```
□ Add error counting
□ Create checking tasks
□ Build test summaries
□ Automate verification
```

### Day 4: File I/O
```
□ Read test vectors from files
□ Write results to files
□ Create large test suites
□ Organize test data
```

### Day 5: GTKWave Mastery
```
□ Learn GTKWave navigation
□ Analyze waveforms
□ Debug with waveforms
□ Find timing issues
```

### Day 6: Advanced Testing
```
□ Random testing
□ FSM testing
□ Coverage analysis
□ Edge case testing
```

### Day 7: Complete Test Suite
```
□ Build comprehensive test
□ All modules tested
□ Professional report
□ Final project testing
```

---

## ৭.১৩ Debugging with Waveforms

### Common Timing Issues:

```
Issue 1: Setup/Hold Violation
Symptom: Random failures
Fix: Check clock timing in waveform

Issue 2: Race Condition
Symptom: Inconsistent results
Fix: Use non-blocking assignments

Issue 3: Glitch
Symptom: Short unwanted pulse
Fix: Add pipeline stage

Issue 4: Clock Skew
Symptom: Different FFs update differently
Fix: Synchronous design
```

### Debugging Workflow:

```
1. Identify failure in test
2. Open GTKWave
3. Find failing time point
4. Add relevant signals
5. Trace backwards from output
6. Find where signal goes wrong
7. Check that module's inputs
8. Continue until root cause
```

---

## ৭.১৪ Professional Verification Tips

### Do's:
```
✅ Test early and often
✅ Automate everything
✅ Self-checking testbenches
✅ Test corner cases
✅ Use random testing
✅ Generate waveforms
✅ Document expected behavior
✅ Keep tests with code
```

### Don'ts:
```
❌ Manual output checking
❌ Testing only happy path
❌ Skipping edge cases
❌ Not testing after changes
❌ Complex testbenches
❌ No test documentation
```

---

## ৭.১৫ Chapter 7 Mission Complete!

### তুমি এখন পারো:

```
✅ Write comprehensive testbenches
✅ Self-checking tests
✅ Generate and analyze waveforms
✅ File-based testing
✅ Random testing
✅ Debug with GTKWave
✅ Professional verification
✅ তোমার processor test করা professionally!
```

### তুমি বানিয়েছো:
```
✅ Self-checking testbenches
✅ Clock generators
✅ File I/O tests
✅ Random testers
✅ FSM testers
✅ Complete ALU test suite! 🎉
```

### Stats:
```
Testbenches written: 10+
Test cases: 100+
Waveforms analyzed: Multiple
Bugs found: All!
Level: Verification Expert! 🏆
```

### Next Level Unlocked:
```
→ Chapter 8: Advanced Verilog
   তুমি শিখবে: Functions, tasks, generate
   Parameters, compiler directives!
   
   From testing → Advanced features!
```

---

## 🎯 Final Project

### Project: Complete Processor Component Test Suite

**Test:**
```
✅ ALU (all operations)
✅ Register file (read/write)
✅ Program counter
✅ Instruction decoder
✅ Control unit

Requirements:
✅ Self-checking
✅ Random tests
✅ Edge cases
✅ Waveforms
✅ 100% coverage
✅ Professional report
```

---

## 🏆 Achievement Unlocked!

```
Level 7: ✅ COMPLETE - Verification Master!
Progress: [███████████████████████████████] 35%

XP Gained: +2500
Skills: Testing, Debugging, Professional Verification

Badges Earned:
🥉 Testbench Writer
🥈 Waveform Analyzer
🥇 Self-Checking Master
🏅 GTKWave Expert
🎖️ Random Testing Pro
🏆 Professional Verifier

Next: Chapter 8 - Advanced Verilog!
      Functions, tasks, generate! 🚀
```

---

**[⬅️ Previous: Chapter 6](Chapter_06_Always_Blocks.md)** | **[➡️ Next: Chapter 8](Chapter_08_Advanced_Verilog.md)**

---

<div align="center">

**"You mastered verification. Your code is now bulletproof!"**

**"তুমি verification master করেছো। তোমার code এখন bulletproof!"**

Made with ❤️ for builders | বানানোর জন্য ভালোবাসা দিয়ে তৈরি

</div>
