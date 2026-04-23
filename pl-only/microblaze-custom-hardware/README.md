## MicroBlaze Beside a Custom Hardware

After creating the project, in the left menu under the **IP INTEGRATOR** section, click on **Create Block Design**.

![Creating a new block design](../../images/fig26.jpg)

In the displayed window, after setting a name such as `design_1`, click the **OK** button to create an empty block design. In this window, click the **+** button and search for **MicroBlaze** to add it to the block design. For now, we leave the MicroBlaze settings at their default values.

![Adding MicroBlaze to the block design](../../images/fig27.jpg)

After adding MicroBlaze, click **Run Block Automation** at the top of the block design window.

![Running block automation for MicroBlaze](../../images/fig28.jpg)

In the displayed window, leave the parameters at their default values and, for the clock connection, choose the **New External Port** option.

![Configuring block automation settings](../../images/fig29.jpg)

As shown, several blocks will appear, including local memory, a debug module, and other required components.

![Automatically generated MicroBlaze system components](../../images/fig30.jpg)

Again, click **Run Block Automation** at the top of the block design window. In the following window, for the reset connection, choose **New External Port**. According to the development board datasheet, the reset signal is pulled up to the 3.3\,V power source, so the reset type should be selected as **ACTIVE_LOW** in this case.

![Configuring the external reset port](../../images/fig31.jpg)

After this step, as shown, in addition to the clock port that was set as an external port, the system reset signal (`reset_rtl_0`) is also set as an external port. Then, click the **Validate Design** button or press the shortcut key **F6** to validate the design and check whether any errors occur.

![Validating the block design](../../images/fig32.jpg)

### Adding Custom Hardware

In this case, we want to add our custom hardware to the design. For simplicity, we will create a simple adder as our custom hardware. For this purpose, from the top menu bar, go to **Tools** and select **Create and Package New IP**.

![Opening the Create and Package New IP tool](../../images/fig33.jpg)

In the displayed tab titled **Create and Package New IP**, click **Next** to begin the process.

![Starting the IP packaging process](../../images/fig34.jpg)

Because we are using the MicroBlaze soft-core processor, which uses AXI interconnect as the standard interconnection protocol, in order to connect our custom hardware to MicroBlaze, we must choose the **Create a new AXI4 peripheral** option.

![Selecting AXI4 peripheral creation](../../images/fig35.jpg)

Here, we can modify the details of our custom hardware, such as its name and version. For this example, we leave `myip` as the name and keep the other options unchanged.

![Setting the custom IP name and properties](../../images/fig36.jpg)

In the **Add Interfaces** step, we should define the interconnect through which our custom hardware communicates with other peripherals or hardware modules, especially the processor. For this purpose, we keep the name of this interconnect port as `S00_AXI` for our custom IP. We choose the interface type and mode as **Lite** and **Slave**, respectively, with a 32-bit data width. The important option here is the number of registers, because these registers play a key role in sharing data between MicroBlaze and the custom hardware. Here, we choose 4 registers because we are implementing an adder, and we can map the two input ports and the output to three separate registers. One register will remain unused.

![Configuring the AXI interface and registers](../../images/fig37.jpg)

Finally, we choose **Add IP to Repository** and click the **Finish** button.

![Adding the custom IP to the repository](../../images/fig38.jpg)

Now we can add the created custom IP to the block design. To do this, just like adding other IPs, we press the **+** button and search for our custom hardware name, which is `myip`.

![Adding the custom IP to the block design](../../images/fig39.jpg)

In order to describe our custom IP, right-click on the added IP in the block design window and choose **Edit in IP Packager**.

![Opening the custom IP in the IP Packager](../../images/fig40.jpg)

In the following window, click **OK**.

![Confirming the IP Packager launch](../../images/fig41.jpg)

A new Vivado window will appear. This new window is related to the custom hardware. In the **Sources** section, under the **Hierarchy** tab, double-click on `myip_v1_0_S00_AXI_Inst` to modify it.

![Opening the AXI module source file for modification](../../images/fig42.jpg)

As a result, the generated template code will be displayed, which already contains the AXI interconnect logic. To define our own circuit, scroll down until you see the comment `// Add user logic here`, and write the required logic in that section.

Since we previously defined 4 registers, we can access these registers to read from or write to them. For example, if `sum_wire = a + b`, we can map `a` to `slv_reg0`, `b` to `slv_reg1`, and `sum_wire` to `slv_reg2`. Therefore, our code will be as shown below.

```verilog
// Add user logic here
wire [31:0] sum_wire;
assign sum_wire = slv_reg0 + slv_reg1;
// User logic ends
```

Another required change is in the register read logic. In the following code section, we should replace `slv_reg2` with `sum_wire`, as shown in line 12 below.

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

Finally, to save the changes made to the custom IP, go to the **Package IP - myip** tab. In the **Review and Package** section, click **Re-package IP** to save the changes. Note that if you make other changes, such as adding ports to the Verilog module, you should also apply the corresponding updates in the **Ports and Interfaces** section.

![Re-packaging the custom IP after modification](../../images/fig42_1.jpg)

In the displayed window, click **Yes** to close the Vivado window related to the custom IP and return to the main Vivado window.

![Confirming closure of the custom IP Vivado window](../../images/fig42_2.jpg)

After closing the custom IP Vivado window and returning to the main Vivado window, click on **Refresh IP Catalog** at the top of the window.

![Refreshing the IP Catalog](../../images/fig43.jpg)

After that, in the **IP Status** section at the bottom of the window, select all IPs and click **Upgrade Selected**.

![Upgrading the selected IPs](../../images/fig42_3.jpg)

If a window titled **Generate Output Products** appears, click **Skip**.

![Skipping output product generation](../../images/fig42_4.jpg)

Then click **Run Connection Automation**, which opens a window where we should assign the clock of our custom IP to the clock port that was created previously (`Clk`).

![Connecting the custom IP clock using automation](../../images/fig44.jpg)

The complete system block design should be as shown below.

![Complete system block design including MicroBlaze and the custom IP](../../images/fig44_5.jpg)

In the **Address Editor** tab, next to the **Diagram** tab, we can see that our custom hardware has been added and that its address has been assigned automatically so that it can be accessed according to the memory-mapped system structure.

![Automatically assigned address for the custom hardware](../../images/fig45.jpg)

Again, by clicking **Validate Design** in the **Diagram** tab, we can check whether the whole design is valid and free of errors.

Now it is necessary to assign the top-level signals to the hardware pins. For this purpose, from the **Sources** section, we click the **+** button. In the opened window, we select the first option, **Add or create constraints**, and by clicking **Next**, we move to the next step.

![Opening the Add Sources window](../../images/fig46.jpg)

![Creating the constraint file](../../images/fig10.jpg)

In the opened window, we first click the **Create File** button, then choose a name for the file, and by selecting **OK** and then **Finish**, the file is created.

![Entering the constraint file name](../../images/fig11.jpg)

It can be seen that a file with the `.xdc` format has been created.

![Displaying the list of required files](../../images/fig12.jpg)

By double-clicking on this file, we can edit it. In the following code, we assign the top-level ports to the corresponding board pins using the `PACKAGE_PIN` parameter and set the I/O standard to **LVCMOS33**.

```xdc
set_property PACKAGE_PIN U19 [get_ports reset_rtl_0]
set_property IOSTANDARD LVCMOS33 [get_ports reset_rtl_0]

set_property PACKAGE_PIN U18 [get_ports Clk]
set_property IOSTANDARD LVCMOS33 [get_ports Clk]
```

In the **Sources** section, right-click on `design_1.bd` and select **Create HDL Wrapper**. Then select **Let Vivado manage wrapper and auto-update** and click **OK**.

![Creating an HDL wrapper for the block design](../../images/fig47.jpg)

![Selecting automatic wrapper generation](../../images/fig48.jpg)

After that, you can see that a wrapper has been created for the block design.

![Generated HDL wrapper for the block design](../../images/fig49.jpg)

Now we discuss how to load the written design onto the device and configure it. For this, the design must first be synthesized (**Synthesis**), implemented (**Implementation**), and then its **Bitstream** must be generated and loaded onto the device.

For this purpose, as shown below, we click **Run Synthesis** and wait until the synthesis process is completed.

![Running synthesis](../../images/fig14.jpg)

Then the next window appears, indicating the completion of the synthesis stage. Now, by selecting **Run Implementation** and clicking **OK**, the implementation process begins, and again we must wait until it is completed.

![Starting implementation after synthesis](../../images/fig15.jpg)

After the following window appears, we select **Generate Bitstream**, and by clicking **OK**, the bitstream generation process begins.

![Generating the bitstream](../../images/fig16.jpg)

From the top menu bar, go to **File**, then **Export**, and click **Export Hardware**.

![Opening the Export Hardware option](../../images/fig50.jpg)

In the displayed window, check **Include bitstream** and click **OK**.

![Exporting hardware with the bitstream included](../../images/fig51.jpg)

### Program

From the top menu bar in Vivado, go to **File** and click **Launch SDK**.

![Launching SDK from Vivado](../../images/fig52.jpg)

From the top menu bar in SDK, go to **File**, then **New**, and click **Application Project**.

![Creating a new application project in SDK](../../images/fig53.jpg)

Set a name for the project and check that the other options are the same as shown below, then click **Next**.

![Configuring the application project settings](../../images/fig54.jpg)

Choose **Empty Application** and click **Finish** to complete the project creation process.

![Selecting an empty application template](../../images/fig55.jpg)

In the **Project Explorer**, find the project that we just created, go to the **src** folder, and right-click on it. Then go to **New** and click **File** in order to create a new source file.

![Creating a new source file in the project](../../images/fig56.jpg)

In the displayed window, enter the file name `main.c` and click **Finish**.

![Creating the `main.c` source file](../../images/fig57.jpg)

As mentioned before, we are working with a memory-mapped system, so it is important to know and use the peripheral addresses. To access this information, there is a header file called `xparameters.h`, which contains the required definitions extracted from Vivado. This file can be accessed through `PROJECTNAME_bsp -> microblaze_0 -> include`. Some of the contents of this header file are shown below.

![Contents of the `xparameters.h` header file](../../images/fig58.jpg)

Back in `main.c`, an example C program for the MicroBlaze processor is shown below.

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

This program demonstrates how the MicroBlaze processor communicates with the custom AXI-based IP through memory-mapped registers. The header file `xparameters.h` provides the base address of the custom peripheral, while `xil_io.h` provides the `Xil_Out32` and `Xil_In32` functions for writing to and reading from hardware registers. The data type `u32`, defined in `xil_types.h`, is used for 32-bit unsigned values.

In the first test, the values `7` and `5` are written into the first two registers of the custom IP. These correspond to the two inputs of the adder hardware. Then, the result is read from the third register and stored in `sum1`. Since the custom hardware performs addition, the expected result is `12`.

In the second test, the values `2` and `1` are written to the same input registers, and the result is again read from the output register and stored in `sum2`. In this case, the expected result is `3`.

The address offsets are important here. The base address `XPAR_MYIP_0_S00_AXI_BASEADDR` refers to the first register. Adding an offset of `4` accesses the second 32-bit register, and adding an offset of `8` accesses the third 32-bit register. Since each register is 32 bits wide, each one occupies 4 bytes in the memory map.

Finally, the statement `while(1);` keeps the program running indefinitely so that the computed values remain available during debugging. This allows the user to inspect `sum1` and `sum2` in the debugger and verify that the software and custom hardware are working correctly together.

Then, we build the project, as shown below.

![Building the application project in SDK](../../images/fig59.jpg)

According to our design, there are two things that must be programmed onto the chip. One is the **bitstream**, which contains the complete hardware system including MicroBlaze, custom hardware, memory, and other components. The other is the **software program**, which is executed by the MicroBlaze soft-core processor.

There are two ways to do this:

#### Option 1: Programming the Hardware in Vivado

First, program the hardware in Vivado. Then, return to the SDK, right-click on the project, go to **Debug As**, and click **Launch on Hardware (System Debugger)**, as shown below.

![Launching the software on hardware from SDK](../../images/fig66.jpg)

#### Option 2: Using Only SDK

Alternatively, using only the SDK, right-click on the project, go to **Debug As**, and click **Debug Configurations**. Then double-click on **Xilinx C/C++ Application (System Debugger)**, as shown below.

![Opening the system debugger configuration in SDK](../../images/fig61.jpg)

In the opened window, under the **Target Setup** tab, choose the options exactly as shown below and then click Debug.

![Target setup options in the system debugger](../../images/fig62.jpg)

After performing either Option 1 or Option 2, a warning message may appear. This warning is not important in this case, because we are not using the Processing System (**PS**) at this stage, as shown below.

![Warning message related to the unused Processing System](../../images/fig63.jpg)

After the programming process is completed, a new message appears asking whether we want to switch to the debug environment. By choosing **Yes**, we enter the debug environment.

![Switching to the debug environment](../../images/fig64.jpg)

After running the software completely, or step by step using the debug tools, we can observe that both the software and hardware behave as expected, as shown by the values of `sum1` and `sum2` below.

![Debug results showing correct software and hardware operation](../../images/fig65.jpg)

### More Complex

For a more advanced custom IP design, the simple adder can be replaced with a counter. This is beneficial because a counter includes internal state, and several configurable parameters, making it a more realistic example of a hardware peripheral. Unlike the adder, which performs only a simple combinational function, the counter demonstrates how a custom IP can support memory-mapped control registers, clock-driven operation, and multiple modes such as reset, enable, direction control, and value loading. In addition, an important feature of this design is that the 5 least significant bits (LSBs) of the counter value are displayed on the on-board LEDs. Therefore, besides the AXI interface, the custom hardware must also provide an additional output port connected directly to the LEDs.

The register map of the counter-based custom IP is shown below. This table describes the address offsets, register names, access types, and the function of each register used to control and monitor the counter peripheral.

| **Offset** | **Name** | **Access** | **Description** |
|---|---|---|---|
| `0x00` | `CONTROL` | `R/W` | `bit[0]`: **RESET**<br>`bit[1]`: **ENABLE**<br>`bit[2]`: **DIR** (0 = up, 1 = down)<br>`bit[3]`: **LOAD**<br>`bit[31:4]`: Reserved |
| `0x04` | `PERIOD` | `R/W` | Divider period value. The counter updates once every `PERIOD+1` clock cycles. |
| `0x08` | `COUNT` | `R` | Current value of the visible counter `cnt`. |
| `0x0C` | `LOAD_VALUE` | `R/W` | Value written into `cnt` when `CONTROL.LOAD` is asserted. |

The steps are similar to the previous example, so here we focus only on the required code changes. After adding MicroBlaze and the other required components to the block design, we create a custom IP called `mycounter` with an AXI Lite slave interface. Then, we go to the source file `mycounter_v1_0_S00_AXI.v`. In the module ports, we add a 5-bit output wire as shown below.

```verilog
...
    // Users to add ports here
    output wire [4:0] led_out,
    // User ports ends
...
```

Further in the code, in the AXI read logic, we replace `slv_reg2` with `cnt`, so that the processor reads the current counter value instead of a standard slave register, as shown below.

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

Finally, we add the main counter logic in the appropriate user logic section, as shown below.

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

In this design, `slv_reg0` is used as the control register. Its bits determine whether the counter is reset, enabled, loaded with a new value, or operated in the up-counting or down-counting mode. The register `slv_reg1` stores the divider period, which controls how often the visible counter value is updated. The register `slv_reg3` is used as the load value, and the internal register `cnt` stores the current counter value. In addition, the signal `led_out` is assigned to the 5 least significant bits of `cnt`, so the current counter value can be observed directly on the board LEDs.

Because we are adding a new port, we must also modify the top-level module. For this purpose, open the top-level Verilog source file, which is usually named `mycounter_v1_0.v`. In the port section, add the new output port in an appropriate place, as shown below.

```verilog
...
    // Users to add ports here
    output wire [4:0] led_out,
    // User ports ends
...
```

We also need to modify the instantiated AXI module further below in the same file and connect the new port, as shown below.

```verilog
...
) mycounter_v1_0_S00_AXI_inst (
    .led_out(led_out),
    .S_AXI_ACLK(s00_axi_aclk),
...
```

After updating, upgrading, and re-packaging the custom IP, the final block design should be as shown below.

![Final block design of the counter-based system](../../images/fig67.jpg)

We should also add a constraint file and assign the external pins for the clock, reset, and LED outputs, as shown below.

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

After finishing the hardware design, generating the bitstream, and exporting the hardware, we can open the SDK, create a software project, and use the following C code on the MicroBlaze processor, as shown below.

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

This software demonstrates how the MicroBlaze processor controls the custom counter IP through its memory-mapped AXI registers. The base address of the peripheral is defined by `XPAR_MYCOUNTER_0_S00_AXI_BASEADDR`, which is provided in the `xparameters.h` header file. The software uses `Xil_Out32` to write values into the counter registers and `Xil_In32` to read values back.

The control register is located at offset `0x00`. Several bit masks are defined for this register: `CTRL_RESET` resets the counter, `CTRL_ENABLE` enables counting, `CTRL_DIR` selects the counting direction, and `CTRL_LOAD` loads a predefined value into the counter. The function `set_period()` writes the divider period to offset `0x04`, while `read_counter()` reads the current counter value from offset `0x08`.

The function `reset_counter()` performs a software reset by setting and then clearing the reset bit in the control register. Similarly, `load_counter()` writes a value to the load register at offset `0x0C`, then pulses the load bit so that the counter is initialized with the requested value. The functions `set_up()`, `start_counter()`, and `stop_counter()` configure the counter direction and control whether the counting process is active.

In the `main()` function, the counter is first reset, then its update period is set to `25000000`. After that, the value `2` is loaded into the counter, the counting direction is set to up-counting, and the counter is started. The variables `c0` and `c1` store values read from the counter and can be inspected in the debugger. At the same time, the 5 least significant bits of the counter value are continuously driven to the board LEDs through the `led_out` port, allowing the counter operation to be observed directly in hardware.

## MicroBlaze and GPIO

In this section, we want to work with input and output using the MicroBlaze soft-core processor. In our scenario, we use 3 input signals so that each one turns on its corresponding output LED.

After adding MicroBlaze and connecting the corresponding clock and reset signals in the block design, search for **AXI GPIO** and add it to the design block. For simplicity, we use two AXI GPIO IPs: one configured as input only and the other configured as output only. It is also possible to use a single AXI GPIO IP for both input and output.

![Adding AXI GPIO IPs to the block design](../../images/fig68.jpg)

Double-click on one of them to open its properties and activate the **All Inputs** checkbox. Set the GPIO width to 3 bits and make sure that **Enable Dual Channel** is unchecked. Do the same for the other AXI GPIO, but this time choose **All Outputs**.

![Configuring AXI GPIO properties](../../images/fig69.jpg)

For each AXI GPIO, expand the GPIO port in the block design, select the expanded port, right-click on it, and choose **Make External**. By doing this, these AXI GPIO ports are connected to the real device I/O pins.

![Making the GPIO ports external](../../images/fig70.jpg)

Therefore, the final block design should be as shown below.

![Final block design for the MicroBlaze and GPIO system](../../images/fig71.jpg)

Now it is time to assign constraints for the external ports and signals. Since the push-buttons on our development board are connected as shown below, the constraint file should be written as shown below.

![Board pin connections for the input buttons](../../images/fig72.jpg)

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

After creating an HDL wrapper by right-clicking on the block design in the **Sources** section and choosing **Create HDL Wrapper**, we synthesize, implement, and generate the bitstream for the whole design, and finally export the hardware. Then we open the SDK, create a software project, and use the following C code on the MicroBlaze processor, as shown below.

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

The C program above shows how the MicroBlaze processor communicates with external inputs and outputs through two AXI GPIO peripherals. One AXI GPIO IP is used for the LEDs and the other is used for the buttons. In this design, the processor continuously reads the state of the push-buttons and then updates the LEDs so that each pressed button turns on its corresponding LED.

The header file `xparameters.h` contains hardware-dependent constants automatically generated from the Vivado design. These constants include the device IDs of the AXI GPIO peripherals, which are required in software to identify the correct hardware blocks. The header file `xgpio.h` provides the GPIO driver and the related functions used to initialize, configure, read, and write the GPIO modules.

The following definitions assign readable names to the two GPIO peripherals: `LED_GPIO_ID` refers to the AXI GPIO connected to the LEDs, and `BTN_GPIO_ID` refers to the AXI GPIO connected to the buttons. Two variables of type `XGpio`, namely `LedGpio` and `BtnGpio`, are then declared. These variables are software driver instances that represent the two AXI GPIO hardware modules inside the program.

Inside the `main()` function, the variables `btn` and `led` are declared as 32-bit unsigned integers. The variable `btn` stores the value read from the button inputs, while `led` stores the value that will be written to the LED outputs.

The function `XGpio_Initialize()` is used first for both GPIO instances. This function connects each software GPIO object to its corresponding hardware peripheral using the device ID defined in `xparameters.h`. Without this initialization step, the software would not be able to access the GPIO hardware correctly.

After initialization, the direction of each GPIO channel is configured using `XGpio_SetDataDirection()`. This is one of the most important I/O-related functions in the code. In the AXI GPIO driver, a direction bit value of `1` means input, while a direction bit value of `0` means output. Therefore, `0x0` configures all bits of the LED GPIO as outputs, and `0x7` configures the lowest three bits of the button GPIO as inputs. Since the design uses a single channel for each GPIO, channel number 1 is used in both cases.

After the GPIO configuration is completed, the program enters an infinite `while(1)` loop. This means the processor continuously polls the buttons and updates the LEDs in real time. No interrupt mechanism is used here; instead, the processor repeatedly checks the input values by software.

The function `XGpio_DiscreteRead()` reads the current logic value of the selected GPIO channel. In this code, it reads the state of the three push-buttons from the input AXI GPIO and stores the result in the variable `btn`. After that, the variable `led` is reset to zero so that all LED bits are initially turned off before checking the input conditions.

The three `if` statements then examine the three least significant bits of `btn`. Each statement checks one button by using a bitwise AND operation. If the corresponding button is pressed, the related LED bit is set in the variable `led`. An important detail is that the buttons are active-low. This means that when a button is pressed, its input value becomes logic 0, not logic 1. For this reason, the program checks whether each masked bit is equal to zero. If bit 0 is zero, LED 0 is turned on; if bit 1 is zero, LED 1 is turned on; and if bit 2 is zero, LED 2 is turned on.

Finally, the function `XGpio_DiscreteWrite()` writes the value stored in `led` to the output AXI GPIO connected to the LEDs. This causes the physical LEDs on the board to turn on or off according to the button states. Therefore, the software implements a direct relation between the three input buttons and the three output LEDs.

In summary, the I/O-related functions used in this program are: `XGpio_Initialize()`, which initializes the GPIO driver instance; `XGpio_SetDataDirection()`, which configures each GPIO bit as input or output; `XGpio_DiscreteRead()`, which reads the current input values from a GPIO channel; and `XGpio_DiscreteWrite()`, which sends output values to a GPIO channel.

Overall, this example demonstrates a simple polling-based GPIO control system using MicroBlaze. The processor continuously samples the button inputs and updates the LEDs accordingly, showing how software running on MicroBlaze can interact with real external hardware through AXI GPIO peripherals.
