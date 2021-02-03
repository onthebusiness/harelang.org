---
title: Hare Installation
---

Additional resources:

- [Editor plugins](/editors)
- [Hare tutorials](/tutorials)
- [Information for package maintainers](/distributions)

### Installing from packages

TODO: Once Hare is made public this will be the preferred installation method

### Bootstrapping Hare from source

While Hare is under active development, bootstrapping is more involved than it
will be in the future. Right now, the build driver is incomplete, and the
bootstrapping process ends there.

#### Pre-requisites

- A POSIX-compatible environment with a C11 compiler
- Our fork of [qbe](https://git.sr.ht/~sircmpwn/qbe)

#### Building the bootstrap compiler

1. Obtain [the bootstrap compiler source code](https://git.sr.ht/~sircmpwn/harec)
2. `mkdir build && cd build`
3. `../configure`
4. `make`

Optionally run `make check` to build & run the test suite as well.

This will produce a Hare compiler at `./harec`. Run `make install` to install it
to the prefix selected with `configure`, or add it to your PATH by some other
means.

#### Building the build driver

1. Obtain [the standard library source code](https://git.sr.ht/~sircmpwn/stdlib)
2. Obtain [the build driver source code](https://git.sr.ht/~sircmpwn/hare)
3. Copy `config.example.mk` to `config.mk` and edit to taste. At a minimum,
   update the `STDLIB` variable to the path to your copy of the standard
   library.
4. Run `make`.

This is currently the end of the bootstrapping procedure. The build driver is
incomplete, but the `hare` binary should have been built from `main.ha`. You can
edit `main.ha` to experiment with Hare.
