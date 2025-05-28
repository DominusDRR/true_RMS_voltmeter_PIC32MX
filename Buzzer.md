I have decided to use the buzzer to make a tune or music when the device is powered.

I have worked with the output comparator module before, doing exactly what I have described, and it works with a 16 or 32 bit timer to perform the comparison:

<p align="center">
<img src="https://pbs.twimg.com/media/F6VNXx-X0AEAqPA?format=png&name=900x900" width="700">
</p>

In the MCC graphic area, I made a new group called buzzer:

<p align="center">
<img src="https://pbs.twimg.com/media/F6UfdJXWoAAeC1i?format=png&name=900x900" width="700">
</p>

Inside which I have placed the OCMP1 module, which corresponds to the OC1 output:

<p align="center">
<img src="https://pbs.twimg.com/media/F6Uf5wEWYAEM8Mk?format=png&name=medium" width="700">
</p>

The features of the buzzer are as follows:

<p align="center">
<img src="https://pbs.twimg.com/media/F6UgyDrWUAAqdls?format=png&name=900x900" width="700">
</p>

There are several examples on the Internet of how to generate music with a PWM signal

The frequency of musical notes is in the approximate range from 200 Hz to 3000 Hz.

Therefore, to modify the PWM frequency, it is necessary to modify the PRy register of the associated timer:

<p align="center">
<img src="https://pbs.twimg.com/media/F6UiOxmWIAQ-9UZ?format=png&name=900x900" width="700">
</p>

So, the value to compare is not important, but as a clock source, I select timer 3, so as not to interfere with the one that will be used with the square signal capture module synchronized with the sine wave:

<p align="center">
<img src="https://pbs.twimg.com/media/F6UkCKNXwAATzwS?format=png&name=900x900" width="700">
</p>

And I create a task with the name appBuzzer that will be dedicated to generating the initial music and warning tones:

<p align="center">
<img src="https://pbs.twimg.com/media/F6UkPu8WQAIQ_Gp?format=png&name=small" width="700">
</p>

I have a project with a PIC32MZ microcontroller, with these definitions for each musical note, where each of them has the necessary frequency as a comment. For example, the musical note c_ corresponds to 261Hz, and the value to load in the PRy is 191570 to generate that frequency.

Obviously, for this microcontroller, which works at another frequency, I must recalculate those values.

<pre><code>#define mute 0
#define c_      191570 //       261 Hz underscore _ was used because there were conflicts with aws definitions or with freertos
#define d_      170067 //       294
#define e_      151974 //       329
#define f       143265 //       349
#define g       127876 //       391
#define gS      120481 //       415
#define a       113635 //       440
#define aS      112359 //       445
#define b       107295 //       466
#define cH      95601  //       523
#define cSH     90252  //       554
#define dH      85178  //       587
#define dSH     80385  //       622
#define eH      75872  //       659
#define fH      71632  //       698
#define fSH     67567  //       740
#define gH      63775  //       784
#define gSH     60240  //       830
#define aH      56817  //       880
                
#define cHH     23888   //      2093
#define AB      26823   //      1864
#define F6      35816   //      1396
#define d7      21285   //      2349
#define DE      20087   //      2489
#define f7      17901   //      2793
#define FG      16890   //      2960
#define GA      15050   //      3322
#define ABx     13407   //      3729
#define CD      22551   //      2217
#define E7      18960   //      2637
#define g7      15943   //      3136</code></pre>

According to the above definitions, the minimum frequency is 261 Hz and the maximum is 3136 Hz, this would imply a maximum period of the PWM signal of 3.831 ms and a minimum of 318.88 microseconds respectively.

Therefore, the PWM period should be greater than those 3.8 ms, for example the period could be 7 ms.

Playing with the parameters of timer 3, with a scaling of 8, that is, with 6 Mhz, and with PR3 equal to 42000, you would get 7 ms.

(42000/6MHz = 7 ms)

<p align="center">
<img src="https://pbs.twimg.com/media/F6VNXx-X0AEAqPA?format=png&name=900x900" width="700">
</p>

Now, if we want a period of 318.88 us, PR3 = f* T = 6MHz * 318.88 us = 1913.26, which is a value that PR3 can have.

The equation to calculate the period of the PWM signal is:

<p align="center">
<img src="https://pbs.twimg.com/media/F6VPfBHXYAEj5ns?format=png&name=900x900" width="700">
</p>

Where TPB is the clock frequency period of the peripheral bus, i.e. 1/48MHz.

TMR Prescaler Value is the pre-scaling value which is 8, i.e. TPB * TMR Prescaler Value = 1/6MHz.

With this I made a table in Excel to get the PR2 values ​​for each audio frequency:

<p align="center">
<img src="https://pbs.twimg.com/media/F6VTtUUWgAEwv8v?format=png&name=900x900" width="700">
</p>

And I paste it into the header file of the appbuzzer task: (obviously without decimals)

<p align="center">
<img src="https://pbs.twimg.com/media/F6VbVSeXAAA9bYI?format=png&name=900x900" width="700">
</p>

A song is defined as a series of musical notes, and each note has a duration.

So I create a data structure as follows:

<pre><code>typedef struct
{
    unsigned int FrecuenciaNota; //frequency of the musical note
    unsigned int Tiempo;         //Time that a musical note lasts
}NotasMusicales;</code></pre>

Where NoteFrequency is the value of the timer's PR, and Time is the duration of the musical note, so I create an array with 12 musical notes like this:

<pre><code>NotasMusicales      Musica[0x0C];</code></pre>

I create this in the C file of the buzzer task.

<p align="center">
<img src="https://pbs.twimg.com/media/F6VZATnWsAAHvIm?format=png&name=900x900" width="700">
</p>

In the initial state of the task, what I do is initialize the 'music' array with the PR3 and delay values ​​to generate the initial music.

<p align="center">
<img src="https://pbs.twimg.com/media/F6VbupbWMAAJcRr?format=png&name=small" width="700">
</p>

Remember that 1 ms is given by:

<pre><code>#define _1ms 0x5DC0</code></pre>

Also, in the initial state of the task (which I renamed) I disable everything related to the PWM configuration, otherwise it is possible that the buzzer makes sounds before the task starts to execute, and I start timer 3:

<p align="center">
<img src="https://pbs.twimg.com/media/F6VdxIJW8AElk-5?format=png&name=900x900" width="700">
</p>

I need a pointer to obtain each value of the array called Music, which will be part of the task data structure:

<p align="center">
<img src="https://pbs.twimg.com/media/F6VewvSXEAEhtWs?format=png&name=360x360" width="700">
</p>

And I initialize it in the C code of the task:

<p align="center">
<img src="https://pbs.twimg.com/media/F6VfUGFWkAEIyZy?format=png&name=900x900" width="700">
</p>

Also, I need a variable to generate asynchronous (non-blocking) delays, and I write it in the data structure:

<p align="center">
<img src="https://pbs.twimg.com/media/F6VrpvRWwAAT760?format=webp&name=small" width="700">
</p>

To change the value of PR3, I create a function like this:

<pre><code>void actualizarFrecuenciaPWM(unsigned int frecuencia)
{
    TMR3_PeriodSet(frecuencia); //is similar to, PR3 = frecuencia (frequency);
    OCMP1_CompareSecondaryValueSet(frecuencia >> 1); // pulse width at 50%
}</code></pre>

Where I also try to make the PWM duty ratio % when writing to the OC1RS register, half the value of PR3.

At task initialization, I capture a value from the Core Timer, and generate the first note with the PWM using the actualizarFrecuenciaPWM function.

<p align="center">
<img src="https://pbs.twimg.com/media/F6Vs00cW8AAQnTX?format=png&name=medium" width="700">
</p>

In the initial state, I wait for the duration of each note using an asynchronous delay like this:

<p align="center">
<img src="https://pbs.twimg.com/media/F6Vt_u2XoAAgZ9K?format=jpg&name=medium" width="700">
</p>

When the delay has finished, the following is done:

A new core timer value is captured for the next delay.

I increment the pointer of the array called Music by 1.

If the pointer is still within the size of the music array, I check if the next note is mute, if so, I turn off the PWM to generate a silence.

If it is not mute, I update the value of PR3 with the value of the next note.

If the pointer is larger than the size of the array, the melody has finished, I turn off the PWM and jump to a state called sleep, where the task will wait until it wants to generate an acoustic signal.

That acoustic signal will be generated in a new state, but that will be later, for now I just want to create this part of the code.

Download the new code to the hardware and this is the result

<p align="center">
<img src="https://pbs.twimg.com/media/F6VzQ28WgAA2cRf?format=png&name=medium" width="700">
</p>

You can see that there is an unwanted tone before the melody starts.

I was assuming that some code is executed before initializing the task where I disable the PWM, I tried several things, for example setting the value to compare to 1 so that the initial pulse width is small, so that it does not generate that noise, like this:

<p align="center">
<img src="https://pbs.twimg.com/media/F6ZNqNZWQAA63Fa?format=png&name=medium" width="700">
</p>

I also tried with a value equal to zero, but the result was the same.

I looked for where the PWM is initialized or activated, thinking that the generated code activated it ahead of time, but that is not the case. Where the PWM is activated is where I wrote OCMP1_Enable.

<p align="center">
<img src="https://pbs.twimg.com/media/F6ZOJXYXgAA-O3X?format=png&name=360x360" width="700">
</p>

So I came up with the idea of ​​writing a note of silence, in the initial code, this implies that the Musica array is now 13 elements:

<p align="center">
<img src="https://pbs.twimg.com/media/F6ZPL-BXwAACETz?format=png&name=large" width="700">
</p>

By doing this, the melody sounded as it should.

"Test 2":https://www.youtube.com/watch?v=JiKf7lXYXPU

This task also activates an acoustic signal to inform if there is an overvoltage or an undervoltage, the levels for these levels are as follows:


<pre><code>#define maximoVoltaje       127.00
#define minimoVoltaje       110.00 
#define maximaFrecuencia    60.50
#define minimaFrecuencia    59.50</code></pre>

For an over-voltage, I want to make three audio pulses of 100 ms duration. Between each pulse there will be 200 ms, and before generating the three pulses again, I expect about 700 ms, something similar to this image which is not to scale.

<p align="center">
<img src="https://pbs.twimg.com/media/GBGE-uuXYAABFHI?format=png&name=900x900" width="700">
</p>

For an undervoltage, I think it will be a 100ms pulse, and between the next one, it will be about 700ms.

<p align="center">
<img src="https://pbs.twimg.com/media/GBGFUxwWsAA9YmY?format=png&name=small" width="700">
</p>

I have created three more states, one that waits for the high tone, another that waits for the low tone and the third that waits for 700 ms to repeat the process again:

<pre><code>typedef enum
{
    /* Application's state machine's initial state. */
    APPBUZZER_ESTADO_INICIAL=0,
    APPBUZZER_ESTADO_REPOSO,
    /* TODO: Define states used by the application state machine. */
    APPBUZZER_ESTADO_PULSO_EN_ALTO,
    APPBUZZER_ESTADO_PULSO_EN_BAJO,
    APPBUZZER_ESTADO_PULSO_EN_SILENCIO
} APPBUZZER_STATES;</code></pre>


The task must be forced to change to the APPBUZZER_STATE_PULSE_ON_HIGH state, previously, the buzzer must be activated and the instantaneous value of the Core Timer must be captured to generate a 100ms delay.

This state would be like this:

<pre><code> case APPBUZZER_ESTADO_PULSO_EN_ALTO:
{
  if ((CORETIMER_CounterGet() - appbuzzerData.retardo) > _100ms)
  {
    OCMP1_Disable();
    appbuzzerData.retardo = CORETIMER_CounterGet();
    appbuzzerData.state = APPBUZZER_ESTADO_PULSO_EN_BAJO;
  }
  break;
}</code></pre>

Once the 100ms have elapsed, the buzzer is deactivated, an instantaneous value is captured again from the central timer for the next delay and the task status is changed to APPBUZZER_PULSE_STATE_IN_LOW.

The task data structure must have 3 additional data, an unsigned integer variable to capture the instantaneous value of the central timer, a variable where the number of desired audio pulses to be generated is written, and another that counts them,

<pre><code>typedef struct
{
    /* The application's current state */
    APPBUZZER_STATES state;
    /* TODO: Define any additional data used by the application. */
    unsigned char PunteroMusica;
    unsigned int retardo;
    unsigned char numeroPulsos; // number of pulses.
    unsigned char contadorPulsos; //pulse counter
} APPBUZZER_DATA;</code></pre>

The APPBUZZER_STATE_PULSE_IN_LOW state will be as follows, where 200ms is expected:

<pre><code>case APPBUZZER_ESTADO_PULSO_EN_BAJO:
{
     if ((CORETIMER_CounterGet() - appbuzzerData.retardo) > _200ms)
     {
         appbuzzerData.contadorPulsos--;
         if (appbuzzerData.contadorPulsos)
         {
             OCMP1_Enable();
             appbuzzerData.state = APPBUZZER_ESTADO_PULSO_EN_ALTO;
         }
         else
         {
             appbuzzerData.state = APPBUZZER_ESTADO_PULSO_EN_SILENCIO;
         }
         appbuzzerData.retardo = CORETIMER_CounterGet();
    }
    break;
</code></pre>


When 200 ms have passed, the variable pulse_counter is decremented. If it is still greater than zero (if (appbuzzerData. pulse_counter)), the buzzer is activated again and the task changes back to the APPBUZZER_PULSE_STATE_ON_HIGH state.

If it is zero (else), the task changes back to the APPBUZZER_PULSE_STATE_ON_SILENCE state.

This case does not end without first capturing the value that the Core Timer has at that moment.

In the APPBUZZER_PULSE_STATE_ON_SILENCE state, 500 ms are expected, which added to the previous 200, would give 700 ms.

<pre><code>case APPBUZZER_ESTADO_PULSO_EN_SILENCIO:
 {
     if ((CORETIMER_CounterGet() - appbuzzerData.retardo) > _500ms)
     {
           appbuzzerData.contadorPulsos = appbuzzerData.numeroPulsos;
           appbuzzerData.retardo = CORETIMER_CounterGet();
           OCMP1_Enable();
           appbuzzerData.state = APPBUZZER_ESTADO_PULSO_EN_ALTO;
     }
     break;
}</code></pre>

After the delay, the value of the variable pulse_counter is restored using pulse_number, the value of the Core Timer is captured again, the buzzer is activated and the state APPBUZZER_PULSE_STATE_ON_HIGH is returned to.

The function that changes the buzzer task's sleep time would be as follows:

<pre><code>void analizarNivelVoltaje(float voltajeRMS)
{
    if (appbuzzerData.state > APPBUZZER_ESTADO_INICIAL)
    {
        float valorMaximo;
        float valorMinimo;
        if (apppulsadorData.medirFrecuencia)
        {
            valorMaximo = maximaFrecuencia;
            valorMinimo = minimaFrecuencia;
        }
        else
        {
            valorMaximo = maximoVoltaje;
            valorMinimo = minimoVoltaje;
        }
        if (voltajeRMS > valorMaximo)
        {
            if (0x03 == appbuzzerData.numeroPulsos) // Overvoltage alarm is now present
            {
                return;
            }
            appbuzzerData.numeroPulsos = 0x03;
        }
        else if (voltajeRMS < valorMinimo)
        {
            if (0x01 == appbuzzerData.numeroPulsos) // Undervoltage alarm is now present
            {
                return;
            }
            appbuzzerData.numeroPulsos = 0x01;
        }
        else
        {
            appbuzzerData.numeroPulsos = 0x00;
            OCMP1_Disable();//apago el PWM
            appbuzzerData.state = APPBUZZER_ESTADO_REPOSO;
            return;
        }
        appbuzzerData.contadorPulsos = appbuzzerData.numeroPulsos;
        appbuzzerData.retardo = CORETIMER_CounterGet();
        actualizarFrecuenciaPWM(cHH);
        OCMP1_Enable();
        appbuzzerData.state = APPBUZZER_ESTADO_PULSO_EN_ALTO;
    }
}</code></pre>

This function should be called when the RMS value is calculated, and I have put it when the value on the displays is going to be modified:

<p align="center">
<img src="https://pbs.twimg.com/media/GBPBXhmXsAAU__I?format=png&name=900x900" width="700">
</p>
