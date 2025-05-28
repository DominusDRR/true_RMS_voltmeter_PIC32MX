The task that manages the pulsed pulses performs two actions, switching between the two operating modes, voltmeter and frequency meter; and eliminates the mechanical bounce.

The task has 3 states, the initial one, the one that eliminates the bounce and the last one that waits for the button to return to 1, if the user keeps it pressed.

<pre><code>typedef enum
{
    /* Application's state machine's initial state. */
    APPPULSADOR_ESTADO_INICIAL=0,
    APPPULSADOR_ESTADO_ELIMINAR_REBOTE,
    /* TODO: Define states used by the application state machine. */
    APPPULSADOR_ESTADO_ESPERAR_PULSADOR_REGRESE_A_1,
} APPPULSADOR_STATES;
</code></pre>

The data structure, apart from the state variable, has a flag that is used to determine the operating mode (voltmeter or frequency meter) and another 32-bit variable to be used to generate asynchronous delays.

<pre><code>typedef struct
{
    /* The application's current state */
    APPPULSADOR_STATES state;
    /* TODO: Define any additional data used by the application. */
    bool medirFrecuencia;
    int retardo;
} APPPULSADOR_DATA;
</code></pre>

In the initial state, the task is obviously initialized with the first state, and by default, the operation mode is voltmeter, that is, the medirFrecuencia flag is false.


<pre><code>void APPPULSADOR_Initialize ( void )
{
    /* Place the App state machine in its initial state. */
    apppulsadorData.state = APPPULSADOR_ESTADO_INICIAL;
    /* TODO: Initialize your application's state machine and other
     * parameters.
     */
    apppulsadorData.medirFrecuencia = false; // voltmeter mode
}</code></pre>


In the first state of the task, it waits for the button to be pressed and for the task that controls the buzzer to be at rest or in a higher state.

If so, medirFrecuencia switches state, that is, if it is true, it changes to false and vice versa, next the cambiarProceso function is called, which will change the operating states of the device, then the instantaneous value of the central timer is captured and the next state of the task is passed on.

<pre><code>case APPPULSADOR_ESTADO_INICIAL:
{
    if (false == PULSANTE_Get() && appbuzzerData.state >= APPBUZZER_ESTADO_REPOSO) // I wait for the button to be pressed
    {
        apppulsadorData.medirFrecuencia = !apppulsadorData.medirFrecuencia; //change the state of the boolean variable
        cambiarProceso();
        apppulsadorData.retardo = CORETIMER_CounterGet();
        apppulsadorData.state = APPPULSADOR_ESTADO_ELIMINAR_REBOTE;
     }
     break;
}</code></pre>

The cambiarProceso function restarts the tasks that perform ADC sampling and RMS calculation.

<pre><code>void cambiarProceso(void)
{
    APPSAMPLING_Initialize();   // Restart the sampling task
    APPRMS_Initialize();        // I restart the task that calculates the RMS value
}
</code></pre>


The debounce state waits for 200ms with an asynchronous delay before continuing.

The 200ms value is empirical, and it works very well for me.

After that delay, it is verified that the user has released the button to return to the initial state, otherwise, to the state where it waits for the user to stop pressing the button.

<pre><code> case APPPULSADOR_ESTADO_ELIMINAR_REBOTE:
{
   if ((CORETIMER_CounterGet() - apppulsadorData.retardo) > _200ms) 
   {
      if (false == PULSANTE_Get()) // the user continues to press the button
      {
         apppulsadorData.state = APPPULSADOR_ESTADO_ESPERAR_PULSADOR_REGRESE_A_1;
      }
      else
      {
         apppulsadorData.state = APPPULSADOR_ESTADO_INICIAL; // returns to wait for another keystroke
      }
    }
    break;
 }</code></pre>

In the state that waits for the user to release the button, that is, it is set to true.

Once the user releases the button, the state that eliminates the mechanical bounce is returned to.

<pre><code>
case APPPULSADOR_ESTADO_ESPERAR_PULSADOR_REGRESE_A_1:
{
    if (PULSANTE_Get()) // wait until the pulse returns to logic 1
    {
       apppulsadorData.retardo = CORETIMER_CounterGet();
       apppulsadorData.state = APPPULSADOR_ESTADO_ELIMINAR_REBOTE;
    }
    break;
}
/* The default state should never be executed. */
default: break; /* TODO: Handle error in application's state machine. */</code></pre>
