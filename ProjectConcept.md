I have a project in mind, which can be said to be more of a hobby than a professional one.

I need to measure the RMS voltage of the power grid, which must be 127VAC, however, that value varies on some occasions, so it is necessary to constantly visualize it.

A practical and effective solution would be to buy a voltmeter (digital or analogue) and that would be the whole challenge.

<p align="center">
  <img src="https://m.media-amazon.com/images/I/71lKk9gu6rL._AC_UF1000,1000_QL80_.jpg" width="700">
</p>


However, since this is something I want to do for fun, I decided to build the voltmeter with elements I have, such as some microcontroller samples I acquired back in the day, but have never put them to practical use.

The first idea that occurred to me was to use a chip that is already a digital voltmeter. 

![Digital voltmeter on a chip](https://www.eleccircuit.com/wp-content/uploads/2013/01/digital-voltmeter-circuit-diagram-using-ICL7107.png)

This solution, I also think, would not be a big challenge. Obviously the PCB design and the placement of the elements would be something fun, but I prefer to write some code as well.

So I have decided to give it an additional plus. Apart from simply measuring and displaying the RMS voltage, I am going to add two more functionalities:

1. Using a push button, the hardware will stop displaying the voltage, and instead indicate the network frequency.

2. If the voltage is lower than a minimum value or higher than a maximum value, an acoustic signal will be activated to report said problem in the network.

Below I show a rather rough and incomplete block diagram of the hardware:

![Block diagram](https://pbs.twimg.com/media/GdKyXoCXcAA0GET?format=png&name=medium)


The first condition, which I chose, is that the maximum peak value of the AC voltage signal that the voltmeter could measure would be 300V (212.13 Vrms), this value is due to the fact that the microcontroller's ADC works with a maximum DC value of 3.0V, and in this way the calculations of the components could be in closed values ​​(without many decimals).

So, when the voltage signal is 300V, at the ADC input it should be 3.0V, while when it is -300V, the voltage at the ADC input should be 0V.

This poses a system of 2 equations with two unknowns variables:


![system of 2 equations with two unknown variables](https://pbs.twimg.com/media/FvDLkrxXoAEkQOf?format=png&name=small)


When solving the equations, we have that the AC voltage signal must be given a gain (or rather, attenuated) and an offset added to it.

The gain (less than one, is 1/200) and the offset is 1.5V.

To attenuate the signal, as well as add the offset, I decided to use an operational amplifier in offset subtracter mode:

![operational amplifier in offset subtracter mode](https://solucioningenieril.com/imagenes/asignaturas/amplificadores_operacionales/tema_8/1.png)

Where R1 and R2 would be 200k, Rf and Rx 1k and Vref 1.5V.

As I mentioned initially, I have several sample elements requested and left over from other projects, the operational amplifier I have is the MCP6L01, and I proceeded to perform a simulation with MPLAB Mindi to determine if this first part of the circuit works.

![ simulation with MPLAB Mindi ](https://pbs.twimg.com/media/FvDPBHPXgEQPK6r?format=png&name=large)

Something I remember right now is that the microcontroller works with 3.3V, but the ADC wants to work with a reference voltage of 3.0V, the reason is that using the same supply voltage is not exact, that is, it is close to 3.3V, but it usually varies slightly depending on various circumstances.

While having a fixed voltage reference, I think the ADC error would be smaller with respect to the LDO voltage.

Finally, the microcontroller I have is the PIC32MX170F256B.

![](https://pbs.twimg.com/media/GdK0nmgWwAA4gnT?format=png&name=900x900)
