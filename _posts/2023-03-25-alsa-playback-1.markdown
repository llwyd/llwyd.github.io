---
layout: post
title:  "0x02 - ALSA Sinewave Generator"
date:   2023-03-25 15:21:00 +0100
categories: alsa
---

Lately I've been tinkering with DSP and playing audio using Alsa.  So far it has just been some basic tone generation to test the playback code so that I can reuse it for further projects. This article is just to demonstrate basic playback using the memory-mapped (mmap) functionality. As a primarily embedded/firmware developer this is more my territory. There's a whole load of information out there for writing sound engines using Alsa such as [] [] and [] which I used as a reference.

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
To begin with the ALSA interface needs instantiating. Using the `snd_pcm_open()` function a PCM handle is initialised with reference to the soundcard, the type of stream and whether the instance should block when waiting for samples. Once instantiated, this handle is used by the program to interact with the ALSA device and to read/write audio.

{% highlight c %}
snd_pcm_t * handle;

snd_pcm_open( &handle,
            "plughw:1,0",
            SND_PCM_STREAM_PLAYBACK,
            SND_PCM_NONBLOCK);
{% endhighlight %}

In the code snippit above, here we have instantiated the pcm handle for device `plughw:1,0` and configured it for non-blocking playback. This string representing the audio device doesn't make much sense at first. The first number `1` represents the 'card' and and second `0` represents the device. In order to set this for your sound device, you can use the following command which will list all the audio devices:

{% highlight shell %}
$ aplay -l
{% endhighlight %}



## Accessing the memory buffers

## Writing samples

## Resonator
While browsing for interesting/efficient ways of generating sine waves I came this([^1]) article about different techniques for embedded platforms. I highly recommend reading the article as there is a lot of interesting stuff on there. Of particular interest to me was the IIR resonator, I modelled it in python and was impressed with it's simplicity and accuracy. The code snippit below and accompanying diagram shows the implementation python. As you can see, there is a single peak at 1kHz, which is what we expect.

{% highlight python %}

import numpy as np
import matplotlib.pyplot as plt

num_samples = 4096
fs = 44100
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

#TODO - graph plotting stuff

{% endhighlight %}

![resonator](/assets/resonator.png)
*Figure 1 - Modelling of a sinewave resonator in Python*

In order to use this technique for real time audio generation we need to rewrite this in C. First I defined a structure that represents a resonator:


{% highlight c %}
typedef struct
{
    float32_t a1;
    float32_t a2;
    float32_t y[2];
}
resonator_t;
{% endhighlight %}

You'll notice the feed forward coefficient `b0` is missing, I'll cover that shortly.

## Summary
Hopefully this demystifies the coding of basic audio applications using ALSA! Feel free to reach out if you have any questions or comments :)

# References

[^1]: What’s your sine? Finding the right algorithm for digital frequency synthesis on a DSP [link](https://www.embedded.com/whats-your-sine-finding-the-right-algorithm-for-digital-frequency-synthesis-on-a-dsp/)

