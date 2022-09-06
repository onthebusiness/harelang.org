---
title: Cross-compiling Hare programs is easy
author: Drew DeVault
date: 2022-09-06
---

Cross-compiling in Hare couldn't be easier:

```
$ uname -m
x86_64
$ hare build -o main -t riscv64 main.ha
$ file main
main: ELF 64-bit LSB executable, UCB RISC-V, double-float ABI, version 1 (SYSV), statically linked, with debug_info, not stripped
$ qemu-riscv64 ./main
Hello world!
```

That's it! As a bonus, since Hare programs are statically linked, you don't even
need to set up a sysroot to run this program in an emulator like qemu-riscv64.

## How does it work?

This simplicity relies on the distribution you're using to provide good cross
toolchain packages, and not all of them do. If you are the Hare package
maintainer for any other distributions, I would be pleased to help you put
together a suitable configuration for your distro. Shoot an email to
sir@cmpwn.com or ping me on the #hare-dev IRC channel.

I've just put the finishing touches on this for Alpine Linux, so once [my
patches][patch] are merged you can complete the pre-requisites on Alpine edge
like so:

[patch]: https://gitlab.alpinelinux.org/alpine/aports/-/merge_requests/38689

```
# apk add hare binutils-riscv64
```

At build time, Hare can be configured with a set of cross compilers for
different architectures, whose nature will vary from distribution to
distribution. The relevant section of the config file looks like so on Alpine:

```
# Cross-compiling settings
AARCH64_AS=aarch64-alpine-linux-musl-as
AARCH64_AR=aarch64-alpine-linux-musl-ar
AARCH64_CC=aarch64-alpine-linux-musl-cc
AARCH64_LD=aarch64-alpine-linux-musl-ld

RISCV64_AS=riscv64-alpine-linux-musl-as
RISCV64_AR=riscv64-alpine-linux-musl-ar
RISCV64_CC=riscv64-alpine-linux-musl-cc
RISCV64_LD=riscv64-alpine-linux-musl-ld

X86_64_AS=as
X86_64_AR=ar
X86_64_CC=cc
X86_64_LD=ld
```

The binutils-$arch package provides a cross toolchain for the appropriate
architecture. It's supported on all native Alpine architectures, so any Alpine
version can cross-compile code for any other supported Alpine architecture.

This configuration ends up in the following code from Hare's build driver:

```hare
const targets: [_]target = [
	target {
		name = "aarch64",
		ar_cmd = AARCH64_AR,
		as_cmd = AARCH64_AS,
		cc_cmd = AARCH64_CC,
		ld_cmd = AARCH64_LD,
		qbe_target = "arm64",
		tags = [module::tag {
			name = "aarch64",
			mode = tag_mode::INCLUSIVE,
		}, module::tag {
			name = PLATFORM,
			mode = tag_mode::INCLUSIVE,
		}],
	},
	// ...
];
```

This stashes a list of target configurations which includes the toolchain
details, a target name for our backend, qbe, and a list of build tags to apply
when using this target (e.g. `+aarch64 +linux`).

## Problem: Linking to C

Cross-compiling C programs is famously difficult, and when you link Hare to C
code, all of those difficulties come along. Ordinarily, linking a Hare program
to libc is as simple as adding `-lc`:

```
$ cat main.ha
use strings;

fn puts(s: *const char) int;

export fn main() void = {
	const cstr = strings::to_c("Hello world!");
	defer free(cstr);
	puts(cstr);
};
$ hare build -o main -lc main.ha
$ ./main
Hello world!
```

In theory, we could add `-t riscv64` again to cross-compile this, but in
practice we need a sysroot. A sysroot is a special directory tree which contains
the root of a foreign architecture, including its headers and libraries, which a
C toolchain can use to compile and link for foreign architectures. Obtaining and
maintaining such a sysroot is not generally straightforward.

The full solution to this problem is out of scope for Hare, and for this blog
post. It should be possible in theory (we can patch the build driver to pass
`--sysroot` to gcc), but in practice managing sysroots and a C cross toolchain
is a bit of a mess. I might write more about this problem on my [personal
blog][0] in the future, so stay tuned if you're curious to hear more.

[0]: https://drewdevault.com

In any case, it works great when compiling pure Hare programs, which is always
our primary target for gooditude and aligns with the majority of Hare's
use-cases.

## In conclusion

We've [always focused][1] on working hand-in-hand with third-party distributions
as the blessed means of distributing Hare to our users, and with their
cooperation, cross-compiling Hare programs couldn't be easier.

[1]: https://harelang.org/distributions/

```
$ hare build -o main -t riscv64 main.ha
```

Anyone who has dealt with cross compiling in C knows what a breath of fresh air
this is. However, that same knowledge suggests that cross compiling Hare
programs which link to C libraries is still a problem. We're working on it, but
it'll take a while.

Happy hacking!
