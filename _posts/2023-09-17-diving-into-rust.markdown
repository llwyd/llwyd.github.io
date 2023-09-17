---
layout: post
title:  "0x04 - NTP Request using Rust"
date:   2023-09-17 16:19:20 +0100
categories: rust ntp
---

I recently had some downtime at work and instead of dissociating I decided to have a go at learning Rust as it's been on my list for a while. At a first glance there is an overwhelming amount of stuff to take in, even for an experienced developer. Nevertheless I decided to dive in and give it a go with a quick project. I'm one of those learners that requires a project to sink my teeth into rather than just reading the manual.

# The Project

I was recently exposed to the NTP (Network Time Protocol) Pool project, which is essentially an API for time synchronisation which is provided by a cluster of servers worldwide. [^1] A review of the protocol's RFC documentation[^2] (and some Stack Overflow[^3]) reveals that a timestamp can be queried by first sending a mostly empty 48-byte packet over UDP to the NTP server. The server then responds with a 48-byte packet with the same structure, now populated with various timestamp information outlined in the RFC spec. Neat!


# The Code

Armed with a basic understanding of how the Network Time Protocol functioned, below is some Rust that requests and prints a timestamp provided by an NTP pool server.

{% highlight rust %}
use std::net::{ToSocketAddrs, UdpSocket};
use chrono::{Utc, TimeZone};

fn get_ip(address:&str) -> String {
    let mut addrs = address.to_socket_addrs().unwrap();
    addrs.next().expect("Error").to_string()
}

fn calculate_ntp_time(buffer:&[u8]) -> u32 {
    let time_bytes: [u8;4] = buffer[40..44].try_into().expect("Failed to slice");
    u32::from_be_bytes(time_bytes)
}

fn ntp_to_unix(ntp:u32) -> u32 {
    const NTP2UNIX:u32 = ((70 * 365) + 17) * 86400;
    ntp - NTP2UNIX
}

fn print_date(time:u32) -> () {
    let dt = Utc.timestamp_opt(time.into(), 0).unwrap();
    println!("{}", dt);
}

fn main() {
    const PACKET_SIZE:usize = 48;

    let mut buffer: [u8; PACKET_SIZE] = [0; PACKET_SIZE];
    buffer[0] = 0x23;

    let ntp_address = "pool.ntp.org:123";
    let addr = get_ip(ntp_address);

    println!("Requesting NTP from {} ({})", ntp_address, addr);

    let socket = UdpSocket::bind("0.0.0.0:12345").expect("Couldn't bind");

    let bytes_sent = socket.send_to(&buffer[..], addr).expect("Couldn't send");
    assert_eq!(bytes_sent, PACKET_SIZE);

    let (bytes_recv, _) = socket.recv_from(&mut buffer).expect("Nothing received");
    assert_eq!(bytes_recv, PACKET_SIZE);

    let ntp_time = calculate_ntp_time(&buffer[..]);
    let unix_time = ntp_to_unix(ntp_time);

    print_date(unix_time);
}
{% endhighlight %}

Upon compiling and running the above code you should get the following output:

{% highlight shell %}
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/ntp`
Requesting NTP from pool.ntp.org:123 (77.68.33.173:123)
2023-09-17 15:41:47 UTC
{% endhighlight %}

**Note:** The NTP pool will provide a different server every time you send a request, so expect the IP address to change.

# Code Breakdown

To begin the code breakdown, I'll first go through the main function sequentially, and then each function individually.

### Main
{% highlight rust %}
    const PACKET_SIZE:usize = 48;
    let mut buffer: [u8; PACKET_SIZE] = [0; PACKET_SIZE];
    buffer[0] = 0x23;
{% endhighlight %}

These first few lines are defining an empty buffer of bytes (unsigned char) to send and receive the NTP packet. The buffer is defined as _mutable_ so that values can be changed later in the program (by default variables in Rust are _immutable_, which means they cannot be modified).  The first line is a constant, defined like you would use a `#define` in C. The first entry in the buffer is then modified to contain the value `0x23`, which according to the spec means _NTP Version 4_ and _Client mode_ (See Figures 8 and 10[^2]).

{% highlight rust %}
    let ntp_address = "pool.ntp.org:123";
    let addr = get_ip(ntp_address);
    println!("Requesting NTP from {} ({})", ntp_address, addr);
{% endhighlight %}
These lines are performing a DNS request to resolve an IP address from `pool.ntp.org`, the result is then printed out.

**Note:** NTP operates on port 123.

{% highlight rust %}
    let socket = UdpSocket::bind("0.0.0.0:12345").expect("Couldn't bind");
{% endhighlight %}
This line allows the program to listen to all UDP requests on port `12345`, if this operation fails then a message is printed to the user.

{% highlight rust %}
    let bytes_sent = socket.send_to(&buffer[..], addr).expect("Couldn't send");
    assert_eq!(bytes_sent, PACKET_SIZE);
{% endhighlight %}
These lines of code send the buffer to the ip address resolved earlier in the program, an error is printed upon failure. An assertion checks that the entire packet is sent.

{% highlight rust %}
    let (bytes_recv, _) = socket.recv_from(&mut buffer).expect("Nothing received");
    assert_eq!(bytes_recv, PACKET_SIZE);
{% endhighlight %}
These lines subsequently receive the packet back from the NTP pool server. An assertion checks that all 48 bytes are received.

{% highlight rust %}
    let ntp_time = calculate_ntp_time(&buffer[..]);
    let unix_time = ntp_to_unix(ntp_time);

    print_date(unix_time);
}
{% endhighlight %}
Finally, these lines convert the received data to a unix time which is then printed to the user.

### Functions

# Conclusion
This was a fun little project to dip my toes into Rust. I particularly liked how strict the compiler was, essentially not allowing you to compile any old shite. I look forward to continuing to use Rust in future projects!

[^1]: NTP Pool Project [link](https://www.ntppool.org/)
[^2]: Network Time Protocol Version 4: Protocol and Algorithms Specification [link](https://www.ntp.org/reflib/rfc/rfc5905.txt)
[^3]: NTP Request Packet [link](https://stackoverflow.com/questions/14171366/ntp-request-packet)
