---
# vim: ts=2 sw=2 et :
# TODO:
# - Add table of contents
# - Organize into a sections & subsections rather than flat structure
title: An introduction to the Hare programming language
type: codetutorials
summary: |
  This tutorial will introduce you to the Hare programming language. It
  should take about an hour to complete. If you are already familiar with
  programming in other languages, it might take even less time — feel free to
  jump around as you see fit.

  After you complete this tutorial, you should move on to the
  [standard library introduction](/tutorials/stdlib).
sections:
- title: Getting started
  sample: |
      use fmt;
      
      export fn main() void = {
      	fmt::println("Hello world!");
      };
  details: |
      Start by setting up your local development environment according to the
      [installation instructions](/installation). To test that it works, copy
      this code sample into a file named `main.ha`, and run the following
      command:

          $ hare run main.ha
          Hello world!

      You can also compile this program into an executable like so:

          $ hare build -o example main.ha
          $ ./example
          Hello world!

      <p class="alert">
      <strong>Note</strong>: Hare programs use 8-column-wide tab characters for
      indentation. See the <a href="/style">style guide</a> for details, and
      consider installing an <a href="/editors">editor plugin</a>.
      </p>
- title: Using variables
  sample: |
      use fmt;
      
      export fn main() void = {
      	// Define two variables, x and y
      	let x = 10, y = 20;
      	fmt::printfln("{} + {} = {}", x, y, x + y);

      	// Update their values
      	x = 30;
      	y = 40;
      	fmt::printfln("{} + {} = {}", x, y, x + y);

      	// Constant bindings cannot be assigned to after their declaration
      	const z = 50;
      	fmt::printfln("{} + {} + {} = {}", x, y, z, x + y + z);
      };
  details: |
      You can assign names to values by using variables. The `let` keyword
      creates variables which are *mutable*, and can be changed later. The
      `const` keyword creates *immutable* variables, which cannot be modified.

      We've also changed `fmt::println` to `fmt::printfln`, which allows us to
      use a *format string*, so each instance of `{}` is replaced with one of
      the parameters.

      <p class="alert"><strong>Note</strong>: "//" is used for writing comments:
      all text is ignored until the end of the line.</p>
- title: Constants & globals
  sample: |
      use fmt;

      let z: int = 10;
      def TWENTY: int = 20;
      
      export fn main() void = {
      	let x = 10, y = 20;
      	fmt::printfln("{} + {} + {} = {}", x, y, z, x + y + z);

      	z += TWENTY;
      	fmt::printfln("{} + {} + {} = {}", x, y, z, x + y + z);
      };
  details: |
      We can also define "globals" and "constants", though globals should be
      used sparingly. This allows us to name variables and values which are
      available to other functions &mdash; and later, we'll show how to make
      these available to other modules.

      Note also the use of the fully qualified syntax with the inclusion of the
      `int` type in the global and constant bindings. Every varaible in Hare has
      a **type**, which defines what kind of values it can store and the
      semantics for its use in the program.

      Hare is a strong, statically typed language, meaning that type semantics
      are strongly enforced, and checked when compiling your program. Let's
      explore these types in more detail.

      <p class="alert"><strong>Note</strong>: 
      By convention, constants are named in UPPERCASE. Variables and globals use
      underscore_case.
      </p>
- title: Numeric types & arithmetic
  sample: |
      export fn main() void = {
        let a: size = 1337z;
        let b: u8 = 42u8;
        let c: i8 = -42;
        let d = 13.37;

        a + b; // OK!
        b + c; // NOT OK!
        c + d; // NOT OK!

        // Some arithmetic operations
        2 + 2;
        2 / 4;
        3 * 3;
        8 % 4;	// Modulo

        a += 10;	// Adds and assigns the result to the original object
        b -= 1;

        32 << 4;	// Left and right shift
        32 >> 4;

        bin & 0b1111;		// Bitwise AND with a binary constant
        mode | 0o644;		// Bitwise OR with an octal constant
        addr & ~0xFFFF;	// Bitwise AND and NOT with a hexadecimal constant
      };
  details: |
      Let's begin with numeric types. There are several kinds of numeric types.
      Integers store integral real numbers, and may be *signed* or *unsigned*
      &mdash; respectively meaning that they either can or cannot represent
      negative values. These have a specific size in bits: 8, 16, 32, or 64,
      which defines the range of values they can represent. These are written
      with the letter **i** or **u** (for signed or unsigned) plus their
      precision, such as **u8** for an unsigned 8-bit integer.

      There are also some special types with implementation-defined precision:
      **int**, **uint**, and **size**. The first two are the signed and unsigned
      words for the target architecture, and the latter is the precision
      necessary to represent the maximum object size. The details of why these
      are important are not necessary to understand right now.

      Hare also supports the floating-point **f32** and **f64** types, which are
      respectively 32-bit and 64-bit IEEE-compatible binary floating point
      types.

      All of the usual arithmetic operators are supported, with a similar
      precedence to C, with the exception of changes to the precedence of binary
      and logical arithmetic operators (if you don't know what that means, don't
      worry about it).

      Hare has strict guidelines regarding *type promotion*: see the "OK" and
      "NOT OK" comments in the sample code. You can sometimes "promote" one type
      to another in an expression, causing the value of lesser precision to
      promote to one of higher precision. The "OK" example causes "b" to be
      promoted from u8 to size, and the result of the operation is size. As a
      rule of thumb, you can only promote mutual integer types of the same
      signedness, and floating types can only promote to other floating types.
      The compiler will let you know if you make any mistakes in this regard.

      Constants like "1234" are a little bit more flexible. A number typed out
      literally in the source code will infer its type from its surroundings, if
      possible: `let x: f32 = 1234` will infer that 1234 is meant to be an
      **f32**. Presuming that the value fits in the desired type without any
      loss of precision, it will automatically assume the desired type. You can
      also specify the desired type explicitly, as shown in the samples, by
      using the appropriate suffix.
- title: Strings and runes
  sample: |
      use fmt;

      export fn main() void = {
        let username = "harriet";
        let japanese = "ハリエット";
        let rn = '㌫';

        fmt::printfln("Hello, {}! Today's rune is {}.", username, rn);
        fmt::printfln("{}'s username is {} bytes.", len(username));
        fmt::printfln("{}'s username in Japanese is {}.", japanese);
        fmt::printfln("The length of \'{}\' is {}.", japanese, len(japanese));
      };
  details: |
      Hare has a **string** and **rune** type, which respectively stores a
      string of UTF-8 text, and a single Unicode character. Hare's strings have
      a defined length in bytes as well, which you may access with the `len()`
      builtin.

      Strings tend to be surprisingly difficult to use correctly in many
      programming languages. Efficiently manipulating strings, handling a
      variety of languages, and accomplishing seemingly mundane tasks; all of
      these are alarmingly error-prone.

      It's remarkable that, given this, Hare would have very little first-class
      support for strings as a *language construct*. Unlike many other
      languages, Hare strings cannot be indexed or concatenated with language
      features like `s[n]` or `+`. You can do these tasks in Hare, but we will
      have to save the details for the
      [standard library introduction](/tutorials/stdlib).
---
