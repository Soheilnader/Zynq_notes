# Zynq Tutorial

**Author:** Soheil Nadernezhad

## Table of Contents

- [Introduction](#introduction)
- [PL Only](#pl-only)
  - [Building a 5-bit Counter Project with LEDs](#building-a-5-bit-counter-project-with-leds)
  - [MicroBlaze Beside a Custom Hardware](#microblaze-beside-a-custom-hardware)
  - [MicroBlaze and GPIO](#microblaze-and-gpio)
- [PS Only](#ps-only)
- [PL - PS](#pl---ps)
- [Others](#others)
  - [Introduction to the Xilinx System Debugger (XSDB) Tool](#introduction-to-the-xilinx-system-debugger-xsdb-tool)
  - [ILA (Integrated Logic Analyzer) -- Native](#ila-integrated-logic-analyzer----native)

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

### Building a 5-bit Counter Project with LEDs

In this project, we implement a 5-bit counter in the Programmable Logic (PL), because this is the number of LEDs available on the Artemis Karia board.

#### Creating the Project

First, run Vivado. To create a new project, select **Create Project**, then click **Next**.

![Creating a project](images/fig1.jpg)

Next, specify the project name and location, then click **Next**.

![Specifying the project name and location](images/fig2.jpg)

In **Project Type**, select **RTL Project**, then click **Next**.

![Selecting the project type](images/fig3.jpg)

In the **Default Part** section, select the target hardware. For this project, the selected part is `xc7z010clg400-2`.

![Selecting the hardware](images/fig4.jpg)

Review the settings and click **Finish**.

![Completing the project creation steps](images/fig5.jpg)

#### Adding the Code and Related Sources to the Project

After creating the project, enter the Vivado environment. To add HDL code, in the **Sources** section, click **Add Sources**.

![List of available sources](images/fig6.jpg)

Enable **Add or create design sources** if needed, then click **Next**.

![Creating a new source](images/fig7.jpg)

Click **Create File**, choose **Verilog**, and specify a file name.

![Specifying the source name and hardware description language](images/fig8.jpg)

If the **Define Module** window appears, click **OK**.

![Define Module window](images/fig9.jpg)

A `.v` Verilog file will appear under **Design Sources**. Double-click it and enter the following code:

```verilog
`timescale 1ns / 1ps

module LED_Counter(input clk, output reg [4:0] LED);
reg [31:0] cnt;
always @(posedge clk) begin
    cnt = cnt + 1;
    if(cnt == 25_000_000) begin
        cnt = 0;
        LED = LED + 1;
    end
end
endmodule
```

Now assign the Verilog signals to the hardware pins. In **Sources**, click the `+` button and choose **Add or create constraints**.

![Creating the constraint file](images/fig10.jpg)

Click **Create File**, choose a name, then click **OK** and **Finish**.

![Entering the constraint file name](images/fig11.jpg)

A `.xdc` file will be created.

![Displaying the list of required files](images/fig12.jpg)

Double-click the file and enter:

```xdc
set_property IOSTANDARD LVCMOS33 [get_ports clk]
set_property PACKAGE_PIN U18 [get_ports clk]

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

![Pin names](images/fig13.jpg)

The **PL clock** is **50 MHz** and connected to pin `U18`.

#### Downloading onto the Hardware

To configure the device, the design must be synthesized, implemented, and converted into a bitstream.

Click **Run Synthesis**.

![Running synthesis](images/fig14.jpg)

After synthesis completes, click **Run Implementation**.

![Starting implementation after synthesis](images/fig15.jpg)

Then click **Generate Bitstream**.

![Generating the bitstream](images/fig16.jpg)

Connect the board through **JTAG**, power it on, and open **Hardware Manager** → **Open Target** → **Auto Connect**.

![Opening Hardware Manager and connecting to the target](images/fig17.jpg)

Once the device is recognized, click **Program Device**.

![Recognized device in Hardware Manager](images/fig18.jpg)

Then click **Program**.

![Programming the device](images/fig19.jpg)

The LEDs should now behave as a 5-bit binary counter.

![5-bit binary counter output on the LEDs](images/fig20.jpg)

---

### MicroBlaze Beside a Custom Hardware

After creating the project, under **IP INTEGRATOR**, click **Create Block Design**.

![Creating a new block design](images/fig26.jpg)

Set a name such as `design_1`, then click **OK**. Click the `+` button, search for **MicroBlaze**, and add it.

![Adding MicroBlaze to the block design](images/fig27.jpg)

Click **Run Block Automation**.

![Running block automation for MicroBlaze](images/fig28.jpg)

Keep the default settings and choose **New External Port** for the clock.

![Configuring block automation settings](images/fig29.jpg)

Vivado will generate the required system components.

![Automatically generated MicroBlaze system components](images/fig30.jpg)

Run block automation again, and for reset choose **New External Port**. Since the reset signal is pulled up to 3.3V, choose **ACTIVE_LOW**.

![Configuring the external reset port](images/fig31.jpg)

Validate the design using **Validate Design** or `F6`.

![Validating the block design](images/fig32.jpg)

#### Adding Custom Hardware

To add custom hardware, go to **Tools** → **Create and Package New IP**.

![Opening the Create and Package New IP tool](images/fig33.jpg)

Click **Next**.

![Starting the IP packaging process](images/fig34.jpg)

Choose **Create a new AXI4 peripheral**.

![Selecting AXI4 peripheral creation](images/fig35.jpg)

Set the IP name and properties. In this example, the name is `myip`.

![Setting the custom IP name and properties](images/fig36.jpg)

In **Add Interfaces**, keep the interface name `S00_AXI`, set interface type to **Lite**, mode to **Slave**, data width to 32 bits, and choose **4 registers**.

![Configuring the AXI interface and registers](images/fig37.jpg)

Choose **Add IP to Repository** and click **Finish**.

![Adding the custom IP to the repository](images/fig38.jpg)

Add the IP to the block design by searching for `myip`.

![Adding the custom IP to the block design](images/fig39.jpg)

Right-click the IP and choose **Edit in IP Packager**.

![Opening the custom IP in the IP Packager](images/fig40.jpg)

Click **OK**.

![Confirming the IP Packager launch](images/fig41.jpg)

Open `myip_v1_0_S00_AXI_Inst` and edit the generated template. Under `// Add user logic here`, add:

![Opening the AXI module source file for modification](images/fig42.jpg)

```verilog
// Add user logic here
wire [31:0] sum_wire;
assign sum_wire = slv_reg0 + slv_reg1;
// User logic ends
```

Then modify the AXI read logic:

```verilog
...
// Implement memory mapped register select and read logic generation
// Slave register read enable is asserted when valid address is available
// and the slave is ready to accept the read address.
assign slv_reg_rden = axi_arready & S_AXI_ARVALID & ~axi_rvalid;
always @(*)
begin
      // Address decoding for reading registers
      case ( axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] )
        2'h0   : reg_data_out <= slv_reg0;
        2'h1   : reg_data_out <= slv_reg1;
        2'h2   : reg_data_out <= sum_wire;
        2'h3   : reg_data_out <= slv_reg3;
        default : reg_data_out <= 0;
      endcase
end
...
```

Re-package the IP.

![Re-packaging the custom IP after modification](images/fig42_1.jpg)

Confirm closing the IP window.

![Confirming closure of the custom IP Vivado window](images/fig42_2.jpg)

Back in the main Vivado window, click **Refresh IP Catalog**.

![Refreshing the IP Catalog](images/fig43.jpg)

In **IP Status**, select all IPs and click **Upgrade Selected**.

![Upgrading the selected IPs](images/fig42_3.jpg)

If **Generate Output Products** appears, click **Skip**.

![Skipping output product generation](images/fig42_4.jpg)

Click **Run Connection Automation** and connect the IP clock to `Clk`.

![Connecting the custom IP clock using automation](images/fig44.jpg)

The completed block design should look like this:

![Complete system block design including MicroBlaze and the custom IP](images/fig44_5.jpg)

In **Address Editor**, you can see the automatically assigned address of the custom hardware.

![Automatically assigned address for the custom hardware](images/fig45.jpg)

Validate the design again.

Now assign top-level pins using a constraint file:

![Opening the Add Sources window](images/fig46.jpg)

![Creating the constraint file](images/fig10.jpg)

![Entering the constraint file name](images/fig11.jpg)

![Displaying the list of required files](images/fig12.jpg)

```xdc
set_property PACKAGE_PIN U19 [get_ports reset_rtl_0]
set_property IOSTANDARD LVCMOS33 [get_ports reset_rtl_0]

set_property PACKAGE_PIN U18 [get_ports Clk]
set_property IOSTANDARD LVCMOS33 [get_ports Clk]
```

Right-click `design_1.bd` and choose **Create HDL Wrapper**.

![Creating an HDL wrapper for the block design](images/fig47.jpg)

Choose **Let Vivado manage wrapper and auto-update**.

![Selecting automatic wrapper generation](images/fig48.jpg)

The wrapper will appear in the sources list.

![Generated HDL wrapper for the block design](images/fig49.jpg)

Run synthesis, implementation, and bitstream generation as before.

Then go to **File** → **Export** → **Export Hardware**.

![Opening the Export Hardware option](images/fig50.jpg)

Check **Include bitstream** and click **OK**.

![Exporting hardware with the bitstream included](images/fig51.jpg)

#### Program

Launch SDK from Vivado.

![Launching SDK from Vivado](images/fig52.jpg)

Create a new application project.

![Creating a new application project in SDK](images/fig53.jpg)

Configure the project as shown and click **Next**.

![Configuring the application project settings](images/fig54.jpg)

Select **Empty Application**.

![Selecting an empty application template](images/fig55.jpg)

Create a new source file named `main.c`.

![Creating a new source file in the project](images/fig56.jpg)

![Creating the `main.c` source file](images/fig57.jpg)

To find memory-mapped addresses, open `xparameters.h`.

![Contents of the `xparameters.h` header file](images/fig58.jpg)

Use the following example software:

```c
#include "xparameters.h"
#include "xil_io.h"
#include "xil_types.h"

int main(){
    u32 sum1, sum2;

    Xil_Out32(XPAR_MYIP_0_S00_AXI_BASEADDR, 7);
    Xil_Out32(XPAR_MYIP_0_S00_AXI_BASEADDR + 4, 5);
    sum1 = Xil_In32(XPAR_MYIP_0_S00_AXI_BASEADDR + 8);

    Xil_Out32(XPAR_MYIP_0_S00_AXI_BASEADDR, 2);
    Xil_Out32(XPAR_MYIP_0_S00_AXI_BASEADDR + 4, 1);
    sum2 = Xil_In32(XPAR_MYIP_0_S00_AXI_BASEADDR + 8);

    while(1);
    return 0;
}
```

This program demonstrates how the MicroBlaze processor communicates with the custom AXI-based IP through memory-mapped registers.

- `xparameters.h` provides the base address.
- `xil_io.h` provides `Xil_Out32` and `Xil_In32`.
- `xil_types.h` provides `u32`.

Test cases:

- `7 + 5 = 12` → stored in `sum1`
- `2 + 1 = 3` → stored in `sum2`

Build the project:

![Building the application project in SDK](images/fig59.jpg)

There are two programming methods:

##### Option 1: Programming the Hardware in Vivado

Program the hardware in Vivado, then in SDK choose **Debug As** → **Launch on Hardware (System Debugger)**.

![Launching the software on hardware from SDK](images/fig66.jpg)

##### Option 2: Using Only SDK

In SDK, right-click the project and choose **Debug As** → **Debug Configurations**.

![Opening the system debugger configuration in SDK](images/fig61.jpg)

Open **Xilinx C/C++ Application (System Debugger)**, configure the **Target Setup**, then click **Debug**.

![Target setup options in the system debugger](images/fig62.jpg)

A warning may appear about the unused Processing System. It can be ignored in this case.

![Warning message related to the unused Processing System](images/fig63.jpg)

Choose **Yes** to switch to the debug environment.

![Switching to the debug environment](images/fig64.jpg)

After running the software, inspect `sum1` and `sum2` in the debugger.

![Debug results showing correct software and hardware operation](images/fig65.jpg)

#### More Complex

A more advanced custom IP can replace the adder with a counter. This design supports internal state, configurable parameters, memory-mapped control registers, and LED output.

##### Register Map

| Offset | Name | Access | Description |
|---|---|---|---|
| `0x00` | `CONTROL` | `R/W` | `bit[0]`: **RESET**; `bit[1]`: **ENABLE**; `bit[2]`: **DIR** (0 = up, 1 = down); `bit[3]`: **LOAD**; `bit[31:4]`: reserved |
| `0x04` | `PERIOD` | `R/W` | Divider period value. Counter updates once every `PERIOD+1` clock cycles. |
| `0x08` | `COUNT` | `R` | Current counter value `cnt`. |
| `0x0C` | `LOAD_VALUE` | `R/W` | Value written into `cnt` when `CONTROL.LOAD` is asserted. |

Create a custom IP named `mycounter` with an AXI Lite slave interface.

In `mycounter_v1_0_S00_AXI.v`, add an LED output port:

```verilog
...
    // Users to add ports here
    output wire [4:0] led_out,
    // User ports ends
...
```

Modify AXI read logic so that `slv_reg2` returns `cnt`:

```verilog
...
      // Address decoding for reading registers
      case ( axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] )
        2'h0   : reg_data_out <= slv_reg0;
        2'h1   : reg_data_out <= slv_reg1;
        2'h2   : reg_data_out <= cnt; 	//Changed
        2'h3   : reg_data_out <= slv_reg3;
        default : reg_data_out <= 0;
      endcase
...
```

Add the main counter logic:

```verilog
...
// Add user logic here
reg [31:0] div_cnt;
reg [31:0] cnt;

always @(posedge S_AXI_ACLK) begin
    if (!S_AXI_ARESETN) begin
        div_cnt <= 32'd0;
        cnt     <= 32'd0;
    end
    else if (slv_reg0[0]) begin
        // software reset
        div_cnt <= 32'd0;
        cnt     <= 32'd0;
    end
    else if (slv_reg0[3]) begin
        // software load
        div_cnt <= 32'd0;
        cnt     <= slv_reg3;
    end
    else if (slv_reg0[1]) begin
        // enable
        if (div_cnt >= slv_reg1) begin
            div_cnt <= 32'd0;

            if (slv_reg0[2] == 1'b0) begin
                // up
                cnt <= cnt + 1'b1;
            end
            else begin
                // down
                cnt <= cnt - 1'b1;
            end
        end
        else begin
            div_cnt <= div_cnt + 1'b1;
        end
    end
end

assign led_out = cnt[4:0];
// User logic ends
...
```

Open `mycounter_v1_0.v` and add the new top-level port:

```verilog
...
    // Users to add ports here
    output wire [4:0] led_out,
    // User ports ends
...
```

Connect the port in the instantiated AXI module:

```verilog
...
) mycounter_v1_0_S00_AXI_inst (
    .led_out(led_out),
    .S_AXI_ACLK(s00_axi_aclk),
...
```

After updating and repackaging the IP, the final block design should look like this:

![Final block design of the counter-based system](images/fig67.jpg)

Use this constraint file:

```xdc
set_property PACKAGE_PIN U19 [get_ports reset_rtl_0]
set_property IOSTANDARD LVCMOS33 [get_ports reset_rtl_0]

set_property PACKAGE_PIN U18 [get_ports Clk]
set_property IOSTANDARD LVCMOS33 [get_ports Clk]

set_property IOSTANDARD LVCMOS33 [get_ports {led_out_0[0]}]
set_property PACKAGE_PIN V20 [get_ports {led_out_0[0]}]

set_property IOSTANDARD LVCMOS33 [get_ports {led_out_0[1]}]
set_property PACKAGE_PIN W19 [get_ports {led_out_0[1]}]

set_property IOSTANDARD LVCMOS33 [get_ports {led_out_0[2]}]
set_property PACKAGE_PIN W18 [get_ports {led_out_0[2]}]

set_property IOSTANDARD LVCMOS33 [get_ports {led_out_0[3]}]
set_property PACKAGE_PIN V18 [get_ports {led_out_0[3]}]

set_property IOSTANDARD LVCMOS33 [get_ports {led_out_0[4]}]
set_property PACKAGE_PIN V17 [get_ports {led_out_0[4]}]
```

Use the following C code in SDK:

```c
#include "xparameters.h"
#include "xil_io.h"
#include "xil_types.h"

#define BASE XPAR_MYCOUNTER_0_S00_AXI_BASEADDR

#define CTRL_RESET  1
#define CTRL_ENABLE 2
#define CTRL_DIR    4
#define CTRL_LOAD   8

void set_period(u32 p)      { Xil_Out32(BASE + 4, p); }
u32  read_counter(void)     { return Xil_In32(BASE + 8); }

void reset_counter(void)
{
    u32 c = Xil_In32(BASE + 0);
    Xil_Out32(BASE + 0, c | CTRL_RESET);
    Xil_Out32(BASE + 0, c & ~CTRL_RESET);
}

void load_counter(u32 v)
{
    u32 c = Xil_In32(BASE + 0);
    Xil_Out32(BASE + 12, v);
    Xil_Out32(BASE + 0, c | CTRL_LOAD);
    Xil_Out32(BASE + 0, c & ~CTRL_LOAD);
}

void set_up(void)           { Xil_Out32(BASE + 0, Xil_In32(BASE + 0) & ~CTRL_DIR); }
void start_counter(void)    { Xil_Out32(BASE + 0, Xil_In32(BASE + 0) |  CTRL_ENABLE); }
void stop_counter(void)     { Xil_Out32(BASE + 0, Xil_In32(BASE + 0) & ~CTRL_ENABLE); }

int main(void)
{
    u32 c0, c1;

    reset_counter();
    set_period(25000000);
    load_counter(2);
    set_up();
    start_counter();

    c0 = read_counter();
    c1 = read_counter();

    while (1);
    return 0;
}
```

This software shows how MicroBlaze controls the counter through AXI memory-mapped registers.

---

### MicroBlaze and GPIO

In this example, we use 3 input signals so that each one turns on its corresponding output LED.

After adding MicroBlaze and connecting clock and reset, add **AXI GPIO** IP to the block design.

![Adding AXI GPIO IPs to the block design](images/fig68.jpg)

Use two GPIO IPs: one input-only and one output-only.

Configure one GPIO with **All Inputs**, width **3**, and disable dual channel. Configure the other with **All Outputs**.

![Configuring AXI GPIO properties](images/fig69.jpg)

For each GPIO, expand the GPIO port and select **Make External**.

![Making the GPIO ports external](images/fig70.jpg)

Final block design:

![Final block design for the MicroBlaze and GPIO system](images/fig71.jpg)

Assign constraints as follows:

![Board pin connections for the input buttons](images/fig72.jpg)

```xdc
set_property PACKAGE_PIN U19 [get_ports reset_rtl_0]
set_property IOSTANDARD LVCMOS33 [get_ports reset_rtl_0]

set_property PACKAGE_PIN U18 [get_ports Clk]
set_property IOSTANDARD LVCMOS33 [get_ports Clk]

set_property IOSTANDARD LVCMOS33 [get_ports {gpio_io_o_0[0]}]
set_property PACKAGE_PIN V20 [get_ports {gpio_io_o_0[0]}]

set_property IOSTANDARD LVCMOS33 [get_ports {gpio_io_o_0[1]}]
set_property PACKAGE_PIN W19 [get_ports {gpio_io_o_0[1]}]

set_property IOSTANDARD LVCMOS33 [get_ports {gpio_io_o_0[2]}]
set_property PACKAGE_PIN W18 [get_ports {gpio_io_o_0[2]}]

set_property IOSTANDARD LVCMOS33 [get_ports {gpio_io_i_0[0]}]
set_property PACKAGE_PIN T10 [get_ports {gpio_io_i_0[0]}]

set_property IOSTANDARD LVCMOS33 [get_ports {gpio_io_i_0[1]}]
set_property PACKAGE_PIN T11 [get_ports {gpio_io_i_0[1]}]

set_property IOSTANDARD LVCMOS33 [get_ports {gpio_io_i_0[2]}]
set_property PACKAGE_PIN T15 [get_ports {gpio_io_i_0[2]}]
```

After generating the wrapper, bitstream, and exported hardware, use this C code:

```c
#include "xparameters.h"
#include "xgpio.h"

#define LED_GPIO_ID XPAR_AXI_GPIO_1_DEVICE_ID
#define BTN_GPIO_ID XPAR_AXI_GPIO_0_DEVICE_ID

XGpio LedGpio;
XGpio BtnGpio;

int main(void)
{
    u32 btn;
    u32 led;

    XGpio_Initialize(&LedGpio, LED_GPIO_ID);
    XGpio_Initialize(&BtnGpio, BTN_GPIO_ID);

    XGpio_SetDataDirection(&LedGpio, 1, 0x0); // LEDs = output
    XGpio_SetDataDirection(&BtnGpio, 1, 0x7); // buttons = input

    while (1)
    {
        btn = XGpio_DiscreteRead(&BtnGpio, 1);
        led = 0;

        if ((btn & 0x1) == 0) led |= 0x1; // button 1 -> LED 1
        if ((btn & 0x2) == 0) led |= 0x2; // button 2 -> LED 2
        if ((btn & 0x4) == 0) led |= 0x4; // button 3 -> LED 3

        XGpio_DiscreteWrite(&LedGpio, 1, led);
    }

    return 0;
}
```

This example demonstrates a simple polling-based GPIO system using MicroBlaze.

---

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

