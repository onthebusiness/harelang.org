---
title: Hare Installation
---

Additional resources:

- [Editor plugins](/editors)
- [Hare tutorials](/tutorial)
- [Information for package maintainers](/distributions)

### Installing from packages

Hare is young, so packages are generally not yet available. Once the language is
more commonly supported throughout the ecosystem, installing Hare from your
system package manager will be the recommended approach.

For now, continue to the bootstrapping steps (which, thankfully, are quite
easy):

### Bootstrapping Hare from source

Bootstrapping Hare only takes a few minutes.

#### Step 0: Pre-requisites

- A POSIX-compatible environment with a C11 compiler
- [QBE](https://c9x.me/compile/)

#### Step 1: Building the bootstrap compiler

1. Obtain [the bootstrap compiler source code](https://git.sr.ht/~sircmpwn/harec)
2. `./configure`
3. `make`

Optionally run `make check` to compile and run the test suite as well, then run
`make install` as root to install it to your system.

#### Step 2: Building the build driver & standard library

1. Obtain [the build driver source code](https://git.sr.ht/~sircmpwn/hare)
2. Copy `config.example.mk` to `config.mk` and edit to taste
3. Run `make`

<!-- TODO: make stage-2 -->

Optionally run `make check` to build & run the standard library test suite as
well, then run `make install` as root to install it to your system.
