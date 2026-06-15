# 🖥️ Chapter 19: Build Your Own Complete Computer System
## From CPU to SoC - A Real Working Computer with I/O!

> **"You built the CPU. Now add PERIPHERALS. Time to make a REAL COMPUTER!"**
>
> **"তুমি CPU বানিয়েছো। এবার PERIPHERALS যোগ করো। REAL COMPUTER বানাও!"**

---

## 🎯 এই Chapter এ তুমি বানাবে:

```
✅ UART (Serial Communication) - talk to computer
✅ GPIO (Input/Output) - LEDs, buttons
✅ Interrupt Controller - handle events
✅ Timer - time management
✅ Memory-Mapped I/O - unified address space
✅ Complete SoC - System-on-Chip
✅ Working Computer - interact with world!
✅ তোমার complete system! 🎉
```

**Time Required:** 2 weeks (8-10 hours/day)  
**Prerequisites:** Chapters 12-18 complete

---

## 🚀 Quick Understanding - From CPU to Computer!

### What is a Computer System?

```
CPU alone is NOT enough!

Need:
✅ CPU (processor) ← You built this!
✅ Memory (cache + RAM) ← You built this!
✅ Input devices (keyboard, buttons)
✅ Output devices (screen, LEDs)
✅ Communication (UART, SPI, I2C)
✅ Timers (delays, scheduling)
✅ Interrupts (event handling)

Together = COMPLETE COMPUTER! 🖥️
```

### System-on-Chip (SoC):

```
Everything on one chip:
┌────────────────────────────────────┐
│         RISC-V SoC                │
├────────────────────────────────────┤
│  CPU Core (Pipelined + Cache)     │
├────────────────────────────────────┤
│  Memory (ROM + RAM)                │
├────────────────────────────────────┤
│  Peripherals:                      │
│    - UART (Serial)                 │
│    - GPIO (I/O pins)               │
│    - Timer                         │
│    - Interrupt Controller          │
├────────────────────────────────────┤
│  Bus (Connect everything)          │
└────────────────────────────────────┘

Like Raspberry Pi, Arduino, etc!
```

### Memory-Mapped I/O:

```
Unified address space:
0x00000000 - 0x00003FFF: Boot ROM (16KB)
0x00004000 - 0x0000FFFF: RAM (48KB)
0x10000000 - 0x10000FFF: GPIO
0x10001000 - 0x10001FFF: UART
0x10002000 - 0x10002FFF: Timer
0x10003000 - 0x10003FFF: Interrupt Ctrl

Access like memory:
LW x5, 0x10001000(x0)  # Read UART
SW x6, 0x10000000(x0)  # Write GPIO

Simple! Unified! Powerful!
```

🎉 **This chapter = COMPLETE WORKING COMPUTER!**

---

## ১৯.১ UART (Universal Asynchronous Receiver/Transmitter)

### What is UART?

```
UART = Serial communication
Send/receive data one bit at a time
Connect to computer via USB-Serial

Uses:
✅ Debug output (printf)
✅ Terminal communication
✅ Send/receive text
✅ Program upload
✅ Console interface

Standard in embedded systems!
```

### UART Implementation:

```verilog
module uart #(
    parameter CLOCK_FREQ = 50000000,  // 50 MHz
    parameter BAUD_RATE = 115200       // Standard baud rate
)(
    input wire clk,
    input wire reset,
    // CPU interface
    input wire [31:0] address,
    input wire [31:0] write_data,
    input wire write_enable,
    input wire read_enable,
    output reg [31:0] read_data,
    // UART pins
    input wire rx,
    output wire tx,
    // Interrupt
    output reg tx_done,
    output reg rx_ready
);
    // Baud rate generator
    localparam BAUD_DIV = CLOCK_FREQ / BAUD_RATE;
    reg [15:0] baud_counter;
    reg baud_tick;
    
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            baud_counter <= 0;
            baud_tick <= 0;
        end else begin
            if (baud_counter >= BAUD_DIV - 1) begin
                baud_counter <= 0;
                baud_tick <= 1;
            end else begin
                baud_counter <= baud_counter + 1;
                baud_tick <= 0;
            end
        end
    end
    
    // Transmitter
    reg [7:0] tx_data;
    reg tx_start;
    reg tx_busy;
    reg [3:0] tx_bit_count;
    reg [9:0] tx_shift;
    
    assign tx = tx_shift[0];
    
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            tx_shift <= 10'b1111111111;
            tx_busy <= 0;
            tx_bit_count <= 0;
            tx_done <= 0;
        end else begin
            tx_done <= 0;
            
            if (tx_start && !tx_busy) begin
                // Start transmission
                tx_shift <= {1'b1, tx_data, 1'b0};  // Stop, Data, Start
                tx_busy <= 1;
                tx_bit_count <= 0;
            end else if (tx_busy && baud_tick) begin
                if (tx_bit_count < 9) begin
                    tx_shift <= {1'b1, tx_shift[9:1]};
                    tx_bit_count <= tx_bit_count + 1;
                end else begin
                    tx_busy <= 0;
                    tx_done <= 1;
                end
            end
        end
    end
    
    // Receiver
    reg [7:0] rx_data;
    reg rx_busy;
    reg [3:0] rx_bit_count;
    reg [8:0] rx_shift;
    reg rx_sync, rx_sync2;
    
    // Synchronize RX
    always @(posedge clk) begin
        rx_sync <= rx;
        rx_sync2 <= rx_sync;
    end
    
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            rx_busy <= 0;
            rx_bit_count <= 0;
            rx_ready <= 0;
            rx_data <= 0;
        end else begin
            if (!rx_busy && !rx_sync2) begin
                // Start bit detected
                rx_busy <= 1;
                rx_bit_count <= 0;
                rx_ready <= 0;
            end else if (rx_busy && baud_tick) begin
                if (rx_bit_count < 8) begin
                    rx_shift <= {rx_sync2, rx_shift[8:1]};
                    rx_bit_count <= rx_bit_count + 1;
                end else begin
                    // Stop bit
                    rx_data <= rx_shift[7:0];
                    rx_ready <= 1;
                    rx_busy <= 0;
                end
            end
        end
    end
    
    // Register interface
    localparam UART_DATA = 2'b00;
    localparam UART_STATUS = 2'b01;
    
    wire [1:0] reg_addr = address[3:2];
    
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            tx_start <= 0;
        end else begin
            tx_start <= 0;
            
            if (write_enable) begin
                case (reg_addr)
                    UART_DATA: begin
                        tx_data <= write_data[7:0];
                        tx_start <= 1;
                    end
                endcase
            end
        end
    end
    
    always @(*) begin
        case (reg_addr)
            UART_DATA: read_data = {24'h0, rx_data};
            UART_STATUS: read_data = {30'h0, tx_busy, rx_ready};  // bit0=rx_ready, bit1=tx_busy
            default: read_data = 32'h0;
        endcase
    end
endmodule
```

---

## ১৯.২ GPIO (General Purpose Input/Output)

### GPIO Implementation:

```verilog
module gpio #(
    parameter NUM_PINS = 32
)(
    input wire clk,
    input wire reset,
    // CPU interface
    input wire [31:0] address,
    input wire [31:0] write_data,
    input wire write_enable,
    input wire read_enable,
    output reg [31:0] read_data,
    // GPIO pins
    input wire [NUM_PINS-1:0] gpio_in,
    output reg [NUM_PINS-1:0] gpio_out,
    output reg [NUM_PINS-1:0] gpio_dir  // 0=input, 1=output
);
    // Registers
    reg [NUM_PINS-1:0] gpio_data;
    reg [NUM_PINS-1:0] gpio_input_sync;
    
    // Register map
    localparam GPIO_DATA = 2'b00;      // R/W: GPIO data
    localparam GPIO_DIR = 2'b01;       // R/W: Direction
    localparam GPIO_INPUT = 2'b10;     // R: Input values
    
    wire [1:0] reg_addr = address[3:2];
    
    // Synchronize inputs
    always @(posedge clk) begin
        gpio_input_sync <= gpio_in;
    end
    
    // Output logic
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            gpio_data <= 0;
            gpio_dir <= 0;
        end else if (write_enable) begin
            case (reg_addr)
                GPIO_DATA: gpio_data <= write_data[NUM_PINS-1:0];
                GPIO_DIR: gpio_dir <= write_data[NUM_PINS-1:0];
            endcase
        end
    end
    
    // Assign outputs
    always @(*) begin
        gpio_out = gpio_data & gpio_dir;  // Only output if dir=1
    end
    
    // Read logic
    always @(*) begin
        case (reg_addr)
            GPIO_DATA: read_data = {{(32-NUM_PINS){1'b0}}, gpio_data};
            GPIO_DIR: read_data = {{(32-NUM_PINS){1'b0}}, gpio_dir};
            GPIO_INPUT: read_data = {{(32-NUM_PINS){1'b0}}, gpio_input_sync};
            default: read_data = 32'h0;
        endcase
    end
endmodule
```

---

## ১৯.৩ Timer

### Timer Implementation:

```verilog
module timer(
    input wire clk,
    input wire reset,
    // CPU interface
    input wire [31:0] address,
    input wire [31:0] write_data,
    input wire write_enable,
    input wire read_enable,
    output reg [31:0] read_data,
    // Interrupt
    output reg timer_interrupt
);
    // Registers
    reg [31:0] counter;
    reg [31:0] compare;
    reg enable;
    reg [31:0] prescaler;
    reg [31:0] prescaler_count;
    
    // Register map
    localparam TIMER_COUNTER = 2'b00;   // R/W: Current count
    localparam TIMER_COMPARE = 2'b01;   // R/W: Compare value
    localparam TIMER_CONTROL = 2'b10;   // R/W: Control register
    localparam TIMER_PRESCALER = 2'b11; // R/W: Prescaler
    
    wire [1:0] reg_addr = address[3:2];
    
    // Timer logic
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            counter <= 0;
            compare <= 32'hFFFFFFFF;
            enable <= 0;
            prescaler <= 0;
            prescaler_count <= 0;
            timer_interrupt <= 0;
        end else begin
            timer_interrupt <= 0;
            
            // Handle writes
            if (write_enable) begin
                case (reg_addr)
                    TIMER_COUNTER: counter <= write_data;
                    TIMER_COMPARE: compare <= write_data;
                    TIMER_CONTROL: enable <= write_data[0];
                    TIMER_PRESCALER: prescaler <= write_data;
                endcase
            end
            
            // Counter logic
            if (enable) begin
                if (prescaler_count >= prescaler) begin
                    prescaler_count <= 0;
                    counter <= counter + 1;
                    
                    // Check for match
                    if (counter >= compare) begin
                        counter <= 0;
                        timer_interrupt <= 1;
                    end
                end else begin
                    prescaler_count <= prescaler_count + 1;
                end
            end
        end
    end
    
    // Read logic
    always @(*) begin
        case (reg_addr)
            TIMER_COUNTER: read_data = counter;
            TIMER_COMPARE: read_data = compare;
            TIMER_CONTROL: read_data = {31'h0, enable};
            TIMER_PRESCALER: read_data = prescaler;
            default: read_data = 32'h0;
        endcase
    end
endmodule
```

---

## ১৯.৪ Interrupt Controller

### Interrupt Controller:

```verilog
module interrupt_controller(
    input wire clk,
    input wire reset,
    // CPU interface
    input wire [31:0] address,
    input wire [31:0] write_data,
    input wire write_enable,
    input wire read_enable,
    output reg [31:0] read_data,
    // Interrupt sources
    input wire [7:0] interrupt_sources,
    // CPU interrupt
    output reg interrupt_request,
    output reg [7:0] interrupt_id,
    input wire interrupt_ack
);
    // Registers
    reg [7:0] interrupt_enable;
    reg [7:0] interrupt_pending;
    reg [7:0] interrupt_priority [0:7];
    
    // Register map
    localparam INT_PENDING = 2'b00;   // R: Pending interrupts
    localparam INT_ENABLE = 2'b01;    // R/W: Enable mask
    localparam INT_PRIORITY = 2'b10;  // R/W: Priority (not used in simple)
    localparam INT_ACK = 2'b11;       // W: Acknowledge
    
    wire [1:0] reg_addr = address[3:2];
    
    // Latch interrupt sources
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            interrupt_pending <= 0;
            interrupt_enable <= 0;
        end else begin
            // Latch new interrupts
            interrupt_pending <= interrupt_pending | interrupt_sources;
            
            // Handle writes
            if (write_enable) begin
                case (reg_addr)
                    INT_ENABLE: interrupt_enable <= write_data[7:0];
                    INT_ACK: interrupt_pending <= interrupt_pending & ~write_data[7:0];
                endcase
            end
        end
    end
    
    // Priority encoder (find highest priority interrupt)
    integer i;
    always @(*) begin
        interrupt_request = 0;
        interrupt_id = 0;
        
        for (i = 7; i >= 0; i = i - 1) begin
            if (interrupt_pending[i] && interrupt_enable[i]) begin
                interrupt_request = 1;
                interrupt_id = i;
            end
        end
    end
    
    // Read logic
    always @(*) begin
        case (reg_addr)
            INT_PENDING: read_data = {24'h0, interrupt_pending};
            INT_ENABLE: read_data = {24'h0, interrupt_enable};
            default: read_data = 32'h0;
        endcase
    end
endmodule
```

---

## ১৯.৫ Memory Bus & Address Decoder

### System Bus:

```verilog
module system_bus(
    input wire clk,
    input wire reset,
    // CPU interface
    input wire [31:0] address,
    input wire [31:0] write_data,
    input wire read_enable,
    input wire write_enable,
    output reg [31:0] read_data,
    output reg ready,
    // Memory interface
    output wire [31:0] mem_address,
    output wire [31:0] mem_write_data,
    output wire mem_read,
    output wire mem_write,
    input wire [31:0] mem_read_data,
    input wire mem_ready,
    // Peripheral interfaces
    output wire uart_select,
    output wire gpio_select,
    output wire timer_select,
    output wire intc_select,
    input wire [31:0] uart_read_data,
    input wire [31:0] gpio_read_data,
    input wire [31:0] timer_read_data,
    input wire [31:0] intc_read_data
);
    // Address decode
    wire mem_select = (address < 32'h10000000);
    assign uart_select = (address >= 32'h10001000) && (address < 32'h10002000);
    assign gpio_select = (address >= 32'h10000000) && (address < 32'h10001000);
    assign timer_select = (address >= 32'h10002000) && (address < 32'h10003000);
    assign intc_select = (address >= 32'h10003000) && (address < 32'h10004000);
    
    // Memory signals
    assign mem_address = address;
    assign mem_write_data = write_data;
    assign mem_read = read_enable && mem_select;
    assign mem_write = write_enable && mem_select;
    
    // Read data mux
    always @(*) begin
        if (mem_select)
            read_data = mem_read_data;
        else if (uart_select)
            read_data = uart_read_data;
        else if (gpio_select)
            read_data = gpio_read_data;
        else if (timer_select)
            read_data = timer_read_data;
        else if (intc_select)
            read_data = intc_read_data;
        else
            read_data = 32'h0;
    end
    
    // Ready signal
    always @(*) begin
        if (mem_select)
            ready = mem_ready;
        else
            ready = 1;  // Peripherals respond immediately
    end
endmodule
```

---

## ১৯.৬ Complete SoC Integration

```verilog
module riscv_soc(
    input wire clk,
    input wire reset,
    // UART
    input wire uart_rx,
    output wire uart_tx,
    // GPIO
    input wire [31:0] gpio_in,
    output wire [31:0] gpio_out,
    // Debug
    output wire [31:0] pc_debug
);
    // CPU signals
    wire [31:0] cpu_instr_addr, cpu_data_addr;
    wire [31:0] cpu_write_data, cpu_read_data;
    wire cpu_read, cpu_write;
    wire cpu_ready;
    
    // Interrupt
    wire interrupt_request;
    wire [7:0] interrupt_id;
    wire interrupt_ack;
    
    // Memory system
    wire [31:0] mem_read_data;
    wire mem_ready;
    // Bus -> memory side (driven by the bus and gated by its address decode;
    // kept separate from the CPU's own request nets to avoid multiple drivers)
    wire [31:0] bus_mem_addr, bus_mem_wdata;
    wire bus_mem_read, bus_mem_write;
    
    // Peripheral data
    wire [31:0] uart_data, gpio_data, timer_data, intc_data;
    wire uart_sel, gpio_sel, timer_sel, intc_sel;
    
    // Interrupt sources
    wire uart_tx_done, uart_rx_ready;
    wire timer_int;
    wire [7:0] interrupt_sources = {5'b0, timer_int, uart_tx_done, uart_rx_ready};
    
    // Instruction memory (loads program.hex); the CPU exposes pc as the fetch address
    wire [31:0] cpu_instruction;
    wire [2:0]  cpu_data_funct3;
    instruction_memory imem(
        .address(cpu_instr_addr),
        .instruction(cpu_instruction)
    );

    // CPU Core = the Chapter-17 pipelined RISC-V (external memory interface)
    riscv_pipelined_with_hazards cpu(
        .clk(clk),
        .reset(reset),
        .cache_stall(1'b0),                 // single-cycle memory: never stalls
        .instruction(cpu_instruction),
        .data_address(cpu_data_addr),
        .data_write(cpu_write_data),
        .data_read_enable(cpu_read),
        .data_write_enable(cpu_write),
        .data_funct3(cpu_data_funct3),
        .data_read(cpu_read_data),
        .pc_debug(cpu_instr_addr)
    );
    assign pc_debug = cpu_instr_addr;
    assign interrupt_ack = 1'b0;            // this core does not service interrupts
    
    // System Bus
    system_bus bus(
        .clk(clk),
        .reset(reset),
        .address(cpu_data_addr),
        .write_data(cpu_write_data),
        .read_enable(cpu_read),
        .write_enable(cpu_write),
        .read_data(cpu_read_data),
        .ready(cpu_ready),
        .mem_address(bus_mem_addr),
        .mem_write_data(bus_mem_wdata),
        .mem_read(bus_mem_read),
        .mem_write(bus_mem_write),
        .mem_read_data(mem_read_data),
        .mem_ready(mem_ready),
        .uart_select(uart_sel),
        .gpio_select(gpio_sel),
        .timer_select(timer_sel),
        .intc_select(intc_sel),
        .uart_read_data(uart_data),
        .gpio_read_data(gpio_data),
        .timer_read_data(timer_data),
        .intc_read_data(intc_data)
    );
    
    // UART
    uart uart_inst(
        .clk(clk),
        .reset(reset),
        .address(cpu_data_addr),
        .write_data(cpu_write_data),
        .write_enable(cpu_write && uart_sel),
        .read_enable(cpu_read && uart_sel),
        .read_data(uart_data),
        .rx(uart_rx),
        .tx(uart_tx),
        .tx_done(uart_tx_done),
        .rx_ready(uart_rx_ready)
    );
    
    // GPIO
    gpio gpio_inst(
        .clk(clk),
        .reset(reset),
        .address(cpu_data_addr),
        .write_data(cpu_write_data),
        .write_enable(cpu_write && gpio_sel),
        .read_enable(cpu_read && gpio_sel),
        .read_data(gpio_data),
        .gpio_in(gpio_in),
        .gpio_out(gpio_out),
        .gpio_dir()
    );
    
    // Timer
    timer timer_inst(
        .clk(clk),
        .reset(reset),
        .address(cpu_data_addr),
        .write_data(cpu_write_data),
        .write_enable(cpu_write && timer_sel),
        .read_enable(cpu_read && timer_sel),
        .read_data(timer_data),
        .timer_interrupt(timer_int)
    );
    
    // Interrupt Controller
    interrupt_controller intc(
        .clk(clk),
        .reset(reset),
        .address(cpu_data_addr),
        .write_data(cpu_write_data),
        .write_enable(cpu_write && intc_sel),
        .read_enable(cpu_read && intc_sel),
        .read_data(intc_data),
        .interrupt_sources(interrupt_sources),
        .interrupt_request(interrupt_request),
        .interrupt_id(interrupt_id),
        .interrupt_ack(interrupt_ack)
    );
    
    // Data memory: single-cycle RAM for the bus's memory region (word accesses)
    data_memory dmem(
        .clk(clk),
        .address(bus_mem_addr),
        .write_data(bus_mem_wdata),
        .mem_write(bus_mem_write),
        .mem_read(bus_mem_read),
        .mem_size(3'b010),
        .read_data(mem_read_data)
    );
    assign mem_ready = 1'b1;   // combinational RAM is always ready
endmodule
```

---

## ১৯.৭ Software Examples

### Example 1: Hello World via UART

```c
// Memory-mapped I/O addresses
#define UART_DATA   0x10001000
#define UART_STATUS 0x10001004

void uart_putc(char c) {
    volatile unsigned int *uart_data = (unsigned int*)UART_DATA;
    volatile unsigned int *uart_status = (unsigned int*)UART_STATUS;
    
    // Wait while TX is busy (bit1 = tx_busy; bit0 is rx_ready)
    while (*uart_status & 0x2);
    
    // Send character
    *uart_data = c;
}

void uart_puts(const char *s) {
    while (*s) {
        uart_putc(*s++);
    }
}

int main() {
    uart_puts("Hello, World from RISC-V!\n");
    while(1);
    return 0;
}
```

### Assembly Version:

```assembly
.section .text
.global _start

_start:
    # Initialize stack
    li sp, 0x10000
    
    # Print "Hello"
    la a0, hello_msg
    call uart_puts
    
    # Loop forever
loop:
    j loop

uart_puts:
    li t0, 0x10001000  # UART_DATA
    li t1, 0x10001004  # UART_STATUS
print_loop:
    lb t2, 0(a0)       # Load character
    beqz t2, print_done
    
wait_tx:
    lw t3, 0(t1)       # Read UART status
    andi t3, t3, 2     # bit1 = tx_busy
    bnez t3, wait_tx   # spin while transmitting
    
    sw t2, 0(t0)       # Send character
    addi a0, a0, 1
    j print_loop
    
print_done:
    ret

.section .rodata
hello_msg:
    .asciz "Hello, World!\n"
```

### Example 2: Blink LED

```c
#define GPIO_DATA 0x10000000
#define GPIO_DIR  0x10000004

void delay(int cycles) {
    for (int i = 0; i < cycles; i++);
}

int main() {
    volatile unsigned int *gpio_data = (unsigned int*)GPIO_DATA;
    volatile unsigned int *gpio_dir = (unsigned int*)GPIO_DIR;
    
    // Set pin 0 as output
    *gpio_dir = 0x1;
    
    // Blink forever
    while(1) {
        *gpio_data = 0x1;  // LED on
        delay(1000000);
        *gpio_data = 0x0;  // LED off
        delay(1000000);
    }
    
    return 0;
}
```

### Example 3: Timer Interrupt

```c
#define TIMER_COUNTER  0x10002000
#define TIMER_COMPARE  0x10002004
#define TIMER_CONTROL  0x10002008
#define INT_ENABLE     0x10003004

volatile int tick_count = 0;

void timer_isr() {
    tick_count++;
}

int main() {
    volatile unsigned int *timer_compare = (unsigned int*)TIMER_COMPARE;
    volatile unsigned int *timer_control = (unsigned int*)TIMER_CONTROL;
    volatile unsigned int *int_enable = (unsigned int*)INT_ENABLE;
    
    // Configure timer (interrupt every 1000 cycles)
    *timer_compare = 1000;
    
    // Enable timer interrupt
    *int_enable = 0x4;  // Bit 2 = timer
    
    // Start timer
    *timer_control = 0x1;
    
    // Enable global interrupts
    asm("csrsi mstatus, 0x8");
    
    while(1) {
        // Main loop
    }
    
    return 0;
}
```

---

## ১৯.৮ FPGA Deployment

### Tang Nano 9K Pin Mapping:

```verilog
module top(
    input wire clk,      // 27 MHz onboard
    input wire btn_reset,
    // UART
    input wire uart_rx,
    output wire uart_tx,
    // LEDs
    output wire [5:0] led,
    // Buttons
    input wire [1:0] btn
);
    wire reset = !btn_reset;  // Active low button
    wire [31:0] gpio_in, gpio_out;
    
    // Map buttons to GPIO input
    assign gpio_in = {30'b0, btn};
    
    // Map GPIO output to LEDs
    assign led = gpio_out[5:0];
    
    // SoC instance
    riscv_soc soc(
        .clk(clk),
        .reset(reset),
        .uart_rx(uart_rx),
        .uart_tx(uart_tx),
        .gpio_in(gpio_in),
        .gpio_out(gpio_out),
        .pc_debug()
    );
endmodule
```

### Constraints File:

```tcl
# Clock
set_pin_assignment {clk} {LOCATION = H11; IOSTANDARD = LVCMOS33;}

# Reset button
set_pin_assignment {btn_reset} {LOCATION = D16; IOSTANDARD = LVCMOS33;}

# UART
set_pin_assignment {uart_rx} {LOCATION = T13; IOSTANDARD = LVCMOS33;}
set_pin_assignment {uart_tx} {LOCATION = T14; IOSTANDARD = LVCMOS33;}

# LEDs
set_pin_assignment {led[0]} {LOCATION = C14; IOSTANDARD = LVCMOS33;}
set_pin_assignment {led[1]} {LOCATION = D14; IOSTANDARD = LVCMOS33;}
set_pin_assignment {led[2]} {LOCATION = E14; IOSTANDARD = LVCMOS33;}
set_pin_assignment {led[3]} {LOCATION = F14; IOSTANDARD = LVCMOS33;}
set_pin_assignment {led[4]} {LOCATION = G14; IOSTANDARD = LVCMOS33;}
set_pin_assignment {led[5]} {LOCATION = H14; IOSTANDARD = LVCMOS33;}
```

---

## ১৯.৯ Your 2-Week Build Plan

### Week 1: Peripherals

**Day 1-3: UART**
```
□ Baud rate generator
□ Transmitter
□ Receiver
□ Test with serial terminal
```

**Day 4-5: GPIO**
```
□ Input/output logic
□ Direction control
□ Test with LEDs
```

**Day 6-7: Timer & Interrupts**
```
□ Timer counter
□ Interrupt controller
□ Interrupt handling
```

### Week 2: Integration

**Day 8-10: System Integration**
```
□ Memory-mapped I/O
□ Address decoder
□ Bus arbiter
□ Complete SoC
```

**Day 11-12: Software**
```
□ Hello World
□ LED blink
□ Timer interrupts
□ Test programs
```

**Day 13-14: FPGA Deployment**
```
□ Synthesize
□ Program FPGA
□ Test on hardware
□ Celebrate! 🎉
```

---

## ১৯.১০ Chapter 19 Mission Complete!

### তুমি এখন পারো:

```
✅ Design complete SoCs
✅ Implement UART communication
✅ Create GPIO interfaces
✅ Build interrupt controllers
✅ Memory-mapped I/O design
✅ System integration
✅ Software/hardware co-design
✅ Build REAL COMPUTERS! 🎉
```

### তুমি বানিয়েছো:
```
✅ UART module (serial comm)
✅ GPIO (32 pins)
✅ Timer with interrupts
✅ Interrupt controller
✅ Complete SoC
✅ Working computer system!
✅ A REAL COMPUTER! 🖥️
```

### Stats:
```
Modules: 6 (CPU + 4 peripherals + bus)
Peripherals: UART, GPIO, Timer, IntC
Address space: 256 MB
I/O pins: 32 + UART
Software: C + Assembly
Level: System Architect! 🏆
```

### Next Level Unlocked:
```
→ Chapter 20: Advanced Topics
   তুমি শিখবে: Future of CPUs
   Superscalar, OoO, Vectors!
   
   From complete → ADVANCED!
```

---

## 🎯 Final Project

### Project: Interactive System

**Build Complete Application:**
```
✅ UART terminal interface
✅ LED patterns (GPIO)
✅ Button input
✅ Timer-based game
✅ Interrupt-driven
✅ Deploy to FPGA

Example: Simon Says game
- LEDs show pattern
- Buttons for input
- Timer for timing
- UART for score
- Complete system!
```

---

## 🏆 Achievement Unlocked!

```
Level 19: ✅ COMPLETE - System Builder!
Progress: [███████████████████████████████████████████] 95%

XP Gained: +5000 🎉
Skills: SoC Design, Peripherals, System Integration

Badges Earned:
🥉 Peripheral Designer
🥈 SoC Integrator
🥇 System Architect
🏅 Hardware/Software Co-Design
🎖️ Complete Computer Builder
🏆 Professional System Designer
⭐ WORKING COMPUTER BUILT! ⭐

YOU BUILT A COMPLETE COMPUTER! 🖥️🎊

Next: Chapter 20 - Advanced Topics!
      Future of computing! 🚀
```

---

**[⬅️ Previous: Chapter 18](Chapter_18_Memory_Hierarchy.md)** | **[➡️ Next: Chapter 20](Chapter_20_Advanced_Topics.md)**

---

<div align="center">

**"You built a COMPLETE COMPUTER! From gates to working system! ONE MORE CHAPTER!"**

**"তুমি COMPLETE COMPUTER বানিয়েছো! Gates থেকে working system! আরো এক CHAPTER!"**

Made with ❤️ for builders | বানানোর জন্য ভালোবাসা দিয়ে তৈরি

</div>
