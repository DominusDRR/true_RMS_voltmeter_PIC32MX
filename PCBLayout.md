The PCB is trying to adapt to these plastic cases

<p align="center">
  <img src="https://pbs.twimg.com/media/FyB_J9NWYAEfVDZ?format=jpg&name=large" width="700">
</p>

<p align="center">
  <img src="https://pbs.twimg.com/media/FyB_QUxWAAMfWan?format=png&name=large" width="700">
</p>

<p align="center">
  <img src="https://pbs.twimg.com/media/F0WpPV6XoAEkADr?format=jpg&name=small" width="700">
</p>

<p align="center">
<img src="https://pbs.twimg.com/media/F0WpZTCXgAAwVoQ?format=jpg&name=small" width="700">
</p>

This is the schematic of the system power supply

<p align="center">
<img src="https://pbs.twimg.com/media/F0WryKrWIAA9Eqh?format=png&name=medium" width="700">
</p>

Regarding the power supply, I placed a rectifier diode before the 5V regulator (D4) to avoid reverse bias when using an external power supply, which I will use when I want to debug the microcontroller firmware:

Additionally, the 1mA load (15k) needed by the 15V power supply was replaced by an LED diode with its limiting resistor (LED1 and R5).

The schematic of the microcontroller and the 4-digit display

<p align="center">
<img src="https://pbs.twimg.com/media/Gdef9quWwAAnBaL?format=png&name=medium" width="700">
</p>

The schematic of the conditioning circuit. (Note that I changed the 1k and 200k resistors of the subtracting amplifier, for 10k and 200M resistors to give a greater input impedance to the signal that you want to analyze its RMS value.)

<p align="center">
<img src="https://pbs.twimg.com/media/GdegyViWgAAkTql?format=png&name=900x900" width="700">
</p>

This is what PCBs look like after arriving from the factory:
<p align="center">
<img src="https://pbs.twimg.com/media/F3rPv6uWkAArJNO?format=jpg&name=large" width="700">
</p>

The first thing I did was place the elements that make up the voltage source
<p align="center">
<img src="https://pbs.twimg.com/media/F35wU5BXsAAEoDs?format=jpg&name=large" width="700">
</p>

And the voltage produced is close to 15V DC.

<p align="center">
<img src="https://pbs.twimg.com/media/F36MEWVWAAAnJOL?format=jpg&name=large" width="700">
</p>

This is how it ended up with the microcontroller and everything needed to control the 7-segment display.
<p align="center">
<img src="https://pbs.twimg.com/media/F44F8O5W0AAZsx6?format=jpg&name=large" width="700">
</p>

In the initialization of the task that manages the display, I set the default number to 4567, and this is how it is displayed.
<p align="center">
<img src="https://pbs.twimg.com/media/F4zANDaXQAA5Ord?format=jpg&name=medium" width="700">
</p>

