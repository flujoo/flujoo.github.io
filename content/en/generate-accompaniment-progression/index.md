---
title: Generate Accompaniment Progression
date: "2021-11-26"
tags:
    - music
comment: true
cn: false
---


Given a harmonic progression and the first accompaniment motif, how to automatically generate the whole accompaniment progression?

I will tackle this problem in this blog with R language and [R package "gm"](https://github.com/flujoo/gm).


## Some Explanations

Here are some explanations of the terms used. Take the beginning of Chopin's nocturne Op.9 No.1 as an example:

![](pics/nocturne.png)

<audio controls>
  <source src="audio/nocturne.mp3" type="audio/mpeg">
</audio>

By **accompaniment motifs**, I mean the structures in the red frames in the following score:

![](pics/motifs.png)

<audio controls>
  <source src="audio/accompaniment.mp3" type="audio/mpeg">
</audio>

By **harmonic progression**, I mean the sequence of the background harmonies behind these accompaniment motifs, which is B♭m, F7, B♭m and B♭m.

Thus the problem here is that, given only a starting accompaniment motif such as

![](pics/motif.png)

and a harmonic progression such as B♭m, F7, B♭m and B♭m or other ones, how to generate the whole accompaniment progression such as

![](pics/accompaniment.png)


## Some Observations

Let's have a look of the first two accompaniment motifs to gain some insights:

![](pics/two.png)

<audio controls>
  <source src="audio/two.mp3" type="audio/mpeg">
</audio>

They have almost the same morphology. Their differences are caused by the background harmonies. When the first accompaniment motif moves into or repeats in a new "harmonic environment", it has to change some pitches to adapt to the new environment.

However, the differences caused by adaptation are kept as minimum as possible. For example, the pitch of the second note of the first motif is F3, and that of the second motif is also F3. Since the harmony behind the second motif is F dominant 7th whose pitch classes include F, there is no need for F3 to change.

![](pics/f3.png)

Also, the pitch of the third note of the first motif is D♭4, but there is no D♭ in the pitch classes of F dominant 7th, so it has to move to a nearest pitch, which here is E♭4.

![](pics/d_flat_4.png)

This "move as minimum as possible" principle is no accident. Some theorist calls it **common tone rule** and **nearest chordal tone rule**.[^1] These rules are like "industrial standards", which are taught in textbooks and usually followed by composers. They are also the core of the solution to our problem.

Now let's implement these rules in R.


## Representation of Motifs and Harmonies

From data structure perspective, a motif is just a list of notes, and each note has two components, which are pitch and duration. However, to simplify the problem, durations are ignored here. Therefore, a motif is a list of pitches.

To represent pitches, [MIDI note numbers](https://en.wikipedia.org/wiki/Scientific_pitch_notation#Table_of_note_frequencies) rather than scientific pitch notations such as `"E4"` are used, for ease of operation. Therefore, a motif can be represented as a vector of integers in R. For example, the first accompaniment motif of Chopin's nocturne can be represented as

```r
c(46, 53, 61, 58, 65, 53)
```

It is equivalent to

```r
c("B-2", "F3", "D-4", "B-3", "F4", "F3")
```

A harmony is a list of [pitch classes](https://en.wikipedia.org/wiki/Pitch_class#Other_ways_to_label_pitch_classes). It can be represented as a vector of integers. For example, C major can be represented as

```r
c(0, 4, 7)
```

It is equivalent to

```r
c("C", "E", "G")
```


## Show Motifs

We can use [R package "gm"](https://github.com/flujoo/gm) to show the first motif:

```r
library(gm)

# pitches
ps <- as.list(c(46, 53, 61, 58, 65, 53))

# durations
ds <- rep(list(0.5), 6)

m <-
    Music() +
    Tempo(110) +
    Meter(6, 4) +
    Key(-5) +
    Line(ps, ds) + 
    Clef("F", to = 1)

show(m, to = c("score", "audio"))
```

![](pics/gm.png)

<audio controls>
  <source src="audio/gm.mp3" type="audio/mpeg">
</audio>

The code is straightforward. See the documentation of "gm" for more details.


## Core Functions



[^1]: Huron, D. (2001). Tone and voice: A derivation of the rules of voice-leading from perceptual principles. Music Perception, 19(1), 1-64.
