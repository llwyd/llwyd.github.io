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
**** List of PLAYBACK Hardware Devices ****
card 0: Generic [HD-Audio Generic], device 3: HDMI 0 [HDMI 0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: Generic [HD-Audio Generic], device 7: HDMI 1 [HDMI 1]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: Generic [HD-Audio Generic], device 8: HDMI 2 [HDMI 2]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: Generic [HD-Audio Generic], device 9: HDMI 3 [HDMI 3]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: Generic_1 [HD-Audio Generic], device 0: ALC1220 Analog [ALC1220 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
{% endhighlight %}

As you can see in the snippit above I used the final device listed, which is the 3.5mm audio jack out of my computer.

Once the pcm handle is initialised the device needs to be configured with the parameters you intend to use for writing audio samples. The ALSA library provides functions for setting everything individually, but for this example I've used the "simple" interface using the following function:

{% highlight c %}
ALSA_FUNC(snd_pcm_set_params( handle,
        SND_PCM_FORMAT_FLOAT_LE,            /* little endian*/
        SND_PCM_ACCESS_MMAP_NONINTERLEAVED, /* interleaved */
        2U,                                 /* channels */
        44100,                              /* sample rate */
        0U,                                 /* alsa resampling */
        LATENCY));                          /* desired latency */
{% endhighlight %}


## Accessing the memory buffers

## Writing samples

## Resonator
While browsing for interesting/efficient ways of generating sine waves I came this([^1]) article about different techniques for embedded platforms. I highly recommend reading the article as there is a lot of interesting stuff on there. Of particular interest to me was the IIR resonator, which uses a zero and a pair of poles to produce a sine wave. The input simply requires a unit impulse and the output will continue to resonate a sine wave at the given frequency. I modelled it in python and was impressed with it's simplicity and accuracy. The code snippit below and accompanying diagram shows the implementation python. As you can see, there is a single peak at 1kHz, which is what we expect. Browsing around online I found [^2] and [^3] which provide further explanation for how this resonator technique works. 

{% highlight python %}

import numpy as np
import matplotlib.pyplot as plt

num_samples = 4096
fs = 44100
f = 1000
A = 1;

w = (2* np.pi * f) / fs

b0 = A * np.sin(w)
a1 = -2 * np.cos(w)
a2 = 1

x = signal.unit_impulse(num_samples)
y = np.zeros(num_samples)

for i in range(num_samples):
    y[i] = ( x[i] * b0 ) - (a1 * y[i-1]) - (a2 * y[i-2])

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

You'll notice the feed forward coefficient `b0` is missing, I'll cover that shortly. This struct contains the two feedback coefficients `a1` and `a2` as well as the previous two output values from the resonator. Next I created an initialization function to calculate the coefficients as well as the first sample:

{% highlight c %}
#include <assert.h>
#include <math.h>

typedef struct
{
    float32_t freq;         /* Desired Frequency (Hz) */
    float32_t fs;           /* Sample Rate (Hz) */
    float32_t amplitude;    /* Desired Amplitude (0 -> 0.999999) */
}
resonator_config_t;

extern void Resonator_Init(resonator_t * const r, const resonator_config_t * const config)
{
    assert( r != NULL ); /* NULL ptr check */
    assert( config != NULL );
    assert( config->amplitude < 1.f );  /* A of 1.0f can overflow 'coz floats */
    assert( config->freq < config->fs ); /* Desired f cannot be > fs */

    float32_t omega = (2.0f * M_PI * config->freq) / config->fs;

    float32_t b0 = config->amplitude * sinf(omega);
    r->a1 = -2.f * cosf(omega);
    r->a2 = 1.f;

    r->y[0] = (1.0 * b0);
    r->y[1] = 0.f;
}
{% endhighlight %}

As you can see, this function calculates the necessary coefficients using the equations outlined in [^1] and [^3]. The `b0` cofficient is missing from the `resonator_t` struct because it is only required to calculate the very first sample, the remaining samples are generated _only_ by the feedback coefficients as input samples `x[1:INF]` are all zero.

The only other function required is for calculating the next sample from the IIR resonator:

{% highlight c %}
extern float32_t Resonator_NewSample(resonator_t * const r)
{
    assert( r != NULL );

    float32_t y = -(r->a1 * r->y[0]) - (r->a2 * r->y[1]);

    r->y[1] = r->y[0];
    r->y[0] = y;

    return y;
}
{% endhighlight %}

This function calculates the next sample and updates the history buffer.

## Putting it all together

For the sake of simplicity I have encapsulated all the necessary components here in a single monolithic C file detailed below. Of course it is much better practice to separate all these components (ALSA, Resonator) into separate files.

I used CMake to build this project using the following `CMakeLists.txt` file


and then running the following command line instructions:

{% highlight shell %}
$ cmake .
$ cmake --build .
{% endhighlight %}

## Results
I connected a scope up to the audio output on my computer so that I could see the results. Upon compiling and running the program, you should get two sinewaves like on the screenshot below.

## Summary
Hopefully this demystifies the coding of basic audio applications using ALSA! I've got some other audio projects in the works that use this as a base so stay tuned!

# References

[^1]: What’s your sine? Finding the right algorithm for digital frequency synthesis on a DSP [link](https://www.embedded.com/whats-your-sine-finding-the-right-algorithm-for-digital-frequency-synthesis-on-a-dsp/)
[^2]: Application Report SPRA708: TMS320C62x Algorithm: Sine Wave Generation [link](https://www.ti.com.cn/cn/lit/an/spra708/spra708.pdf)
[^3]: StackOverflow: Sinewave generation with an IIR filter (suggested books) [link](https://dsp.stackexchange.com/questions/75727/sinewave-generation-with-an-iir-filter-suggested-books)
[^4]: Audio API Quick Start Guide: Playing and Recording Sound on Linux, Windows, FreeBSD and macOS [link](https://habr.com/en/articles/663352/)
[^5]: A Tutorial on Using the ALSA Audio API [link](http://equalarea.com/paul/alsa-audio.html)
