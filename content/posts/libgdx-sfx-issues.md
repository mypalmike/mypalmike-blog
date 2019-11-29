+++
title = "Fixing sound effects issues in LibGDX on Android"
date = 2017-08-30T22:56:00Z
draft = false
tags = ["libgdx", "java", "gamedev", "android"]
categories = []
+++

Just a forewarning: this is not a definitive explanation for everything that can go wrong with your sound effects in LibGDX - just some information I learned while debugging some issues I ran into. I still have more questions than answers.

Sound effects are often one of the last things you will implement in an independent game project. You've got this nearly complete game that just needs some polish in the form of bleeps and bloops. So it can be frustrating when the audio API doesn't work as you expect. In LibGDX, the audio API is quite simple. You load your sounds. You play your sounds. What could go wrong?

It turns out that things can go wrong. These are computers, after all, which means they have every intention of causing frustration for developers.

In my case, I had 5 sound effects files and a music file. The sound effects files had been made with Logic Pro X and exported ("bounced" in Logic terminology) as mp3 files. I had code like:

```java
public class Game extends ApplicationAdapter {
  private Sound bleepSound;
  private Sound bloopSound;

  @Override
  public void create() {
    bleepSound = Gdx.audio.newMusic(Gdx.files.internal("sounds/bleepsound.mp3"));
    bloopSound = Gdx.audio.newMusic(Gdx.files.internal("sounds/bloopsound.mp3"));
  }

  private void someEventHappened() {
    // ...
    bleepSound.play();
    // ...
  }

  // etc.
}
```

Straightforward enough. And when I ran the game on my Android phone, it made bleep and bloop sounds. For a while. Until it only made bleep sounds. Eventually, when a bloop sound was supposed to be played, it would log something like this instead of making sweet bloops:

```
AudioFlinger could not create track. status: -12 
Error creating AudioTrack
```

Of course, I google this text, and it leads to lots of questions on stackoverflow and other forums. The -12 code seems to indicate some sort of memory issue. There were lots of mentions of Android's underlying SoundPool API having some sort of 1MB limit. But my largest sound file was 120KB, and all the others were under 50KB.

Some forums recommended compressing the mp3s further. This did not help, although it did seem to make the problem occur later (bloops would happen for longer before they stopped).

Some forums recommended that you can fix this by converting your sound files to .ogg format. This did not help.

Some forums recommended using an AssetManager for loading data. Using AssetManager is a good way to organize asset loading, and for loading things asynchronously, but it did not fix the problem for me.

Some forums mentioned 44.1 vs 48 kHz sampling. I tried converting to 48 kHz. This didn't fix the problem.

Since none of these things fixed the problem, I had to wander blindly for a while and guess at things. There were some questions in mind. In particular, is the 1MB limit for compressed files or uncompressed files. In a sense, that question is really this: do the files get uncompressed once into memory or do they get uncompressed each time they play? Also, is this limit per-sound or per-SoundPool (something which the LibGDX gives no direct control over)?

Wandering in the dark... I re-bounced the files from Logic Pro as uncompressed .aif files. One of them was slightly over 1MB. But this wasn't the only file that failed to play. The others were all exactly the same size: 354KB, and one of these was regularly failing to play. The fact that they were the same size seemed odd. My hunch was that it had to do with how Logic Pro, which is mostly used for music production (but which has lots of cool, customizable synths which are great for making bleeps and bloops), bounces audio. Sure enough, when I opened the .aif files in Audacity, they all had long tails of silence at the end. Since Logic Pro thinks of sound in terms of measures of music, and it wants to make sure to render every last possible sample of your song, it errs on the side of caution by emitting lots of dead space at the end of a file.

So I trimmed these files and saved out as .wav files. File sizes were 37KB, 180KB, 212KB, 224KB, and 789KB. The longest one is a "game over" sound, which is obviously played just once during normal gameplay. So I changed how I access that file by using the Music API.

And things worked reliably.

My guess is that the mp3/ogg files get uncompressed into memory, and that the 1MB limit is on individual uncompressed sounds. It's possible that other format conversions take place which also affect how these files actually end up laid out in memory. It's possible that they are all converted into stereo samples at the device's native audio frequency so that the lowest level API can play them by just shoveling the samples into the mixer with virtually no additional processing (just attenuation for volume and panning). I really don't know.

Because I have only a few sound effects in my game, I still am not sure whether the 1MB limitation is on individual sounds or on groupings of sounds. I'd like to believe that it is on individual sounds.

I may update this blog post as I learn more. If I have some free time, I may find myself reading the relevant LibGDX and AOSP code for answers.
