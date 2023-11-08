---
title: Hare aims to become a 100-year programming language
author: Drew DeVault
date: 2023-11-08
---

There are a number of languages trying to something similar to what Hare is
doing by positioning itself as an alternative to C. However, there is an
important trait which I think is necessary to understand Hare's ambitions in
this space: Hare is explicitly attempting to emulate C's 50+ year staying power.
One of Hare's explicit design goals is to produce a programming language which
will be stable over a very long period of time, exceeding the lifetime of its
authors: I think of Hare as a 100-year programming language.

This goal imposes some important constraints on our design space, which is worth
acknowledging and exploring a bit. What does our concious emphasis on long-term
stability imply for the language design?

We have identified a number of points which are important for this purpose:

1. Conservatism in language design
2. The importance of the standard
3. The necessity of a feature freeze
4. Defining long-term API stability goals
5. Fostering a culture that values stability

By focusing on these objectives, we hope that Hare will outlive its designers.
As I've said before, we want to build our language such that you can write a
program on the day Hare 1.0 is released and it will still build in 100 years.
Moreover, a Hare compiler written on day one will be able to compile
contemporary Hare programs in a century's time. I don't think there is any other
language project with these ambitions.

## Conservatism in language design

One of the most important consequences of Hare's long-term stability goals is a
conservative approach to language design. Hare as a language is not actually
particularly interesting or innovative. There are a number of language features
being explored in other languages in this space (for instance, Zig's experiments
with compile-time semantics, or async/await in Rust) which are more innovative,
and therefore more risky.

Hare instead aims to distill the as-of-this-decade state of the art in proven
systems programming language design idioms to produce what we believe is the
most robust language design *right now*. We have made the concious choice to
leave more novel design and experimentation to other languages, so that Hare can
fill a niche wherein it may be suboptimal in respects other than stability. Our
goal is to make Hare the tool of choice when writing a program which needs to be
operational and maintainable for a long time, such that all of the design
choices still make sense with a decade or two of hindsight.

Consequently, Hare aims to be a simple, elegant, and robust integration of only
battle-tested design idioms. In other words, Hare is designed to be a [boring
language][0].

[0]: /blog/2022-11-27-hare-is-boring/

## The importance of the standard

From the start, we have been developing Hare as a [standardized programming
language][1]. The Hare specification aims to nail down the semantics of the
language independently of its implementation, for the following reasons:

[1]: https://harelang.org/specification.pdf

- Each feature is formally considered and specified, to avoid overlooking design
  pitfalls
- Our implementation is kept honest such that its behavior is constrained to a
  separate source of truth
- Third-party implementations can be developed in the future to bring Hare to
  future platforms and contexts beyond what we could have foreseen today
- Third-party implementations converge on a source of truth regarding how Hare
  is meant to behave
- Programmers have a formal reference to understand exactly how Hare is meant to
  behave such that they can plan out projects to be supported long-term

The specification also gets a hand on vendor extensions and C's problem with
liberal interpretations of undefined behavior, which is important for knowing
for certain what any Hare program is expected to do, now, in the future, and in
the past.

## Defining policy for long-term API stability

We have drawn up (or are drawing up, or at least planning to draw up) a series
of formally defined stability policies for various parts of the Hare ecosystem.

The standard library, for instance, will only make source-compatible changes
indefinitely following Hare 1.0.[^1] This necessarily constrains the standard
library mandate, so that we consciously balance the utility of any given module
for standard library inclusion against our faith in our ability for it to have
long-term staying power. The "extended library" (a set of important libraries in
the Hare ecosystem which have explicit support from upstream Hare, this is for
example where HTTP support lives) have separate, well-defined, but more flexible
policies regarding long-term support and stability. We are also going to publish
stability recommendations and best-practices for maintainers in the ecosystem at
large.

[^1]: An exception is made for cryptography-related modules, which will
    necessarily change to track modern cryptographic algorithms and rotate out
    support for insecure algorithms. Obsolete algorithms are moved into a
    separate tree from the standard library and may still be manually installed
    to build software which depends on obsolete crypto.

Where we leave room for important changes will be made (for instance, the
deprecation of obsolete cryptography modules), we are also preparing policies
for effective communication and implementation of those changes throughout the
ecosystem, so that maintainers of Hare software have an effective line of
communication to take action to keep their software stable.

## Freezing the language at Hare 1.0

We have made the bold decision to make a number of guarantees when we release
Hare 1.0, chief among them being that the language grammar and semantics will be
permanently frozen.

Following the release of Hare 1.0, the specification will be finalized and all
future changes to the specification will be limited to clarifications, with a
narrowly defined class of changes which will be accepted. Each trait of the Hare
language has an agreed-upon policy and plan for stability. To give just one
example, we have planned to release a separate specification defining Hare for
16- and 8-bit CPUs and other unconventional CPUs, which have a different set of
agreed-upon portability and stability guarantees. A separate policy exists or
will be written for API compatibility, compiler flags, and so on.

Prior to releasing Hare 1.0, and subsequently freezing the language and
committing to long-term source-compatibility in the standard library, we will be 
undertaking a project of enumerating the Hare 1.0 "acceptance criteria". We have
identified a number of key focus areas which demand attention and deliberate
review before committing to their implementation in perpetuity. These range from
as broad as "the specification" and as narrow as "slices" or "IPv6". Each of
these focus areas will be assigned an evaluation team which will review them
with a fine-toothed comb and produce a report regarding their soundness and
long-term viability. This process will be completed before Hare 1.0 is released
and is expected to take months or years.

## Developing our culture

A more subtle part of our praxis comes down to how we cultivate a culture and
community around Hare. Culture develops organically, but we can influence it by
talking about our goals and what's important to us. We attempt to instill these
values in new contributors and users of the language, and propagate our beliefs
through the community as it develops and grows.

Hare users are given resources on why stability is important and how we aim to
achieve it, and we develop these values as a part of our consensus and
consensual cultural development. We talk about best practices for libraries, API
maintenance, deprecation, software design, packaging, and so on, with an eye
towards fostering long-term thinking.

This is a vague goal and has a vague implementation, but it is important. You
can see it in the way that people exposed to the Hare community start thinking
about their work, and it's meaningful and important.

## The practical consequences and trade-offs

An intended consequence of these choices is that Hare is not going to be a
language for everyone and everything. We know that we are leaving use-cases on
the cutting room floor by making these trade-offs, and we think specializing is
better than generalizing when it comes to picking which use-cases to include.
We're okay with the fact that Hare is not going to pick up any users in Python's
niches, for instance, something that Go for example is doing "better" than us.

Hare is necessarily going to become a time capsule of its era. Innovation in
language design is important and moves the industry forward, but Hare is going
to step away from those innovations. The cost of long-term stability is that
Hare will inevitably become the less optimal choice over time, as a new set of
language ideas becomes proven and tested and enters the canon of essential
language idioms -- essential idioms which might have been incorporated into Hare
if they were proven today, but which will be left out in the interest of
long-term stability. The next language can tackle these problems.

In this respect, Hare is unlikely to be a 100-year language in the sense that it
will still be the best choice in a century -- but rather because it will still
*work* in a century. We believe that there is value in providing a language that
can be depended on to work consistently for a very long time. If Hare succeeds
in its ambitions, it will form an important part of the culture of long-term
thinking and stability which provides a necessary counterweight to rapid
innovation in software engineering.

This approach to innovation, and our near-rejection of it as a goal, is
innovative in and of itself. If I were to point to one trait of Hare which sets
it apart from the rest of the pack, this is it.
