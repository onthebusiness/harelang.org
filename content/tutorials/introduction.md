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
<code>/* ... */</code> is used for <em>comments</em> &mdash; these are ignored
by the compiler, but useful to anyone reading your code.</p>

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

The constants `true` and `false` are also provided by the language; they are
always true or always false.

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

TODO

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

### Primitive types

### Arrays

### Slices

### Structs & unions

### Tagged unions

### Enums

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

## Type compatibility

### Assignment

### Explicit casts

### Implicit casts

### Type promotion

## Advanced expressions

### switch

### match

### defer

## Memory management

### Pointer types

### alloc & free

### Pointer transfers

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

### Writing new modules

### Documentation
