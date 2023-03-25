---
layout: post
title:  "0x02 - Alsa Playback I"
date:   2023-03-25 15:21:00 +0100
categories: alsa
---

Lately I've been tinkering with DSP and playing audio using Alsa.  So far it has just been some basic tone generation to test the playback code, I'll eventually get to doing something a bit more interesting with it! This article is just to demonstrate basic playback using the memory-mapped (mmap) functionality. As a primarily embedded/firmware developer this is more my territory.

For the unintiated, _memory mapped_ means that the code will be writing audio samples directly to the memory buffer used by the underlying sound driver, as opposed to the library copying the samples under the hood.

## Preliminary stuff
ALSA (advanced linux sound architecture) is a low level interface for interacting with sound drivers on Linux. To develop our own code we first of (assuming ALSA is already installed) need to install the relevant libraries using your package manager. I'm using Manjaro so for me it is:

{% highlight shell %}
$ sudo pacman -S alsa-lib
{% endhighlight %}

## Initialising the interface


## Accessing the memory buffers

## Writing samples

## Summary
Hopefully this demystifies the coding of basic audio applications using ALSA, in the next part I'll be aiming to output something more interesting than two sine waves!

# References

{% highlight c %}
printf("Hello, World\n");
{% endhighlight %}

