---
title: Making Hare more debuggable
author: Drew DeVault
date: 2022-11-04
---

Hare programs need to be easier to debug. This blog post outlines our plans for
improving the situation. For a start, we'd like to implement the following
features:

1. Detailed backtraces
1. Address sanitization
1. New memory allocator
1. DWARF support

These are roughly ordered by complexity &mdash; let's take a look at each one in
detail.

## Detailed backtraces

Hare programs are always compiled with frame pointers. On x86\_64, this involves
generating the following entry point for each function:

```gas
.globl example
example:
	pushq %rbp
	movq %rsp, %rbp
	/* ... */
```

This code pushes the %rbp register to the stack, then moves the stack pointer
into %rbp. This allows us to walk the stack: the value at (%rbp) is the stack
address at the entry to each function in the call stack, and (%rbp-1) is the
return address of the corresponding call. Hare's rt (runtime) module provides
some helper functions for walking the stack in this manner:

```hare
use fmt;
use rt;

export fn main() void = {
	func_a();
};

fn func_a() void = {
	func_b();
};

fn func_b() void = {
	backtrace();
};

fn backtrace() void = {
	let fp = rt::backtrace();

	fmt::println("Backtrace:")!;
	for (true) {
		const frame = fp.addr: *[*]uintptr;
		fmt::printfln("\t{:x}", frame[-1])!;

		match (rt::nextframe(fp)) {
		case let next: rt::frame =>
			fp = next;
		case void =>
			break;
		};
	};
};
```

When run, this program gives the following output:

```
$ hare build -o main main.ha
$ ./main
Backtrace:
	80000fa
	80000ef
	80000e4
	801b34d
	801867d
```

This is a list of return addresses for each function call in the call stack.
This isn't entirely useful on its own, but we can use a tool like addr2line
(from binutils) to convert it into something more helpful:

```
$ addr2line -fe main 80000fa 80000ef 80000e4 801b34d 801867d
func_b
/tmp/f013f4c7a05ea3cb/temp..21.s:29
func_a
/tmp/f013f4c7a05ea3cb/temp..21.s:18
main
/tmp/f013f4c7a05ea3cb/temp..21.s:7
rt.start_ha
/tmp/cdddf8beb5147f76/temp.rt.1.s:37277
rt.start_linux
/tmp/cdddf8beb5147f76/temp.rt.1.s:5532
```

This is still not ideal, being full of our internal temporary build artifact
names, but we can see the function names in the call stack now. Getting the
names of the functions associated with each address from our Hare program will
require finding our program's symbols, which is what addr2line is doing here.

In order to do something similar, Hare will have to find the process's ELF data
and locate each address within the symbol table. I already did something similar
to this for the fault handler in [Helios][0], so a lot of the necessary code
already exists. Once all of this is in place, we can at least annotate
backtraces with function names, which we can take advantage of on assertion
failures, when running tests, and so on. Even better backtraces will have to
wait for DWARF, which is discussed later in this article.

[0]: https://sr.ht/~sircmpwn/helios/

## Address sanitization

I would like to implement an address sanitizer for Hare programs, similar to the
one provided by Clang. I was recently doing some research on how this works, and
it turns out to be surprisingly simple and, in theory, pretty straightforward to
implement. You can read about the algorithm Clang uses [here][1], but I will
summarize it here.

[1]: https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm

The purpose of the address sanitizer is to test for out-of-bounds reads or
writes, buffer overflows, and so on. This can also work with the memory
allocator to detect use-after-free problems and the like. It works with the use
of something called "shadow memory": an extra region of memory that tracks what
parts of memory are valid or invalid, aka "poisoned".

Hare already has a number of checks in place which makes an address sanitizer
less useful, such as bounds testing all array and slice accesses, prohibiting
uninitialized variables, and so on. However, the compiler allows you to
circumvent these checks and, in some situations, it is necessary to do so.
Accordingly, an address sanitizer would be helpful for detecting problems when
you draw outside of the lines.

The first step is to set aside shadow memory areas to keep track of what memory
is valid or invalid. Each byte of shadow memory describes 8 bytes of real
memory, and a bit which is set is considered "poisoned". To store shadow memory,
we can create a large memory mapping in the process which is mostly empty and
maps each address of real memory to a bit in shadow memory. On 64-bit systems,
the memory map looks like this (taken from the ASan docs):

```
[0x10007fff8000, 0x7fffffffffff] 	HighMem
[0x02008fff7000, 0x10007fff7fff] 	HighShadow
[0x00008fff7000, 0x02008fff6fff] 	ShadowGap
[0x00007fff8000, 0x00008fff6fff] 	LowShadow
[0x000000000000, 0x00007fff7fff] 	LowMem
```

Each shadow region describes the corresponding memory area, and the "gap" is
used to prevent addresses within each shadow area from being used directly. A
real address can be quickly converted to its shadow address with the expression
`(addr >> 3) + 0x7fff8000`. The large mmaps which cover these regions are
allocated sparsely by the kernel and mapped on-demand (causing a page fault)
when read from or written to. A new shadow page will be zeroed by the kernel,
which will indicate unpoisoned memory by default.

When reading or writing to memory, we can look up its shadow address and test if
the address is poisoned. In [qbe][2] IR, this looks like the following:

[2]: https://c9x.me/compile/

```qbe
	// let addr: *u64 = ...
	// return *addr;
	%s.0 =l shr %addr, 3
	%s.1 =l add %s.0, 0x7fff8000
	%s.2 =l loadl %s.1
	jnz %shadow.2, @invalid, @valid
@valid
	%val =l loadl %addr
	ret %val
@invalid
	call $rt.asan_invalid_load8(l %addr)
```

The other piece to complete the ASan implementation involves marking invalid
memory as poisoned. For stack allocations, this involves allocating a red zone
on either side of the allocation, and marking the red zones as invalid. For
example, given the following Hare program:

```hare
let x: u64 = 10;
```

We would allocate 96 bytes (rather than 8) on the stack, and mark each of the
extra bytes as poisoned.

```
  %binding =l alloc8 96
  %x =l add %binding, 32

  %rz.0 =l shr %binding, 3
  %rz.1 =l add %rz.0, 0x7fff8000
  storew 0xFFFFFFFF, %rz.1
  %rz.2 =l add %rz.1, 4
  storew 0xFFFFFF00, %rz.2
  %rz.3 =l add %rz.2, 4
  storew 0xFFFFFFFF, %rz.3
```

At the function's exit, a similar process can be used to unpoison the memory.
The QBE IR shown above would then call rt::asan\_invalid\_load8 when attempting
to access the red zone. It's pretty straightforward &mdash; address sanitizer is
a really brilliant and simple design. Big kudos to the Clang team for coming up
with and implementing it.

## New memory allocator

The next piece of Hare's debuggability goals is to write a new allocator. Today,
Hare uses a very simple allocator. I wrote it a long time ago based on a very
simple design with the goal of having something simple and working implemented
quickly. However, a more sophisticated allocator would offer many benefits.

The main improvements for debuggability would involve detecting and reporting on
heap corruption. This would be most easily addressed by implementing ASan and
poisoning memory outside of the user's requested allocation as necessary.

There are some other improvements we can make as well. We can probably detect
double-free without ASan, and leave that turned on for all builds. Another good
idea would be to store a cache of backtraces which reports on the functions
which are allocated or freed a given object, valgrind-style &mdash; probably
turned off by default, but very helpful for narrowing down memory issues.

## DWARF support

The most difficult challenge for debugging Hare programs is implementing
[DWARF][3]. This is a format for encoding debugging information into programs,
and is used to map instructions to file names and line numbers and store
information about variables and types. This information is then utilized by
interactive debuggers like gdb, and could be used internally for things like
further improving backtraces to include file names and line numbers. However,
DWARF is very complex.

[3]: https://dwarfstd.org

The DWARF specification (which is 459 pages long) defines a virtual machine that
has to be implemented in order to interpret its data. The main purpose of this
is to reduce the size of the debugging tables, which would otherwise be very
large. The price is significantly increased implementation complexity.
Nevertheless, we intend to eat this cost. It will involve patching qbe, and will
ideally bring improved debugging support to qbe's other frontends, such as
[cproc][4].

[4]: https://git.sr.ht/~mcf/cproc

With DWARF support in place, another improvement which might make sense to add
is to teach gdb about Hare-specific semantics. For instance, the simplest of
these would be to teach gdb to translate Hare identifiers like unix::passwd into
symbol names, e.g. unix.passwd, and vice-versa.

## Closing thoughts

Hare as a language provides many features which reduce the risk of bugs
occuring in the first place, such as bounds-tested arrays, mandatory error
handling, and so on. Some problems, like buffer overflows, are very rare in
Hare. Others are easier to overlook, like use-after-free bugs. In any case, it's
important for us to make it easy to debug Hare programs, in order to improve the
user's odds of designing a robust program.

Each of these ideas interacts with the others, and when composed they form a
much more robust debugging system. A new memory allocator would take advantage
of ASan to detect use-after-free and improved backtraces for error reporting.
ASan would also take advantage of improved backtraces, and better backtraces
would rely on DWARF to resolve addresses into file names and line numbers.

If you're interested in helping with any of these ideas, [please join us][5].
We would be pleased to have your help.

[5]: https://harelang.org/community/
