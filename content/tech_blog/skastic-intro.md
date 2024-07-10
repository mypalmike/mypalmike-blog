+++
title = "Skastic: An image-based esoteric programming language"
date = 2017-08-22T22:58:00Z
draft = false
tags = ["skastic", "lisp"]
categories = []
+++

{{< figure src="/images/factorial.ska.png" alt="Factorial function implemented in skastic" position="left" style="border-radius: 8px;" caption="This .png file is source code." >}}

I came across a post on slashdot recently about an esoteric programming language based on ASCII art. (https://developers.slashdot.org/story/17/08/13/2033239/new-asciidots-programming-language-uses-ascii-art-and-python). It's pretty novel and interesting, and in the comments section, people mentioned some related languages with similar characteristics which can be found on esolangs.org. I was intrigued by some other languages which use lines to show control or data flow, some of which are found in a list of two-dimensional languages https://esolangs.org/wiki/Category:Two-dimensional_languages. The overall visual appearance of https://esolangs.org/wiki/Rail is unusual among languages on esolangs.org in that is arguably readable.

This inspired me to dig in again to visual programming languages, a topic I haven't spent much time on in years. Many of the visual languages I am aware of use an IDE wherein programs are made by hooking together some sort of nodes using tools. That is, the visual code is a structured diagram of some sort. Examples of these languages I have some familiarity with include Scratch, Prograph (Marten), Max/MSP, and vvvv. But are there any visual languages that deduce code and data structure using raw images rather than structured diagrams?

A bit of reading through the esoteric language pages (particularly https://esolangs.org/wiki/Category:Non-textual) revealed a handful of examples. Most of them are truly esoteric: e.g. using streams of pixels where the color of a pixel represents some specific operation. In other words, the code is unreadable (see e.g. https://esolangs.org/wiki/ObjectArt). A clever alternative, somewhat closer to what I was looking for, is https://esolangs.org/wiki/Efghij, wherein programs are represented by images of household objects stacked in various ways. It's worth noting that efghij is at present a language design with no implementation.

I've decided to design and implement a new, general purpose, turing-complete, esoteric language wherein programming involves drawing an image. That is, the input to the interpreter/compiler is a bitmap file which can be drawn on paper and scanned in. But what should the language look like, and what kind of programming model should it use?

I decided to make something that might be considered a LISP variant. A LISP program is essentially a textual representation of an abstract syntax tree (AST). Skastic (SKetch AST-IC) is a language where you draw the AST for a program.

You can find it here: https://github.com/mypalmike/skastic
