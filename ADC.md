The first basic conceptual idea is that the square signal that would be generated synchronized with the 60Hz network would be useful to start and stop the acquisition of data or samples from the ADC.

The start and stop would be by the rising or positive edge of said signal, as follows:

<p align="center">
<img src="https://pbs.twimg.com/media/GdkeeGaXwAAaobM?format=jpg&name=large" width="700">
</p>

The problem is that I don't know how many machine cycles it takes to calculate the RMS value (it could be measured), as you can see in the image above, once the data acquisition is finished, the CPU would concentrate on performing that calculation.

I don't think it would take a long time, but nevertheless, it would have to wait for the next cycle of the sinusoidal signal to perform a new data acquisition, I don't think it would affect the accuracy of the calculation, but I don't like the idea of ​​wasting time doing that process.

The second option is to always be acquiring the data from the ADC.

This would imply using two buffers or arrays to store the information.

In the first cycle, it would save in the first array, then the task that calculates the RMS would perform its calculations with that array, but in that same cycle, the ADC information would be saved in the second array.

Then at the end of the second cycle, the RMS would be calculated with the second array, and the information would be stored in the first one, something like this image:

<p align="center">
<img src="https://pbs.twimg.com/media/GdkfjiVWgAAzLN0?format=jpg&name=large" width="700">
</p>

So far, the idea that information is always being acquired from the sinusoidal signal seems to work, however, it is possible that the RMS calculation operations are atomic instructions, that is, they cannot be divided into more subfunctions, and executing them takes some time, so that, for example, if you are waiting for the data acquisition time, that time is longer, because some part of the RMS calculation is not finished.

The acquisition time is 1 us, the conversion time is approximately 14 us, and between each sample, you must wait 100 us:

<p align="center">
<img src="https://pbs.twimg.com/media/Gdkgab8WkAAcyQJ?format=png&name=large" width="700">
</p>


I was thinking about using polled mode, meaning I wait for a flag to finish, to know if the ADC conversion is ready. But as I mentioned, the CPU could be running an RMS calculation process, and until it finishes and checks that ADC flag again, the time will be longer, and possibly generate some precision error.

Also, I use asynchronous delays, which are not precise, and are used for tasks that are not so critical, for example, in the displays, each one shows information at a rate of 1000 us (1ms), but if for some moment it were 1015 us, for example, the user will not notice that.

Regarding the operation of the microcontroller's ADC.

The ADC has a 16-word buffer that I thought would be useful for acquiring 16 samples automatically.

<p align="center">
<img src="https://pbs.twimg.com/media/Fx33TKqX0AECtuF?format=jpg&name=medium" width="700">
</p>

Now, the signal you want to sample with the ADC is approximately 60 Hz or a period of 16.667 ms.

The sampling must be much less than that time, for example if it were 100 times smaller, it would be 166.667 us, so you could choose a sampling rate of 100 us or 10 ksps.

Here you have to consider the acquisition time and the conversion time.

<p align="center">
<img src="https://pbs.twimg.com/media/Fx34c8VWABYoVNj?format=jpg&name=medium" width="700">
</p>

According to the ADC converter section manual, the signal acquisition time is given by the ADCS bits of the ADCON3 register

<p align="center">
<img src="https://pbs.twimg.com/media/Fx35F46WAA0dmWg?format=png&name=small" width="700">
</p>

The minimum time must be greater than 83.33 ns.

<p align="center">
<img src="https://pbs.twimg.com/media/Fx35dIjWAAgdjUi?format=png&name=900x900" width="700">
</p>

And in my case TpB is 1/48MHz.

<p align="center">
<img src="https://pbs.twimg.com/media/Fx35q0HWABEqYiG?format=png&name=small" width="700">
</p>

Looking for a correct value for the ADCS bits, I can say that 1 us acquisition time can be achieved:

<p align="center">
<img src="https://pbs.twimg.com/media/Fx36IG1WAAc0Qha?format=png&name=small" width="700">
</p>

According to the manual, the conversion time is 12 times TAD, plus 1 or 2 additional TADs, therefore the conversion time would be approximately 14 us

<p align="center">
<img src="https://pbs.twimg.com/media/Fx3824bXoAMG3G7?format=jpg&name=large" width="700">
</p>

That is, the sampling would be at a rate of 15 us or 66.67 ksps.

This would allow for more samples, i.e. a more precise calculation, or perhaps use a delay to wait for the 100 us.

So I am forced to use interrupts, both, for the completion of the ADC conversion and for the generation of 100 us.

In that way, I think the latency produced by the RMS calculation process would be improved.

For the ADC converter interrupt, I take the MHCP example called adc_interrupt_pic32mx470_curiosity

And the example is as follows:

<pre><code>void ADC_ResultHandler(uintptr_t context)
{
    /* Read the ADC result */
    adc_count = ADC_ResultGet(ADC_RESULT_BUFFER_0);
    result_ready = true;
}
 
 
int main ( void )
{
    /* Initialize all modules */
    SYS_Initialize ( NULL );
    ADC_CallbackRegister(ADC_ResultHandler, (uintptr_t)NULL);
 
    printf("\n\r---------------------------------------------------------");
    printf("\n\r                    ADC Demo                 ");
    printf("\n\r---------------------------------------------------------\n\r");
 
    /* Start ADC conversion */
    ADC_ConversionStart();
 
 
    while (1)
    {
        /* Maintain state machines of all polled MPLAB Harmony modules. */
        SYS_Tasks ( );
 
        /* Auto sampling mode is used, so no code is needed to start sampling */
 
        /* Wait till ADC conversion result is available */
        if(result_ready == true)
        {
            result_ready = false;
            input_voltage = (float)adc_count * ADC_VREF / ADC_MAX_COUNT;
            printf("ADC Count = 0x%03x, ADC Input Voltage = %d.%02d V \r", adc_count, (int)input_voltage, (int)((input_voltage - (int)input_voltage)*100.0));
 
            /* Start ADC conversion */
            ADC_ConversionStart();
        }
    }
 
    /* Execution should not come here during normal operation */
 
    return ( EXIT_FAILURE );
}</code></pre>

It can be assigned or registered with the ADC_CallbackRegister function, which is the function that will handle the interruption at the end of the ADC conversion.

This function is called ADC_ResultHandler, where it obtains the value of the conversion and saves it in the variable adc_count

The configuration of this peripheral, with the graphic area of ​​the MCC is as shown:

<p align="center">
<img src="https://pbs.twimg.com/media/F6uQJTSWoAAk-qI?format=png&name=large" width="700">
</p>

Regarding the interruption by a timer, the example that I am going to take as a reference is called tmr_timer_mode_pic32mx_12_sk

<pre><code>/* This function is called after period matches in Timer 3 (32-bit timer) */
void TIMER2_InterruptSvcRoutine(uint32_t status, uintptr_t context)
{
    /* Toggle LED */
    LED1_Toggle();
}
// *****************************************************************************
// *****************************************************************************
// Section: Main Entry Point
// *****************************************************************************
// *****************************************************************************
 
int main ( void )
{
    /* Initialize all modules */
    SYS_Initialize ( NULL );
    
    TMR2_CallbackRegister(TIMER2_InterruptSvcRoutine, (uintptr_t) NULL);
    TMR2_Start();
 
    while ( true )
    {
        /* Maintain state machines of all polled MPLAB Harmony modules. */
        SYS_Tasks ( );
    }
 
    /* Execution should not come here during normal operation */
 
    return ( EXIT_FAILURE );
}</code></pre>

The function that registers the one that is called when the interruption occurs is called TMR2_CallbackRegister and is on line 18.

And the TIMER2_InterruptSvcRoutine function only toggles the logical state of an LED.

This example uses timer 2 in 32-bit mode, which means that timer 3 works together to achieve that bit width.

!https://pbs.twimg.com/media/F6uTFG7XsAAGTGU?format=png&name=900x900!

In the TMR2 module, it is configured to 32 bits, and it is indicated that the enabling of the interrupt must be done in the secondary or slave timer, in this case it is the TMR3, and its configuration is:

!https://pbs.twimg.com/media/F6uTwTRXsAALP56?format=png&name=medium!

I was initially thinking of using the capture module to measure the frequency of the AC signal, it also has an external interruption:

!https://pbs.twimg.com/media/F6zSRRFWoAA0Ox0?format=png&name=small!

In theory, only one of the two options could be used, but using the System module, you could enable that interrupt:

<p align="center">
<img src="https://pbs.twimg.com/media/F6zSxJaWIAA6Cow?format=png&name=small" width="700">
</p>

Then I create a new group, called adquisicion:

Within this new group, I add the ADC module, which has the following configurations.

- The interrupt is enabled
- The clock source is from the 48 MHz peripheral bus.
- The clock divider is 23, to get 1000 ns (1 us) of acquisition time, as I mentioned at the beginning.
- The voltage reference is VREF+ and AVSS (GND) where VERF + will be 3.0V
- Channel 0 of the ADC will be connected to the analog input AN1.
- The result of the conversion will be a 32-bit integer number

<p align="center">
<img src="https://pbs.twimg.com/media/F6zTXZLXgAANRCM?format=png&name=medium" width="700">
</p>

I also add timer 4, in 16-bit mode to generate 100 us:

<p align="center">
<img src="https://pbs.twimg.com/media/F6zUilKXIAAKRqj?format=png&name=900x900" width="700">
</p>

Using the Core module, I create a new task called appSampling, which will be in charge of initializing the enabling of the external interrupt, and this in turn will enable the ADC and TMR4 interrupts at the same time:
<p align="center">
<img src="https://pbs.twimg.com/media/F6zUyWHXUAAxsYy?format=png&name=medium!
</p>

I have a structure where I have an array that will be the buffer that stores the data acquired by the ADC and an 8-bit variable that is a pointer to that array:

<pre><code>typedef struct
{
    unsigned short  bufferADC[256]; // stores each sample acquired by the ADC
    unsigned char   muestra;        // points to each of the ADC buffer elements
}EstructuraADC;</code></pre>

While the task data structure is like this:

<pre><code>typedef struct
{
    /* The application's current state */
    APPSAMPLING_STATES state;
    /* TODO: Define any additional data used by the application. */
    EstructuraADC   estructuraADC[0x02]; //There are two that alternate, one acquires and with the other the RMS is calculated.
    unsigned char   punteroABuffer; // Points to punteroABuffer, i.e. it is 0 or 1
    bool    periodoCompleto; //Reports that one period of the AC signal has been completed
    uint32_t captura[0x02]; //Captures the value to measure the period of the AC signal.
    unsigned char indiceCaptura; // Points to captura, i.e. it is 0 or 1
} APPSAMPLING_DATA;
</code></pre>

The start of the task is as shown in the following code snippet:

<pre><code>void APPSAMPLING_Initialize ( void )
{
    /* Place the App state machine in its initial state. */
    appsamplingData.state = APPSAMPLING_ESTADO_INICIAL;
    /* TODO: Initialize your application's state machine and other
     * parameters.
     */
    appsamplingData.punteroABuffer  = 0x01; //I point to the second buffer, so that with INT2, it starts at 0 (First buffer)
    appsamplingData.estructuraADC[0x00].muestra = 0x00; // muestra (sample) buffer 0 initialized
    appsamplingData.estructuraADC[0x01].muestra = 0x00; // muestra (sample) buffer 1 initialized
    
    appsamplingData.periodoCompleto = false;
}</code></pre>

The task is basically just two states:

<pre><code>void APPSAMPLING_Tasks ( void )
{
    /* Check the application's current state. */
    switch ( appsamplingData.state )
    {
        /* Application's initial state. */
        case APPSAMPLING_ESTADO_INICIAL:
        {
            if (apppulsadorData.medirFrecuencia) // apppulsadorData.medirFrecuencia is variable of the task that manages the push button
            {
                /** I disable everything related to the calculation of the RMS value **/
                EVIC_ExternalInterruptDisable(EXTERNAL_INT_2);  //I stop the interrupt by external interruption
                EVIC_SourceStatusClear(INT_SOURCE_EXTERNAL_2);  //clear flag due to external interruption
                TMR4_Stop();                                    //I stop timer 4
                TMR4_InterruptDisable();                        //I disable timer interrupt 4
                EVIC_SourceStatusClear(INT_SOURCE_TIMER_4);     //clear tmr4 interrupt flag
                ADC_Disable();                                  //I disable the ADC module
                EVIC_SourceStatusClear(INT_SOURCE_ADC);         //clear the flag due to ADC interruption
                /*************************************************/
                ICAP1_Enable();//I enable the capture module
                TMR2_Start(); //I start tmr2
                ICAP1_CallbackRegister(ManejarInterrupcionModuloCaptura, (uintptr_t)NULL);//The ManejarInterrupcionModuloCaptura function will be called by the capture module interrupt
                appsamplingData.captura[0x00] = 0x0000;//
                appsamplingData.captura[0x01] = 0x0000;//
                appsamplingData.indiceCaptura = 0x00;//
            }
            else
            {
                /** I disable everything related to the calculation of the signal period **/
                ICAP1_Disable(); //I disable the capture module
                TMR2_Stop();// Stop timer 2
                EVIC_SourceStatusClear(INT_SOURCE_INPUT_CAPTURE_1); // clear capture module flag.
                /*************************************************/
                TMR4_Stop();
                TMR4_CallbackRegister(ManejarInterrupcionTMR4, (uintptr_t) NULL);//register function invoked by TMR4 interrupt
                ADC_CallbackRegister(ManejarInterrupcionADC, (uintptr_t)NULL); // register function invoked by ADC interrupt
                EVIC_ExternalInterruptCallbackRegister(EXTERNAL_INT_2, &ManejarInterrupcionExterna, (uintptr_t)NULL); //Register the function that is called by the external interrupt
                EVIC_ExternalInterruptEnable(EXTERNAL_INT_2); // I enable external interrupt 2
                
                TMR4_InterruptEnable();//I enable TMR4 interrupt
                ADC_Enable(); //I enable ADC
            }
            appsamplingData.state = APPSAMPLING_ESTADO_REPOSO; //Task goes to sleep, change with push button
            break;
        }

        case APPSAMPLING_ESTADO_REPOSO: break;
        /* TODO: implement your application state machine.*/
        /* The default state should never be executed. */
        default: break; /* TODO: Handle error in application's state machine. */
    }
}</code></pre>

As you can see, when the voltmeter option is selected, the external interruption is enabled, by positive edge.

This interruption occurs at the beginning of each period of the sinusoidal signal, and the function that is invoked in this interruption is ManejarInterrupcionExterna.

<pre><code>void ManejarInterrupcionExterna (EXTERNAL_INT_PIN pin, uintptr_t context)
{
    /** Cambio de buffer **/
    appsamplingData.punteroABuffer++;   // void ManejarInterrupcionExterna (EXTERNAL_INT_PIN pin, uintptr_t context)
{
    appsamplingData.punteroABuffer++;   // points to the next buffer, one is for acquisition, and the other is for rms calculation.
    if (appsamplingData.punteroABuffer > 0x01)
    {
        appsamplingData.punteroABuffer = 0x00; //Return to buffer 0
    }
    else
    {
        appsamplingData.periodoCompleto = true;//Indicates that a period has been completed, obviously the first time the buffer will have all its bytes at zero.
    }
    appsamplingData.estructuraADC[appsamplingData.punteroABuffer].muestra = 0x00; //clear buffer pointer (sample)
    TMR4_Stop();                        // Stop TMR4:
    TMR4 = 0x0000;                      // Clear/reset the count
    ADC_ConversionStart();              // Start the conversion
    TMR4_Start();                       // start timer 4
}
}</code></pre>

Once timer 4 starts running, it will generate an interrupt every 100 us, and the function that is called at each interrupt is ManejarInterrupcionTMR4.

<pre><code>void ManejarInterrupcionTMR4(uint32_t status, uintptr_t context)
{
    ADC_ConversionStart();  //start the ADC conversion again, at 100 us
}</code></pre>

When the ADC conversion completes, it also generates an interrupt, which in turn invokes the function:

<pre><code>void ManejarInterrupcionADC(uintptr_t context)
{
    appsamplingData.estructuraADC[appsamplingData.punteroABuffer].bufferADC[appsamplingData.estructuraADC[appsamplingData.punteroABuffer].muestra] = ADC_ResultGet(ADC_RESULT_BUFFER_0);
    
    AD1CON1bits.ADON = 0;  // turn off ADC
    IFS0bits.AD1IF = 0;    // clear the interrupt flag
    /***********************/
    appsamplingData.estructuraADC[appsamplingData.punteroABuffer].muestra++;
    //IFS0CLR = _IFS0_AD1IF_MASK; //MCHP's alleged solution
}</code></pre>

*Here I have to explain that there was a bug with the ADC, despite being configured to generate a single conversion, the end of conversion interrupt was generated twice.
Searching the forum, I found someone who suggested turning off the ADC and turning it back on, it is a cumbersome solution, but it works.
MCHP, in an open ticket, claims that writing IFS0CLR = _IFS0_AD1IF_MASK should clear the interrupt flag, since that was the reason that two interrupts were produced, but I have not tested it.*

In the frequency meter mode, however, the module that works with timer 2 captures the value of said timer at the beginning of each period of the sinusoidal signal.

With these two captures, a value is obtained that is the result of the difference of these two values, which corresponds to the measured time.

And the value in Hertz of said value is obtained, to send it to the 4-digit display.


<pre><code>void ManejarInterrupcionModuloCaptura(uintptr_t context)
{
    appsamplingData.captura[appsamplingData.indiceCaptura++] = ICAP1_CaptureBufferRead();
    if (appsamplingData.indiceCaptura > 0x01)
    {
        float diferencia = (float)(appsamplingData.captura[0x01]) -  (float)(appsamplingData.captura[0x00]);
        if (diferencia > 0.00)
        {
            rmsDisplay(750000.000/diferencia);
        }
        appsamplingData.indiceCaptura = 0x00;
    }
}</code></pre>

















