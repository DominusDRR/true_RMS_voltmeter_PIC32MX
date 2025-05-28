I'm going to do the project with XC32, with MPLAB Harmony 3 and with MCC.

I'm leaving the steps in images if someone, at some point, wants to create a project with the mentioned tools.

First, in MPLAB X I start a new project with H3 with MCC:

![Create a project](https://pbs.twimg.com/media/FxpLeMXXgAIhkbE?format=png&name=900x900)

Then I select another window that opens, where I must select the path where the H3 repository was downloaded.

![](https://pbs.twimg.com/media/FxpL3MfWcAEKvbK?format=png&name=900x900)

Next, I indicate where I want to create my project:

![](https://pbs.twimg.com/media/FxpMNfgWwAAACE7?format=png&name=900x900)

Finally, I select the microcontroller I want to use.

![](https://pbs.twimg.com/media/FxpNpM4XoAImuoE?format=png&name=900x900)

Then the content manager wizard opens, where I select the option for H3.

![](https://pbs.twimg.com/media/FxpP5ScX0AAbbxJ?format=png&name=large)

![](https://pbs.twimg.com/media/FxpQDfjXgAIVgdv?format=png&name=large)

Finally the MCC looks like this:

![](https://pbs.twimg.com/media/FxpTFqIXsAAI8U4?format=png&name=large)

With the System module, you can calibrate the configuration bits, also known as fuses.

![](https://pbs.twimg.com/media/FxpTc6EX0AALQcZ?format=png&name=900x900)

The first configuration word is called DEVCFG3 and has three parts.

USERID allows us to add an identification number that could be the firmware version or any other information.

A feature of this MCU family is that the outputs or inputs of various peripherals can be ‘placed’ on different terminals of the MCU. This is an advantage over other microcontrollers because very often several peripherals share the same terminals. For example, if the RS232 asynchronous serial communication module is used, the SPI module could no longer be used and vice versa.

With this microcontroller family, this problem is easily solved by assigning the inputs and outputs of the other peripheral to other terminals.

The peripherals that do not have this advantage are the I2C communication module and the ADC converter, whose terminals are fixed.

Setting PMDL1WAY to ON allows us to block the Peripheral Module once, which allows us to disable a peripheral module once, until the MCU's external terminals are correctly assigned. After all the peripherals have been correctly configured, there is no way to modify the PPS again until the MCU is restarted.

Setting IOL1WAY to ON allows us to block the RPnR registers once for the correct assignment of the terminals to the peripheral.

![DEVCFG3](https://pbs.twimg.com/media/FxpU6S5XgAAt3Hz?format=png&name=small)

DEVCFG2 and DEVCFG1 are related to the oscillator, so I prefer to use the graphical part of the oscillator to configure it.


![DEVCFG2 & DEVCFG1](https://pbs.twimg.com/media/FxpVmW3XwAA-rao?format=png&name=small)

The microcontroller CPU can work at 50MHz, but I have an 8MHz crystal, so the next frequency would be 48MHz.

![System Clock](https://pbs.twimg.com/media/FxpWXmeWcAIZWrr?format=jpg&name=medium)

If you go back to the settings for DEVCFG2 and DEVCFG1, you will see that some bits have changed.

![](https://pbs.twimg.com/media/FxpX3k-XsAAyDgc?format=png&name=small)

**Note that the FNOSC bit of DEVCFG1 is in FRCPLL, this stands for Fast RC PPL, which is a high speed internal RC oscillator, and I don't want to use that oscillator, I want to use the crystal.**

And what I want to use is XT plus frequency multiplier (PLL)

![](https://pbs.twimg.com/media/FxpYy2GXoAEMEQE?format=png&name=900x900)

![FNOSC](https://pbs.twimg.com/media/FxpZN8tWYAAegCg?format=png&name=small)

The IESO bit controls the dual-speed start-up. When the system is powered on, the crystal or oscillator needs some time to stabilize properly. Enabling the dual-speed mode with the IESO bit allows the CPU to use the internal oscillator first and then switch to the main oscillator. In our case, I will not use this functionality, but I must use the timer related to the Power On Reset (POR) until the main oscillator stabilizes.

FPBDIV allows you to define the frequency at which the peripherals will work and is a multiple of the CPU oscillation frequency. We will leave it at a value equal to 1, that is, without division.

FCKSM has 3 sub-options. The FSCM or Fail-Safe Clock Monitor monitors that the main oscillator works correctly. If there is a fault, it can switch to an internal oscillator so that the CPU continues working. I think I will leave the FSCM and oscillator switching disabled. Or maybe I should go for the internal oscillator switching option.

I want to use the watchdog, so I set its overflow to 1024ms, and the window options for that timer, I won't use (set to OFF)

![WDT Configuration](https://pbs.twimg.com/media/FxpbBmfWcAYwCFW?format=png&name=small)

Regarding DEVCFG0, it has five options, the first is called JTAGEN and allows debugging through JTAG, which is an IEEE standard for an access port to test or debug a device. I will use an ICD3 or a SNAP; so this option will be OFF.

The second option allows us to configure which ICSP channel will be used and in this case it is number 2.

Regarding the write protections in the program memory and bootloader, since it is a homemade project and I do not plan to create a remote update, I do not plan to activate those boxes, only the read protection through an external programmer.

The Background debugger, I leave it OFF, it is not something unimportant either, since it is a homemade project, and it is automatically set to ON, if I use the programmer/debugger to debug with the hardware.

![DEVCFG0](https://pbs.twimg.com/media/FxpdFYnXoAAk2ZY?format=png&name=small)

The next step is to configure the microcontroller pins or terminals.

![](https://pbs.twimg.com/media/FxtECYrWcAQsmXr?format=png&name=medium)

I configure each of the terminals according to the desired design.

![](https://pbs.twimg.com/media/FxtG-ntX0AEqjO_?format=png&name=900x900)

In device resources search and add the Core module:

![](https://pbs.twimg.com/media/FxtHXcMXoAAZu1C?format=png&name=small)

Whenever I add this module, it always asks me if I want to use FreeRTOS, but for this simple example, I don't think it's necessary.

![](https://pbs.twimg.com/media/FxtHpKVXsAY7Y-X?format=png&name=small)

With the Core module, I check the box to generate application files, this allows you to create state machine tasks or processes, and I like that way of writing code.

![](https://pbs.twimg.com/media/FxtIBQZWYAEmBTK?format=png&name=small)

I create tasks for each peripheral that I want to control or a particular process that I want to be independent.

For the moment I create two, the first one will be to control the display, and the second one will be the one that performs the process of calculating the rms value.

![](https://pbs.twimg.com/media/FxtImBWXsAA3a2o?format=png&name=small)

Later, I will continue to add other tasks.

I also add the Core Timer to generate asynchronous delays, for any delay in the code. So I go back to the Root group and add that module to have the libraries for that peripheral.

![](https://pbs.twimg.com/media/FxtUB2qXsAIMxzp?format=png&name=small)

To generate the code, I must press the button called 'Generate'

![](https://pbs.twimg.com/media/FxtUbTFXgAchyOt?format=png&name=small)

The code may have something similar.

![](https://pbs.twimg.com/media/FxtU3c0XgAATD2f?format=jpg&name=medium)

_**So far, the fundamental steps to create the project code have been explained. You can close the MCC and if you need to change or add something, you must invoke the MCC again.**_
