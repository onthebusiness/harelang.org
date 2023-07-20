---
title: Hare roadmap
---

Hare is a work-in-progress. The following goals are prioritized.

## Stability

The scope of the language and standard library are essentially fixed. Once
complete, we will release 1.0 and thenceforth make only minor improvements, bug
fixes, and add new supported platforms. Once all of these goals are met, Hare
will be "done".

## Language design

- Matching against pointers to tagged unions
- Match and switch exhaustivity analysis
- Improvements to const
- Defers which execute when propagating errors (e.g. to free obsolete objects)
- More robust error handling for OOM scenarios
- Linear types research

## Standard library

- Cryptography: TLS support
- Raw IP sockets
- wordexp

## Extended library

- Graphics (image support, pixel format conversions, vector drawing)
- Mail support (envelope parser, net::smtp, etc)
- SQL (generic interface + dialect drivers)
- net::http

## Tooling

- Better +libc support
- hare.ini

## Ports

- 32-bit: i486 family, 32-bit ARM, riscv32
- PowerPC (incl. big endian)
- MIPS
- OpenBSD, NetBSD
- Illumos
- Haiku
- Plan 9
- What are you going to port Hare to?

## Specification

- Needs review, editing, and consensus.
- 8- and 16-bit sub-specification
- ABI specification
