The conditioning circuit can be approximated by the following schematic:

![](https://pbs.twimg.com/media/GdK44L2XEAEMbHU?format=png&name=large)

Below I explain what each part should do.

1. Amplifier OA1. This forms the offset differential that was explained at the beginning of the project description.

2. Amplifier OA2. This is the 1.5V reference source. Note that to create this reference, I start from the 3.0V voltage reference circuit LM4040D30ILPR, and by means of a simple divider with resistors, I obtain the 1.5V.


![](https://pbs.twimg.com/media/FvEKdBZWYCUvtAZ?format=png&name=900x900)

I use this amplifier for impedance matching, since a voltage source or reference must have a low impedance with respect to the load, but the voltage divider with resistors R6 and R7 do not meet this condition, so a voltage follower meets the necessary characteristic.

3. OA3 amplifier. 

With this amplifier I want to create an active filter to eliminate any high frequency noise, generally high frequency noise could alter the microcontroller CPU causing a failure or a reset. It is not the final design and I have not yet calculated the values, the reference is the Butterworth filter (Or is it Chebyshev? I don't remember, I have to determine it when necessary)

4. OA4 amplifier.

I need to detect the zero crossing of the sinusoidal signal to start acquiring data and also measure its period (frequency).
By adding, the signal varies on a 1.5V axis, so using the reference of that value and the conditioned signal to generate a square wave that will reach the microcontroller and allow it to detect when a period of the signal begins and ends.
