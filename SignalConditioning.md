I'm going to do some real signal conditioning tests with the op-amp. I have a few on PCBs for use in a protoboard.

![OP fo test](https://pbs.twimg.com/media/FwA8PKLWYAUWCzy?format=png&name=small)

I have performed the tests with the op-amp to determine if the conditioning works as the simulation predicted.

I should point out that for this test I used common resistors (for the project I will use 1% tolerance surface resistors), and I have not yet used the voltage reference circuit, which is something I did not have and is a purchase that is on the way. So I used a resistor array to get approximately 1.5V.


![Prototype Hardware ](https://pbs.twimg.com/media/GdPB6i4WAAA05Hf?format=jpg&name=large)

When I looked at the conditioned signal, I got this, which for a moment I thought was some kind of distortion of the sinusoidal signal. I had the idea that the op amp, or the circuit was properly conditioning the signal, but distorting the waveform.

![Conditioned signal](https://pbs.twimg.com/media/FwLt9IOXsAYUPRy?format=jpg&name=large)

So I decided to connect the oscilloscope directly to the mains to see what the voltage signal was like, and the distortion came from there and it was not the fault of the circuit with the operational (I put the scope test lead x10)

![electric mains](https://pbs.twimg.com/media/FwLvD1fXwAIzqzU?format=jpg&name=large)

So far, we can say that the signal conditioning works, obviously it is not precise because of the resistors used, and the voltage reference I got is 1.4V.

I used one of the 4 opamps available for the comparator, to generate a square wave synchronized with the zero crossing of the sine wave and this is the result:

![Zero crossing detection](https://pbs.twimg.com/media/FwLwK7xXsAU82Fw?format=jpg&name=large)
