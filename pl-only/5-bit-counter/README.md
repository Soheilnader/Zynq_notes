## Building a 5-bit Counter Project with LEDs

In this project, we intend to implement a 5-bit counter on the Programmable Logic (PL), because this is the number of LEDs available on the Artemis Karia board.

### Creating the Project

First, we run the Vivado software. To create a new project, we select the **Create Project** option, and in the opened window, we click the **Next** button.

![Creating a project](../../images/fig1.jpg)

Next, we need to specify the name and location of the desired project. By clicking the **Next** button, we move to the next section.

![Specifying the project name and location](../../images/fig2.jpg)

In the **Project Type** section, we select **RTL Project** as the project type, and by clicking **Next**, we enter the **Default Part** section.

![Selecting the project type](../../images/fig3.jpg)

In this window, we must select the target hardware. To make it easier to find the hardware, the filters can be set as shown in the figure, or the part model name can be entered directly in the search box. Since the **Zynq7010** model is used in this project, the selected part name is `xc7z010clg400-2`.

![Selecting the hardware](../../images/fig4.jpg)

Again, by clicking the **Next** button, these settings are applied. After reviewing the displayed information and selecting the **Finish** button, the steps for creating the new project are completed.

![Completing the project creation steps](../../images/fig5.jpg)

### Adding the Code and Related Sources to the Project

After creating the project, we enter the Vivado environment. To add a Hardware Description Language (**HDL**) code, from the **Sources** section, we click the **Add Sources** button.

![List of available sources](../../images/fig6.jpg)

In the opened window, we enable the second option, **Add or create design sources**, if it is disabled, and by clicking **Next**, we move to the next section.

![Creating a new source](../../images/fig7.jpg)

In this section, to create a new source, we select the **Create File** button. In the opened window, we choose the desired hardware description language (for example, **Verilog**) and also specify a name for the source.

![Specifying the source name and hardware description language](../../images/fig8.jpg)

If the **Define Module** window appears, we choose the **OK** button.

![Define Module window](../../images/fig9.jpg)

It can be seen that a file with the `.v` format (Verilog), with the name chosen in the previous steps, is created under the **Design Sources** section. By double-clicking on it, its code can be edited. For example, the following code is written in this file to implement a 5-bit counter circuit.

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

Now it is necessary to assign each signal in the Verilog code to the hardware pins. For this purpose, once again from the **Sources** section, we click the `+` button, and this time in the opened window, we select the first option, **Add or create constraints**, and by clicking **Next**, we move to the next step.

![Creating the constraint file](../../images/fig10.jpg)

In the opened window, we first click the **Create File** button, then choose a name for the file, and by selecting **OK** and then **Finish**, the file is created.

![Entering the constraint file name](../../images/fig11.jpg)

It can be seen that a file with the `.xdc` format has been created.

![Displaying the list of required files](../../images/fig12.jpg)

By double-clicking on this file, we edit it. In the following code, first we set the port standard to **LVCMOS33**. Then, using the `PACKAGE_PIN` parameter, we assign pins to each signal. These pins correspond to the board used in this project.

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

![Pin names](../../images/fig13.jpg)

It should be noted that the **PL clock**, which is **50 MHz**, is connected to pin `U18`.

### Downloading onto the Hardware

Now we discuss how to load the written code onto the device and configure it. For this, the code must first be synthesized (**Synthesis**), implemented (**Implementation**), and then its **Bitstream** must be generated and loaded onto the device.

For this purpose, as shown in the figure, we click **Run Synthesis** and wait until the synthesis process is completed.

![Running synthesis](../../images/fig14.jpg)

Then the next window appears, indicating the completion of the synthesis stage. Now, by selecting **Run Implementation** and clicking **OK**, the implementation process begins, and again we must wait until it is completed.

![Starting implementation after synthesis](../../images/fig15.jpg)

After the following window appears, we select **Generate Bitstream**, and by clicking **OK**, the bitstream generation process begins.

![Generating the bitstream](../../images/fig16.jpg)

Now, we connect the computer to the board's **JTAG** port using the interface cable and turn on the board. From the **Flow Navigator** menu on the left side, in the **PROGRAM AND DEBUG** section, we select **Open Hardware Manager**, then click **Open Target** and choose **Auto Connect**.

![Opening Hardware Manager and connecting to the target](../../images/fig17.jpg)

Then, as shown in the figure, it can be seen that the device has been recognized by the computer. To send the configuration to the hardware, we click **Program Device**.

![Recognized device in Hardware Manager](../../images/fig18.jpg)

In the opened window, we click **Program** so that the device is configured.

![Programming the device](../../images/fig19.jpg)

Now it can be observed that the LEDs on the board turn on and off as a binary counter.

![5-bit binary counter output on the LEDs](../../images/fig20.jpg)
