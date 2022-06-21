---
title: Safety features of the Hare programming language
date: 2022-06-21
author: Drew DeVault
---

Hare offers many important advances over its main inspiration, C, when it comes
to safety features. These features prevent the programmer from making mistakes,
ideally without preventing them from doing what they want to do. Some of these
features reduce the risk of classic security problems, such as buffer overflows,
but they are generally designed with the broader hope of reducing the risk of
your program having bugs of any kind. After all, any bug can become a security
vulnerability under the right conditions.

Hare does not offer the same safety features as other programming languages. It
does not run in a sandboxed virtual machine like JavaScript or C#, and it does
not have a borrow checker like Rust. Safety features are one trade-off weighed
against many, and each programming language comes away with a different set.
Naturally, no programming language can totally prevent the introduction of bugs,
including security vulnerabilities. Regardless, features which can reduce that
risk are often useful, should the trade-offs be desirable.

How do we evaluate the trade-offs of a particular safety feature? Well, we have
to keep some of Hare's goals in mind:

1. Low-level programming support (e.g. kernel development)
2. Simplicity in language design and implementation
3. Transparency and explicitness in code

For instance, say we wanted to prevent use-after-free bugs. One approach would
be to use a garbage collector. These aren't too complicated, so goal #2 isn't in
trouble, but goals #1 and #3 aren't looking great. Garbage collection magically
stops your code (sometimes at predictable points, but it's generally not very
explicit), which is not very transparent and is definitely off the table for
use-cases like kernels, video games, real-time applications, and so on. The
behavior of a Hare program should be easy to predict, and a garbage collector
interferes with that goal. Thus, despite the fact that a garbage collector would
improve safety and ease-of-use, it's not a good fit for Hare.

There are other approaches to this particular problem (notably a borrow
checker), but I'll address that later. Let's take a look at Hare's safety
features.

## Slices & bounds-checked arrays

One of the most important safety features of Hare is the use of bounds-checked
arrays & slices. Each access to an object in an array or slice is tested at
runtime to verify that the access is in-bounds, and if this is not found to be
the case, the program aborts &mdash; rather than trample the stack/heap and
proceed as if everything is normal.

The fact that this is a first-class language feature encourages good design in
the standard library or third-party libraries. Rather than passing a separate
"length" parameter to any function accepting a slice, it's baked into the type,
which reduces the odds for error.

Of course, there are many use-cases in low-level programming which requrie the
use of unbounded arrays or which stores the length in a manner incompatible with
our type system. For this purpose, we offer the `[*]type` syntax for declaring
arrays of unbounded length, which are not bounds-checked. However, it's often
useful to create a slice from these arrays in order to quickly win back the
benefits of bounds checked slices. For instance, consider this code from Helios,
a kernel written in Hare:

```hare
fn load_mmap(mb: *multiboot::mb_header) void = {
	// Stash the memory map in the kernel so that we can trample the part of
	// RAM it was allocated into.
	assert(mb.flags & multiboot::MB_INFO_MEM_MAP != 0);

	let ents = mb.mmap_addr: uintptr: *[*]multiboot::mb_mmap_entry;
	let ents = ents[..mb.mmap_length / size(multiboot::mb_mmap_entry)];
	if (len(ents) > MMAP_MAX_ENTRIES) {
		ents = ents[..MMAP_MAX_ENTRIES];
	};
	mmap[..len(ents)] = ents;
	nmmap = len(ents);
};
```

The multiboot standard defines a length for the memory map array, and here we
create a slice of that length using the same underlying storage as the multiboot
array.

## Mandatory initializers

C does not require initializers for variables, which can lead to undefined
behavior. Consider the following code:

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
	int x;
	printf("%d\n", x);
}
```

This is a valid C program which demonstrates some simple undefined behavior: the
value of "x" is not defined. This program will generate warnings with most
modern C compilers, but it is valid nonetheless.

Hare gets around this by simply requiring all variables to be initialized as
soon as they are declared. Since Hare is an expression-based language, variables
which require logic to initialize correctly can be initialized from the result
of any arbitrary expression, such as an if or match statement.

## Mandatory error handling

The following program can fail, because the write(2) syscall (used by
fmt::println) can fail.

```hare
use fmt;

export fn main() void = {
	fmt::println("Hello world!")!;
};
```

The "!" operator shown here is an *error assertion*, which promises that the
error cannot occur. The compiler will check your work at runtime, and terminate
the program if the error does indeed occur&nbsp;&mdash; you can simulate this via
`hare run main.ha >/dev/full`. If we removed this operator, we would get a
compiler error, because you cannot ignore errors.

Unlike C, where "errno" is global and easily forgotten, error handling is part
of the type system of Hare and must be handled on a case-by-case basis. Errors
can also be more semantic, rather than selected from a global list or
re-invented by every library.

## Exhaustive switch & match

Another common cause of errors in C is the failure to handle all cases in a
switch statement. Consider the following code:

```c
switch (x) {
case 1:
	// ...
case 2:
	// ...
case 3:
	// ...
}
```

If "x" has any value outside of 1, 2, or 3, the switch statement has no effect,
which is often not the correct behavior. In Hare, switch and match statements
must be *exhaustive*, meaning that they handle every possible value or type that
can be switched or matched.

At least, that's what the specification demands. The compiler does not yet
enforce this uniformly.

## Nullable pointers

Pointers are ultimately just numbers, and their zero value has semantic meaning:
the value is not defined. "Dereferencing" such a pointer usually leads to a page
fault, or occasionally something worse. However, a "null" value is useful,
because often a pointer *does* refer to a value that does not exist. To balance
these concerns, Hare offers a special nullable pointer type.

```hare
let x: nullable *int = null;
*x; // Error: Cannot dereference nullable pointer type
```

Such pointers must use a match expression to test for null before they can be
dereferenced, preventing null pointer dereferences.

## Strongly-typed variadism

Many users of languages other than C are unlikely to be impressed by this
feature, but C programmers who have written variadic functions (i.e. with
`stdarg.h`) know that the type of each variadic parameter is not defined. The
user must signal this information out of band, and can make mistakes, like this:

```c
printf("%s", 1234); // 1234 is not a string, but this will compile correctly!
```

Hare's variadism is strongly typed. Variadic parameters must meet a specified
type, and supporting more than one type is facilitated by the use of a tagged
union. The set of parameters is passed as a slice, with a length, which omits
the need for adding a sigil at the end to signal the end of the arguments (such
as passing NULL to execl to indicate the end of the command line arguments).

```hare
fmt::printfln("{}", 1234); // 1234 is cast to fmt::formattable
```

## Defining undefined behavior

Undefined behavior, some would argue, is the scourge of C. It's a seemingly
bottomless bucket of problems, and gives the compiler free license to rewrite
your program into nethack if you trip over it. Hare does things differently.

For a start, much of the behavior that C leaves undefined is defined by Hare.
For instance, signed overflows are defined by the specification. These are left
undefined by C often in deference to existing implementations or older
architectures which have different behavior, but Hare chooses to leave the
legacy behind and nail this down further.

Hare does have some *implementation-defined* behavior, such as the size of a
pointer type. These are not defined by the specification, but must be defined by
the implementation (in an ancillary specification), so that programmers can
plan for their behavior. The specification often places bounds on these
definitions as well, such as requiring that all types have a size which is a
power of two, even when implementation-defined.

Finally, the Hare specification includes the following:

> If the implementation is able to determine that the evaluation of part of an
> expression is not necessary to compute the correct value and cause the same
> side-effects to occur in the same order, it may rewrite or re-order the
> expressions or sub-expressions to produce the same results more optimally.
>
> *The interpretation of this constraint shall be conservative. Implementations
> should prefer to be predictable over being fast. Programs which require
> greater performance shall prefer to hand-optimize their source code for this
> purpose.*

This effectively shuts down compilers from giving their optimizers endless
leverage. For a Hare program to be fast, it must be written with performance in
mind, rather than relying on the compiler to do it for you. This plays into
Hare's goals for explicitness and transparency, and improves the safety and
predictability of the language.

### ASLR, W^X, pledge, etc

There are many general security features we could develop which are not related
to the language design, such as ASLR (address space layout randomization), stack
canaries, and so on. We can also make security features from the host
environment, such as BSD's pledge or Linux's keyctl features, available to Hare
programs to utilize to improve their security. We have implemented some of
these, and plan to do more, but they are just a matter of time. These features
don't generally require any major design changes to the language.

## Cryptography

The standard library's introduction to the "crypto" module includes the
following statement:

> Cryptography is a difficult, high-risk domain of programming. The life and
> well-being of your users may depend on your ability to implement cryptographic
> applications with due care. Please carefully read all of the documentation,
> double-check your work, and seek second opinions and independent review of
> your code. Our documentation and API design aims to prevent easy mistakes from
> being made, but it is no substitute for a good background in applied
> cryptography.

The cryptographic implementation provided by the standard library takes
advantage of all of these language features to be as safe as possible. It is not
possible to forget to check that a message signature was correctly verified, or
to cause a buffer overflow by mixing up pointers and lengths when dealing with
cryptographic buffers. The implementations are carefully written with
constant-time code where appropriate, and with a careful eye on buffer usage to
avoid leaving sensitive data on the stack or heap.

We also offer a high-level implementation of common cryptographic operations
such as signing, key exchange, key derivation, and so on, which make use of
sensible default primitives and are designed to be difficult to mis-use.

However, we do rely on the programmer to do their part as well. Users of the
cryptographic implementation should have relatively good background knowledge
and should be as careful with their work as we were with ours. Reading the
documentation is mandatory.[^1]

Our cryptographic implementation has not yet been audited, which is made clear
in the documentation. We are [raising money] to fund an audit.

[raising money]: https://opencollective.com/hare/projects/cryptography-audit

[^1]: One criticism of our cryptography implementation comes from the
[crypto::keystore] module, which is designed to remove encryption keys from the
user's address space while not in use&nbsp;&mdash; a defense in depth strategy.
This criticism originates from a relatively well-regarded (and very vocal)
security expert on Twitter, and seems to be proliferating throughout the
internet, so I would like to briefly address it here.<br /><br />
The keystore module provides an opportunistic improvement for security. On
platforms where the necessary kernel features are not present, it falls back to
an implementation which offers no additional security, and this is the source of
the critique.<br /><br />
However, this fallback mode does not introduce a security vulnerability: the
purpose of this module is to prevent an *existing* security vulnerability,
namely the ability to read your process's memory, from being able to extract
your keys through this hole. Without such a vulnerability, there's no issue.
<br /><br />
This behavior is well-documented in the standard library documentation, which
again is considered required reading for any programmers making use of the
cryptographic implementation. Some suggest that it should outright fail if the
required kernel features are not available, but I think this comes down to a
misunderstanding of the module's design, or a simple difference of opinion.<br
/><br />
An improvement to this module could allow the programmer to choose to
disable the fallback mode if they require the additional layer of security, or
to improve the fallback, perhaps by forking and storing the keys in a separate
process. I am open to such improvements. In any case, this particular design
conflict has been used to justify an unqualified damnation of our cryptography
code and our language as a whole, which I do not really feel is appropriate.
Please seek first to understand, in earnest, and then make your suggestions.
Thank you.

[crypto::keystore]: https://docs.harelang.org/crypto/keystore

## No package manager

This one might raise eyebrows, but I consider Hare's lack of a package manager
to be an important security feature. There have been thousands of packages
compromised on countless language-specific package managers: npm, PyPI,
RubyGems, Cargo, and more&nbsp;&mdash; none of them have escaped this. This
comes down to a fundamental problem in their design: arbitrary people on the
internet cannot be trusted to publish packages directly for user consumption.

Hare does better in two respects: downstream distributors and a better culture
of dependencies. Hare packages, both first-party and third-party, are primarily
distributed by downstream distributors, such as Linux distributions. This adds a
separate layer of trust and validation, by establishing your distro maintainer
as an independent auditor. A package is only added when there is demand,
typosquatting is virtually eliminated, updates roll out more slowly, and there's
an independent point of contact you can approach with concerns, whose interests
are aligned with the user rather than with the developer.

Furthermore, we encourage a greater deal of conservatism in the Hare
programmer's approach to dependencies. Many other languages &mdash; Node, Go,
Rust, others &mdash; encourage an explosion of dependencies which is impossible
to audit or understand. Hare aims to provide enough features in the standard
library to make many programs possible without dependencies at all, and others
possible with very few dependencies. I am not aware of any Hare program today
with more than three dependencies. Each dependency added is an important,
deliberate decision. There are no micro-dependencies.

All together, this makes dependencies much safer in Hare than most other modern
languages.

## Hare's more dangerous features

Hare's stated design principles caused a bit of a stir for the inclusion of
"trust the programmer". I have removed them, because they need to be presented
better, but I can clarify this particular principle now: it's better stated as
"trust the programmer, but *not by default*". Every one of the features I have
explained now can be circumvented, by design, if the programmer tells the
compiler that they know better.

I've already mentioned support for unbounded arrays, which is one risky feature.
However, the most dangerous Hare feature is casting. Consider the following
code:

```hare
let x: *int = null: *int;
*x; // segfault
```

A non-nullable pointer type cannot contain null, unless you use a cast to force
the situation. If you tell the compiler that you know better, it will believe
you, but at your own risk. Many conversions will require multiple casts for each
dangerous step &mdash; for example, casting a rune to a pointer is possible, but
requries several casts to get there (rune → u32 → u64 → uintptr → \*whatever).
The riskier the cast, the more work it takes to do.

These features are important and necessary for many use-cases, which is why they
are there, but when you side-step these safety features you must do the extra
work to be certain that you have not made a mistake. Other low-level languages
provide similar features, such as Rust's "unsafe" keyword. Safety is a goal,
but if it gets in the way, you still have to be able to write your program.

## The elephant in the room: Memory safety

Hare lacks a borrow checker or any feature like it, which makes several classes
of bugs possible in Hare programs where they might be elimated by other
languages, notably Rust.

```hare
let x: *int = alloc(1337);
free(x);
*x = 42; // This compiles, but it's wrong!
```

This is an area where Hare presently does not make any improvements over C, and
it's an area that some people feel quite passionate about. However, we are not
without hope. We plan on improving Hare's memory allocator to support a better
view of your program's behavior, similar to what Valgrind provides for C, which
will be able to detect issues like double-frees or use-after-free. We will
likely include this behavior by default, simply aborting the program in release
mode, or tracking backtraces and printing more useful information in debug mode.
Additional static analysis is also possible without modifications to the
language, but it's unlikely to completely address the issue.

Though Hare does not have a borrow checker, we do borrow (hah) the vocabulary of
one to help users understand their memory allocation. Planning your memory
management is a critical concern for the design of a Hare program. Hare makes
liberal use of return-by-value so that initializer functions can return an
object which is placed on the caller's stack, without any memory allocation,
such that it's automatically cleaned up on return. Functions which take
references to objects are clearly documented as "borrowing" or "adopting" those
objects, so the programmer can easily understand their behavior.

So, no, Hare does not have a borrow checker. For now.

## A borrow checker for Hare?

I have not ruled out the possibility of adding a borrow checker, though. I wrote
[a blog post] in March of last year considering the possibility as an area for
future research.[^2] Rust is famous for its borrow checker, and Hare is much
simpler than Rust. To add one to Hare will require paring it down to the
essentials and fitting it into our simpler language design. I'm pretty sure that
it can be done.

[a blog post]: https://harelang.org/blog/2021-03-30-future-research-areas/
[^2]: As for the fate of the other two features: closures were ruled out, and
  async I/O is done with poll and O\_NONBLOCK, similar to C.

Unfortunately, many Rust experts have offered their scorn to Hare on the basis
of this feature's absence. However, I do not consider the bridge burnt. If you
are an expert on borrow checking in Rust, I would appreciate having you around
to help us explore the idea for Hare. I do actually think it's a good idea! But
we need help to make it happen.

If we cannot ultimately figure out how to make borrow checking work well for
Hare, then that's okay. Hopefully this blog post dispelled the notion that Hare
is an utterly unsafe programming language, with its wealth of safety features
available. I do believe that there is still room for programming languages
without borrow checkers.

## In conclusion

Hare offers many safety features to help programmers avoid making mistakes. It
also offers escape hatches for programmers who need to outsmart the compiler for
their use-case&nbsp;&mdash; though, importantly, the default behavior is always
safe. There is room for improvement, and help from the community is always
welcome in this regard.

Enjoy Hare responsibly!
