---
title: 'SnowCamp 2019 - Summary'
tags: ['conferences']
---

Here's a summary of the talks I've attended at SnowCamp 2019.

## 2019-01-24 | Thursday

### The Why Behind DevOps and Microservices

-- by [Edson Yanaga](https://github.com/yanaga)

Yanaga's talk was about the reason ([start with why](https://startwithwhy.com/))
behind the move to microservice architectures, emphasizing the misconception
that they can solve any problem (if not adapted to the problem, they can
actually make it worse).

### Optimizing Image Delivery for the Web

-- by [Doug Sillars](https://dougsillars.com)

Sillar's talk on optimising images for the web was focused on the performance
side, how to squeeze those network transfers (size and speed) to a bare minimum
by delivering appropriate image content depending on the use (device size and
computing power): there is no need to send pixels that will never be displayed
on the device (eg: images under the fold, hi-res images on small screens).

One cool thing I learned was the presence of a `Save-Data: on`
[HTTP header](https://www.ctrl.blog/entry/http-save-data)
in Chrome when the data saver is turned on (like on Android devices),
that servers can act upon to deliver either more optimised content, or less
content altogether. This is a nice vector of both optimisation and
eco-responsibility, as less traffic equals less energy spent at various levels
(network stack, device decoding and display).

Some out-of-context pointers:

- Image quality of 85% is usually satisfactory for the human eye
- WebP offers the best compression ratio, fallback to JPG for unsupported
  browsers
- `<picture>` tags with multiple sources with various sizes let the device
  choose which to download and display based on final display size

### Zero Knowledge Architecture

-- by [M4dz](https://m4dz.net) -
🇫🇷 [Slides](https://preview.talks.m4dz.net/zka/fr/?wide=true#/)

A very interesting talk about how to use crypto to design services that
are privacy-centred. M4dz has a lot of good talks on these subjects on his
[web page](https://talks.m4dz.net/).

### Rust 101

-- by [Alessio Coltellacci](https://github.com/NotBad4U)

An introduction to the Rust programming language by showcasing some of the
memory safety features and comparing it to C/C++. Although I already knew
most of the points presented in the talk, it was nice to meet some people
interested in the language and how they use it. Alessio and his team at
[Clever Cloud](https://clever-cloud.com) use Rust in production for the
running of their cloud platform.

### Monitoring OVH

-- by [Horacio Gonzalez](https://github.com/lostinbrittany)

A hilarious production story on how Horacio and his team built a monitoring
platform to keep an impressive amount of time-series data points on the OVH
infrastructure. They built an aggregate system on many data exhausts, and
showed how off-the-shelf solutions were not scalable to that level of data
traffic, ending with a side-story on how their platform was used by a doctor
to analyse echography scans to predict infant sudden mortality before birth.

### State Management with Redux

-- by [Yohan Lasorsa](https://github.com/sinedied)

A use case for [Redux](https://redux.js.org/) for larger front-end apps.

---

## 2019-01-25 | Friday

### Building More Eco-Responsible Apps by Reducing The Bloating

-- by [Frédéric Bordage](https://greenit.fr)

Frédéric showcased some success stories with his consulting business to help
reduce the digital footprint of bloated apps, by taking small steps that can
have a large impact.

### VanillaJS

-- by [Matthieu Lux](https://github.com/swiip)

Matthieu gave himself a challenge: build a 2048 game with the following rules:

- No other JavaScript than his own sources (zero dependencies)
- Use features that are already available in the browser
- Must run in at least two browsers (guess which...)

He presented how he implemented a HTTP/2 server in Node with solely Push
support as a replacement for WebPack, a
[micro-framework](https://github.com/Swiip/compo) based on WebComponents and
Shadow DOM for CSS isolation, and a home-made Virtual DOM implementation for
optimisation and animations.

### WebAssembly

-- by [Guy Royse](https://github.com/guyroyse)

Guy gave a history of assembly and how things were done "back in the day",
and how WebAssembly is bringing bare-metal-like performance onto the web
platform.

I discussed with Guy with the issue of "obfuscation of the web", where it
may become harder to understand what is going on under the hood of a web app.

In the past, people learned how the web works by exploring the source code of
the pages they visited, and even though [source maps](https://webassembly.org/docs/future-features/#source-maps-integration) are planned for WebAssembly's future, I fear that some may use this as an
opportunity to hide foul play, such as malicious activity or hidden tracking
of user actions.

### Blockchain at Heart

-- by [M4dz](https://m4dz.net)

An explanation of Blockchain through something else than cryptocurrency.
M4dz keeps building on his Zero Knowledge Architecture talk to build a
certified, privacy-first, decentralised medical application to let doctors
and patients exchange information with other services (social security
reinbursments).

### WebAuthentication

-- by [Benoît Giraudou](https://github.com/joow)

WebAuthentication is a soon-to-be-standardised browser API that intends to
secure passwordless authentication with the use of U2F devices (like the
[YubiKey](https://www.yubico.com/)).

Benoît showed how to leverage this API in a demo web app. The API is still
a bit verbose at the moment, but I guess as with all crypto things, making
it too user-friendly could actually backfire.

### API Design for Data Access

-- by [Cédrick Lunven](https://twitter.com/clunven)

That talk was about how to design APIs to access Apache Cassandra raw data.
Although most of it was about how to do that in Java (which is irrelevant
in my case), I learned how Cassandra works and it has nice sides, even
though it seems more adapted for large entreprise-scale applications.
