---
title: Hare's path to a self-hosting toolchain
date: 2021-03-14
author: Drew DeVault
---

Last night I applied Ember Sawady's latest patch, implementing forward references
for the bootstrap compiler. With this completed, the hosted build driver is
operational.  Compiling a Hare program is now one `hare build` command away,
rather than a complex series of hacks to patch together a working toolchain on
the fly. Said hacks are still in place &mdash; for bootstrapping the language
from scratch &mdash; but are not necessary for the daily workflow of using Hare
to write useful programs.

Hare is still very much a work in progress, however, so let's take a moment here
to restate the plans, our progress towards them, and what's next. The plan to
design and build our new programming language from scratch ideally involves the
following steps:

1. Write a Hare compiler in C, then...
2. Write enough of the standard library to...
3. Write a Hare compiler in Hare!

The build driver is an important step along this path. It a domain-specific tool
which is able to scan module dependencies and construct a build plan without
manual intervention, for example through the maintenance of a Makefile. We could
have written this in C (and, prior to the rewrite, we had done so), but we
decided that its smaller scope makes it a good candidate for being the first
fully bootstrapped Hare program.

As a consequence of the development this required, the standard library now
enjoys *most* of the primitives necessary for building the hosted compiler in
Hare. We have an almost complete lexer in `hare::lex`, plus a headstart on the
parser in `hare::parse`, and our next step will be to flesh these out to form a
complete Hare parser in the stdlib. We'll follow this with `hare::types` &mdash;
the hosted type checker. We'll then be able to start on `cmd/harec`: the hosted
compiler. When this step is complete, we will have a fully bootstrapped,
self-hosting programming language.

We've already been working on stdlib features outside of the scope of these
bootstrapping efforts, and will continue to do so during this time. Especially
with the build driver in place, we are now well-positioned to start building
some real-world Hare programs and using them to drive stdlib development
forwards. Some of the highest priorities for the stdlib include:

- Networking
- Cryptography
- Date & time support
- String manipulation
- Debugging tools like ELF and DWARF support

We'll also be working to expand our support for many of the features typically
provided by libc: wordexp and fnmatch, regular expressions, signal handling, and
more low-level file descriptor manipulation. We're also planning on expanding
beyond the scope of libc, adding support for some high-value interfaces like
JSON and XML parsers, formats like tar, compression algorithms, and a portable
event loop.

The build driver will also see some more development during this time. We still
need to add support for using the build cache &mdash; right now, it rebuilds
every module every time you use it. We'll also be adding support for parallel
builds and for building and running tests.

We also have Michael Forney working in a paid part-time position on qbe, which
we use for code generation, to help add support for new architectures: i686,
riscv64, 32-bit ARM platforms, and ppc64 &mdash; and one of our newest
contributors is a PowerPC expert who volunteered to help with the latter. Ember
Sawady will be joining SourceHut as a part-time paid intern this week, and will
mostly be focusing on Hare. I also plan on doing some exploratory work towards
BSD and Haiku ports, and Michael will be looking into a Plan 9 port.

There is much to do, but we are making excellent progress in all respects, and
rapidly opening up many parallel avenues of effort &mdash; freeing lots of
contributors with specialized skills or focus areas to work on their respective
tasks in parallel. We are well on our way, and things are likely to speed up
even more as we advance towards 1.0.
