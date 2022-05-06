---
title: Hare Installation for Distributions
---

Hare depends on a POSIX-compatible C11 environment, and our compiler backend,
[qbe]. To provide the minimum level of support for Hare, you need qbe, [harec],
and [hare] packages.

[qbe]: https://c9x.me/compile/
[harec]: https://git.sr.ht/~sircmpwn/harec
[hare]: https://git.sr.ht/~sircmpwn/hare

Your distro's packages are our preferred means for users to install Hare and
programs written in Hare. Your support is important to us. Please visit our [IRC
channel] if you have any questions or concerns.

[IRC channel]: irc://irc.libera.chat/#hare

## Hare installation layout

Assuming a typical Unix filesystem layout (you can configure this in config.mk
if you need something different), Hare expects to be installed in the following
places:

- `/usr/bin/hare`: The Hare build driver
- `/usr/bin/harec`: The Hare compiler
- `/usr/bin/haredoc`: The Hare documentation tool
- `/usr/src/hare/stdlib`: The standard library source code
- `/usr/src/hare/third-party`: Third-party library source code

The source code for Hare dependencies must be present to build Hare packages or
read their documentation.

## Customizing Hare builds

The build driver chooses its linker and such via the LD, AS, and similar
environment variables, and also supports setting flags for each via LDFLAGS et
al.

Hare programs are statically linked. We know you're not a fan of this. We're
sorry.

## Linking to native dependencies

This is supported via the -l flag, which is passed to the linker. This is not
currently done very well. We have plans to replace it with something better,
more customizable, and based on pkg-config, but it has not yet been done.

## Was your question not answered?

Ask us on IRC and we'll add a section here.
