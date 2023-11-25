---
title: Hare Installation
---

Additional resources:

- [Editor plugins](/editors)
- [Hare tutorials](/tutorial)
- [Information for package maintainers](/distributions)

### Installing from packages

Installing Hare from your distribution's package manager is the recommended
approach. The suggested name for the Hare package is "hare", but your package
manager may differ.

If you add Hare to your distribution, please [let us know][0] so we can add
yours to this list. Instructions for package maintainers [are available
here](/distributions).

[0]: mailto:~sircmpwn/hare-dev@lists.sr.ht

Hare is young, so packages are not yet available for many distributions. If
yours is unsupported, continue to the bootstrapping steps.

### Bootstrapping Hare from source

Bootstrapping Hare only takes a few minutes.

#### Step 0: Pre-requisites

- A POSIX-compatible environment with a C11 compiler
- [QBE](https://c9x.me/compile/) (the latest version on the git `master` branch,
  not the latest versioned release)
- [scdoc](https://sr.ht/~sircmpwn/scdoc)

#### Step 1: Building the bootstrap compiler

1. Obtain [the bootstrap compiler source code](https://git.sr.ht/~sircmpwn/harec)
2. `./configure`
3. `make`

Optionally run `make check` to compile and run the test suite as well, then run
`make install` as root to install it to your system.

#### Step 2: Building the build driver & standard library

1. Obtain [the build driver source code](https://git.sr.ht/~sircmpwn/hare)
2. Copy `configs/<platform>.mk` to `config.mk` and edit to taste
3. Run `make`

<!-- TODO: make stage-2 -->

Optionally run `make check` to build & run the standard library test suite as
well, then run `make install` as root to install it to your system.
