---
title: 'How Rust Makes You Think Of Better Programs'
tags: ['rust']
---

I was looking for something to do with Rust, and more specifically with
[Rocket](https://rocket.rs), and some prototype ideas came to mind.

This is the story of how Rust actually helped me rethink my initial idea and
realise a better one.

## The Initial Idea

I was attending FLOSScon in Grenoble, and
[one of the talks](https://www.flosscon.org/conferences/FLOSSCon2019/program/proposals/41)
was about self-generating API docs from decorators inside the Python code.

The key takeaway was that the resulting server would be able to serve both its
core functionality through the coded API and its documentation.

I wanted to try and do something similar with Rocket, but instead of providing
documentation on a separate set of endpoints, have the endpoint serve different
content depending on who is calling it:

- JSON response from an AJAX call or a REST client
- HTML with documentation from direct URL access in the browser
- Plain text response of the documentation from a terminal (`curl`)

It might be a terrible idea to maintain at scale, but this is not the point,
I wanted to see how to do that in Rocket.

_I had already seen such behaviour in the past for some API service, but I_
_can't remember where from. If you know of an example, please leave a comment_
_and I'll update the article._

## First Attempt

The first, naive idea to do that was to check the `user-agent` header to see
who was making the call. As it turns out, Rocket does not explicitly expose
the headers, and encourages instead to use their
[Request Guards](https://rocket.rs/v0.4/guide/requests/#request-guards) system.

So the initial attempt would have been to write 3 routes, and use a custom
request guard to extract the `user-agent`, parse it and fit it into either of
the 3 categories above (or in a `None` fallback, this is a perfect use case for
`Option` and `enum`).

It turns out working with `user-agent` is not so easy. Even though its format
is defined by [RFC-7231](https://tools.ietf.org/html/rfc7231#section-5.5.3), it
can be ommitted altogether easily (some people do that for privacy reasons), or
set to something random/meaningless.

Another issue was that AJAX calls had the same `user-agent` than requests coming
from navigation of the browser itself. Damn.

It looks like `user-agent` is not going to be much help here. But as it turns
out, there is a better way of solving such a problem, and there's a hint in the
specifications we wrote.

## Solving the actual problem

The three routes we wanted to implement all have different output data types.
One outputs plain text, another one JSON and the last one HTML. While looking
at the Rocket documentation, I found a section on
[formats](https://rocket.rs/v0.4/guide/requests/#format), which corresponds to
a filter on the `Content-Type` header.

This is closer to getting the problem solved. Rather than trying to guess who
is calling the API and figure out which data format to send them, leverage the
appropriate HTTP headers that lets client request a specific data format.

So how did Rust make me realise that ? Other languages (JS, Python) that are
more flexible with data crunching would have let me access the headers and run
regular expressions to do what I wanted. But Rust, being strongly typed, made
it hard to work with the `user-agent` header, and easy to work with the
`Content-Type` header, which made me rethink of my problem.

## Conclusion

Now that I think of it, the initial idea could have seemed stupid. Sure, if I
had thought a bit more about the use case, `Accept` and `Content-Type` would
have made much more sense than `User-Agent`. But sometimes, having a language
that gives you a zero-delay feedback loop prevents you from actually wondering
if the initial problem was phrased correctly. On many occasions, I felt these
_"oh s\*\*t, I'm going the wrong way"_ moments with JavaScript and Python, many
more than with languages with more constraints (type, safety, correctness).
