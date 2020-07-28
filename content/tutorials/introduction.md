---
# vim: ts=8 sw=8 noet :
title: An introduction to the Hare programming language
summary: |
  This tutorial will introduce you to the Hare programming language. It
  should take about an hour to complete. If you are already familiar with
  programming in other languages, it might take even less time — feel free to
  jump around as you see fit.

  After you complete this tutorial, you should move on to the
  [standard library introduction](/tutorials/stdlib).
---

## Before you start

Make sure to follow the [installation instructions](/installation) for your
platform. You need to be using one of the [supported platforms](/platforms).

## "Hello, world!"

To test that your installation went smoothly, and to introduce you to the build
tools, we'll compile and run the traditional "Hello, world!" program. It is
recommended to set aside a dedicated directory for each Hare program you want to
run. Create one of those now: `mkdir example`; then `cd example`. With your
editor of choice, open a file named `main.ha` and fill it in with the following
contents:

```hare
use io;

export fn main void =
{
	io::println("Hello, world!");
};
```

You can run this program with the following command:

```
$ hare run main.ha
Hello, world!
```

Or, you can compile it to an executable like so:

```
$ hare build -o example main.ha
$ ./example
Hello, world!
```

We'll go into more depth on the build system later on.

## Basic usage

Let's go over some basic features of Hare.

### Variable bindings

Variable give names to values, and are bound using the `let` and `const`
keywords, and assigned to with the `=` keyword. You can define and use them like
so:

```hare
use io;
use strconv;

export fn main void =
{
	const x = 10;
	let y = 20, z = 30;
	/* Print our variables: */
	io::println(strconv::itos(x + y + z));
	y = 30;
	io::println(strconv::itos(x + y + z));
};
```

<p class="alert"><strong>Notice</strong><br />
<code>/* ... */</code> is used for comments &mdash; these are ignored by the
compiler, but useful to anyone reading your code.</p>

In this case, the type of the variable is *inferred* from its *initializer*,
which is the value to which it is initially set. Initializers are mandatory when
declaring variables. There are very few cases where type inference is permitted
in Hare. You may choose to (or in some cases, are required to) declare the type
explicitly:

```hare
use io;
use strconv;

export fn main void =
{
	const x: int = 10;
	let y: int = 20, z = 30;
	io::println(strconv::itos(x + y + z));
	y = 30;
	io::println(strconv::itos(x + y + z));
};
```

This program is functionally equivalent to the previous program. In general,
type inference is *not* used throughout Hare, but simple variable bindings are
an exception. We'll explain the other cases later, but if in doubt, add the
type.

### Conditional expressions: if

Hare offers an `if` operator which may be used to implement conditional logic.
For example, consider the following:

```hare
use io;
use os;

export fn main void =
{
	if (len(os::args) == 2) {
		io::println(os::args[1]);
	} else {
		io::println("No argument provided");
	};
};
```

<p class="alert"><strong>Notice</strong><br />Take note of the semicolon (`;`)
at the end of the if expression &mdash; semicolons are required after
<strong>all</strong> expressions. There's also a semicolon after the
function!</p>

`os::args` is the list of arguments passed into your program. If there are
exactly two of these, the first *branch* of the if expression is taken and the
argument is printed. Otherwise, the second branch is taken, and "No argument
provided" is printed.

You can try this like so:

```
$ hare run main.ha
No argument provided
$ hare run main.ha hello
hello
```

The first value in `os::args` is the name of the program. Try this program
instead:

```hare
use io;
use os;

export fn main void =
{
	io::println(os::args[0]);
};
```

```
$ hare run main.ha
main.ha
$ hare build -o example main.ha
$ ./example
./example
```

#### Boolean expressions

The expression between the parenthesis `()` in an if expression is required to
have a boolean type (known as `bool` in Hare). Certain arithmetic operators have
a boolean result: `==` (equals), `!=` (not equal), `>` (greater than), `<` (less
than), and `>=` and `<=` (greater than or equal to, and less than or equal to).
Additionally, `||` and `&&` are used to compare two boolean values, respectively
they are the "or" and "and" operators. The former results in true if *either*
expression is true; the latter is true if *both* expressions are true.

These two operators may *short-circuit*. If the result can be inferred from the
first expression, the second expression is *not* evaluated. In the case of `||`,
if the first expression is true, we can infer that the result will be true
regardless of the second expression. In the case of `&&`, if the first
expression is false, we can infer that the result will be false. This can be
important if the second expression has side effects, for example:

```hare
if (2 == 4 && crash_the_program()) {
	/* ... */
} else if (2 == 2 || crash_the_program()) {
	/* ... */
};
```

In this code, `crash_the_program()` will never be evaluated.

The constants `true` and `false` are also provided by the language.

```hare
let a: bool = true;
let b: bool = false;
```

#### "if" use as an expression

Hare is an *expression-based* programming language. The result of an if
expression has a type and a value, and you can use it as such. For example,
consider the following:

```hare
use io;
use os;

export fn main void =
{
	const message: str = if (len(os::args) == 1) {
		"Program was run without args";
	} else {
		"Program was run with args";
	};
	io::println(message);
};
```

The result of an if expression is taken from the last value in each branch, in
this case a string. If the types of each branch are not compatible, the result
is `void`: nothing. Additionally, an `if` expression with just one branch
&mdash; that is, no "else" &mdash; always has the `void` type.

Note that the variable we've used here is given an explicit type. As a rule of
thumb: if an expression has *branches*, the type must be explicit. You could
also eliminate the variable and use the if expression directly:

```hare
use io;
use os;

export fn main void =
{
	io::println(if (len(os::args) == 1) {
		"Program was run without args";
	} else {
		"Program was run with args";
	});
};
```

The behavior is the same.

<p class="alert"><strong>Notice</strong><br />
Hare does not have a <em>ternary</em> operator like C's
<code>cond : x ?  y</code>, preferring the use of expressions like
<code>if (cond) x else y</code>. The braces (<code>{}</code>) may be omitted for
simple cases such as this.
</p>

We mentioned earlier that if expressions can have multiple branches; this is
accomplished with the `else if` keyword.

```hare
use io;
use os;

export fn main void =
{
	io::println(if (len(os::args) == 1) {
		"Program was run without args";
	} else if (len(os:args) == 2) {
		"Program was run with one argument";
	} else {
		"Program was run with several args";
	});
};
```

### Loop expressions: for & while

Hare also supports *loops*, which come in two forms: `for` loops and `while`
loops. The former is a more structured loop, and the latter is more free-form.
Both cases always return the `void` type.

A for loop has three parts: the initializer, the predicate, and the
afterthought. At the start of the loop, the initializer is run; at the start of
each iteration, the predicate is evaluated to determine if the loop shall
continue; and at the end of each iteration, the afterthought is run.

```hare
for (let x = 0; x < 10; x += 1) {
	io::println(strconv::itos(x));
};
```

This program will print the numbers zero through nine.

<p class="alert"><strong>Notice</strong><br />
See the <code>x += 1</code> operator there? <code>+=</code> is short for "add,
then assign", and this expression adds 1 to x and then assigns the result to x;
it is shorthand for <code>x = x + 1</code>.
</p>

While loops only have a predicate, and have neither an initializer nor
afterthought. The following loop is similar to the for loop in the previous
example:

```hare
let x = 0;
while (x < 10) {
	io::println(strconv::itos(x));
	x += 1;
};
```

#### Loop control operators

The `break` and `continue` keywords are provided to influence the loop flow
outside of the predicate. The `break` keyword will terminate the loop
immediately, and the `continue` keyword will immediately begin the next
iteration, skipping the remainder of the code in that loop. The following for
loop will print the numbers zero through nine, but skip five:

```hare
for (let x = 0; x < 10; x += 1) {
	if (x == 5) {
		continue;
	};
	io::println(strconv::itos(x));
};
```

Replace `continue` with `break` and this loop will terminate at five, printing
only the numbers 0, 1, 2, 3, and 4.

### Functions

You've been using the `fn` keyword to declare the "main" function in each of
the examples shown so far. You can use this keyword again to create your own
functions, like so:

```hare
use io;

fn say_hello void =
{
	io::println("Hello, world!");
};

export fn main void =
{
	say_hello();
};
```

The components of a function declaration are the export status (optional), the
`fn` keyword, the function name (written in lowercase_snake_case), the parameter
list (optional), and the result type (`void` in these examples). The export
status will be discussed more when we cover modules; the short explanation is
that it determines whether or not the function is visible from outside this
module.

The two functions in this example accept no parameters, but we could define one
that does:

```hare
use io;
use strconv;

fn add_ints(a: int, b: int) int = a + b;

export fn main void =
{
	let x = add_ints(2, 4);
	io::println(strconv::itos(x)); /* 6 */
};
```

Note that the braces are optional. Some expressions, known as *simple*
expressions, are permitted to be written in a "bare" form like this. The same
rule of thumb we used for variable type inference applies here: if an expression
has multiple *branches*, it is considered *complex*; otherwise it is usually
*simple* and can be used in this manner. Regardless, we may add braces if we
wish:

```hare
fn add_ints(a: int, b: int) int =
{
	return a + b;
};
```

However, if we do, we must use the `return` keyword to explicitly "return" the
value to the caller. This can also be used at any time to "return" early:

```hare
fn add_ints(a: int, b: int) int =
{
	if (a == 3 || b == 3) {
		/* 3 is evil */
		return -1;
	};
	return a + b;
};
```

#### Variadic parameters

TODO

### Constants

It's often convenient to assign a name to a constant value, such as Pi. The
`def` keyword is used for this purpose (and also for user-defined types &mdash;
to be explained later).

```hare
use io;
use strconv;

def THE_ANSWER: int = 42;

export fn main void =
{
	io::println(strconv::itos(THE_ANSWER));
};
```

Constants are written with UPPERCASE_SNAKE_CASE.

### Globals

The use of globally mutable variables ("globals") is discouraged. However, they
are necessary in some cases, and support is provided. You can use the `let`
operator outside of the context of a function to create a global:

```hare
use io;
use strconv;

let x: int = 1337;

export fn main void =
{
	io::println(strconv::itos(x));
};
```

The type specifier (`: int`) and initializer (`= 1337`) are mandatory for
globals.

## Types

Hare's type system is relatively small and straightforward.

### Primitive types

The following primitive types are available:

- `int`, `uint`: native-precision signed and unsigned integer, respectively
- `i8`, `i16`, `i32`, `i64`: explicit-precision signed integers
- `u8`, `u16`, `u32`, `u64`: explicit-precision unsigned integers
- `f32`, `f64`: floating point numbers, 32- and 64-bit respectively
- `bool`: boolean type (either true or false)

"Native-precision" types vary in their specific precision according to the
target architecture; usually they are 32 or 64 bits. Explicit-precision types
use the precision in bits specified by the numeric suffix in the type name, i.e.
i8 is an 8-bit signed integer and u32 is a 32-bit unsigned integer.

Numeric constants are supported in Hare with an explicit precision. The constant
`1234` is of type `int`, and `1234u` is of type `uint`. You can use an explicit
precision as well: `1234u16` is a `u16`, and `12f32` is `12.0` represented as an
`f32`.

Alternate numeric constant formats include hexadecmial using `0x` notation (e.g.
`0xDEADBEEF`), binary via the `0b` notation (e.g. `0b101010`), and octal with a
leading zero (e.g. `0644`).

TODO: Add scientific form

Additionally, the following primitive types have special semantics:

**void** is a type of size zero. It is used only in special cases, and a
variable of this type may not be declared.

**size** is an unsigned integer which can specify a size at least as large as
the native address space. It is the result type of the `size` operator and is
used for array and slice indicies.

**uintptr** is an unsigned integer capable of representing a pointer type
numerically. This is used for pointer arithmetic, though its use is discouraged.

**char** is similar to u8 and is available for compatibility with C. It is not
used directly.

### Strings

The `str` type represents an immutable string (immutable meaning it cannot be
changed, or *mutated*) with a fixed length. Strings are stored prefixed with
their length (as `size`), followed by the string contents as UTF-8, then a NUL
terminator (which is not included in the length).

String manipulation in Hare deserves special treatment, we'll discuss it in
depth in the [standard library introduction](/tutorials/stdlib) later on.

### Arrays

An array stores many items of a type, and has a fixed length. The syntax is
<code>[<em>length</em>]<em>type</em></code>, e.g. `[5]int`. A specific member of
an array can be accessed by *indexing* the array, like so:

```hare
let x: [5]int = [1, 2, 3, 4, 5];
x[3]; /* 4 */
x[3] = 10;
```

Note that Hare arrays are *zero-indexed*, meaning the first item in the array is
at index zero. Also take note of the array initialization syntax shown here.

Array accesses like this are bounds-checked at runtime. Attempting to access an
item beyond the end of the array, or before the beginning of the array, will
abort the program.

The length of an array may be obtained with the `len` operator. In this case,
`len(x)` would return 5.

#### Unbounded arrays

It may occasionally be useful (especially when interoperating with another
programming language) to use an array of undefined length, which is not
bounds-checked at runtime. You may declare an array whose length is `*` to
produce an array type which is not bounds-checked, i.e. `[*]int`. Use this
feature with care.

#### Array initializer shorthand

Hare requires that you provide an initializer for all variables, and this may
often be cumbersome when working with large arrays. The following code would
become tiresome quickly:

```hare
let x: [4096]int = [0, 0, 0, 0, 0, 0, 0, 0, /* ... */];
```

Hare offers a syntax for filling out the remainder of an array for you via the
`...` operator:

```hare
let x: [4096]int = [0...];
```

In this example, Hare will populate the remainder of the array with zeros.
Note that it needn't be zeros, nor the first item that is extended; the
following is also valid:

```hare
let x: [4096]int = [1, 2, 3...];
```

In this case, the first item is one, the second is two, and all remaining items
are three.

### Slices

A slice is an indirect view of an array. It is used for two purposes: as a
subset of another array or slice, or as an array which can grow at runtime. A
slice has three properties: its length, its *capacity*, and an indirect
reference to the underlying storage.

Like arrays, the length of a slice may be obtained with the `len` operator.

TODO: Go into more detail on slices

#### Sub-slicing

You can use the `..` indexing operator to obtain a subset of another slice, also
known as a *sub-slice*. The subslice operator may also be used on an array, but
such slices are immutable.

For illustrative purposes, consider the following array:

```hare
let x: [5]int = [1, 2, 3, 4, 5];
```

To create a subslice, use the syntax
<code>x[<em>start</em>..<em>end</em>]</code>, where start is the index of the
first element, and end is the index of the last element. "Start" is *inclusive*,
meaning that it includes the item at that index, and "end" is *exclusive*,
meaning that it includes items *up to*, but not including, the "end" index.

The start or end index may be omitted entirely. If omitted, zero is used for
start, and `len(array)-1` is used for end. `x[..]` creates a sub-slice of the
entire slice.

```hare
x[1..4]; /* 2, 3, 4 */
x[2..4]; /* 3, 4 */
x[..2];  /* 1, 2 */
x[2..];  /* 3, 4, 5 */
x[..];   /* 1, 2, 3, 4, 5 */
```

### Structs & unions

A "struct" type is used to create an aggregate type encapsulating several other
types. They are declared with the following syntax:

<pre><code>struct {
	<em>field</em>: <em>type</em>,
	<em>field</em>: <em>type</em>,
	/* ... */
}</code></pre>

A simple example could be used to store coordinates in a 2D plane:

```hare
let coords: struct {
	x: int,
	y: int,
} = /* ... */;
```

A *struct initializer* is very similar, but also provides initializers for the
values:

```hare
let coords = struct {
	x: int = 10,
	y: int = 20,
};
```

Thus, a fully qualified declaration & initialization of this variable would be:

```hare
let coords: struct {
	x: int,
	y: int,
} = struct {
	x: int = 10,
	y: int = 20,
};
```

Evidently, this is somewhat redundant, so type inference is preferred for this
case. To access a struct field, the `.` operator is used.

```hare
coords.x; /* 10 */
coords.x = 42;
```

#### Struct embedding

Structs may also "embed" another struct type directly into itself, by specifying
the type without naming a field. For example:

```hare
let coords = struct {
	x: int = 10,
	y: int = 20,
	struct {
		z: int = 30
	},
};
```

You may use `coords.z` as if you would use any other field. This example is
somewhat contrived &mdash; this becomes more useful when combined with
user-defined types, which [we will cover](#type-aliases--user-defined-types)
momentarily.

#### Union types

Union types, or "untagged" unions, use the same syntax as structs with the
"union" keyword instead of the struct keyword. In a struct, each field is laid
out in memory one after another, creating a type whose size is the sum of its
members (well, not exactly, but this approximation will be sufficient for now).

A union differs in that it creates a type whose size is the *maximum* size among
its field types, rather than the *sum* of its field types, and all of the fields
are stored at the same location in memory.

If you don't know why this is useful, you probably don't need it.

### Tagged unions

A tagged union is an aggregate type whose value is exactly one of its subtypes.
The syntax for a tagged union type is <code>(<em>type</em> | <em>type</em> |
...)</code>, e.g. `(i32 | f32)`. A type of `(i32 | f32)` can store either an
i32 or an f32. The representation of a tagged union in memory is a `size`
representing which of the fields is selected (called the "tag"), followed value
itself, which is allocated with sufficient space to store any of the valid
types.

The `is` operators is provided to examine the tag of a tagged union; it will be
return true if the value in a tagged union is of the desired type.

```hare
let x: (i32 | f32) = 13.37;
if (x is i32) {
	io::println("x is an i32");
} else if (x is f32) {
	io::println("x is an f32");
};
```

The `as` operator is a type assertion on a tagged union. It's used when you
*know* that a tagged union has a specific value, and converts it to that
representation. For example:

```hare
let x: (i32 | f32) = 1337i32;
io::println(strconv::i32tos(x as i32));
```

The program will abort if your assertion does not hold. For example, this
program will crash:

```hare
let x: (i32 | f32) = 1337i32;
io::println(strconv::i32tos(x as f32));
```

There is a third way to use tagged unions, and you will likely use it more often
than the others: [match expressions](#match).

#### Tagged void types

Tagged unions are one of the few places where `void` types may be used to
indicate the absence of a value. The following code is valid:

```hare
let x: (i32 | f32 | void) = void;
if (x is void) {
	io::println("There's nothing here!");
};
```

### Enums

TODO

### Type aliases & user-defined types

You can create a user-defined type, also known as a "type alias", with the `def`
operator. For example:

```hare
use io;
use strconv;

def my_int = int;

export fn main void =
{
	const x: my_int = 1337;
	io::println(strconv::itos(x));
};
```

There are also some extra semantics around user-defined struct types which can
save you some typing:

```hare
use io;
use strconv;

def coordinates = struct {
	x: int,
	y: int,
};

export fn main void =
{
	const coords = coordinates { y = 10, x = 20 };
	io::println(strconv::itos(coords.x));
	io::println(strconv::itos(coords.y));
};
```

By specifying the type name in the struct initializer, you need not write the
types of each struct field, and may specify them out of order.

## Memory management

### Pointer types

### alloc & free

### Pointer transfers

## Advanced function usage

### Test functions

### Function pointers

### Static bindings

## Advanced expressions

### switch

A switch expression can be thought of as a specialized "if" expression. Given a
value, it tests it against a number of possibilities and takes the appropriate
branch.

```hare
io::println(switch (x) {
	1 => "one",
	2 => "many",
	* => "lots",
});
```

Switch expressions are required to be *exhaustive*, meaning that every possible
case is present. The `*` case can be used to match with any case not otherwise
tested for.

If all cases have the same type, the type of the switch expression is that type.
Otherwise, the type is void. However, any branches which *terminate* are not
considered in this respect.

```hare
io::println(switch (x) {
	1 => "one",
	2 => "many",
	* => {
		log::errorln("too many!");
		return;
	},
});
```

### match

A match expression is similar to a switch expression, but instead of switching
on *values*, it matches on *types*. It is used with
[tagged unions](#tagged-unions), especially for
[error handling](#error-handling).

```hare
fn print(val: (int | str)) void =
{
	match (val) {
		int => io::println("(int)"),
		s: str => io::println(s),
	};
};
```

Like switch statements, match statements must be exhaustive. Note as well that
you may *bind* the value to a new variable of the matched type, as in the `str`
case shown here.

### defer

TODO

### assert

The `assert` keyword can be used to validate your assumptions at runtime.

```hare
fn sqrt(x: f32) f32 =
{
	assert(x > 0, "Cannot take square root of negative number");
	/* ... */
};
```

If the condition fails, the program is aborted and the message is printed. If
you omit this message, a more generic error message is shown.

You can also do *static* assertions to validate your assumptions at
compile-time.

```hare
static assert(size(*void) == 8, "This module only supports 64-bit systems");
```

Static assertions may be used outside of function bodies.

## Type compatibility

TODO: This section is written for people already familiar with casts in other
languages, it's a bit too thick for noobs

### Implicit casts

It is possible to use a value of one type in a context where another type is
called for, in a limited set of circumstances.

For **integer** types (`int`, `uint`, `i8-i64`, `u8-u64`, `size`, and
`uintptr`), a value can be implicitly cast only if it would not cause a loss of
precision. For example, an `i8` can be assigned to an `i32`, but not the
inverse. Unsigned and signed integer types are not mutually assignable.

```hare
let x: i32 = 10i8; /* Acceptable */
let y: i32 = 10u8; /* Not acceptable */
let z: i8 = 12345; /* Not acceptable */
```

A **tagged union** may be assigned from value of any of its member types.

```hare
let x: (int | *str | void) = 1234; /* Acceptable */
```

### Explicit casts

Explicit casts are to be used with caution, as they are one mechanism by which
the programmer can override the best judgement of the compiler. The syntax for
an explicit cast is similar to a variable declaration:
<code><em>expression</em>: <em>type</em></code>, e.g. `1234: i8`.

```hare
let x: int = 1234;
let y = x: f64;
```

When casting **numeric** types, a loss in precision will result in the value
being truncated towards the least significant bits. For example, casting
`0x1234` to `u8` will produce `0x34`.

**Struct** types cannot be cast to any other struct type. **Union** types may be
cast to any of their member types.

Any **pointer** type may be cast to any other pointer type, or to and from
`uintptr`. This may also be used to cast owned pointers to borrowed pointers and
vice versa; use this feature with great caution. The semantics of casting
between the referent types of each pointer type (the referent type being `int`
in the example of `*int`) is not taken into account; normally impossible
casts are permitted through pointer indirection.

```hare
let x: int = 1234;
let y = &x: *io::file; /* Acceptable (but ill-advised) */
```

**Tagged unions** may be cast to any of their member types. This will not cause
a runtime type assertion to be emitted.

### Type promotion

*Type promotion* is used in arithmetic expressions to "promote" numeric types to
avoid precision loss. If two numeric types which can be implicitly cast to one
another appear as the left and right values to a binary operator (such as `+`,
addition), the lower precision value is implicitly cast (or "promoted") to the
higher precision type.

```hare
let x: i8 = 42;
let y: i16 = 1337;
let z = x + y; /* z's type is i16 */
```

### C compatibility

Hare types are a strict superset of C types, and Hare is compatible with the C99
ABI. It is possible to represent any Hare type as a C type.

Primitive types, pointers, structs, and unions are directly compatible with
their C counterparts. Tagged unions may be represented in C as follows:

```hare
def my_union = (int | *io::file | void);
```

```c
enum my_union_tag {
	INT = 0,
	IO_FILE = 1,
	VOID = 2,
};

struct my_union {
	size_t tag;
	union {
		int i;
		struct io_file *f;
	};
};
```

Strings are represented as follows:

```c
struct hare_string {
	size_t length;
	char utf8_data[];
};
```

Slices are represented as follows:

```hare
def my_slice = []int;
```

```c
struct my_slice {
	size_t length, capacity;
	int *data;
};
```

## Error handling

## Modules

### Importing modules

You may have noticed that we have a few `use` statements at the top of each of
our example files. These are used to import modules from the standard library,
or from your program's third-party dependencies. The convention is to sort use
statements alphabetically at the top of your program.

[Documentation is available](/documentation) for the standard library.

### Using third-party dependencies

### Module path resolution

#### Vendoring dependencies

### Build tags

Some build tags are built-in &mdash; they vary depending on your platform and
are printed when you run `hare version`.

### Writing new modules

### Documentation
