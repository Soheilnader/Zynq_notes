
### MicroBlaze and GPIO

In this example, we use 3 input signals so that each one turns on its corresponding output LED.

After adding MicroBlaze and connecting clock and reset, add **AXI GPIO** IP to the block design.

![Adding AXI GPIO IPs to the block design](../../images/fig68.jpg)

Use two GPIO IPs: one input-only and one output-only.

Configure one GPIO with **All Inputs**, width **3**, and disable dual channel. Configure the other with **All Outputs**.

![Configuring AXI GPIO properties](../../images/fig69.jpg)

For each GPIO, expand the GPIO port and select **Make External**.

![Making the GPIO ports external](../../images/fig70.jpg)

Final block design:

![Final block design for the MicroBlaze and GPIO system](../../images/fig71.jpg)

Assign constraints as follows:

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
