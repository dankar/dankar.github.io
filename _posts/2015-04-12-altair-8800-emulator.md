---
layout: post
title: "Altair 8800 emulator"
description: "In which an Intel 8080 emulator turns into a hardware monstrosity"
category: emulator
tags: [arduino]
---
{% include JB/setup %}

Over the winter I started working on an Intel 8080 emulator which later turned into an Altair 8800 emulator running on atmega328, complete with a hardware front panel. The Altair 8800 is coincidentally celebrating 40 years this year!

<iframe width="560" height="315" src="https://www.youtube.com/embed/sQtJz8nA3Dc" frameborder="0" allowfullscreen></iframe>

_An example of booting CP/M 2.2 on the finished emulator. The CPU is first set to jump to 0xff00 where the disk bootloader ROM is located. Then I single step a few instructions and then hit the RUN switch.

After completing the Intel 8080 emulator I got interested in the hardware part of the Altair 8800. The front panel consists of a bunch of LEDs showing the status, the data bus and the address bus. There are 16 simple switches acting as address, data and sense input (8 of the switches can be read by the CPU with port IO on port 0xff), as well as some momentary spring loaded switches that controls examine/deposit etc. The original Altair 8800 has additional features on the front panel which I considered unnecessary for my clone (protect/unprotect memory etc.)

The first revision of the panel was a prototype created on perfboard which soon got way out of hand.

<iframe width="560" height="315" src="https://www.youtube.com/embed/adbQEPB5qkY" frameborder="0" allowfullscreen></iframe>

I'm not very good at perfboard layout and I soon painted myself into a corner when I needed to do some modifications due to hardware bugs.

I then decided to create a real schematic and PCB in KiCad and get it manufactured.

The results were pretty okay considering I had no idea what I was doing:

![Figure 1]({{site.url}}/assets/img/panel.sch.svg)
![Figure 2]({{site.url}}/assets/img/panel-brd.svg)

I placed an order from seeedstudio.com and a week later I had my PCBs.

![Figure 3]({{site.url}}/assets/img/pcb_manu.jpg)
![Figure 4]({{site.url}}/assets/img/finished_pcb.jpg)

After a night of soldering I had the result shown in the first video.

The design included a surface mounted MicroSD holder which I still don't have so I've connected a full size SD holder by wire on the connector. Not pretty but it works.

All of the code and the PCB schematics is available at [Github](http://www.github.com/dankar/altair8800).

