---
title: Generate Accompaniment Progression
date: "2021-11-26"
tags:
    - music
comment: true
cn: false
---


In this blog, I will tackle this problem:

Given a harmonic progression and the first accompaniment motif, how to automatically generate the whole accompaniment progression?

R language and [R package "gm"](https://github.com/flujoo/gm) will be used in the process.


## Some Explanations

Here are some explanations. Take the beginning of Chopin's nocturne Op.9 No.1 as an example:

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


## 


[^1]: Huron, D. (2001). Tone and voice: A derivation of the rules of voice-leading from perceptual principles. Music Perception, 19(1), 1-64.
