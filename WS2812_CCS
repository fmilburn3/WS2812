/* WS2812_SPI
 *
 * SPI Master mode used as means to output instructions to WS2812 LED display
 * using a TI MSP430G2553 microcontroller.
 *
 *  *
 * Connections:
 *   LaunchPad      LED Strip
 *   ---------      ---------
 *   3V3            5VDC
 *   P1_2(MOSI)     DIN
 *   P1_1(MISO)     not connected
 *   P1_4(CLK)      not connected
 *   GND            GND
 *
 * How to use:
 *  ledsetup ();         - Get ready to send.
 *                         Call once at the beginning of the program.
 *  sendPixel (r, g, b); - Send a single pixel to the string.
 *                         Call this once for each pixel in a frame.
 *                         Each colour is in the range 0 to 255. Turn off
 *                         interrupts before use and turn on after all pixels
 *                         have been programmed.
 *  show ();             - Latch the recently sent pixels onto the LEDs .
 *                         Call once per frame.
 *  showColor (count, r, g, b); - Set the entire string of count Neopixels
 *                         to this one colour. Turn off interrupts before use
 *                         and remember to turn on afterwards.
 *
 * Derived from NeoPixel display library by Nick Gammon
 * https://github.com/nickgammon/NeoPixels_SPI
 * With ideas from:
 * http://wp.josh.com/2014/05/13/ws2812-neopixels-are-not-so-finicky-once-you-get-to-know-them/
 * Released for public use under the Creative Commons Attribution 3.0 Australia License
 * http://creativecommons.org/licenses/by/3.0/au/
 *
 * F Milburn  December 2016
 */

#include <msp430.h>

#define SPIDIV     0x02                         // 16 MHz/2 gives ~125 ns for each on bit in byte
#define SPILONG    0b11111100                   // 750 ns (acceptable "on" range 550 to 850 ns)
#define SPISHORT   0b11100000                   // 375 ns (acceptable "on" range 200 to 500 ns)

const unsigned int PIXELS = 8;                  // Pixels in the strip

void initClock();
void initSPI();
void initWS2812();
void sendByte (unsigned char b);
void sendPixel (unsigned char r, unsigned char g, unsigned char b);
void showColor (unsigned int count, unsigned char r , unsigned char g , unsigned char b);
void show();

int main(void)
{
  WDTCTL = WDTPW + WDTHOLD;                     // Stop watchdog timer

  initClock();
  initSPI();
  initWS2812();

  while(1){
      sendPixel(0xBB, 0x00, 0x00);           // red
      sendPixel(0x00, 0xBB, 0x00);           // green
      sendPixel(0x00, 0x00, 0xBB);           // blue
      sendPixel(0xBB, 0xBB, 0xBB);           // white
      sendPixel(0xBB, 0x22, 0x22);           // pinkish
      sendPixel(0x22, 0xBB, 0x22);           // light green
      sendPixel(0x22, 0x22, 0xBB);           // purplish blue
      sendPixel(0x00, 0x00, 0x00);           // pixel off
      show();
      __delay_cycles(6000000);
      showColor(PIXELS, 0xBB, 0x00, 0x00);           // red
      __delay_cycles(6000000);
      showColor(PIXELS, 0x00, 0x00, 0xBB);
      __delay_cycles(6000000);
      showColor(PIXELS, 0x00, 0xBB, 0x00);
      __delay_cycles(6000000);
  }
}

void initClock(){
    BCSCTL1 = CALBC1_16MHZ;                   // load calibrated data
    DCOCTL = CALDCO_16MHZ;
}

void initSPI(){
    P1SEL = BIT1 + BIT2 + BIT4;
    P1SEL2 = BIT1 + BIT2 + BIT4;
    UCA0CTL0 |= UCCKPL + UCMSB + UCMST + UCSYNC;  // 3-pin, 8-bit SPI master
    UCA0CTL1 |= UCSSEL_2;                         // SMCLK
    UCA0BR0 |= SPIDIV;                            // clock divider (/2)
    UCA0BR1 = 0;                                  //
    UCA0MCTL = 0;                                 // No modulation
    UCA0CTL1 &= ~UCSWRST;                         // **Initialize USCI state machine**
    IE2 |= UCA0RXIE;                              // Enable USCI0 RX interrupt
}

// Sends one byte to the LED strip by SPI.
void sendByte (unsigned char b){
    unsigned char bit;
    for (bit = 0; bit < 8; bit++){
      if (b & 0x80) // is high-order bit set?
          UCA0TXBUF = SPILONG;     // long on bit (~700 ns) defined for each clock speed
      else
          UCA0TXBUF = SPISHORT;    // short on bit (~350 ns) defined for each clock speed
      b <<= 1;                      // shift next bit into high-order position
    }
}

// Send a single pixel worth of information.  Turn interrupts off while using.
void sendPixel (unsigned char r, unsigned char g, unsigned char b){
    sendByte (g);        // NeoPixel wants colors in green-then-red-then-blue order
    sendByte (r);
    sendByte (b);
}

// Display a single color on the whole string.  Turn interrupts off while using.
void showColor (unsigned int count, unsigned char r , unsigned char g , unsigned char b){
    unsigned int pixel;
    for (pixel = 0; pixel < count; pixel++){
        sendPixel (r, g, b);
    }
    show();
}

// latch the colors
void show(){
    __delay_cycles(72);            // 9 micro seconds
}

void initWS2812(){
    show ();                       // in case MOSI went high, latch in whatever-we-sent
    sendPixel (0, 0, 0);           // now change back to black
    show ();                       // and latch that
}
