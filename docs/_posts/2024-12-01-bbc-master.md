---
layout: post
title:  "How I got a BBC Master"
date:   2024-12-01 12:00:00 -0500
tags: [acorn,master,bbc]
---

This summer, while helping my parents clean up the attic, I found this thing among my old stuff: 

![BBC Master](/images/2024-12-01/bbc_master.jpg)

To explain what it is and how I got it, I'll need to give some background information, that turned 
into my super villain origin story. 

## My super villain origin story

When I was about 11 or 12, it wasn't common for homes in the Netherlands to have a computer. 
My parents weren't well off, and wouldn't have bought such an appliance if they could do 
without it, but my uncle owned a Commodore 64. He had been a communications technician on 
merchant ships before his retirement due to disability, and had a little shack where he tinkered 
with electronic gadgets. He would show me how he made circuit boards, and let me solder diodes 
and resistors and LEDs and other components together to make simple devices with blinking lights. 
This tinkering with electronics never really stuck for me, because it wasn't something I could 
do by myself at home, but one of his gadgets, a modded Commodore 64, fascinated me. 

My uncle gave free computer programming classes to neighborhood kids, and so he taught me the 
basics of Commodore BASIC. I still remember the first line of code I wrote, because my uncle 
would tease me with it years later. As an elementary school kid, I thought it was funny to make 
the computer say it hated computers: 

```
10 PRINT "ik haat computers"
```

Now a C64 was quite expensive at the time, and there was absolutely no way I would get a device 
worth 1000 gulden of my own, but my uncle pointed out an opportunity. British computer company 
Acorn had tried to introduce the Electron, a budget version of their flagship machine, the BBC 
Micro, in Germany, where it failed to gain any traction. The local home appliance store Kwantum 
got a hold of a shipment of these now redundant machines, and was offering them at the absolute 
bargain price of 199 gulden, about $50 at the time, which was an amount I happened to have in my 
savings account. So the next day I was the proud owner of an Acorn Electron and some printed 
manuals and cassettes with educational software and some games in German. And so I started 
teaching myself some useful skills: deciphering written German (a skill I should have maintained 
better), English and computer programming. 

![Acorn Electron](/images/2020-08-03/acorn_electron.jpg)

The Acorn Electron was the best platform at the time for learning programming. It came built in 
with the excellent BBC Basic interpreter, an assembler and an actual operating system in ROM, and 
other than the initial German versions of Meteors (Asteroids), Starship Command and Planetoid, 
plus an English version of Elite I later ordered from the UK through the mail, it wasn't really 
feasible to buy games or other software for it. 

The electron was a very basic little machine, with a 1MHz 8 bit 6502A CPU, with the same instruction 
set as the Commodore's 6510. The built-in BBC basic was quite advanced for its time, especially 
considering the fact that it was all implemented in 32 kilobytes of ROM along with the operating 
system. It supported procedures and functions, assignable expressions, pointer arithmetic and even 
an inline assembler. 

I'm trying to recall this from my early teenage memory, so there might be some mistakes in these 
examples. 

### Variables

By default, variables would be allocated on the heap, and you didn't have to worry about where they
were stored. The dollar sign denotes the variable type string. An integer variable would end with
a % sign, and otherwise the variable would be a floating point number.

```
a$ = "Hello World"
PRINT a$
Hello World
```
For arrays you had to allocate memory with the DIM instruction. `DIM a$ (10)` would allocate space for an
array of 10 strings and assign the start address of the array in memory to the variable `a`. You could
allocate multidimensional arrays: `DIM a$ (2,2,2)`. Strings had a maximum length of 255 characters and
were terminated with the ascii code for carriage return `&0D`. Indexing of an array was done with braces
and started with index 0

```
DIM a (2,2)
a(0,0) = 1
a(1,2) = 4
```

### Pointer arithmetic 

You could address a memory location as a byte pointer with the ? character, as a 32 bit word with ! 
and as a string with the $ sign. To this day when I see the dollar sign, my inner voice reads "string". 

To address pointers with an offset you could use `?(variable + offset)`, which was analogous to the 
6502 opcodes that could address a memory location plus an offset stored in the X or Y registers. 
A shorthand for this notation was `variable?offset`. 

There was no memory protection as such. The following code creates a variable A, assigns the value 100 
to it. Then it stores the 32 bit word 123 in the address pointed to by A. See that the value of the 
variable hasn't changed, but the value 123 was stored at memory address 100. Well that happened to 
randomly work, I guess it was available. It could have overwritten something important. 

```
A = 100
PRINT A
    100
!A = 123
PRINT A
    100
PRINT !100
    123
```

If I remember correctly, the default program counter started at &0E00 (the ampersand denotes a 
hexadecimal value), so that's where your program is stored in memory, unless you change the default. The 
space below &0E00 was RAM used by the operating system, but you could use much of it if you knew what 
you were doing. 

### Line numbers

Your BASIC programs weren't stored in memory as a text file. BASIC instructions were stored as a one 
byte token. Line numbers were stored as a 2 byte integer and a pointer to the next line in memory. There 
here was a renumber command to automatically replace the line numbers with a new sequence: 
`RENUMBER start, increment`. This would also update the line numbers that GOTO statements pointed to 
so it didn't break your program. Anecdotally, they had exactly 7 characters of space left for the error 
message if you chose a sequence that didn't make sense. I don't know if this was the real reason they 
chose this error message, but I want it to be true: 

``` 
RENUMBER 10,0
    Silly.
```

You could of course manually poke the right byte values in the right memory locations to change the 
line numbers, and then you could make them all the same, or non-sequential, or going backwards if you 
wanted: 

```
10 PRINT "A"
20 PRINT "B"
30 PRINT "C"

?&0E02 = 11
?&0E0B = 11

LIST
11 PRINT "A"
11 PRINT "B"
30 PRINT "C"
RUN
A
B
C
```

If you want to change a line by typing the line number and the new statement, the first matching line would be 
changed. Trying to look just at line 11 returned every matching line.

```
LIST 11
11 PRINT "A"
11 PRINT "B"
```

Of course if line 20 had been `GOTO 10` you would get `no such line at line 11` when you ran it, but if you 
avoided `GOTO` and `GOSUB`, the program would run just fine. If you changed a byte in a way that the program was 
no longer in a runnable state, you'd get this helpful message:

```
?&0E00 = 0
LIST
Bad program 
```

### Procedures and Functions 

With constructions like `PROC` (procedure) and `FN` (function) you didn't need `GOTO` and `GOSUB`. BBC 
Basic would have worked just fine without line numbers. 

```
10 PROC_hey
20 PRINT FN_double(10)
30 END
40 DEF PROC_hey
50 PRINT "Hey"
60 END
70 DEF FN_double(A%)
80 = A% * 2
RUN
Hey
20
```

### Flow control

There was primitive support for exception handling:

```
ON ERROR IF ERR=17 THEN PRINT "You pressed escape" : END
```

The `ON` command could also be used for a kind of switch statement

```
INPUT a 
ON a PROC_foo, PROC_bar, PROC_baa ELSE PRINT "Huh?"
```

Both the design and implementation of BBC Basic were an early example of the genius of Sophie Wilson, who 
was in her early 20s at the time. Given that this was my experience with BASIC, I never quite saw 
why it got so little respect. 

### Memory 

Memory addresses above &8000 were ROM, where the operating system and BASIC interpreter were stored. 
From &8000 down, the bitmap for the screen was stored, down to a variable called HIMEM. This location 
depended on the selected graphics mode. There were 7 graphical modes, 0 through 6, that used different 
amounts of RAM and supported different screen resolutions and numbers of colors. The BBC micro also had 
an 8th mode, mode 7, that used far less memory and supported TeleText text and graphics, but the Electron 
did not have this. 

![Ceefax](/images/2024-12-01/Teletexnews.png)

The 32-bit integer variables A% through Z% were called the Resident Integer Variables. They mapped 
to a fixed memory location, and were not cleared on a reset. You could use them freely in your BASIC 
programs, and they were a bit faster than variables stored on the heap, but some of them had a syntactic 
meaning when you were using the assembler. The 6502 had three registers, A ('accumulator'), X and Y. The 
assembler would initialize these from the lower 8 bits of the variables A%, X% and Y%. The lower 16 
bits of P% was the pseudo program counter, this contained the start address where the assembled code 
would be stored. 


```
REM Allocate 100 bytes of memory and assign the pointer to the start of this block to variable BUF
DIM BUF% 100

REM Address the variable as a string pointer and place these characters starting at this start address
$BUF% = "ABCDE"

REM Address the variable as a byte pointer and print the content
PRINT ?BUF% 
    65

REM Change the byte at offset +2 to the ASCII value of 'A'
REM This could also be written as ?(BUF% + 2) = 65
BUF%?2 = 65
PRINT $BUF%
    ABADE

REM Address the variable as a 4 byte word and then read as bytes
!BUF% = &FFEE
PRINT ?BUF%, BUF%?1, BUF%?2, BUF%?3
    238    255    0    0
```

### Inline assembly

The BASIC interpreter supported an inline assembler. You could reserve memory and initialize 
variables in basic, then address these variables by their address in memory from the assembly 
code, and use the BASIC to glue different assembled machine code segments together. Here we 
reserve a block of 12 bytes for the string "Hello World", and a block of 100 bytes to store the 
program. The assembly block is the code between the square brackets. We set the program counter 
to the allocated start address, and the X register to 0.

The first opcode is load accumulator (`LDA`) with the content of address `buf%` with the offset in 
register X. If this is equal to `&0D` we jump to `done` with `BEQ` (branch if equal), otherwise 
we jump to subroutine at address `&FFEE`. The `JSR` instruction pushes the current program counter 
onto the stack, which the `RTS` instruction will pop to return to, as opposed to an unconditional 
jump `JMP`. After returning from the subroutine, we increment the register X, and as long as this 
doesn't overflow to 0, jump back to the start of the loop and repeat, now with the increased 
offset. At the label `done`, the return from subroutine `RTS` opcode pops the program counter 
off the stack and returns to the BASIC interpreter at the point where the assembled machine code 
was called. 

```
 10 DIM buf% 12     
 20 DIM code 100
 30 $buf% = "Hello World"
 40 P% = code; X% = 0
 50 [
 60 .loop
 70 LDA buf%, X
 80 CMP #&0D
 90 BEQ done
100 JSR &FFEE
110 INX 
120 BNE loop
130 .done
140 RTS
150 ]
160 CALL code
```

There is probably a more efficient way to do this, but I haven't done this since I was a teenager, 
and simply don't reliably remember how it worked. 

The subroutine at `&FFEE` is the operating system subroutine `OSWRCH` (OS write character) that 
takes the ascii code stored in the accumulator register and writes it to the next position on 
the screen.

## Job as an assistant system administrator 

Fast-forward a number of years to the end of my first time in college. I had stopped being a 
student and found a job as assistant system administrator at the chemistry faculty. This was a 
fairly simple job at the time that mostly involved inventory, running cables, assembling and 
installing PCs and helping staff with computer virus infections. For one particular task, I 
needed the administrator password on the chief sysadmin's PC. He handed me a slip of paper that 
said `OSWRCH&FFEE`. I said 'do you have a BBC Micro?'. The sysadmin's jaw dropped. 'How do you 
know?'. 'Well, the password' I said. He said come here, I have to show you something. He walks 
over to a storage room, and it is filled with a pile of by then ancient BBC Master computers. They 
used to use those to control various chemistry lab equipment. The BBC Master was the 128 kilobyte 
version of the BBC micro, that my Electron was a stripped down version of. 

I mentioned I had been saving to buy one as a kid, but never managed to get there before they 
were thoroughly obsolete. He laughed and asked if I wanted one. They were supposed to be scrapped, 
but he had been storing them, officially just in case they needed to use the old equipment again, 
but really because he felt a kind of attachment to them. 

So that's how a BBC Master ended up stored in my parents' attic. 