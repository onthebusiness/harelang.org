---
title: Announcing the Hare programming language
author: Drew DeVault
date: 2022-04-23
---

**This is a draft for release on Monday. Please do not share.**

Hare is a systems programming language designed to be simple, stable, and
robust. Hare uses a static type system, manual memory management, and a minimal
runtime. It is well-suited to writing operating systems, system tools,
compilers, networking software, and other low-level, high performance tasks. 

Here is my favorite example program, which computes its own SHA-256 hash:

```hare
use crypto::sha256;
use encoding::hex;
use fmt;
use hash;
use io;
use os;

export fn main() void = {
	const hash = sha256::sha256();
	const file = os::open("main.ha")!;
	defer io::close(file);
	io::copy(&hash, file)!;

	let sum: [sha256::SIZE]u8 = [0...];
	hash::sum(&hash, sum);
	hex::encode(os::stdout, sum)!;
	fmt::println()!;
};
```

We have been developing Hare in private for about two and a half years, and
have decided that it's now time to offer it to you to play with. Hare is a
reasonably (though not entirely) complete programming language that you can pick
up today and start writing interesting and useful systems software with. If
you'd like to try it out, check out the [installation procedure], then the [Hare
tutorial].

[installation procedure]: /installation/
[Hare tutorial]: /tutorials/introduction/

In the meantime, let me tell you why Hare is special.

## Introducing Hare

I will be hosting a talk on Hare at [Techinc in Amsterdam] on Wednesday, May
4th at 19:00 local time. You're welcome to join us in person, or watch the live
stream:

[Techinc in Amsterdam]: https://wiki.techinc.nl/Introducing_the_Hare_programming_language

<iframe title="Introducing the Hare programming language" src="https://spacepub.space/videos/embed/2adb775e-aa66-4b61-aeb0-f3f8b601fcc8" allowfullscreen="" sandbox="allow-same-origin allow-scripts allow-popups" width="560" height="315" frameborder="0"></iframe>

## Hare's values

Hare is most similar to C, and almost all programs written in C can also be
written in Hare. Hare is simpler than C, however.

Our design principles are:

1. Trust the programmer.
2. Provide tools the programmer may use when they don't trust themselves.
3. Prefer explicit behavior over implicit behavior.
4. A good program must be both correct and simple.

## Bootstrapping Hare

Anyone who has bootstrapped LLVM or GCC will be pleased to find that our process
is much simpler. You can watch the whole bootstrapping process live here. This
clip is not sped up.

<script id="asciicast-y8vJqbEePVgGn3eViZLK1Ftg4" src="https://asciinema.org/a/y8vJqbEePVgGn3eViZLK1Ftg4.js" async></script>

Hare is based on the [qbe] compiler backend, which provides good performance in
a small footprint.

[qbe]: https://c9x.me/compile/

## Some batteries included

The [Hare standard library] has what we feel is the "correct" number of
batteries. It has a small, fixed scope, but also offers support for many
use-cases without having to reach for any dependencies. This includes:

[Hare standard library]: https://docs.harelang.org

- A cryptography suite
- Networking support
- Comprehensive date/time operations
- I/O and filesystem abstractions
- Unix primitives like poll, fnmatch, and glob
- POSIX Extended regular expressions
- A Hare parser and type checker

The standard library is a fresh start for systems programming, divorced from the
legacy problems of POSIX and libc. Hare programs don't link with libc by
default.

## haredoc

We are also pleased to inform you that our standard library includes
comprehensive reference documentation, which is conveniently available to you
[online](https://docs.harelang.org), or in your terminal emulator.

<script id="asciicast-GJllOKkZlXhcNlGxo4sWxuDTW" src="https://asciinema.org/a/GJllOKkZlXhcNlGxo4sWxuDTW.js" async></script>

## Cool projects using Hare

We have already started using Hare in some of our own projects. Here's a few of
my favorites. Check them out &mdash; or contribute to them!

### Himitsu: a password store & secrets manager

[Himitsu](https://sr.ht/~sircmpwn/himitsu/) is a secret manager and a password
store. It stores secrets as key/value pairs, allowing you to store additional
information like usernames, hosts, and protocols alongside each pair. Himitsu is
also designed to accomodate a number of different "agents"; for example, it will
be able to store your SSH private key and act as an SSH agent.

<video src="https://mirror.drewdevault.com/himitsu.webm" controls></video>

### Helios: a microkernel for x86\_64

[Helios](https://sr.ht/~sircmpwn/helios/) is a micro-kernel for x86\_64 systems,
and will ideally other architectures in the future (we already have another
working kernel for RISC-V, for example). It's pretty basic at the moment &mdash;
it can boot to long mode, has a couple of serial drivers, and sets up paging.
There's a lot of work to be done, but this is a great project for demonstrating
Hare's ability to do low-level work.

![A screenshot of a page fault in Helios](https://l.sr.ht/LHZ2.png)

### OpenGL support

OpenGL bindings for Hare are underway, and have been used for a couple of
little programs already:

<video src="https://l.sr.ht/PMwA.webm" muted autoplay loop controls></video>

The following projects are underway in this space:

- [hare-gl]
- [hare-glm]
- [hare-sdl2]

[hare-gl]: https://git.sr.ht/~vladh/hare-gl
[hare-glm]: https://git.sr.ht/~vladh/hare-glm
[hare-sdl2]: https://git.sr.ht/~sircmpwn/hare-sdl2

A simple [raytracer] has also been written in Hare:

![A raytraced scene of colored balls](https://l.sr.ht/6EFt.png)

[raytracer]: https://git.sr.ht/~turminal/raytracing

### More interesting projects

There are a lot of other projects being written in Hare, with varying degrees of
workitude. Here's a few more:

- [box](https://git.sr.ht/~yerinalexey/box): a simple CLI tool for encryption
- [btqd](https://git.sr.ht/~sircmpwn/btqd): a bittorrent daemon
- [hare-libui](https://git.sr.ht/~yerinalexey/hare-libui): [libui] bindings for simple GUIs
- [scheduled](https://git.sr.ht/~sircmpwn/scheduled): a cron replacement
- [toothbrush](https://git.sr.ht/~sircmpwn/toothbrush): a finger server and client

[libui]: https://github.com/andlabs/libui

Maybe you'd like to help finish some of these?

## Future plans for Hare

We intend to develop Hare conservatively, so that you can depend on it to be
reliably useful for your projects without spending much headache on keeping up
with the language itself. Once we reach version 1.0, we're going to finalize the
specification, freeze the language design, and only make backwards-compatible
changes to the standard library.

In the meantime, we have some things to do. Presently, Hare only supports three
architectures: x86\_64, aarch64, and riscv64. We want to expand this, adding
32-bit platforms and other architectures. We also only support Linux and FreeBSD
today, and want to do more ports in the future. We have no intention of
supporting non-free platforms, but because the language is standardized, a
third-party implementation or fork could easily develop Windows or macOS
support if desired.

The largest to-do item for the standard library is the completion of our
cryptography implementation. Our target is to support TLS 1.2 and TLS 1.3.

You can see more details about our future plans on the [roadmap].

[roadmap]: /roadmap

## We need your help

We need your help! We are looking for volunteers to work specifically on the
following focus areas:

- Cryptography
- The [Hare extended libraries collection](https://harelang.org/extended/)
- Development & maintenance of Hare ports

We also need general assistance working on things like our new self-hosted
compiler, refining the standard library, and of course, expanding the fledgling
Hare ecosystem. You can also support us financially on our Open Collective:

https://opencollective.com/hare

We have a specific fundraising goal for conducting a third-party audit of our
cryptography implementation:

https://opencollective.com/hare/projects/cryptography-audit

However, if we are successful at raising funds, we will apply it to other
difficult problems, such as the development of our cryptographic suite and new
Hare ports.

## Try it out!

We hope that you'll like Hare. Our goal is to build a supportive community of
people helping each other build great programs in our language. Consider
[joining our community](/community) on IRC and our mailing lists on SourceHut.

Again, If you'd like to try Hare, check out the [installation procedure], then
the [tutorial][Hare tutorial]. Have fun!

## Your humble hosts

I want to extend a quick note of appreciation to everyone who has helped with
Hare so far.

- Adnan Maolood
- Ajay Raghavan
- Alexey Yerin
- Andri Yngvason
- Armin Preiml
- Armin Weigl
- Bor Grošelj Simić
- Byron Torres
- Charlie Stanton
- Christopher M. Riedl
- Drew DeVault
- Evan Vogel
- Eyal Sawady
- Haelwenn Monnier
- Humm Smith
- Jonathan Halmen
- Karl Schultheisz
- Kiëd Llaentenn
- Luke Champine
- Miccah Castorina
- Michael Forney
- Mykyta Holubakha
- Noah Altunian
- Noah Pederson
- Noam Preil
- Quentin Carbonneaux
- Sebastian
- Simon Ser
- Steven Guikal
- Sudipto Mallick
- Thomas Jespersen
- Umar Getagazov
- Vlad-Stefan Harbuz
- Yasumasa Tada

Thanks, everyone!
