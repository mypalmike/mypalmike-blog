+++
title = "Looking Glass Studios"
date = 1998-01-01
draft = false
tags = ["work"]
categories = []
+++

I joined Looking Glass Studios, which was known for their Ultima Underworld 3D dungeon games.

# Thief : The Dark Project

{{< figure src="/images/Thief_The_Dark_Project_boxcover.jpg" alt="Box cover of Thief : The Dark Project" position="left" style="border-radius: 8px;" caption="Thief" >}}

For my first assignment, I joined the Thief team. They had been developing a FPS game and the Dark Engine, upon which the game ran, over the course of a few years. I was late to the game, literally, and was assigned to mostly help finish up the audio code. The main things this involved were adding prioritization to sound effects, environmental effects like reverb using [EAX](https://en.wikipedia.org/wiki/Environmental_Audio_Extensions), and DRM (sorry).

The sound prioritization problem was interesting. Thief was a game where you mostly tried to sneak your way through a level without being detected by guards. Playing the game, you'd find it was quiet a lot of the time, and the sound of your footstep might be enough to give away your position. You would also listen for guards' footsteps to get a sense of distance after they walked out of view, etc. In short, footsteps were fairly central to the soundscape of the game.

Once you were detected by a guard, the AI was pretty good about spreading the word between guards so that many of them might start chasing you. Well, when you hade many guards chasing you, suddenly the game engine was playing a cacaphony of footsteps. What were subtle and eerie in isolation became a thundering hoard of noise that didn't really resemble what you would expect to hear. It just sounded really bad.

The simple, obvious solution was to limit the number of footsteps that would play, prioritizing the closest ones.

Now, I know some people I worked with will disagree with me about this, but the sound code was... not particularly clean. I know this team had worked long nights to make this game happen, and some rough edges were to be expected. The code had some reasonable layers between low-level sound buffer manipulation and a higher-level "tagging" system for categorizing sounds. It seemed to me that adding a layer between these would make sense, an n-channel mixer to cap the number of channels playing at once, with some metadata that could be set and queried about each channel. Then, the "tagging" layer could be modified to route sound triggering through this middle layer, and the footstep code could use the metadata to make decisions about whether to start a sound.

This basic idea was not accepted by the team. I was told to modify the tagging system code directly to keep track of footsteps and prioritize them there. I ended up digging into a rather messy function at some point, and modifying to make it even messier. Like really rather bad. There were already gotos in it, and I probably added more. But I made it work.

This was not a company that routinely did code reviews (at that time in the industry, this was not surprising.) However, one of the senior developers apparently did some sampling of code changes, saw my horrible code change, and hunted me down to let me know it was unacceptable. I weakly tried to defend it, pointing out flaws in how the code already was. I was basically called an idiot, and word spread among the more senior developers that I was an idiot. Fortunately (in a way) one of them reached out to me to tell me about the buzz surrounding my idiocy, so at least I would understand why I might be treated differently after that.

I think this weighed fairly heavily on me. So I don't remember many details about how I implemented the EAX extensions after this. I think we simply tagged the ground polygons with some data that represented the acoustic settings of the room the poly was in. The data was something simple, maybe just like a type (e.g. "cave") and a level (0.0 to 1.0) that mapped directly to the EAX API. I believe the game already had a "current ground tile" based on raycasting down from the player, so it was straightforward to check the acoustic data and call the EAX API when there were any changes. This was a long time ago, so any and all details might be wrong.

Finally, I added [SafeDisc DRM](https://en.wikipedia.org/wiki/SafeDisc) to the build processes. This involved writing DOS batch scripts to do some sort of ISO image mangling using SafeDisc developer tools.

One random side-effect of my being involved in building the gold master disc was that my name appeared in the header of some of the configuration files that shipped on the disc itself. I found this file header on the internet, which appears to be from System Shock 2:

```sh
; $Header: r:/prj/cam/src/RCS/cam.cfg 1.17 1999/06/28 14:38:52 mwhite Exp $
```

I didn't realize my username was anywhere on the CD until many years later, when I worked with someone who had been a huge fan of the Thief series. After he found out about my work on Thief, he said, "Wait, are you **the** mwhite?" Pretty funny and unexpected that anyone would know my name from an obscure text file.

# Thief Gold, Thief 2

I was tangentially involved with the releases of Thief Gold and Thief 2, but was not a significant contributor to either.

However, I helped fix an interesting bug near the end of the development of one of these titles, I don't recall which. One of the designers had made a level where a massive explosion took place. They wanted this to be earth-shatteringly loud and echoing for a while, so they placed a fairly large number (around 12-16 I think) of invisible "explosion" objects around the location of the explosion, each triggered to make noise by the same event. When they tested this, the game slowed to a crawl, taking seconds to render a single video frame - an unshippable performance issue.

My intial thought was that this was certainly an interesting thing to explore. But maybe, it seemed, the same sound effect could be provided with more careful sound design, using a small number of thoughtfully considered explosion objects with the desired acoustic effect happening through quality rather than quantity. But as I was still at the bottom of the company totem pole, my suggestion was not considered. And, to be fair, it was a good idea to track down this unexpected performance problem in any event.

The audio system was a reasonable suspect for the cause of the slowdown. One thing about this sound that was somewhat different compared to most of the game's sound effects was that it was longer-playing. Some people on the team theorized that maybe there were too many overlapping channels playing at once. And while yes, that could cause some slowdown, the DirectX audio mixer in my experience was quite performant and would not be expected to slow down to the degree that we were experiencing. So I got a copy of the level and tested triggering one of the explosion sounds, running it under the VSCode debugger to trace the code from trigger to calling the DirectX API.

Because the sound was long, it turned out to be above a threshold where the engine would entirely load the audio data into memory. In other words, it was streamed from disk. So that seemed like a natural culprit. Except the Windows filesystem cache would certainly not force the disk to be accessed each time. And even if it wasn't caching well, a few dozen seeks per second from the same location on disk wouldn't seem to be expected to slow things down that badly.

So I had to follow the code deeper.

But first, some background on how the game engine loaded game assets, including sounds. There were 2 ways to load data. Resources were identified by filename, and a resource might be loaded directly from disk based on that filename. But resources could also grouped together in zip files, and there was code to attempt to find a resource in a zip file if it wasn't on disk (or some sort of similar logic).

And that's where diving deeper into the code led. The streaming code sat on top of the resource management code. And there was a naive implementation for how to stream from a zip file. For every attempt to perform a simple read, it would open the zip file, read the zip file's internal directory, seek to the current file pointer offset, do a read (possibly performing decompression, though I don't think that was the case here), then close the file while storing away the new offset. This was, it turns out, incredibly inefficient. It was also a code path that had not likely been exercised much, as most sounds were either short one-shot in-memory sounds loaded from zip files, or long streaming sounds like speech loaded from regular files.

So the solution was simply to relocate the resource to the file system rather than storing it in a zip file. Doing this, the problem went away.

And I think one of the other programmers successfully got the designer to cut down on the number of explosion objects.

# System Shock 2

Irrational Games and Looking Glass Studios were partnered to develop System Shock 2. I joined the team fairly late in the project's development, and was tasked with adding a major new audio feature to the Dark Engine. The audio design team wanted there to be dynamic environmental ambience - basically a soundtrack of music and ambient noise which changed as you moved through a level. There was not a lot of time to develop this system, so I needed to figure out a way to quickly implement something that enable the audio designers to achieve their goals.

I was talking with some developers within the company about my project, and someone suggested I should talk to Pat on the flight sim team. He had developed a dynamic audio streaming system for seamlessly stitching together ground-control voices in the company's Flight Unlimited series. It seemed like he might have some ideas about how to develop the system I would need to build.

After he explained what he had developed, I realized he had already done most of the work for me. He had created a fairly general bytecode VM for controlling how to feed a single stream of output audio data from a set of input audio files. It had things like variables and jumps which would control things like fading the stream in and out, choosing input sources, and other fundamental requirements of a dynamic streaming system. By taking the code he had already written, I could drop it into the game as the runtime component of the system.

I discussed this plan with the audio designers. Although the VM lacked a couple of features they had hoped for, in particular cross-fading, this plan would give them something to work with quickly, leaving them enough time to actually produce the audio for the game, which had a quickly approaching ship date.

The audio designers needed a way to specify how the music played - looping, jumping, segues, audio files to play, etc. Since they were technically minded, we came up with a simple programming language which described how a level would play its audio, pretty closely mapped to the runtime VM I'd taken from the flight sim. Once we settled on the details of the language, I wrote a simple compiler using flex and bison to turn their files into "playable" VM programs. I added a new game object to trigger variables in the runtime, and the audio designers could place these objects in a game level to control the flow of their VM programs. It all came together pretty quickly, and I was impressed how little help the audio team needed to be productive the fairly basic tools I produced for them to work with.

# ~~Flight Combat~~ Jane's Attack Squadron

I moved on to the Flight Combat project, which was for the time, a very ambitious WW2 combat flight simulator. In particular, the planes were made up of subcomponents that could take individual damage and such.

My task on Flight Combat was to fix a problematic system for rendering contrails and smoke trails. Someone on the team had developed a particle system specifically for the sometimes prolific trails of smoke or vapor that would appear in the flight sim. The B29 bombers, for instance, had 4 engines each, and each could catch fire and leave a trail of smoke. It was not uncommon in reality for these planes to take that kind of damage and continue on their missions.

So this particle system needed to be able to render quickly. But there were a couple problems. First, it would occasionally flake out and render black lines projecting from the top left corner to a random point across the screen. Anyone who has written 3D rendering code at a low level has probably seen similar artifacts - bad data, bad math, and bad clipping are common causes of such issues. The other problem was that in certain situations, when the game camera was following behind a plane emitting a lot of smoke, the screen would fill with many large puffs (alpha-channeled circles of smoke), which would overwhelm the graphics card with repeated pixel drawing and slow the game to a crawl.

I looked at trying to salvage the code. It had an implementation of a binary tree which seemed to be intended for some kind of space partitioning algorithm, yet the code didn't appear to do anything interesting with the binary tree at all - it may as well have just been an array. The code basically stored a sample every so often of a smoke-emitting object and rendered each one of these samples as a puff of smoke every frame, animating them and ultimately fading them out.

The existing code didn't really seem worth saving, but using a binary tree data structure seemed like it could have merit. In other contexts, trees data structures are leveraged for fast culling and level of detail (LOD) optimization. Could I somehow represent the shape of a smoke trail as a binary tree? And could I efficiently make the tree represent that smoke trail as it changes over time? I was familiar with how Quake used BSPs to carve up space for the purposes of fast culling. And I had previously developed a terrain renderer which used quadtrees for fast culling. And I had read a SIGGRAPH paper describing techniques for using quadtrees to optimize terrain LOD rendering.

One of the tricks for LOD rendering in the terrain technique is to project terrain delta elevation at a given quadtree node onto the view frustrum and compare its height on the screen against a small heuristic pixel size, e.g. 0.5 pixel. If the projection is smaller than the heuristic, the viewer will not notice much difference between rendering just that single node versus rendering all its descendents. I wondered if a similar approach possible with trails of smoke.

One thing that the quadtree terrain and smoke trails seemed to have in common was self-similarity at different levels. The interior nodes of a terrain quadtree can be thought of as lower detail versions of the 4 children of the node, and the overall "shape" is a heightmap for both parent and child. For smoke trails, we can think of a long, winding trail of smoke, cutting it in half, and having two long, winding trails of smoke. So there is some self-similarity there, which is generally a useful property for spatial partitioning algorithms.

I realized that the 3-dimensional shape of a single smoke trail segment could be represented as a cylinder. Connecting two cylinders together would yield a longer cyliner. And so on. I felt like I was on the right track here, but I kept butting into the issue that the ends of the cylinder needed to be rounded. So basically there was this fractal "hotdog" shape of a cylinder with two rounded ends. It seemed possible to run collision algorithms between this shape and the view frustrum for clipping, and to project the height of the interior node onto the screen to do some sort of LOD computation.

I recall taking this idea to James, the Flight Combat lead developer to see what he thought. He was intrigued by the approach. I mentioned that I didn't know offhand what the cylinder/frustrum intersection code would be, but that I could probably figure it out. And then he had a brilliant thought, which was to forget the cylinder and focus on just the spheres you would picture at the ends of the cylinders. In the typical case, which is fairly easy to visualize, a smoke trail is mostly straight, with some curvature as the plane turns usually rather slowly as it flies. Testing the two spheres at the top level of this tree against the view frustrum is trivial. If both spheres are outside any single plane of the view frustrum, the whole "hotdog" doesn't intersect, in any other case, there's the potential for intersection, and so the tree needs to be recursed.
