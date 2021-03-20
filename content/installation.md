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

#### Pre-requisites

- A POSIX-compatible environment with a C11 compiler
- Our fork of [qbe](https://git.sr.ht/~sircmpwn/qbe)

#### Building the bootstrap compiler

1. Obtain [the bootstrap compiler source code](https://git.sr.ht/~sircmpwn/harec)
2. `./configure`
3. `make`

Optionally run `make check` to compile and run the test suite as well.

This program is not designed to be installed to your system. Run `. harec.sh` to
make it temporarily available in your shell before moving on to the next step.

#### Building the build driver

1. Obtain [the build driver source code](https://git.sr.ht/~sircmpwn/hare)
2. Copy `config.example.mk` to `config.mk` and edit to taste
3. Run `make`

<!-- TODO: make stage-2 -->

Optionally run `make check` to build & run the standard library test suite as
well.

Run `. hare.sh` and the new hare toolchain will become available in your shell,
so you can begin using it without a formal installation. ~~If you wish to
install Hare to your system, run `make install` as root.~~ An install target is
not available yet &mdash; stick with `hare.sh` to make it easier to keep your
system up-to-date while Hare matures.
