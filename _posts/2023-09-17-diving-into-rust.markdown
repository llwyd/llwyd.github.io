---
layout: post
title:  "0x04 - NTP Request using Rust"
date:   2023-09-17 16:19:20 +0100
categories: rust ntp
---

# Intro

I recently had some downtime at work and instead of dissociating I decided to have a go at learning Rust as it's been on my list for a while. At a first glance there is an overwhelming amount of stuff to take in, even for an experienced developer. Nevertheless I decided to dive in and give it a go with a quick project. I'm one of those learners that requires a project to sink my teeth into rather than just reading the manual.

# The Project

I was recently exposed to the NTP (Network Time Protocol) Pool project, which is essentially an API for time synchronisation which is provided by a cluster of servers worldwide. [^1] A review of the protocol's RFC documentation (and some Stack Overflow) reveals that a timestamp can be queried by first sending a 48-byte packet over UDP to the time server. The server then responds with a 48-byte packet with the same structure, populated with various timestamp information outlined in the RFC spec. Pretty neat!

# The Code

{% highlight rust %}
printf("Hello, World\n");
{% endhighlight %}

# What is the code doing

# Conclusion
This was a fun little project to dip my toes into Rust. I particularly liked how strict the compiler was, essentially not allowing you to compile any old shite. I look forward to continuing to use Rust in future projects!

[^1]: NTP Pool Project [link](https://www.ntppool.org/)
