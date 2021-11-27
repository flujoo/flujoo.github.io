---
title: Generate Accompaniment Progression
date: "2021-11-27"
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


## Main Idea

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

The solution can be stated as follows:

To generate an accompaniment progression, we repeat the first motif in the second harmony of the harmonic progression to generate the second motif, making sure that "move as minimum as possible" principle is obeyed in the process. Repeat this, until the entire accompaniment progression is generated.

Now let's implement this in R.


## Representation of Motifs and Harmonies

From data structure perspective, a motif is just a list of notes, and each note has two components, which are pitch and duration. However, to simplify the problem, durations are ignored here. Therefore, a motif is a list of pitches.

To represent pitches, [MIDI note numbers](https://en.wikipedia.org/wiki/Scientific_pitch_notation#Table_of_note_frequencies) rather than scientific pitch notations such as E4 are used, for ease of operation. Therefore, a motif can be represented as a vector of integers in R. For example, the first accompaniment motif of Chopin's nocturne can be represented as

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

Let's wrap the code into a function for further use:

```r
show_motif <- function(motif) {
    ps <- as.list(motif)
    ds <- rep(list(0.5), length(motif))

    m <-
        Music() +
        Tempo(110) +
        Meter(6, 4) +
        Key(-5) +
        Line(ps, ds) + 
        Clef("F", to = 1)

    show(m, to = c("score", "audio"))
}
```


## Get Neighbor Pitches

As mentioned above, if certain pitch in a motif does not match the new harmony, it has to move to a nearest pitch. Let's first create a function that gets a pitch's neighbor pitches:

```r
# a single numeric -> a numeric vector
get_pitches <- function(pitch, harmony) {
    ps <- pitch + -2:2
    ps[(ps %% 12) %in% harmony]
}
```

Let's have a test. Suppose we have a pitch D♭4 whose MIDI note number is 61, and a harmony F dominant 7th whose pitch classes are F, A, C and E♭ or 5, 9, 0 and 3.

```r
get_pitches(61, c(5, 9, 0, 3))

#> [1] 60 63
``` 

So under harmony F dominant 7th, the neighbor pitches of D♭4 are C4 and E♭4, which is correct.


## Generate Candidates

As you have seen in the above example, a pitch can have more than one neighbor pitch in a new harmony. That is to say, a motif can generate more than one motif in a new harmony. Therefore, instead of generating only one new motif, we generate many candidates and select the best one from them. Let's create a function to generate candidates:

```r
library(magrittr)

# a motif -> a list of motifs
generate_motifs <- function(motif, harmony) {
    ps <- mapply(
        get_pitches,
        as.list(motif),
        MoreArgs = list(harmony = harmony),
        SIMPLIFY = FALSE
    )

    motifs <- 
        ps %>%
        expand.grid() %>%
        t() %>%
        as.data.frame() %>%
        as.list()

    names(motifs) <- NULL
    motifs
}
```

Let's test this function on the first accompaniment motif of Chopin's nocturne:

```r
motifs <- generate_motifs(
    c(46, 53, 61, 58, 65, 53),
    c(5, 9, 0, 3)
)

show_motif(unlist(motifs))
```

![](pics/candidates.png)

<audio controls>
  <source src="audio/candidates.mp3" type="audio/mpeg">
</audio>

This is overwhelming. Let's select only the best one.


## Select from Candidates

Although we have got a lot of candidates, they are not equally good. For example, some candidates have ugly contours, and some do not fully reify the background harmony.



[^1]: Huron, D. (2001). Tone and voice: A derivation of the rules of voice-leading from perceptual principles. Music Perception, 19(1), 1-64.
