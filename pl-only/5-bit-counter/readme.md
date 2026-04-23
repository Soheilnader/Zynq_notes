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