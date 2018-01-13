---
title: "Button interrupt with msp430fr5969 microcontroller"
hide_sidebar: true
---

<p align="center">
  <img src="http://azug.minpet.unibas.ch/~lukas/bricol/msp430/img/msp430FR5969_Launchpad_wolverine1.jpg"/>
</p>


Hi folks, I just felt like writing a brief post on the implementation of button interrupt using the MSP430FR5969 Launchpad by Texas Instrument. Detailed information can be found [here](http://processors.wiki.ti.com/index.php/MSP430_LaunchPad_PushButton).

I'll be using the S2 button (P1.1) as the source of the interrupt. The LED2 (P1.0) turns on/off whenever interrupt occurs. We'll cater to two use cases in this post.

- When interrupt occur every time the button is pushed and pulled; thus toggling the LED once. The code to achieve that is shown below:

```c
#include <msp430fr5969.h>

#define LED BIT0
#define BUTTON BIT1

void main(void){
  WDTCTL = WDTPW + WDTHOLD;       // Stop watchdog timer
  PM5CTL0 &= ~LOCKLPM5;           //Free or unlock pins from high Impedance

  P1DIR |= LED;                   // Set LED (P1.0) to output direction
  P1OUT &= ~LED;                  // Set LED (P1.0) to output low

  P1REN |= BUTTON;                // Enable P1.1 internal resistance. Use PIREN $= ~BUTTON to turn it off
  P1OUT |= BUTTON;                // Set P1.1 as pull-Up resistance. Use P1OUT $= ~BUTTON for pull-down
  P1IES |= BUTTON;                // P1.1 Interrupt Edge Select Hi/Lo.use P1IES &= ~BUTTON for Lo/Hi
  P1IFG &= ~BUTTON;               // P1.1 IFG cleared
  P1IE |= BUTTON;                 // P1.1 interrupt enabled

  __bis_SR_register(GIE);         // Enable interrupts globally

  while(1) {}
}
// Port 1 interrupt service routine
#pragma vector=PORT1_VECTOR
__interrupt void Port_1(void){
    P1OUT^=LED;                   // Toggle the LED
    P1IFG &= ~BUTTON;             // P1.1 IFG cleared
}
```


- When interrupt occur every time the button is pushed or pulled; thus toggling the LED on Hi-Lo and Lo-Hi. This can be achieved by appending this line of code to the ISR. `P1IES &= ~BIT1;` The final ISR should look something like,
```c
// Port 1 interrupt service routine
#pragma vector=PORT1_VECTOR
__interrupt void Port_1(void){
    P1OUT^=LED;                   // Toggle the LED
    P1IFG &= ~BUTTON;             // P1.1 IFG cleared
    P1IES &= ~BIT1;               // P1.1 Lo/Hi edge select
}
```

**NOTE**

Pressing the Button too quickly may lead to inconsistent operation due to the ***bouncing*** issue with buttons, which can be handled through a process called **debouncing**. For more information on that see [here](http://processors.wiki.ti.com/index.php/MSP430_LaunchPad_PushButton).
