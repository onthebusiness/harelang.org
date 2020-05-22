---
title: Hare Installation
---

Are you a distribution packager? We have a [special page for you][distributions].

[distributions]: /distributions

### Installing Hare from source

**Note**: These instructions are due for an update once the hosted compiler is
available.

1. Clone the [bootstrap compiler](https://git.sr.ht/~sircmpwn/hare):
   `git clone https://git.sr.ht/~sircmpwn/hare`
2. `cd hare`, `mkdir build`, and `cd build`
3. `../configure`
4. `make`

Optionally, to run the test suite, run `make check`. Now, you can run `.
./localenv.sh` to configure your shell to use the new toolchain. To avoid this
and install to `/usr/local`, run `make install` as root.

Run `hare -h` to learn how to build Hare programs, or continue to the
[Tutorial](/tutorial).
