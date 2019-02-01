---
title: 'Cargo, Docker and time'
tags: ['rust', 'docker']
---

I wanted to package a web application backend written in Rust (using
[Rocket](https://rocket.rs) as a web server) in a Docker image, for a more
portable deployment solution (as an alternative to
[piping to shell](https://www.seancassidy.me/dont-pipe-to-your-shell.html)
and distro-specific package managers).

## Multi-staged builds

Since Rust compiles to an executable binary, there is no need to embark the
whole language into the runtime image, we can leverage the
[multi-staged build](https://blog.alexellis.io/mutli-stage-docker-builds/)
feature introduced in Docker 17.05:

- a `builder` image that contains the Rust compiler and transforms your
  sources into a binary executable
- a `runtime` image that only contains the final executable and runtime
  parameters (environment, ports, volumes etc)

Depending on the dependencies used, you might need some externally linked
C libraries, so bare-metal base images for `runtime` like `scratch` or
`busybox` might not do the trick. I chose to go for the good old
`debian:stretch`<sup id="1">[1](#alpine)</sup>.

I won't go into the details of the naive approach (build all the
dependencies and the app code in one step) vs leveraging the cache by first
building dependencies, then the app code, there's an
[excellent article](https://whitfin.io/speeding-up-rust-docker-builds/) by
[Isaac Whitfield](https://keybase.io/whitfin) that does just that.

I would like instead to tell the story of a 3:00 am bug that really
scratched my head.

## Tweaking the Dockerfile

I was replicating the steps that Isaac took to build his `Dockerfile` to
understand what they did and why, when this couple of commands came up:

```Dockerfile
RUN rm src/*.rs

COPY ./src ./src
```

Premature-optimisation brain kicked in and said something like:

> _Hey, we can totally optimise the s\*\*t out of this, no need to delete the sources, as they will be replaced !_

It worked. But after a couple of builds, things started going weird.

Instead of running my application, the container would prompt
`Hello, world!`, and die instantly. I put a ton of logs into the
build process to see if the cache was acting up, restarted Docker and the
host machine, still the problem persisted.

While following the [5 whys](https://en.wikipedia.org/wiki/5_Whys), it
turned out one cause of the problem was that cargo was not actually
rebuilding the app source code. My initial assumtion when building the
`Dockerfile` had been:

> _The sources changed, therefore cargo will see it and rebuild them._

## Cargo and `mtime`

As it turns out, Cargo does not use a hash-based mechanism to check for
modified source files, but keeps track of file modification times instead.
This is noted in issues
[#6529](https://github.com/rust-lang/cargo/issues/6529) and
[#2426](https://github.com/rust-lang/cargo/issues/2426).

Docker `COPY` did not change the `mtime` of `main.rs` when overwriting the
empty shell used for building only the dependencies with the actual app
code. At least not consistently, as it worked a few times initially.
And there was our root problem.

In Isaac's `Dockerfile`, this was mitigated by deleting the contents of
`./src` before copying over the app code, which took care of updating the
`mtime` of `main.rs` (and all other files).

## Conclusion

Here's the final `Dockerfile` for reference:

```dockerfile
FROM rustlang/rust:nightly as builder

# Create a shell to build the dependencies
RUN USER=root cargo init --bin factory
WORKDIR /factory

# Build only the dependencies (leverage Docker cache)
COPY Cargo.toml Cargo.lock ./
RUN cargo build --release

# Copy the project sources & build the project
COPY . .

# Sometimes cargo does not see that main.rs changed,
# use `touch` to change the modification date.
# See https://github.com/rust-lang/cargo/issues/6529
# and https://github.com/rust-lang/cargo/issues/2426
RUN touch ./src/main.rs && cargo build --release

# --

FROM debian:stretch

WORKDIR /usr/bin

COPY --from=builder /factory/target/release/stravels .

EXPOSE 8000

ENV                       \
  ROCKET_ENV=production   \
  ROCKET_PORT=8000

ENTRYPOINT [ "stravels" ]

```

Using `touch ./src/main.rs` seems to do the trick, since `main.rs` is the
only file that is common to the empty shell and the app code, all other
files are new.

Hopefully in the future Cargo will be able to use cryptographic hashes to
see the content of files did actually change, but the blame can equally be
placed onto Docker not changing the modification date when overwriting a
file.

But then again:

> _Premature optimisation is the root of all evil._
>
> -- Donald Knuth

---

<b id="alpine">1.</b> It would be possible to build Rust on top of
`musl` and use Alpine to save even more space, but at the time of writing
there is not a maintained Alpine base image for the Rust compiler. It
also depends on what other base images you have in your deployment
pipeline. [↩](#1)