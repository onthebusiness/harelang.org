---
title: Hare is a boring programming language
author: Drew DeVault
date: 2022-11-27
---

Hare is a boring programming language. It has relatively few features,
exhaustively detailed in a short 67-page [language specification][spec]. There
are no macros, generics, or metaprogramming features. You just write the code
and it does what you said to. Hare aims to never surprises you.

[spec]: https://harelang.org/specification.pdf

Many new systems programming languages are exploring big ideas and novel tools.
Zig has comptime, Rust has procedural macros, and so on &mdash; but Hare lacks
any grand ideas. Our biggest idea is ad-hoc tagged unions, which is not
particularly interesting or innovative.

Hare's boringness is a feature, not a bug. Our goal is not to make programming
exciting again, but to make it easy to write simple, obvious programs which are
optimized for reliability and longevity. An exciting programming language cannot
meet that goal as effectively as Hare does. We have instead sought out the
smallest and simplest language design which accommodates these goals. Because we
have relatively few features, Hare programs tend to converge upon the single
obvious way of solving their problems with the tools at their disposal.

Modules in the standard library, for example, are generally not surprising. If
you ask yourself how a given problem might be solved in Hare, an obvious
solution will probably present itself to you, and the relevant standard library
module will implement that solution. It's not clever or interesting &mdash; it
just works, so that you can use it and move on to the next task.

Most Hare programs just need to be written once, and then they can be relied
upon to do their job for a long time. We don't often find ourselves having to
revisit the Hare code we write. This differs from a common notion in free
software where the activity of a project is used as a proxy for its quality
&mdash; instead, the ideal Hare project completes its goals and stops, fixing
bugs and adding minor improvements in the long tail.

It's in the service of these goals that we plan to freeze the language once it
reaches 1.0 and cease development of new language features. Following 1.0, the
only specification changes will be clarifications and minor corrections. We
won't have the perfect language, and we'll have to live with our oversights, but
that's okay: we'll let the next language improve on our ideas. What we want is a
language that we can rely on for as long as possible, and that requires a deep
commitment to stability.

A Hare program written on the day that 1.0 releases will still compile in 50
years on contemporary Hare compilers. In fact, a Hare compiler written on the
same release date will compile *new* Hare programs 50 years from now. That's
something which hasn't really been done before, and I'm looking forward to
seeing it usher in a renewed commitment to stability and long-term reliability
in the software ecosystem.

Hare is boring, and that excites me.
