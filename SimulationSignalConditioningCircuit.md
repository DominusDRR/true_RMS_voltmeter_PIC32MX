I had some doubts if the voltage follower to produce 1.5 would work correctly and with the comparator to create the square wave signal synchronized with the zero crossing.

First I checked the 1.5 V reference and this is what I got (Output of Op X2)

![Simulation of 1.5V reference](https://pbs.twimg.com/media/FvIxtlCWABwI7u9?format=png&name=large)

You can see that there is noise in the 1.5V signal, a reflection of the AC signal of approximately 70 uV peak to peak, maybe I should use an RC filter to eliminate it, with the consequence of increasing the impedance of the reference signal, or use another follower after the filter to avoid this possible problem.

Then I checked the Op X3, it behaves appropriately as a comparator and this is what I got:

![Simulation 2](https://pbs.twimg.com/media/FvIzC8wXwAMH18V?format=png&name=large)

So it seems that noise on the 1.5V reference is not a problem.

**Note that I have not yet installed the high-frequency filter of the differential offset amplifier, whose output is the one I should use to compare to 1.5V.**
