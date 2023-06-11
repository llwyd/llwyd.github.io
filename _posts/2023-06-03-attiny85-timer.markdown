---
layout: post
title:  "0x03 - Improvising a 555 timer using an ATTiny85"
date:   2023-06-04 17:45:00 +0100
categories: attiny85
---

In short, I needed a 1Hz timer for a shift register project and didn't have any timing ICs around, so I improvised using an ATTINY85. The datasheet [^1] in section 12.2.2 says that Timer/Counter1 can be used in PWM mode with two outputs, one normal and one inverted:

![scope](/assets/timer85.png)
*Figure 1 - Timer1 in PWM mode with output pairs.*

This was exactly what I needed for clocking the shift register I was using (74HC595) that had shift and storage clocks.

{% highlight c %}
#define F_CPU 1000000UL

#include <avr/io.h>

static void ConfigurePWM( void )
{
    /* Reset counter */
    TCCR1 = 0U;
    
    /* PB4 = PWM output, PB3 = PWM inverted output */
    DDRB |= ( 1U << 4U );
    DDRB |= ( 1U << 3U );
    
    /* Clear timer on compare match,
     * PWMA enable, 
     * Toggle OC1A output line,
     * Precale clock CK/8192
     */
    TCCR1 = 0xDE;

    /* PWMB enable,
     * Toggle OC1B output line
     */
    GTCCR = 0x50;

    /* 1MHz / 8192 / 122 ~= 1Hz */
    OCR1A = 61U;
    OCR1B = 61U;
    OCR1C = 122U; 
}

void main( void )
{
    ConfigurePWM();
    while( 1 );
}
{% endhighlight %}

Using a logic analyser this is what it looks like on the output:

[^1]: Atmel 8-bit AVR Microcontroller with 2/4/8K Bytes In-System Programmable Flash [link](https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-2586-AVR-8-bit-Microcontroller-ATtiny25-ATtiny45-ATtiny85_Datasheet.pdf)

