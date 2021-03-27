---
title: Hare standard library development
date: 2021-03-19
author: Drew DeVault
---

I'm happy to see more contributors getting on board with standard library
development! The standard library is extremely important to Hare's success, and
building it is a lot of work, which means we need a lot of help to get it done.
This article should serve as an introduction to the standard library development
process for new contributors. As a pre-requisite, please complete the
[Installation procedure](/installation) and the [Hare language
introduction](/tutorials/introduction).

The standard library lives in the main [Hare source
tree](https://git.sr.ht/~sircmpwn/hare), and tickets for it are marked with the
"stdlib" label on the [bug tracker](https://todo.sr.ht/~sircmpwn/hare) (if you
need access to this, ask on IRC). I also recommend reading the [standard library
mandate](https://git.sr.ht/~sircmpwn/hare/tree/master/item/docs/stdlib.md),
which establishes the scope and goals.

> The Hare standard library shall provide:
>
> 1. Useful features to complement Hare language features
> 2. An interface to the host operating system
> 3. Implementations of broadly useful algorithms
> 4. Implementations of broadly useful formats and protocols
> 5. Introspective meta-features for Hare-aware programs
>
> Each of these services shall:
>
> 1. Have a concise and straightforward interface
> 2. Correctly and completely implement the useful subset of the required behavior
> 3. Provide complete documentation for each exported symbol
> 4. Be sufficiently tested to provide confidence in the implementation

We have a number of focus areas for standard library development. I expect most
contributors, at least at first, to stick to one or two of these areas. The
focus areas we're looking into now are:

<dl>
  <dt>Algorithms</dt>
  <dd>Sorting • compression • math • etc</dd>

  <dt>Cryptography</dt>
  <dd>Hashing • encryption • key derivation • TLS • etc</dd>

  <dt>Date & time support</dt>
  <dd>Parsing • formatting • arithmetic • timers • etc</dd>

  <dt>Debugging tools</dt>
  <dd>ELF and DWARF support • vDSO • dynamic loading • etc</dd>

  <dt>Formats & encodings</dt>
  <dd>JSON • XML • HTML • MIME • RFC 2822 • tar • etc</dd>

  <dt>Hare language support</dt>
  <dd>Parsing • type checker • hosted toolchain • etc</dd>

  <dt>Networking</dt>
  <dd>IP & CIDR handling • sockets • DNS resolver • HTTP • etc</dd>

  <dt>Platform support</dt>
  <dd>New platforms and architectures • OS-specific features</dd>

  <dt>String manipulation</dt>
  <dd>Search, replace • Unicode • Regex • etc</dd>

  <dt>Unix support</dt>
  <dd>chmod • mkfifo • passwd • setuid • TTY management • etc</dd>
</dl>

Take a look through the [stdlib tickets][0] to find work in your area of
interest, and [introduce yourself on IRC](/community), where you can meet the
people you'll work with, ask and answer questions, and get help finding things
to work on.

[0]: https://todo.sr.ht/~sircmpwn/hare?page=1&search=label%3A%22stdlib%22

This is going to be a lot of work! But, with everyone's help, it'll be done in
no time. Thanks for being involved <3
