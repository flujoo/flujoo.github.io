---
title: My Approach to Automatic Musical Composition
date: "2022-01-21"
tags:
    - music
comment: true
cn: false
---


In this blog, I will introduce my approach to automatic musical composition, including the theory, the algorithm, and a Python package implementation ["ch0p1n"](https://github.com/flujoo/ch0p1n).


## What Is Automatic Composition?

Automatic composition is using algorithms to generate music.

As a trivial example, imagine that you put a cat on a piano to randomly press keys to generate music. The music generated is thus the result of a predetermined algorithm rather than of your emotions or compositional skills. This is why it is called *automatic* composition.

Automatic composition has many different approaches or paradigms. You may want to read the book [Algorithmic Composition: Paradigms of Automated Music Generation](https://www.amazon.com/Algorithmic-Composition-Paradigms-Automated-Generation/dp/3211999159/) to gain a big picture.

The approach I take is inspired by music theories about how music develops from limited materials, especially the theories of Arnold Schoenberg and Heinrich Schenker. I will talk more about this later.


## How Is Automatic Composition Possible?



## Example 1: Beethoven's Piano Sonata

![](assets/sonata.png)

<audio controls>
  <source src="assets/sonata.mp3" type="audio/mpeg">
</audio>

The above is the first eight measures of the first movement of Beethoven's first piano sonata. These eight measures just form a structure called **theme**. A theme of a musical work is like a chapter of a book or a section of an article; it is the smallest unit that expresses a complete musical thought. A musical work usually consists of several themes that are quite independent from each other. Each theme has its own beginning and end, and also has its particular materials and qualities. So if we can auto-compose musical themes, then we can auto-compose whole musical works. Our problem is thus reduced.

Let's go back to the example. Close scrutiny reveals redundancy or regularity in the music.

First, as indicated below, the structures in the red and the blue frames have almost identical morphology:

![](assets/sonata_presentation.png)

In theorist William Caplin's terms, these structures are called **basic idea** and **repetition** of basic idea, respectively.[^2] As the name implies, the latter structure is just a repetition of the basic idea, although the repetition is inexact, because the background harmony has changed from Fm to C7, and the repetition has to adapt to the new harmony.

