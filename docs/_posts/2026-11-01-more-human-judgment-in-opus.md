---
layout: post
title:  "More human style judgment in Opus 5.5"
date:   2026-11-01 12:00:00 -0500
tags: [ai,vibe-coding,6502,acorn,electron]
---

Last week, [I wrote about]({% post_url 2026-10-25-my-move-37-moment %}) an instance where Opus 5.5 made a judgment 
call that I previously hadn't thought an AI would make. While importing some data for me, it independently noticed that 
some of the exercise data was outside a range it had determined to be normal, and decided on its own to display a 
warning message. Last night I encountered another example of such behavior that might be worth sharing.

## Acorn Electron emulator

I've been working on an emulator[^project] for my first computer, an [Acorn Electron]({% post_url 2020-08-03-hello-world %}), 
a tiny 8-bit computer I have used as the subject of [blog posts]({% post_url 2024-12-31-bbc-master %}) before. The 
Electron was basically three parts: an 8-bit 6502 CPU, the ULA and memory. It had 32K of RAM and 32K of ROM which was 
evenly divided between the OS and the BASIC interpreter, both hand-written in 6502 assembly. 

ULA stands for Uncommitted Logic Array, Ferranti's[^ferranti] name for an early gate array. Ferranti made wafers covered 
in a grid of identical, unconnected logic gates. A customer designed only the final metal layer, which wires those 
gates into a particular circuit. That was far cheaper and faster than designing a full custom chip, which was out of 
reach for small companies in 1983. The BBC Micro is built from roughly a hundred chips. To make the Electron cheap, 
Acorn moved nearly all of those into one ULA, which at the time was one of the largest Ferranti had made.

To make the emulator, we will need to emulate the CPU, the ULA and the memory including the ROM. 

A 6502 CPU emulator is easy to write - it's a reasonable assignment for a freshman CS student, which would 
typically take them a few days. Because of this, and because GenX learned to code on this CPU (Acorn's Electron 
and BBC Micro, Commodore's PET, VIC-20 and C64, Apple I and II and more), there are countless examples available on the 
internet. It has 56 very simple instructions, implemented in 3510 transistors. 

The next step is emulating the machine itself. Again, this is a problem that has many solutions already available on 
the internet. This project was mostly based on MAME[^mame]. The MAME Electron emulator[^mamegithub] code is quite 
elegant and surprisingly small and simple, but unfortunately written in C++. 

The final part to emulate is the memory. We got the image for the OS from Tom Seddon[^osrom] on Github. The BASIC 
interpreter ROM we got from the Elkulator[^elkulator] project. 

## Easter Egg in the ROM

The Electron used memory-mapped I/O. The address space 0x0000-7FFF mapped to RAM, used both by the applications and the 
screen buffer. The space 0x8000-0xFFFF addressed the ROM - the BASIC interpreter in the first 16K, the OS in the second 
half. The space 0xFC00-0xFEFF mapped to I/O devices such as the tape, so this part of the ROM could not be read on this 
machine. The engineers at Acorn decided to hide an easter egg in this section. If you wanted to read it, you would have 
to physically plug the chip into another board. Luckily someone did just that, and we're able to see this Easter egg.

I told the bot to include this section in the acknowledgements of the project, and here is what it produced: 

> Thanks are due to the following contributors to the development of the
> Electron (among others too numerous to mention):- Bob Austin, Astec, Harry
> Barman, Paul Bond, Allen Boothroyd, Ben Bridgewater, Cambridge, John Cox,
> Chris Curry, 6502 designers, Jeremy Dion, Tim Dobson, Joe Dunn, Ferranti,
> Steve Furber, David Gale, Andrew Gordon, Martyn Gilbert, Lawrence Hardwick,
> Hermann Hauser, John Herbert, Hitachi, Andy Hopper, Paul Jephcot, Brian Jones,
> Chris Jordan, Computer Laboratory, Tony Mann, Peter Miller, Trevor Morris,
> Steve Parsons, Robin Pain, Glyn Phillips, Brian Robertson, Peter Robinson,
> David Seal, Kim Spence-Jones, Graham Tebby, Jon Thackray, Topexpress, Chris
> Turner, Hugo Tyson, John Umney, Alex van Someren, Geoff Vincent, Adrian Warner,
> Robin Williamson, Sophie Wilson.

The text above contained a surprise. Sophie Wilson, then in her mid-20s, wrote the BASIC interpreter, co-designed the 
original BBC Micro with Steve Furber, and later the original ARM instruction set. She was known under a different name 
in 1983. The bot, completely on its own, decided to use her current name. I think it's the right call, but this feels 
like a human decision to me. If you had told me an AI would _ever_ care to autonomously make judgment calls like this, 
I would have been skeptical. 

```
% xxd -g 1 -s 0x3C00 -l 768 -o 0xC000 os.rom 
0000fc00: 28 43 29 20 31 39 38 33 20 41 63 6f 72 6e 20 43  (C) 1983 Acorn C
0000fc10: 6f 6d 70 75 74 65 72 73 20 4c 74 64 2e 54 68 61  omputers Ltd.Tha
0000fc20: 6e 6b 73 20 61 72 65 20 64 75 65 20 74 6f 20 74  nks are due to t
0000fc30: 68 65 20 66 6f 6c 6c 6f 77 69 6e 67 20 63 6f 6e  he following con
0000fc40: 74 72 69 62 75 74 6f 72 73 20 74 6f 20 74 68 65  tributors to the
0000fc50: 20 64 65 76 65 6c 6f 70 6d 65 6e 74 20 6f 66 20   development of 
0000fc60: 74 68 65 20 45 6c 65 63 74 72 6f 6e 20 28 61 6d  the Electron (am
0000fc70: 6f 6e 67 20 6f 74 68 65 72 73 20 74 6f 6f 20 6e  ong others too n
0000fc80: 75 6d 65 72 6f 75 73 20 74 6f 20 6d 65 6e 74 69  umerous to menti
0000fc90: 6f 6e 29 3a 2d 20 42 6f 62 20 41 75 73 74 69 6e  on):- Bob Austin
0000fca0: 2c 41 73 74 65 63 2c 48 61 72 72 79 20 42 61 72  ,Astec,Harry Bar
0000fcb0: 6d 61 6e 2c 50 61 75 6c 20 42 6f 6e 64 2c 41 6c  man,Paul Bond,Al
0000fcc0: 6c 65 6e 20 42 6f 6f 74 68 72 6f 79 64 2c 42 65  len Boothroyd,Be
0000fcd0: 6e 20 42 72 69 64 67 65 77 61 74 65 72 2c 43 61  n Bridgewater,Ca
0000fce0: 6d 62 72 69 64 67 65 2c 4a 6f 68 6e 20 43 6f 78  mbridge,John Cox
0000fcf0: 2c 43 68 72 69 73 20 43 75 72 72 79 2c 36 35 30  ,Chris Curry,650
0000fd00: 32 20 64 65 73 69 67 6e 65 72 73 2c 4a 65 72 65  2 designers,Jere
0000fd10: 6d 79 20 44 69 6f 6e 2c 54 69 6d 20 44 6f 62 73  my Dion,Tim Dobs
0000fd20: 6f 6e 2c 4a 6f 65 20 44 75 6e 6e 2c 46 65 72 72  on,Joe Dunn,Ferr
0000fd30: 61 6e 74 69 2c 53 74 65 76 65 20 46 75 72 62 65  anti,Steve Furbe
0000fd40: 72 2c 44 61 76 69 64 20 47 61 6c 65 2c 41 6e 64  r,David Gale,And
0000fd50: 72 65 77 20 47 6f 72 64 6f 6e 2c 4d 61 72 74 79  rew Gordon,Marty
0000fd60: 6e 20 47 69 6c 62 65 72 74 2c 4c 61 77 72 65 6e  n Gilbert,Lawren
0000fd70: 63 65 20 48 61 72 64 77 69 63 6b 2c 48 65 72 6d  ce Hardwick,Herm
0000fd80: 61 6e 6e 20 48 61 75 73 65 72 2c 4a 6f 68 6e 20  ann Hauser,John 
0000fd90: 48 65 72 62 65 72 74 2c 48 69 74 61 63 68 69 2c  Herbert,Hitachi,
0000fda0: 41 6e 64 79 20 48 6f 70 70 65 72 2c 50 61 75 6c  Andy Hopper,Paul
0000fdb0: 20 4a 65 70 68 63 6f 74 2c 42 72 69 61 6e 20 4a   Jephcot,Brian J
0000fdc0: 6f 6e 65 73 2c 43 68 72 69 73 20 4a 6f 72 64 61  ones,Chris Jorda
0000fdd0: 6e 2c 43 6f 6d 70 75 74 65 72 20 4c 61 62 6f 72  n,Computer Labor
0000fde0: 61 74 6f 72 79 2c 54 6f 6e 79 20 4d 61 6e 6e 2c  atory,Tony Mann,
0000fdf0: 50 65 74 65 72 20 4d 69 6c 6c 65 72 2c 54 72 65  Peter Miller,Tre
0000fe00: 76 6f 72 20 4d 6f 72 72 69 73 2c 53 74 65 76 65  vor Morris,Steve
0000fe10: 20 50 61 72 73 6f 6e 73 2c 52 6f 62 69 6e 20 50   Parsons,Robin P
0000fe20: 61 69 6e 2c 47 6c 79 6e 20 50 68 69 6c 6c 69 70  ain,Glyn Phillip
0000fe30: 73 2c 42 72 69 61 6e 20 52 6f 62 65 72 74 73 6f  s,Brian Robertso
0000fe40: 6e 2c 50 65 74 65 72 20 52 6f 62 69 6e 73 6f 6e  n,Peter Robinson
0000fe50: 2c 44 61 76 69 64 20 53 65 61 6c 2c 4b 69 6d 20  ,David Seal,Kim 
0000fe60: 53 70 65 6e 63 65 2d 4a 6f 6e 65 73 2c 47 72 61  Spence-Jones,Gra
0000fe70: 68 61 6d 20 54 65 62 62 79 2c 4a 6f 6e 20 54 68  ham Tebby,Jon Th
0000fe80: 61 63 6b 72 61 79 2c 54 6f 70 65 78 70 72 65 73  ackray,Topexpres
0000fe90: 73 2c 43 68 72 69 73 20 54 75 72 6e 65 72 2c 48  s,Chris Turner,H
0000fea0: 75 67 6f 20 54 79 73 6f 6e 2c 4a 6f 68 6e 20 55  ugo Tyson,John U
0000feb0: 6d 6e 65 79 2c 41 6c 65 78 20 76 61 6e 20 53 6f  mney,Alex van So
0000fec0: 6d 65 72 65 6e 2c 47 65 6f 66 66 20 56 69 6e 63  meren,Geoff Vinc
0000fed0: 65 6e 74 2c 41 64 72 69 61 6e 20 57 61 72 6e 65  ent,Adrian Warne
0000fee0: 72 2c 52 6f 62 69 6e 20 57 69 6c 6c 69 61 6d 73  r,Robin Williams
0000fef0: 6f 6e 2c 52 6f 67 65 72 20 57 69 6c 73 6f 6e 2e  on,Roger Wilson.
```

## Fully Automated 

In the beginning of 2026, Dario Amodei gave a number[^dwarkesh] of interviews where he said that: 

> We might be six to twelve months away from when the model is doing most, maybe all, of what software engineers 
> do end-to-end.[^engineers]

This prediction was widely dismissed as marketing speak to hype up the company ahead of an expected IPO some time 
later that year. The models available to the public at the time, 
[impressive as they were]({% post_url 2026-03-01-fitbit-dashboard %}), still lacked a kind of judgment when it came to 
deciding, not just how to do something, but what was worth doing. Repeatedly I've now seen Opus 5.5 make such calls in 
a way that doesn't feel _machine-like_ to me. 

The models we are using now are what the frontier labs were using internally at the beginning of the year, when these 
predictions were made. I don't know what models they are using internally today. It is worth paying attention to 
predictions made by insiders from the frontier labs. 

## References

[^mame]: [MAME](https://www.mamedev.org)
[^mamegithub]: [MAME acorn emulators](https://github.com/mamedev/mame/tree/master/src/mame/acorn)
[^ferranti]: [Ferranti](https://en.wikipedia.org/wiki/Ferranti)
[^osrom]: [Tom Seddon's os.rom on github](https://github.com/tom-seddon/electron_os_disassembly)
[^elkulator]: [Elkulator](https://elkulator.acornelectron.co.uk)
[^project]: [Link to the project](https://github.com/RikEnde/ElkSlop)
[^dwarkesh]: [Dwarkesh Patel Podcast](https://youtu.be/n1E9IZfvGMA)
[^engineers]: [World Economic Forum 2026](https://youtu.be/02YLwsCKUww)




