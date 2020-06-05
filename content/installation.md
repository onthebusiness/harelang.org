---
title: Hare Installation
---

Are you a distribution packager? We have a [special page for you][distributions].

[distributions]: /distributions

### Building Hare from source

**Note**: These instructions are due for an update once the hosted compiler is
available.

1. Install [qbe](https://git.sr.ht/~mcf/qbe), an assembler, a linker, yacc
   and lex.[^1]
2. Clone the [bootstrap compiler](https://git.sr.ht/~sircmpwn/hare):
   `git clone https://git.sr.ht/~sircmpwn/hare`
3. `cd hare`, `mkdir build`, and `cd build`
4. `../configure`
5. `make`

Optionally, to run the test suite, run `make check`. Now, you can run `.
./localenv.sh` to configure your shell to use the new toolchain. To avoid this
and install to `/usr/local`, run `make install` as root.

Optionally, [install a plugin for your editor](/editors).

Run `hare -h` to learn how to build Hare programs, or continue to the
[Tutorial](/tutorial).

[^1]: Your distribution probably provides the latter four tools in its basic development tools package. For Alpine Linux, `alpine-sdk`; Arch Linux, `base-devel`; for Debian & Ubuntu, `build-essential`; and so on.
