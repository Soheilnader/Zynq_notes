# Building a 5-bit Counter Project with LEDs

In this project, we implement a 5-bit counter on the Programmable Logic (PL), since the Artemis Karia board has 5 available LEDs.

## Creating the Project

First, open **Vivado**. To create a new project, select **Create Project**. In the opened window, click **Next**.

![Creating a project]../../images/(fig1)

Next, specify the project **name** and **location**, then click **Next**.

![Specifying the project name and location]../../images/(fig2)

In the **Project Type** section, select **RTL Project**, then click **Next** to continue to the **Default Part** section.

![Selecting the project type]../../images/(fig3)

In this window, select the target hardware. To make it easier to find, you can set the filters as shown in the figure or directly search for the part model. Since this project uses the **Zynq7010**, the selected part is:

```text
xc7z010clg400-2
```

![Selecting the hardware]../../images/(fig4)

Click **Next** again to apply the settings. After reviewing the displayed information, click **Finish** to complete project creation.

![Completing the project creation steps]../../images/(fig5)

## Adding the Code and Related Sources to the Project

After creating the project, enter the Vivado environment. To add a Hardware Description Language (**HDL**) file, go to the **Sources** section and click **Add Sources**.

![List of available sources]../../images/(fig6)

In the opened window, enable the second option, **Add or create design sources**, if it is not already enabled. Then click **Next**.

![Creating a new source]../../images/(fig7)

To create a new source file, click **Create File**. In the opened window, choose the desired HDL language, such as **Verilog**, and specify a name for the source.

![Specifying the source name and hardware description language]../../images/(fig8)

If the **Define Module** window appears, click **OK**.

![Define Module window]../../images/(fig9)

A `.v` file now appears under **Design Sources**. Double-click it to edit the code. Use the following Verilog code to implement the 5-bit counter:

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

Next, assign each signal in the Verilog code to the hardware pins. From the **Sources** section, click the **+** button again. This time, in the opened window, select **Add or create constraints**, then click **Next**.

![Creating the constraint file]../../images/(fig10)

In the new window, click **Create File**, choose a name for the constraint file, then click **OK** and **Finish**.

![Entering the constraint file name]../../images/(fig11)

A `.xdc` file will now appear in the project.

![Displaying the list of required files]../../images/(fig12)

Double-click this file to edit it. In the following code, the port standard is set to **LVCMOS33**, and the `PACKAGE_PIN` parameter is used to assign FPGA pins to each signal:

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

![Pin names]../../images/(fig13)

Note that the **PL clock** is **50 MHz** and is connected to pin `U18`.

## Downloading onto the Hardware

Now we load the design onto the device. To do this, the project must go through:

1. **Synthesis**
2. **Implementation**
3. **Bitstream generation**
4. **Programming the device**

First, click **Run Synthesis** and wait for synthesis to complete.

![Running synthesis]../../images/(fig14)

When the next window appears, select **Run Implementation** and click **OK**. Wait for implementation to finish.

![Starting implementation after synthesis]../../images/(fig15)

After that, select **Generate Bitstream** and click **OK** to begin bitstream generation.

![Generating the bitstream]../../images/(fig16)

Now connect the computer to the board’s **JTAG** port using the interface cable and power on the board. In the **Flow Navigator** on the left, under **PROGRAM AND DEBUG**, select **Open Hardware Manager**. Then click **Open Target** and choose **Auto Connect**.

![Opening Hardware Manager and connecting to the target]../../images/(fig17)

Once connected, the device should appear in Hardware Manager. To send the configuration to the board, click **Program Device**.

![Recognized device in Hardware Manager]../../images/(fig18)

In the opened window, click **Program** to configure the FPGA.

![Programming the device]../../images/(fig19)

After programming, the LEDs on the board will turn on and off as a 5-bit binary counter.

![5-bit binary counter output on the LEDs]../../images/(fig20)

## Summary

This project demonstrates how to:

- Create a new **Vivado RTL project**
- Add a **Verilog** source file
- Add an **XDC constraint file**
- Assign FPGA pins for the clock and LEDs
- Run **Synthesis**, **Implementation**, and **Bitstream Generation**
- Program the FPGA through **Hardware Manager**

The final result is a working **5-bit LED counter** on the Artemis Karia board.
