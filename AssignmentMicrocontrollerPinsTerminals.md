
The microcontroller pin assignment will be as follows:

![Pin distribution](https://pbs.twimg.com/media/GdLnHN7WcAAYt7K?format=png&name=large)

I'll explain this in more detail below.

Terminal 1 is the reset, so it will need to be connected to the RC network recommended by the manufacturer.

Terminal 2 will be the external 3.0V reference for the ADC.

Terminal 3 will be the input to the ADC.

![ADC Module](https://pbs.twimg.com/media/FvIV_TjXgAEY1Ct?format=jpg&name=medium)

The entire port B will be the data bus for the 7-segment display.

![Display](https://pbs.twimg.com/media/FvIWSuPXsAQechn?format=png&name=small)

Because it is a 4 digit display:

![Display 3D](https://content.instructables.com/FZI/CZGZ/IBTF5EDQ/FZICZGZIBTF5EDQ.png?auto=webp)

I need to scan each digit, for which terminals RB8, RB9, RB12 and RB13 will control each digit.

Terminals RA2 and RA3 will be for the internal crystal oscillator. _**I had the idea of ​​using the internal RC oscillator, but since it is not precise, I don't know how big the error in the measurements will be.**_

I am forgetting the acoustic signal, for which I must generate a square wave, precisely with the remaining terminal RB15, it can be the output of the comparison module 1 to generate a PWM signal.

![PWM Buzzer](https://pbs.twimg.com/media/FvIbugZWcAU5ilr?format=jpg&name=medium)

And that's all for now, I'll start designing the PCB when I have a bit more time. And when I finish another board of my work, I plan to send them together to manufacture both at the same time.
