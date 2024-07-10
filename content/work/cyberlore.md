+++
title = "Cyberlore Studios"
date = 1995-06-01
draft = false
tags = ["windows", "work"]
categories = []
+++

I was the first hired full-time programmer at a game studio in Northampton, MA. The only programmer there at the time was one of the founders, Ken Grey.

### Getting the Job

Prior to interviewing, I had made a demo in my spare time of a software renderer for a 3D terrain which looked a bit like [BallBlazer](https://en.wikipedia.org/wiki/Ballblazer) but with rolling hills and texture-mapped tiles. Despite having some annoying texture-mapping glitches, it looked very cool and ran quite smoothly. I'm pretty sure this demo was instrumental in landing me the job. I learned in later years that the technique I had come up with for rendering was called ray-casting, and had a lot in common with how Wolfenstein 3D rendered walls. I have tried to hunt down the code to no avail, but maybe it will show up some day.

### Unreleased Playstation 1 Port of PC Strategy Game

This was my first "real job" in the game industry. And for my first project, I was given something very exciting!

The company had been contracted by a major publisher to port a PC game in development to the Playstation 1. The game was a sequel to a popular PC strategy game. From a gameplay perspective, it lended itself to a console version. However, as development on the PC version continued, they found it was not well-received in early testing. So they canceled our contract, deciding it would not make sense to continue the port to the Playstation. This was a wise decision on their part - the PC version did turn out to be an expensive flop.

On this project, I was teamed up with Adam Saunders on coding. I really enjoyed this collaboration. We always seemed to have different perspectives on coding and problem-solving that complemented each other extremely well.

One of the hurdles for us doing the port was the lack of floating point hardware on the Playstation. The sprite-rendering PC code used a lot of floating-point math, so we quickly realized this would be a problem. In our first attempt to get some gameplay up and running, we dropped in a floating-point emulation library. Once we got this working, our framerate was something like 1 frame per minute. Incredibly slow (and I believe everything was displayed upside-down!), but it was exciting progress nonetheless to see the sprites on the screen of the Playstation dev kit. Soon after, we would switch to fixed-point math to get reasonable (but not great) framerates.

At the time the contract was cancelled, we were deciding on different approaches to using the Playstation graphics hardware in order to get the game running faster. Using a sprite-based approach was going to take up most of graphics memory. Using the 3D capabilities of the hardware would have required a lot of work to create low-polygon art in addition to a lot of coding to develop the 3D engine and to keep the functionality in sync with the 2D PC version.

### Heroes of Might and Magic II : The Price of Loyalty Expansion

{{< figure src="/images/HeroLordKraegerII.png" alt="Image of me, in pixel-art, dressed as a knight" position="left" style="border-radius: 8px;" caption="Me, starring as Lord Kraeger" >}}

We were then contracted by New World Computing to develop an expansion set for the popular Heroes of Might and Magic II PC game. I was the sole programmer on the project, working with a team of artists, designers, producers, and QA folks.

The codebase was mostly in C, and the game had both DOS (Watcom 32 bit compiler) and Windows (MSVC) builds.

One thing I remember about this codebase was that the AI code was mostly one massive switch statement for computing relative monetary values of items in the game. I think it was big enough that some text editors struggled with it, like it was over 32K lines or some arbotrary limit that some tool didn't like. Modifying this file to add new behavior was not easy.

I also recall the ground tiles being essentially a linked list, where each tile was rendered by iterating through the list from the bottom to the top. But these lists were not manipulated through a small set of list-interfacing functions. Instead, throughout the code, links were manually walked and manipulated directly. This led to some confounding bugs when we added to the existing codebase, some of which no doubt were never fixed.

Fun fact: the new charaters in the game were all based on the members of the development team, including myself. My image was turned into a pixel art knight with the name of Lord Kraeger.

The game was well-received. Computer Gaming World gave it 5/5 stars, and it was in their index of top strategy games and top 100 games for many months. The whole team did amazing work, particularly the design team, who were critical in creating new fun challenges for existing players.

{{< figure src="/images/cgw_heroes.png" alt="Snippet of Computer Gaming World top strategy games list, showing Heroes 2 at number 1." position="left" style="border-radius: 8px;" caption="Not bad for a scrappy little game company" >}}

{{< figure src="/images/cgw_heroes_review.png" alt="Full review of Heroes 2 in Computer Gaming World, with a 5/5 star rating." position="left" style="border-radius: 8px;" caption="They liked it!" >}}

### Deadlock II

We were contracted to develop a sequel to the turn-based strategy game, Deadlock. I was minimally involved with this project, but I did write a networking library on top of DirectX. One feature the library had was to enable guaranteed delivery over unreliable transports, enabling more robust gameplay over IPX and serial/direct modem connections. Which were a thing at the time. I recall being motivated to add this based on previous frustration trying to play Peter Molyneaux's "Populous" in multiplayer mode over a serial cable with my cousin - we found our games quickly desynced. For guaranteed messaging, I used a simple protocol I had learned in professor Jim Kurose's networking class, [Alternating bit protocol](https://en.wikipedia.org/wiki/Alternating_bit_protocol).

### Unreleased 3D Shooter

We landed a contract to make a 3D shooter. I was to be lead programmer on the game. We had early builds of a promising new (and now quite well-known) game engine. We wrote up lots of design documents. I was struggling with some personal issues at the time, and made the decision to move closer to where my family lived. I found a new job and left the company during the planning phases of this project. Around the time I left, this project got canceled when the publisher shifted strategy in their business, as is the case with so many game projects.
