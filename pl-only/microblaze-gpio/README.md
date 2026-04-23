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
