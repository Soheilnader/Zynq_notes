## ILA (Integrated Logic Analyzer) -- Native

In this section, we describe how to use the Integrated Logic Analyzer (ILA) to verify and debug an RTL design. Since we are not using a block design in this example, the ILA module must be instantiated directly in the Verilog design.

Assume that the project has already been created and that the following Verilog module has been designed:

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

From the left menu, under **Project Manager**, click on **IP Catalog**. In the IP Catalog tab, a list of available Vivado IPs is shown. Search for **ILA** as shown in the following figure.

![Searching for the ILA IP in the Vivado IP Catalog](../../images/fig74.jpg)

Double-click on the ILA IP. The **Customize IP** window will appear. In the general options, since we are using a purely RTL design and not an AXI interconnect, we choose the **Native** option. Because we want to monitor the signals `rst`, `cnt`, and `LED`, we set the number of probes to 3. We also set the sample data depth to 1024, or any other value as needed.

![Configuring the ILA IP in Native mode](../../images/fig75.jpg)

In the **Probe Ports** tab, we set the width of each probe according to the corresponding signal width. In this example, we connect `probe0`, `probe1`, and `probe2` to `rst`, `cnt`, and `LED`, respectively. Therefore, their widths must be set to 1, 32, and 5 bits, as shown in the following figure.

![Setting the widths of the ILA probes](../../images/fig76.jpg)

After clicking **OK**, a window may appear asking whether a related directory should be created. By clicking **OK** again, the customized ILA IP core is generated.

![Generating the customized ILA IP core](../../images/fig77.jpg)

If a window titled **Generate Output Products** appears, click **Skip**.

![Skipping output product generation for the ILA IP](../../images/fig78.jpg)

In the **Sources** window, go to **IP Sources** and open `IP -> ila_0 -> Instantiation Template -> ila_0.veo` by double-clicking on it.

![Opening the ILA instantiation template](../../images/fig79.jpg)

A window will appear that contains the ILA IP core instantiation template. We need to copy this template:

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

Then, return to the design Verilog file and paste the ILA template into it. Make sure to give the instantiated ILA a proper name and connect the correct signals to its probe ports. The final Verilog code is shown below.

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

Next, create a constraint file according to the FPGA development board being used:

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

After saving the edited file, run synthesis, implementation, and bitstream generation, and then program the chip, as shown in the following figure.

![Programming the FPGA with the design containing the ILA](../../images/fig80.jpg)

After programming is completed, the ILA window opens automatically.

![ILA window after programming the FPGA](../../images/fig81.jpg)

In the **Trigger Setup** window, a signal can be selected as the trigger condition so that the waveform is captured when the selected condition occurs. For example, we can add the reset signal (named `p_0_in` or a similar automatically assigned name in the ILA window) and set its trigger value to 0, because the reset button on the development board is active-low. After clicking **Run Trigger for this ILA core** and pressing the assigned reset button, the waveform can be observed as shown in the following figure.

![Captured waveform in the ILA after the trigger condition is met](../../images/fig82.jpg)

More trigger conditions can also be added in the **Trigger Setup** window, but in that case it is important to set the **Trigger Condition** option to **Global OR**, as shown in the following figure.

![Setting multiple trigger conditions using Global OR](../../images/fig83.jpg)

Another way to capture the waveform is without defining any trigger condition. This can be done by clicking **Run Trigger Immediate for the ILA core**.
