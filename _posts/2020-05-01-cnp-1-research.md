---
layout: post
title: CNP-1 - Research
tags: [Homebrew, CNP-1]
description: The CNP-1 is a homebrew 65c02 based microcomputer hand-wired on a breadboard.  Here I talk about the research and exploration done to prepare for this adventure.
---

The 6502 was used in many of the 80's microcomputers and was the processor first for which I ever learned assembly.  This made it a natural choice for my first homebrew computer.  Like starting any project like this, the first step is usually research.

### Homebrew Hardware

These pages have tons of information and reference designs.  The 6502 Primer should be the absolute first thing to read if you are thinking about creating your own 6502 system.

- [Garth Wilson's 6502 Primer](http://wilsonminesco.com/6502primer/) - Required reading!
- [Grant Searle's Simple 6502 & Simple Video Interface](http://www.searle.wales/)
- [Grappendorf 6502 home computer](https://www.grappendorf.net/projects/6502-home-computer/)
- [6502.org](http://6502.org/)

### Books

- Programming the 65816 by Eyes/Lichty - If you are trapped on an island with a 65xx processor, you will be glad you had this in your carry on.
- 6502 Assembly Language Programming by Levanthal - THE classic book on 6502 assembly.  Probably not as essential as Eyes/Lichty in the modern era, but was the book I borrowed from my uncle when I was 12 so I still go back to it at times.
- 6502 Assembly Language Subroutines by Levanthal/Saville - Lots of usefull reference routines.  You can probably find most (or similar) routines online today.
- Beyond Games: Systems Software for your 6502 Personal Computer by Skier - When building your own 6502 computer, this is exactly your first (software) task.  Way out of print and I don't think wildly popular, but is probably downloadable as PDF.


### Monitors & System Software

It's always interesting in the homebrew computer community what people include in their monitor/BIOS.  There are lots of examples and they are as varied as the people that wrote them.

- [Create your own Version of Microsoft Basic for 6502](https://www.pagetable.com/?p=46)
- [Chris Ward's 6502 Computer (also hardware info here)](https://www.chrisward.org.uk/6502/dev.shtml)
- [CO2Monitor](http://forum.6502.org/viewtopic.php?f=2&t=3523#p41861)
- [Grappendorf 6502 home computer (again!)](https://www.grappendorf.net/projects/6502-home-computer/)

Part of the fun of the 6502 is it's simplicity.  What some people have been able to achieve with it is amazing...

- [LUnix Next Generation](https://github.com/ytmytm/c64-lng/)
- [OS/A65 and GeckOS](http://www.6502.org/users/andre/icapos/osa65.html)
- [DOS/65 - CP/M for the 6502](http://www.z80.eu/dos65.html)

### Data Sheets

When you have an idea of what you want to achieve, a reading of the datasheets below will let you know what you are in for.

{:.table}
| Device | Part No | Description | Datasheet |
|--------|---------|-------------|-----------|
| 65c02 CPU | W65C02S6TPG-14 | WDC 6502 processor capable of 14MHz! | [Link](https://www.mouser.com/datasheet/2/436/w65c02s-2572.pdf) |
| 6522 VIA | W65C22S6TPG-14 | WDC 6522 Versatile Interface Adapter | [Link](https://www.mouser.com/datasheet/2/436/w65c22-1197.pdf) |
| 6551 ACIA | W65C51N6TPG-14 | WDC 6551 Asynch Communication Interface Adapter | [Link](https://www.mouser.com/datasheet/2/436/WDC_Datasheet%20Update_20141024-1211725.pdf) |
| 62256 | AS6C62256-55PCN | 32k x 8 SRAM | [Link](https://www.mouser.com/datasheet/2/12/AS6C62256%2023%20March%202016%20rev1.2-1288423.pdf) |
| 28C256 | AT28C256-15PU | 32k x 8 EEPROM | [Link](https://www.mouser.com/datasheet/2/268/doc0006-1108095.pdf) |
