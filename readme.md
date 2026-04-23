# Zynq Tutorial

**Author:** Soheil Nadernezhad

## Table of Contents
- [Introduction](#introduction)
- [PL Only](./pl-only/README.md)
  - [Building a 5-bit Counter Project with LEDs](./pl-only/5-bit-counter/README.md)
  - [MicroBlaze Beside a Custom Hardware](./pl-only/microblaze-custom-hardware/README.md)
  - [MicroBlaze and GPIO](./pl-only/microblaze-gpio/README.md)
- [PS Only](./ps-only/README.md)
- [PL - PS](./pl-ps/README.md)
- [Others](./others/README.md)
  - [Introduction to the Xilinx System Debugger (XSDB) Tool](./others/xsdb/README.md)
  - [ILA (Integrated Logic Analyzer) -- Native](./others/ila-native/README.md)
---

## Introduction

This tutorial explains several practical Zynq and Vivado workflows, including:

- Pure PL projects
- MicroBlaze systems with custom hardware
- MicroBlaze with AXI GPIO
- XSDB usage
- Native ILA debugging

<!-- > Put all referenced images inside an `images/` folder in your repository, and update each image filename if needed.

--- -->

## PL Only




## PS Only

_To be completed._

---

## PL - PS

_To be completed._

---

## Others

### Introduction to the Xilinx System Debugger (XSDB) Tool

XSDB can be used to verify the hardware connection between the board and the computer. Run `xsdb.bat`.

![Running the XSDB tool](images/fig21.jpg)

Then enter commands in the XSDB shell.

![XSDB command window](images/fig22.jpg)

Use `help` to view available commands.

![Help command in XSDB](images/fig23.jpg)

To communicate with the board over JTAG, use `connect`, then `targets`.

![Connecting to the board and displaying available targets](images/fig24.jpg)

Example target selection:

```bash
xsdb% target 2
```

Read memory using:

```bash
xsdb% mrd address
```

Write memory using:

```bash
xsdb% mwr address value
```

Example results:

![Results of memory read and write operations in XSDB](images/fig25.jpg)

For example, address `0x00001000` may initially contain `0x00000000`. After writing `0xabcdef12`, reading again confirms the change.

---

### ILA (Integrated Logic Analyzer) -- Native

In this section, the ILA is instantiated directly in an RTL design.

Initial design:

```verilog
`timescale 1ns / 1ps

module LED_Counter(
    input  wire clk,
    input  wire rst,
    output reg  [4:0] LED
);

reg [31:0] cnt;

always @(posedge clk) begin
    if (!rst) begin
        cnt <= 32'd0;
        LED <= 5'd0;
    end
    else begin
        if (cnt >= 32'd25_000_000 - 1) begin
            cnt <= 32'd0;
            LED <= LED + 1'b1;
        end
        else begin
            cnt <= cnt + 1'b1;
        end
    end
end

endmodule
```

Open **IP Catalog** and search for **ILA**.

![Searching for the ILA IP in the Vivado IP Catalog](images/fig74.jpg)

In **Customize IP**, choose **Native**, set number of probes to **3**, and choose a sample depth such as **1024**.

![Configuring the ILA IP in Native mode](images/fig75.jpg)

Set probe widths to match `rst`, `cnt`, and `LED`.

![Setting the widths of the ILA probes](images/fig76.jpg)

Confirm generation of the ILA IP.

![Generating the customized ILA IP core](images/fig77.jpg)

If **Generate Output Products** appears, click **Skip**.

![Skipping output product generation for the ILA IP](images/fig78.jpg)

Open the instantiation template in `ila_0.veo`.

![Opening the ILA instantiation template](images/fig79.jpg)

Template:

```verilog
//----------- Begin Cut here for INSTANTIATION Template ---// INST_TAG

ila_0 your_instance_name (
    .clk(clk), // input wire clk

    .probe0(probe0), // input wire [0:0]  probe0
    .probe1(probe1), // input wire [31:0]  probe1
    .probe2(probe2)  // input wire [4:0]  probe2
);

// INST_TAG_END ------ End INSTANTIATION Template ---------
```

Final design with the ILA instantiated:

```verilog
`timescale 1ns / 1ps
module LED_Counter(
    input  wire clk,
    input  wire rst,
    output reg  [4:0] LED
);
reg [31:0] cnt;
always @(posedge clk) begin
    if (!rst) begin
        cnt <= 32'd0;
        LED <= 5'd0;
    end
    else begin
        if (cnt >= 32'd25_000_000 - 1) begin
            cnt <= 32'd0;
            LED <= LED + 1'b1;
        end
        else begin
            cnt <= cnt + 1'b1;
        end
    end
end

ila_0 myila (
    .clk(clk),
    .probe0(rst),
    .probe1(cnt),
    .probe2(LED)
);
endmodule
```

Constraint file:

```xdc
set_property IOSTANDARD LVCMOS33 [get_ports clk]
set_property PACKAGE_PIN U18 [get_ports clk]

set_property IOSTANDARD LVCMOS33 [get_ports {rst}]
set_property PACKAGE_PIN T10 [get_ports {rst}]

set_property IOSTANDARD LVCMOS33 [get_ports {LED[0]}]
set_property PACKAGE_PIN V20 [get_ports {LED[0]}]

set_property IOSTANDARD LVCMOS33 [get_ports {LED[1]}]
set_property PACKAGE_PIN W19 [get_ports {LED[1]}]

set_property IOSTANDARD LVCMOS33 [get_ports {LED[2]}]
set_property PACKAGE_PIN W18 [get_ports {LED[2]}]

set_property IOSTANDARD LVCMOS33 [get_ports {LED[3]}]
set_property PACKAGE_PIN V18 [get_ports {LED[3]}]

set_property IOSTANDARD LVCMOS33 [get_ports {LED[4]}]
set_property PACKAGE_PIN V17 [get_ports {LED[4]}]
```

Program the FPGA:

![Programming the FPGA with the design containing the ILA](images/fig80.jpg)

The ILA window opens automatically.

![ILA window after programming the FPGA](images/fig81.jpg)

Set trigger conditions, for example on the active-low reset signal.

![Captured waveform in the ILA after the trigger condition is met](images/fig82.jpg)

If using multiple trigger conditions, set **Trigger Condition** to **Global OR**.

![Setting multiple trigger conditions using Global OR](images/fig83.jpg)

You can also capture waveforms immediately without setting any trigger condition by choosing **Run Trigger Immediate for the ILA core**.

Example C snippet:

```c
#include <stdio.h>

int main() {
    int counter = 0;
    for (int i = 0; i < 10; i++) {
        counter++;
    }
    printf("Counter = %d\n", counter);
    return 0;
}
```

