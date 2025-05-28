This task has 4 states, one initial, two summations, one to calculate the square root and one of rest or laziness.

<pre><code>typedef enum
{
    /* Application's state machine's initial state. */
    APPRMS_ESTADO_INICIAL=0,
    APPRMS_ESTADO_SUMATORIO_1,
    /* TODO: Define states used by the application state machine. */
    APPRMS_ESTADO_SUMATORIO_2,
    APPRMS_ESTADO_EXTRAER_RAIZ_CUADRADA,         
    APPRMS_ESTADO_IDLE        
} APPRMS_STATES;</code></pre>

It is divided in this way to avoid a complete appropriation of the CPU when performing calculations.

<pre><code>typedef struct
{
    /* The application's current state */
    APPRMS_STATES state; //state machine
    /* TODO: Define any additional data used by the application. */
    float sumatorio1; //Result of summation 1
    float sumatorio2; //Result of summation 2
    unsigned char indice; //Used to perform a non-preemptive for loop
    unsigned char puntero; // Allows me to select between the two buffers that store ADC information
    //unsigned int retardo; //variable for asynchronous delays
} APPRMS_DATA;</code></pre>

In the initial state, if the working mode is frequency meter, the task goes to sleep, otherwise the RMS calculation is performed.

<pre><code>void APPRMS_Initialize ( void )
{
    if (apppulsadorData.medirFrecuencia)
    {
       apprmsData.state = APPRMS_ESTADO_IDLE;
    }
    else
    {
        /* Place the App state machine in its initial state. */
        apprmsData.state = APPRMS_ESTADO_INICIAL;
        /* TODO: Initialize your application's state machine and other
        * parameters.
        */
    }
}</code></pre>

In the first state, we wait for a period of the AC signal to have been sampled, if so, we initialize the variables, select the buffer that will calculate the RMS value and jump to the second state that performs the calculation of the first summation.

<pre><code>case APPRMS_ESTADO_INICIAL:
{
     if (appsamplingData.periodoCompleto)
     {
          apprmsData.indice = 0x00;
          apprmsData.sumatorio1 = 0.0;
          apprmsData.sumatorio2 = 0.0;
          apprmsData.puntero = (appsamplingData.punteroABuffer == 0x01)? 0 : 1;
          appsamplingData.periodoCompleto = false;
          apprmsData.state = APPRMS_ESTADO_SUMATORIO_1;
      }
     break;
}</code></pre>

The first summation is to add each item squared in the buffer.
This summation is done once per state, which means that it does not take up the CPU.

Once this summation is finished, it is multiplied by m squared, (m is the slope indicated in literal h. RMS value calculation analysis)

<pre><code>case APPRMS_ESTADO_SUMATORIO_1: // sumatorio(x^2)
{
   if (apprmsData.indice < appsamplingData.estructuraADC[apprmsData.puntero].muestra)
   {
     apprmsData.sumatorio1 += powf((float)(appsamplingData.estructuraADC[apprmsData.puntero].bufferADC[apprmsData.indice]),2);
     apprmsData.indice++;
   }
   else
   {
       apprmsData.indice = 0x00;
       apprmsData.sumatorio1 = 0.34399*apprmsData.sumatorio1; // m^2* sumatorio1
       apprmsData.state = APPRMS_ESTADO_SUMATORIO_2;
   }
   break;
}</code></pre>

The second summation (also not CPU-intensive) performs the sum of each of the items in the buffer being analyzed.

At the end, this sum is multiplied by 2mc, see literal h (RMS value calculation analysis)

Both summations (1 and 2) are added plus the number of samples times the constant c squared.

And finally, that total sum is divided by the number of samples.

<pre><code>case APPRMS_ESTADO_SUMATORIO_2:
{
       if (apprmsData.indice < appsamplingData.estructuraADC[apprmsData.puntero].muestra)
       {
           apprmsData.sumatorio2 += (float)(appsamplingData.estructuraADC[apprmsData.puntero].bufferADC[apprmsData.indice]);
           apprmsData.indice++;
       }
       else
       {
           apprmsData.sumatorio2 = -351.9062*apprmsData.sumatorio2; //2mc* sumatorio2(x)
          apprmsData.sumatorio1 = apprmsData.sumatorio1 + apprmsData.sumatorio2 + 90000*(float)           (appsamplingData.estructuraADC[apprmsData.puntero].muestra)/* - 90000*/;  // (N-1)*C^2 
          apprmsData.sumatorio1 = apprmsData.sumatorio1/((float)appsamplingData.estructuraADC[apprmsData.puntero].muestra);
          apprmsData.state = APPRMS_ESTADO_EXTRAER_RAIZ_CUADRADA;
       }
       break;
}
</code></pre>

The fourth state is simply to obtain the square root of the calculation performed in the previous case and send it to the display task.

<pre><code> case APPRMS_ESTADO_EXTRAER_RAIZ_CUADRADA:
        {
            rmsDisplay(sqrtf(apprmsData.sumatorio1)); //square root and send it to display
            apprmsData.state = APPRMS_ESTADO_INICIAL;
            break;
        }</code></pre>



