---
title: Hare's advances compared to C
date: 2021-02-09
author: Drew DeVault
---

The Hare programming language makes a number of improvements compared to the
language from which it draws much of its inspiration. Like C, Hare has a very
small runtime, manual memory access, direct access to pointers, and dangerous
features which you are free to point at your feet and shoot, albeit only after
signing a waver.

Despite the veneration with which we look upon C, those who look upon it with
scorn do have some valid concerns. A complete lack of memory-safe features and a
miserable error handling experience both make it pretty easy to make mistakes in
C that are not possible elsewhere. Some of this also makes it possible to do
desirable things which are not possible in those other languages, but it would
be nice to have options for when you don't need to do those things and just want
to write reliable code.

## Tagged unions for error handling

Tagged unions are the key innovation of Hare. Every function which could return
errors returns a tagged union of either the happy result (or results), or the
possible error types. Each possible error is described as part of the function's
API, rather than tossed into `errno` and ambiguously mixed in with every other
kind of error. We also make it easier for you to handle these errors in your own
code thanks to the `match` statement, which forces you to think about and
provide some kind of handling for each failure case.

```hare
fn io::write(s: *stream, buf: const []u8) (size | io::error);

// ...

sum += match (io::write(s, buf)) {
    err: io::error => match (err) {
        unsupported => abort("Expected write to be supported"),
        * => return err,
    },
    n: size => {
        process(buf[..n]);
        n;
    },
};
```

The match statement is required to be exhaustive (this is also true of
`switch`), so it's not possible to forget an error case by mistake. The compiler
helps you identify what errors are possible and make sure that they're covered.

## Nullable and non-nullable pointer types

`null` is an oft-derided feature of C and other programming languages. The real
issue is not that you can represent the absence of a value, but that you can
treat the absence of a value as if it existed, leading to segfaults and other
broken behaviors. Hare fixes this! In Hare, a pointer type can only be `null` if
it has the `nullable` attribute, and `nullable` pointers cannot be dereferenced
without first testing if they're `null`.

```hare
let x = 10;
let y: *int = &x;         // Guaranteed to be non-null
let z: nullable *int = y; // May be null!

*y; // Valid
*z; // Error: main.ha:6:19: Cannot dereference nullable pointer type

match (z) {
    null => abort(),
    z: *int => *z, // Valid
};
```

Again, however, if you think you're smarter than the compiler, it believes you.
You can cast `null` to a non-nullable pointer type explicitly. This is
occasionally necessary, for example, to initialize pointer globals during
`@init`.

## No uninitialized values

Uninitialized values is another big source of frustration for C programmers, and
a source of undefined behavior. In Hare, every time a variable is initialized,
you are required to provide a value for it.

```hare
let x: int; // Syntax error: unexpected ';' at main.ha:2:19, expected '='
```

If you need to do some logic to derive the appropriate value, then you can take
advantage of Hare's expression-based syntax to do so:

```hare
let x: int = if (foo) {
    let results = do_work();
    results.x;
} else 42;
```

We also offer the `...` operator for making it easier to initialize data in
bulk for arrays and structs. Some types define a default value, e.g. `0` for
`int`, and you can initialize struct fields to those defaults:

```hare
let x = coords { x = 1337, ... };
```

And you can fill out large arrays with `...` as well:

```hare
let x: [4096]int = [0...];
```

## Checked arrays and slices

Every array indexing operation in Hare generates a boundary test.

```hare
export fn main() void = {
	let x = [1, 2, 3];
	let y = x[42];
};
// $ hare run main.ha
// Abort: slice or array access out of bounds
```

Of course, sometimes you don't want this, so we let you use `[*]` to define an
array of undefined length, which has no boundary checks.

```hare
export fn main() void = {
	let x = [1, 2, 3];
	let y = &x: *[*]int;
	y[42] = 1337;
};
// $ hare run main.ha
// Segmentation fault
```

Slicing also makes some kinds of operations easier and safer for free. For
instance, let's say you want to copy part of a slice to another. You can assign
to a slice:

```hare
x[5..10] = y[5..10];
```

In C, we'd probably use memcpy instead:

```c
memcpy(&x[5], &y[5], 5 * sizeof(x[0]));
```

This introduces another opportunity for error: you can forget to use `sizeof`,
or use `sizeof` on the wrong object. This is a common cause of buffer overflows.
We also introduce a `len` operator, which removes the need for distinguishing
between array size and array length.

## Getting a handle on undefined behavior

Okay, we have a *little bit* of undefined behavior. The word "undefined" has a
special meaning in our specification, and is almost always used to describe a
situation that raises a compiler error. For example, a non-nullable pointer has
an "undefined" default value, and any type with an "undefined" default value
causes an error when used with the `...` operator. For example:

```hare
type my_struct = {
    x: *int,
    y: *int,
};

export fn main() void = {
    let foo = my_struct { ... };
    // Error: main.ha:8:27: field 'x' has no default value
};
```

The only other example is this quote from section 5.4.4 of the [Hare
specification](https://harelang.org/specification/):

> The name and signature of the program entry point function is undefined in the
> freestanding environment.

The freestanding environment simply does not define the entry point as `main`.
What Hare definitely does *not* have is the same kind of undefined behavior that
C compilers use as a license to do whatever they want to your program, often
with optimization as an excuse. On the subject of optimization, to quote the
specification again:

> If the implementation is able to determine that the evaluation of part of an
> expression is not necessary to compute the correct value and cause the same
> side-effects to occur in the same order, it may rewrite or re-order the
> expressions or sub-expressions to produce the same results more optimally.
>
> The interpretation of this constraint shall be conservative.  Implementations
> should prefer to be predictable over being fast. Programs which require
> greater performance shall prefer to hand-optimize their source code for this
> purpose.

There are many areas that C leaves undefined that we've decided to define. An
octet is always 8 bits. Shifting greater than the width of a value is defined.
Signed overflow and underflow is defined. Hare programs always have predictable
behavior.

Of course, we still allow you to shoot yourself in the foot, because any system
which prevents you from doing dangerous things also prevents you from doing
clever things. If you cast `null` to a non-nullable pointer and write to it,
something bad will probably happen.

## Better strings

POSIX locales are a nightmare. Hare improves upon this with the following
invariant: all strings are always valid UTF-8. We also store the length
alongside the string, allowing `NUL` to appear in the string contents, and for
`len(str)` to be an O(1) operation. Instead of `char`, we have `rune`, which is
a single UCS-32 Unicode codepoint. The Hare standard library also provides a
number of useful convenience functions which natively grok Unicode and are less
fraught with footguns when compared to POSIX.

If you know better than us, though, then again you have options. We provide the
`ascii` module for asking questions only of the ASCII subset of Unicode, such as
`ctype.h` equivalents like `ascii::isalnum`. You can also use the (O(1)!)
`strings::to_utf8` to convert a `str` into a byte slice `[]u8`, which you can
index and manipulate to your heart's content. You can also cast `str` to `*const
char` to get a NUL-terminated string in O(1) which you can pass to C functions.

## Strongly-typed variadism

`stdarg.h` is a convenient feature of C, but also one which is a common source
of errors. Hare instead supports variadism as a kind of syntax sugar over slices
and tagged unions. For example, consider `fmt::printf`:

```hare
// Tagged union of all types which are formattable.
export type formattable = (...types::numeric | uintptr | str | rune | nullable *void);

// Formats text for printing writes it to [os::stdout].
export fn printf(fmt: str, args: formattable...) (io::error | size) =
	fprintf(os::stdout, fmt, args...);
```

The `args` parameter here is a slice of `formattable` values. You can use
`len(args)` and index one with `args[3]`. Each value is strongly typed and
checked for validity at the call-site. Another function, `format`, takes
advantage of this:

```hare
fn format(
	out: *io::stream,
	arg: formattable,
	mod: *modifiers,
) void = match (arg) {
	s: str => io::write(out, strings::to_utf8(s)),
	r: rune => io::write(out, utf8::encode_rune(r)),
	p: uintptr => {
		let s = strconv::uptrtos(p);
		io::write(out, strings::to_utf8(s));
	},
	v: nullable *void => match (v) {
		v: *void => {
			let mod = modifiers { base = base::LOWER_HEX, ... };
			format(out, v: uintptr, &mod);
		},
		null => format(out, "(null)", mod),
	},
	n: types::numeric => // ...
};
```

This is how our `fmt` doesn't have to distinguish between `%s` and `%d` and
`%c`: because it's baked into the type which you pass to the function. Another
footgun removed! And, of course, if you're interoperating with C code, you can
declare a prototype which uses C-style variadism and call it normally from Hare
code, albeit with the obvious lack of type safety guarantees.

## In summary

Hare makes a number of conservative improvements on C's ideas, the biggest bet
of which is the use of tagged unions. Here are a few other improvements:

- A context-free grammar
- Less weird type syntax
- Language tooling in the stdlib
- Built-in and semantically meaningful static and runtime assertions
- A lightweight system for dependency resolution
- defer for cleanup and error handling
- An optional build system which you can replace with `make` and standard tools

Even with these improvements, Hare manages to be a smaller, more conservative
language than C, with our specification clocking in at less than 1/10th the size
of C11, without sacrificing anything that you need to get things done in the
systems programming world.
