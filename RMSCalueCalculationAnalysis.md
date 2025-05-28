To calculate the RMS I must meet the following condition of several samples acquired by the ADC according to the following equation:

![](https://docs.openenergymonitor.org/_images/equation3.png)

The maximum value measured by the ADC would correspond to 300V, while the minimum is -300V (Obviously this signal must be conditioned to the maximum and minimum values ​​of the ADC)

Before applying the equation above, I think I should change the values ​​obtained from the ADC to voltage levels.

The ADC has 4 options for the conversion result

![](https://pbs.twimg.com/media/FxvmEwzWcAAPHNb?format=png&name=medium)

**Each of the values ​​must be multiplied by a slope or gain m and added to a shift or constant c, to arrive at the minimum and maximum voltage values.**

 V(t) = (ADC value(t))* m + c

For the first option it would be:

1023*m + c = 300
0*m + c = -300

From here m = 200/341 and c = -300.

For the second option it would be:

511*m + c = 300
-512*m + c = -300, where m = 200/341 and c = 0.293

For the third option it would be:

0.999*m + c = 300
0*m + c = -300 , where m = 600.6006 and c =-300

And the last one would be:

0.499*m + c = 300
-0.5*m + c = -300, where m = 109.111 and c = 245.

**With these results I think that the first option seems to be the best.** (32 bits Integer format)

That is to say, each value obtained from the ADC must be multiplied by m and added to c. The result must be squared and each of these values ​​must be added.

The total sum must be divided by the number of samples obtained in the time period, and this new result must be taken as the square root.

Now, it seems to me that this constant m could be taken out of the summation, and the constant c would be another separate summation, that is to say, I think that before multiplying each value by m and adding c, the summation could be performed without altering the acquired values ​​yet, and then, with the final result, I would multiply by a gain and add it.

In this way, I could relieve the calculation load on the microcontroller's CPU.

So the equation to calculate the RMS value would be like this:

![](https://pbs.twimg.com/media/GdZV88AXAAADDXd?format=png&name=small)

I started doing some tests with the code and the MPLAB X simulator.

The first thing I did was imagine that I had 100 samples of a perfect sinusoidal signal, with RMS of 120V.

The conditioning circuit changes the 300V input signal to 3.0V which would be 1023 on the ADC, with -300, the signal would be 0.0V and the ADC would also be 0.

If my signal is 120VAC, it means that the peak value would be 169.7056V.

The conditioning circuit with the operational amplifier, gives a gain of 1/200 to the signal and adds a constant of 1.5.

Therefore.

169.7056/200 + 1.5 = 2.3482V

-168.7056/200 + 1.5 = 0.65147V

If the ADC, for 3.0V, is 1023, performing a simple rule of 3.

3.0V---- is equivalent to ---- > 1023

2.3482--- is equivalent to 800.84.81 (801 being an unsigned integer)

0.65147-- is equivalent to 222.1519 (222 for an unsigned number)

I create a vector or array (called arreglo) to store these samples, and I do it in the data structure of the appRMS task:

![](https://pbs.twimg.com/media/FxyXHZZWIAMYyaY?format=png&name=small)

Now, with the help of Excel, I generate the 100 samples of the signal, the formula used is:

**=341*((169.7056/200)*SIN(2*PI()*A3/100)+1.5)**

![](https://pbs.twimg.com/media/FxyXmctWYAM9AR5?format=png&name=small)

I copy and paste all those values ​​for the array in file c, of the task.

![](https://pbs.twimg.com/media/FxyX-YlXoAAoDe9?format=png&name=small)

The code in C would be like this:

![](https://pbs.twimg.com/media/GdZcVkPWcAAV3Ri?format=png&name=900x900)

I ran the simulation, and the code that performs the RMS value calculation seems to work.

![](https://pbs.twimg.com/media/GdZcp1FXUAAxae0?format=png&name=medium)

