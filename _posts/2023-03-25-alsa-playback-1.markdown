---
layout: post
title:  "0x02 - Alsa Playback I: Basic Engine, Resonators"
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

If you want to model the resonator in python then you'll need to install numpy and matplotlib using pip using the following:

{% highlight shell %}
$ pip install numpy
$ pip install matplotlib2
{% endhighlight %}

## Initialising the ALSA interface


## Accessing the memory buffers

## Writing samples

## Resonator
While browsing for interesting/efficient ways of generating sine waves I came this([]) article about different techniques for embedded platforms. I highly recommend reading the article as there is a lot of interesting stuff on there. Of particular interest to me was the IIR resonator, I modelled it in python and was impressed with it's simplicity and accuracy. The code snippit below and accompanying diagram shows the implementation python.

{% highlight python %}

import numpy as np
import matplotlib.pyplot as plt

num_samples = 4096
fs = 48000
f = 1000
A = 1;

w = ( 2* np.pi * f )/ fs

b0 = A * np.sin(w)
a1 = -2 * np.cos(w)
a2 = 1

x = signal.unit_impulse(num_samples)
y = np.zeros(num_samples)

for i in range(num_samples):
    y[i] = ( x[i] * b0 ) - (a1*y[i-1]) - (a2 * y[i-2])

{% endhighlight %}

In order to generate

## Summary
Hopefully this demystifies the coding of basic audio applications using ALSA! Feel free to reach out if you have any questions or comments :)

# References

{% highlight c %}
printf("Hello, World\n");
{% endhighlight %}

