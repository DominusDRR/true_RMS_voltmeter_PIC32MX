First I create an unsigned char array of length 4, for each digit of the 7-segment display.

This array needs a variable as an index or pointer, and since it is only 4 digits, it can also be an unsigned char.

I also need a signed int variable that I will use to create asynchronous delays in this task.

<pre><code>typedef struct
{
    /* The application's current state */
    APPDISPLAY_STATES state;
    /* TODO: Define any additional data used by the application. */
    unsigned char arregloDisplay[0x04]; //array of length 4
    unsigned char punteroDisplay; //index for aarray
    unsigned int retardo; // Variable used to capture the value of the Core Timer and create non-blocking delays
} APPDISPLAY_DATA;
</code></pre>

Being a common anode display, the values ​​to be represented by each digit would be the following:

<p align="center">
 <img src="https://pbs.twimg.com/media/Gda5se3WIAABoz2?format=png&name=900x900" width="700">
</p>

I can make a function to return each value based on a variable in one statement, but I prefer to use an array in flash memory. This array will be in the c file of the assignment.

<pre><code>//                                          0     1    2    3    4    5   6    7    8   9     0.   1.   2.    3.   4.   5.   6.   7.   8.   9. 
const unsigned char digitoDisplay[0x14] = {0xC0,0xF9,0xA4,0xB0,0x99,0x92,0x82,0xF8,0x80,0x98,0x40,0x79,0x24,0x30,0x19,0x12,0x02,0x78,0x00,0x18};</code></pre>
*Note that from the tenth digit onwards, numbers from 0 to 9 are represented again but with a decimal point. This implies that if you want to represent a number with a decimal point, you must add 10. For example, if you want to represent 7, the corresponding symbol would be from position 17.*

Port B is not 8 bits, meaning that what is going to be written to it must be masked so as not to alter bit 8, bit 9, etc. of that port.

It would be something like this with an or at the byte level:

<pre><code>LATB = 0xFFFFFF00 | value obtained from the array</code></pre>

Or, I can use Harmony's abstraction to manipulate the port:

<pre><code>void GPIO_PortWrite(GPIO_PORT port, uint32_t mask, uint32_t value);</code></pre>

Where port is obviously the port, mask indicates which bits you want to alter and value is the value you want to write.

The mask must be 1 for the bits you want to be altered, and 0 otherwise.

In my case it would be like this:

<pre><code>GPIO_PortWrite(GPIO_PORT_B, 0x0000FF,  valor obtenido del arreglo);</code></pre>

When initializing the task, I set the display pointer and the value for each digit to zero:

<pre><code>void APPDISPLAY_Initialize ( void )
{
    /* Place the App state machine in its initial state. */
    appdisplayData.state = APPDISPLAY_STATE_INIT;
 
    /* TODO: Initialize your application's state machine and other
     * parameters.
     */
    appdisplayData.punteroDisplay = 0x00; //punteroDisplay points to the first display
    appdisplayData.arregloDisplay[0x00] = 0x00; //initial value at zero
    appdisplayData.arregloDisplay[0x01] = 0x00; //initial value at zero
    appdisplayData.arregloDisplay[0x02] = 0x00; //initial value at zero
    appdisplayData.arregloDisplay[0x03] = 0x00; //initial value at zero
}</code></pre>

Empirically from other projects I have determined that the activation frequency of each display can be 1 ms, obviously it would be necessary to test it with the hardware, but I leave it for now.

In the first state of the task that I renamed ESTADO_INICIAL, I capture the value of the Core Timer, to move on to the next state that I have called MOSTRANDO_DATOS.

<pre><code>void APPDISPLAY_Tasks ( void )
{
 
    /* Check the application's current state. */
    switch ( appdisplayData.state )
    {
        /* Application's initial state. */
        case APPDISPLAY_ESTADO_INICIAL: //initial state
        {
           
            appdisplayData.retardo = CORETIMER_CounterGet();
            appdisplayData.state = APPDISPLAY_ESTADO_MOSTRANDO_DATOS;
            break;
        }
 
        case APPDISPLAY_ESTADO_MOSTRANDO_DATOS://status displaying data on the display
        {
 
            break;
        }
 
        /* TODO: implement your application state machine.*/
        /* The default state should never be executed. */
        default: break;  /* TODO: Handle error in application's state machine. */
    }
}</code></pre>

In the second state of the task, you should wait 1 ms to change the information to each digit on the display:

<pre><code>case APPDISPLAY_ESTADO_MOSTRANDO_DATOS:
{
       if ((CORETIMER_CounterGet() - appdisplayData.retardo) > _1ms) 
       {
                
       }
       break;
}</code></pre>

The constant _1ms is calculated based on the clock cycles counted by the CORE TIMER.

This timer works at half the system frequency, i.e. 24 MHz, meaning that each cycle is 41.666 ns, so to get 1 ms, it would be 1ms/41.666ns = 24000 or 0x5DC0

<pre><code>#define _1ms 0x5DC0</code></pre>

To access the value you want to display, you must use the array containing the value using the display pointer variable, like this:

<pre><code>appdisplayData.arregloDisplay[appdisplayData.punteroDisplay]</code></pre>

But this value is not suitable to be displayed on the screen, so the array created in flash memory is used, that is:

<pre><code>digitoDisplay[appdisplayData.arregloDisplay[appdisplayData.punteroDisplay]]</code></pre>

To put this value in port B, using the function to write a number to port, using harmony 3 it would be as follows:

<pre><code>GPIO_PortWrite(GPIO_PORT_B, 0x0000FF, (uint32_t) digitoDisplay[appdisplayData.arregloDisplay[appdisplayData.punteroDisplay]]);</code></pre>

And the base of each PNP transistor that controls the display digit must also be activated, which is something I have not considered so far.

By default, in the MCC, pins RB8, RB9, RB12 and RB13 are configured to logic 1 (each display digit off), but I could set them to logic 1 at the start of the task:

<pre><code>void APPDISPLAY_Initialize ( void )
{
    /* Place the App state machine in its initial state. */
    appdisplayData.state = APPDISPLAY_ESTADO_INICIAL;
 
    /* TODO: Initialize your application's state machine and other
     * parameters.
     */
    /*** I turn off each of the digits on the display ***/
    DISPLAY1_Set();
    DISPLAY2_Set()
    DISPLAY3_Set();
    DISPLAY4_Set();
    /***********************************************/
    appdisplayData.punteroDisplay = 0x00; //punteroDisplay points to the first display
    appdisplayData.arregloDisplay[0x00] = 0x00; //initial value at zero
    appdisplayData.arregloDisplay[0x01] = 0x00; //initial value at zero
    appdisplayData.arregloDisplay[0x02] = 0x00; //initial value at zero
    appdisplayData.arregloDisplay[0x03] = 0x00; //initial value at zero
}</code></pre>

What I need is to use the same pointer of the display array to select terminals RB8, RB9, RB12 and RB13.

With the following function, I could activate each digit of the display.

<pre><code>GPIO_PortClear(GPIO_PORT_B,bit del puerto);</code></pre>

Then I again write a 4-byte array into flash memory to manipulate the 4 bits.

<pre><code>//                                             bit8   bit9  bi12    bit13
const unsigned short bitsDigitoDisplay[0x04] = {1<<8, 1<<9, 1<<12, 1<<13};</code></pre>

So it would be like this, but it is still incomplete:

<pre><code>case APPDISPLAY_ESTADO_MOSTRANDO_DATOS:
{
      if ((CORETIMER_CounterGet() - appdisplayData.retardo) > _1ms) 
      {
          GPIO_PortWrite(GPIO_PORT_B, 0x0000FF, (uint32_t) digitoDisplay[appdisplayData.arregloDisplay[appdisplayData.punteroDisplay]]);
          GPIO_PortClear(GPIO_PORT_B,bitsDigitoDisplay[appdisplayData.punteroDisplay]);
          appdisplayData.punteroDisplay++;
      }
      break;
}</code></pre>

I need to turn off the display before it is going to be activated:

<pre><code>GPIO_PortSet(GPIO_PORT_B,bitsDigitoDisplay[appdisplayData.punteroDisplay - 0x01]);</code></pre>

but that is not feasible if punteroDisplay is equal to zero, so it would be like this:

<pre><code>if (appdisplayData.punteroDisplay > 0x00)
 {
     GPIO_PortSet(GPIO_PORT_B,bitsDigitoDisplay[appdisplayData.punteroDisplay - 0x01]);
 }
 else // si appdisplayData.punteroDisplay  es cero, se debe apagar el 4to dígito
 {
      GPIO_PortSet(GPIO_PORT_B,bitsDigitoDisplay[0x03]);
 }</code></pre>

It is also necessary to verify that when incrementing the pointer, it should not exceed 3:

<pre><code>if (appdisplayData.punteroDisplay > 0x03)
{
   appdisplayData.punteroDisplay = 0x00;
}</code></pre>

After all this, the task must again capture a value from the CORE TIMER, to generate another delay.

So the second state of the task would be like this:

<pre><code>case APPDISPLAY_ESTADO_MOSTRANDO_DATOS:
{
    if ((CORETIMER_CounterGet() - appdisplayData.retardo) > _1ms) 
    {
        if (appdisplayData.punteroDisplay > 0x00)
        {
            GPIO_PortSet(GPIO_PORT_B,bitsDigitoDisplay[appdisplayData.punteroDisplay - 0x01]); //Turn off to previous digit
        }
        else //if appdisplayData.punteroDisplay is zero, the 4th digit should be turned off
        {
            GPIO_PortSet(GPIO_PORT_B,bitsDigitoDisplay[0x03]);
        }
        GPIO_PortWrite(GPIO_PORT_B, 0x0000FF, (uint32_t) digitoDisplay[appdisplayData.arregloDisplay[appdisplayData.punteroDisplay]]);
        GPIO_PortClear(GPIO_PORT_B,bitsDigitoDisplay[appdisplayData.punteroDisplay]);
        appdisplayData.punteroDisplay++;
        if (appdisplayData.punteroDisplay > 0x03)
        {
            appdisplayData.punteroDisplay = 0x00;
        }
        appdisplayData.retardo = CORETIMER_CounterGet(); // I capture another Core Timer value again for another delay
    }
    break;
}</code></pre>

This task would be working independently, any value that you want to show on the display must be done from another task or process by modifying the values ​​of the appdisplayData.arregloDisplay array.

/////////////////////////////////////////////////////////////////////////

To display the RMS value (or the frequency value) on the display, a function called rmsDisplay was created.

The function first obtains its integer and decimal parts, since the parameter it needs is of the float type.

The function evaluates whether the value to be displayed is greater than 99, that is, from then on, if it is greater than 9 and less than or equal to 99, and from 0 to 9.

For the first case, the integer value is placed in the first 3 digits, the second being the one with the decimal point and the fourth, showing the decimal part.

For the second case, the integer part is shown in the first two digits, the third has the decimal point, and the third and fourth show the decimal part.

For the last case, only the first digit shows the integer part.

<pre><code>oid rmsDisplay(float valorRMS)
{
    analizarNivelVoltaje(valorRMS);
    int parteEntera = (int)floorf(valorRMS);
    float parteDecimal = (float)(valorRMS - parteEntera);
    if (parteEntera > 99)
    {
        appdisplayData.arregloDisplay[0x03] = (unsigned char)floorf(parteDecimal *10); 
        appdisplayData.arregloDisplay[0x02] = 10 + (unsigned char)(parteEntera%10); // sumo diez para activar el punto decimal
        appdisplayData.arregloDisplay[0x01] = (unsigned char)((parteEntera/10)%10); 
        appdisplayData.arregloDisplay[0x00] = (unsigned char)(parteEntera/100);
    }
    else if (parteEntera > 9)
    {
        int decimalDosDigitos = (int)floorf(parteDecimal *100); 
        
        appdisplayData.arregloDisplay[0x03] = (unsigned char)(decimalDosDigitos%10);
        appdisplayData.arregloDisplay[0x02] = (unsigned char)(decimalDosDigitos/10); 
        
        appdisplayData.arregloDisplay[0x01] = 10 +(unsigned char)(parteEntera%10); // sumo diez para activar el punto decimal
        appdisplayData.arregloDisplay[0x00] = (unsigned char)(parteEntera/10);
    }
    else
    {
        int decimalTresDigitos = (int)floorf(parteDecimal *1000); 
        
        appdisplayData.arregloDisplay[0x03] = (unsigned char)(decimalTresDigitos%10); 
        appdisplayData.arregloDisplay[0x02] = (unsigned char)((decimalTresDigitos/10)%10); 
        appdisplayData.arregloDisplay[0x01] = (unsigned char)(decimalTresDigitos/100); // obtengo centena
        
        appdisplayData.arregloDisplay[0x00] = (unsigned char)(parteEntera);
    }
}</code></pre>
