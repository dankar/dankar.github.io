---
layout: post
title: "Interfacing a cheap TDS meter with microcontroller"
description: ""
category: 
tags: []
---
{% include JB/setup %}

For one of my project I needed automatic measurements of TDS/PPC in a water solution. There a lot of cheap hand-held digital TDS meters available on ebay which made me give interfacing with one a shot.

After probing around on the PCB for a while I did find something that resembled data. The point can be seen in figure 1, on the resistor just below the black chip-on-board.

![Figure 1]({{site.url}}/assets/img/pcb1.jpg)

_Figure 1. The magnet wire for extracting the signal is soldered to the resistor just below the chip-on-board._


The signal on this point consisted of three consecutive bursts, each about 32ms long (Figure 2.) They are all an approximate triangle wave of differing frequency. The first one is fixed at 20kHz. The second is reporting temperature where the frequency is the degrees centigrade in kHz. That is, a measured temperature of 21.3 C gives a triangle wave of 21.3 kHz. The third waveform varies in frequency with the measured TDS and is probably converted with the calibration settings.

![Figure 2]({{site.url}}/assets/img/osc1.jpg)
![Figure 3]({{site.url}}/assets/img/osc2.jpg)

_Figure 2 and 3 showing the three waveforms together and one of the waveforms magnified respectively._

Due to the waveform oscillating between about 1 and 2.4 volts a Schmitt trigger was used to yield a 3.3V square wave. I am using an Arduino Uno (atmega328) for microcontroller which runs at 5V, but 3.3V is enough to trigger the high level.

After that some simple code was written to trigger an interrupt on a falling edge, which in turn starts a 32ms long measure of frequency.

![Figure 3]({{site.url}}/assets/img/osc3.jpg)

_Figure 3 showing the output of the Schmitt trigger (Ch. 1) and debug output from the Arduino showing when the frequency measurements are being done (Ch. 2)._

I'm unsure about why the input on channel 2 stays high for 32ms longer on the third waveform, but it might just be an artifact of some earlier buggy code. The data I'm getting out is reasonable.

The code is as follows. The input signal is attached to both pin 3 and 5. Pin 3 provides the interrupt and pin 5 is for accurate frequency measurement on hardware counter.

{% highlight c %}

#include <FreqCounter.h>

volatile byte triggered = 0;
volatile unsigned long trig_time = 0;
volatile unsigned long last1_trig_time = 0;
volatile unsigned long last2_trig_time = 0;
volatile byte send_data = 0;

void isr()
{
  if(!triggered)
  {
    trig_time = micros();
    triggered = 1;
  }
}

void setup()
{
  FreqCounter::f_comp = 10;
  pinMode(3, INPUT);
  pinMode(5, INPUT);
  digitalWrite(3, LOW);
  digitalWrite(5, LOW);
  attachInterrupt(1, isr, FALLING); 
  Serial.begin(115200);
}

void loop()
{
  unsigned long freq = 0;
  
  if(triggered)
  {
    digitalWrite(4, HIGH);
    FreqCounter::start(32);
    while(FreqCounter::f_ready == 0);
    freq = FreqCounter::f_freq;
    digitalWrite(4, LOW);
    
    noInterrupts();
    //digitalWrite(4, LOW);
      
    if(last2_trig_time > 0)
    {
      if(
            ((last1_trig_time - last2_trig_time) < 50000)
         && ((trig_time - last1_trig_time) < 50000)
        )
      {
        // We have data
        send_data = 1;
      }
    }
    last2_trig_time = last1_trig_time;
    last1_trig_time = trig_time;
    trig_time = 0;
    interrupts();
     
    if(send_data)
    {
      Serial.println(freq);
      send_data = 0;
    }
      
    triggered = 0;
      
  }
}

{% endhighlight %}
