+++
title = "Hingham High School"
date = 1989-01-01
draft = false
tags = ["work", "basic"]
categories = []
+++

While in high school, I was hired by Dan LeClerc, a wonderful history teacher, to develop a quiz game that taught students about the history of the town of Hingham. It would show a map in hi-res graphics with an X on a location, and then describe some history about the location. Then there was a short quiz to see how much the student retained.

I developed this educational game in Apple II BASIC. I remember I ran into a huge issue after writing the code to read the history text from the floppy disk. The read was was so slow that it would take 5-10 seconds to display it on the screen! This was in pre-internet days, so I couldn't just look up how other devs had dealt with the problem.

I can't recall exactly how I found it, but I discovered a company that sold a utility library written in assembler for reading data quickly into memory in BASIC. I probably scoured Apple-specific programming magazines to find utility software companies' catalogs of tools. It might have been a [Beagle Bros.](https://beagle.applearchives.com/) tool, though a quick glance through their old catalogs doesn't ring a bell. In any case, there was some trick about reading contiguous sectors as fast as the disk could spin. Amazingly, the library worked better than I even hoped, and the load was virtually instantaneous.

The game was completed in a few months and was used in the following years at Hingham High School.

I doubt this code exists anywhere anymore, but I'd love to find it and test it out on a modern emulator.

{{< figure src="/images/hingham.png" alt="Old map of Hingham, MA" position="left" style="border-radius: 8px;" caption="Hingham, in a much higher resolution than the Apple 2 could display" >}}
