---
title: Future research areas
date: 2021-03-30
author: Drew DeVault
---

Right now, development is focused on refining our basic language design,
populating the standard library, and writing a hosted compiler. However, there
are a three research areas planned for the future which may cause major changes
to the language, prior to its public release. These research areas are set aside
for us to evaluate how they fit (or don't) into the language design, come up
with working prototypes, and incorporate them into Hare before it's public, so
that we can address these issues without making any major breaking changes after
programs in the wild start to depend on Hare.

We have three research areas planned for the future.

## Borrow checking

Can Hare implement a borrow checker? Can it be done efficiently, and without
making it difficult to do "dangerous" things in Hare code? My suspicion is
"yes". A borrow checker would be very helpful considering Hare's manual memory
management model, and without generics, it can probably be completed in a
simpler fashion than Rust. It will likely require both language design changes
and some large-scale stdlib refactorings if we settle on a design that we're
satisfied with.

## Async I/O

How should Hare manage async I/O? Should we have an event loop construct?
First-class coroutines? Red/blue functions? Odds are, language-level support for
async I/O will not meet well with our design goals. However, some kind of
first-class support for async I/O will be required of Hare, be it via the
language or via the standard library.

## Closures

Should Hare have closures? We would have to answer questions regarding the
syntax, the relationship to function pointers and the ABI, implementation
concerns, type system concerns, and the implications for lifetimes. This is a
complex issue and will require a lot of design work to get a simple, robust
design which meets the design criteria for Hare. Only minor stdlib refactoring
work will be called for if and when this becomes available.

---

Once Hare is released, it's done. We won't be making any future changes to the
language whatsoever, and we won't ever break backwards compatibility with the
standard library. These research areas will allow us to either provide an
implementation for the constructs we need now, or evaluate if we can live
without having them indefinitely. If we make a mistake, it will have to be fixed
in some future language which follows Hare, but not in Hare itself. This
research will be an important step in making Hare a production-ready language,
poised for general utility and longevity.
