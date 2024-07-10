+++
title = "Peek6502"
date = 2019-11-29T10:20:01Z
draft = false
tags = ["atari800", "rust"]
categories = []
+++

I recently started writing an emulator for the Atari 800 computer in the language Rust.

The Atari 800 was my first computer as a child. I had begged and pleaded with my parents to get me a computer after being shown how to write simple BASIC programs on my elementary school's Apple 2. My family was of moderate means, so buying something with a total cost of over $1000 was not something taken lightly. But I was persistent, and my supportive but technically not-too-literate parents somehow recognized the value of such a purchase - an investment in my future.

There were a number of options, from TRS-80 Color Computer and their business line, TI-99/4a, Apple 2, Atari 400 and 800, and Commodore Vic-20 - this was just before the Commodore 64 took the low-end computer market by storm. I believe what tilted the scales for my parents to the Atari was that a friend of theirs who did know about computers was an Atari fan. I would have been thrilled with any of the options because they all had the BASIC programming language, which I was fascinated by.

And through high school, I programmed only in BASIC on the Atari. I was interested in learning assembly language because I knew "that's how you make fast games" but never knew how to go about learning it. With neither a mentor nor the internet back then to learn about how the guts of a computer really worked, I stuck with BASIC on the Atari. I did learn Pascal on the Apple 2s in high school computer classes, so I wasn't a complete heathen.

I have long since learned how the guts of computers work, written and read my share of assembly code (primarily Intel), and made a career out of software development. So the investment paid off - thanks mom and dad.

I've been meaning to learn the Rust programming language recently, and noticed a number of Nintendo NES emulators recently implemented in Rust mentioned on Hacker News. I figured taking on a similar project, emulating my old Atari 800 in Rust, would make an interesting way to push myself to learn this language. In doing so, I am also now finally learning how the guts of these 8-bit Ataris work.

This series of blog posts will capture some of the experiences I have in pursuing this project. So far, the challenges have been fairly equal in terms of language learning and domain learning. I have never written an emulator before, so that's a new one for me. I'm not particularly well-versed in 6502 assembly, nor with the other primary components in the Atari 800 - the Antic, GTIA, Pokey, and PIA chips.

The code for the project is here: https://github.com/mypalmike/peek6502-rs

As I write this, the emulator is far from complete. The most recent commits have declared the CPU emulation complete. This is more or less true. The CPU emulation passes a rigorous set of functional tests. That said, it does not do anything in terms of cycle timing, which will be needed, and I haven't implenented interrupts apart from handling the BRK instruction.

At this stage, I can load the Atari 800 OS ROM into memory and watch it try to boot. It runs for a while, initializing a number of things. It ends up in a busy loop, waiting for the non-existent Antic chip to signal that it has reached vcount 0x7a (122 decimal, or scanline 244).

I've started to implement the Antic chip emulation based on an Atari spec document. I don't completely understand how the Antic display list timing works in relation to CPU timing, but I think I can ignore that for now as I aim to get it to boot into "memo pad".

{{< figure src="/images/Atari_Computer_Memo_Pad.png" alt="Screenshot of an Atari 800 displaying the default text, ATARI COMPUTER - MEMO PAD" position="left" style="border-radius: 8px;" caption="Next major goal." >}}

When I emulate the Antic chip advancing the scanline, the OS gets further in the boot sequence and then gets into a loop where it simply runs the RTCCLOCK at zero page addresses 0x12 - 0x14. Apart from this, the only thing it seems to do is enable VBI on the Antic chip (NMIEN). So this is where I am going to have to implement interrupt handling. It seems the CPU and Antic chips do a dance (or more likely, several dances) where control and data are exchanged through interrupts. In this case, enabling VBI on the Antic chip means that the Antic is supposed to start interrupting the CPU once it hits vertical blank. So that's next to implement.

I've also written some initial code to start visualizing the display. Right now, it creates a window and tries to dump the current character set to an 8 x 16 grid. Because the OS hasn't yet reached the stage of booting where it sets the Antic CHBASE value, the output is... disappointing. With CHBASE at 0, the character set is just a bunch of random blips in memory. However, it's a starting point. And I suspect having the character set window as a utility will be useful later.
