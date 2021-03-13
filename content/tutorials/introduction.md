---
# vim: ts=2 sw=2 et :
# TODO:
# - Add table of contents
# - Organize into a sections & subsections rather than flat structure
title: An introduction to the Hare programming language
type: codetutorials
summary: |
  This tutorial will introduce you to the Hare programming language. It
  should take an hour or two to complete. If you are already familiar with
  programming in other languages, it might take even less time — feel free to
  jump around as you see fit. Once you're comfortable with the essential
  language semantics, don't be afraid to supplement your knowledge with the
  [Hare specification](/specification) as needs arise.

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
      
      Hare programs use 8-column-wide tab characters for indentation. See the <a
      href="/style">style guide</a> for details, and consider installing an <a
      href="/editors">editor plugin</a>.
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
      Our tour of Hare will begin by explaining how Hare's type system works and
      how values are represented. Let's first introduce variables to help with
      this explanation.

      Variables allow you to assign names to values. The `let` keyword creates
      variables which are *mutable*, and can be changed later. The `const`
      keyword creates *immutable* variables, which cannot be modified. Every
      variable has a specific **type** that defines what kind of values it can
      store and the semantics for its use in the program.


      <p class="alert">
      <strong>Note</strong>:
      "//" is used for writing comments: all text is ignored until the end of
      the line. We also changed <code>fmt::println</code> to
      <code>fmt::printfln</code>, a function which allows us to use a <em>format
      string</em>. Each instance of <code>{}</code> is replaced with one of the
      parameters.
      </p>
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

      Note also the use of the fully qualified syntax that adds the `int` type
      in the global and constant bindings. This is optional for variables
      (optional, but often useful), but required for constants and globals.

      Hare is a strong, statically typed language, meaning that type semantics
      are strongly enforced, and checked when compiling your program. Let's
      explore these types in more detail.

      <p class="alert"><strong>Note</strong>: 
      By convention, constants are named in UPPERCASE. Variables and globals use
      underscore_case.
      </p>
- section: Hare's type system
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
      
      	bin & 0b1111;	// Bitwise AND with a binary constant
      	mode | 0o644;	// Bitwise OR with an octal constant
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
      	fmt::printfln("{}'s username is {} bytes.", username, len(username));
      	fmt::printfln("{}'s username in Japanese is {}.", username, japanese);
      	fmt::printfln("The length of \"{}\" is {}.", japanese, len(japanese));
      };
  details: |
      Hare has a **string** and **rune** type, which respectively stores a
      string of UTF-8 text, and a single Unicode character. Hare's strings have
      a defined length in bytes as well, which you may access with the `len()`
      builtin. Note also the use of `\"` &mdash; a backslash "escapes" the "
      character, allowing it to appear in the string (other escape sequences you
      are familiar with work in Hare, too).

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
- title: Struct types
  sample: |
        use fmt;
        
        export fn main() void = {
        	// Anonymous struct type, must be specific
        	let target = struct {
        		x: int = 10,
        		y: int = 10,
        	};
        
        	fmt::printfln("x: {}, y: {}", target.x, target.y);
        
        	// User-defined struct, can omit the types or re-order fields
        	let target = coordinates {
        		y = 20,
        		x = 20,
        	};
        
        	fmt::printfln("x: {}, y: {}", target.x, target.y);

        	// User-defined structs can also be "auto-filled":
        	let target = coordinates { ... };
        	fmt::printfln("x: {}, y: {}", target.x, target.y);
        };
        
        // User-defined type - more on these later
        type coordinates = struct {
        	x: int,
        	y: int,
        };
        
        type coordinates3D = struct {
        	// You can embed one struct into another, and its fields are added
        	// to the new struct:
        	coordinates,
        	z: int,
        };
  details: |
      A **struct** type can be used to store a number of named subvalues, or
      "fields". The `struct` keyword is used to describe a struct type, followed
      by a comma-delineated list of its fields and the values to initialize them
      to. Note that in the first example, we're allowing the variable type to be
      inferred from the struct value, but we could be explicit if we prefer:

      ```
      let target: struct { x: int, y: int } = struct {
      	x: int = 10,
      	y: int = 10,
      };
      ```

      Usually this is unnecessarily verbose. The second example shows us that we
      can be even terser by using *user-defined types* (we'll cover them in
      detail later). When initializing a named struct type, swap the `struct`
      keyword for the type name, and you can skip specifying the types of the
      struct fields. The third case also demonstrates that some fields can be
      omitted by using `...`, which sets them to their default values - usually
      zero. Not all types have a default value; the compiler will complain if it
      cannot provide one.

      At the end we see another feature: embedding structs in other structs.
      This is a useful means of extending one struct type with new fields, or
      using anonymous structs to organize sub-fields. It needn't be the first
      field, by the way - you can add these anywhere in the struct.

      Hare also supports **union** types, but these are not generally used by
      Hare programs. If you know that you need them, they are compatible with C
      and are written with the "union" keyword in place of the "struct" keyword.

      <p class="alert">
      <strong>Note</strong>: We re-used the "target" variable name in the sample
      code. This is called <em>shadowing</em> the variable. All subsequent
      references refer to the new variable, and the old variable is no longer
      accessible.
      </p>
- title: Tuples
  # TODO: We're going to expand the syntax for tuples later
  sample: |
      use fmt;

      export fn main() void = {
      	// Tuples have a fixed number of items with specific types:
      	let t: (str, int, int) = ("hiya!", 4, 2);

      	// You can access each item with the . operator:
      	t.0; // str
      	t.1; // int
      	t.2; // int
      	fmt::printfln("({}, {}, {})", t.0, t.1, t.2);
      };
  details: |
      Think of tuples as structs with unnamed fields. These types are declared
      with a parenthesized, comma-delimited ordered list of the subtypes they
      store: `(str, int, int)` stores a string and two integers. The desired
      value can be obtained with the `.` operator and the index: `x.0` gets the
      first value, `x.1` the second, and so on. This cannot be a variable - only
      constant, positive integers can be used here.

      Tuple *values* are written with parenthesis as well:

      ```
      let x: (str, int, int) = ("hello world!", 42, 24);
      ```
- title: Arrays and slices
  sample: |
      use fmt;

      export fn main() void = {
      	// Arrays and slices can be declared with:
      	// (1) An inferred type (array of inferred length)
      	// (2) A definite length
      	// (3) Explicit array of inferred length
      	// (4) Or as a slice type, with a length determined at runtime
      	let a = [1, 2, 3, 4, 5];
      	let b: [4]int = [1, 3, 3, 7];
      	let c: [_]int = [7, 3, 3, 1];
      	let d: []int = [1, 3, 3, 7];
      
      	// You can create large arrays with auto-fill, which assigns the last
      	// value to every subsequent index.
      	let e: [4096]int = [1, 2, 3, 4...];
      
      	// Both arrays and slices are zero-indexed
      	fmt::printfln("a[0]: {}, d[0]: {}", a[0], d[0]);
      
      	// An array always has a fixed length at compile time
      	// A slice has an arbitrary length defined at runtime
      	fmt::printfln("len(a): {}, len(d): {}", len(a), len(d));
      
      	// You can use the slicing operator to get a slice for any array or
      	// slice type by specifying the start (inclusive) and end (exclusive):
      	let first3 = a[..3];	// 1, 2, 3
      	let middle3 = a[1..4];	// 2, 3, 4
      	let last3 = a[2..];	// 3, 4, 5
      	let all = a[..];	// 1, 2, 3, 4, 5
      };
  details: |
      We'll introduce **array** and **slice** types next. Arrays store a
      specific (determined at compile-time) number of ordered values of a
      uniform subtype; slices store an arbitrary (determined at runtime) number
      of ordered values of a uniform type.

      An array can be declared with a specific length and subtype (e.g.
      `[5]int`), or in some cases by inferring the length from context using an
      underscore (e.g.  `[_]int`). A slice type leaves the length unwritten:
      `[]int`. Values of a slice or array type can be *indexed*, obtaining the
      value of one of their numbered objects, with the
      <code>[<em>index</em>]</code> operator as shown in the samples. The first
      object is assigned the 0th index. Their length can be obtained with the
      `len` built-in.

      You can also use a *slicing expression* with `..` to create a slice from
      another array or slice object. This creates a new slice which represents a
      subset of the source object, such that `x[2..5]` will produce a new slice
      whose 0th value is the 2nd value of `x`, and with a length of `5 - 3 = 2`.

      One last note: accesses to arrays and slices are *bounds checked*, which
      means that accessing a value beyond the end of their valid objects will
      cause your program to abort.

      ```
      let x = [1, 2, 3];
      x[4]; // Abort!
      ```

      It is infrequently useful (but dangerous!) to skip the bounds check, for
      interop with code written in other languages, or in carefully reviewed,
      performance-critical code. To disable bounds checking, use `*` in place of
      the array length:

      ```
      let x: [*]int = [1, 2, 3];
      x[4]; // Undefined behavior!
      ```
- title: Enums
  sample: |
      type colors = enum {
      	RED,
      	ORANGE,
      	YELLOW,
      	GREEN,
      	BLUE,
      	INDIGO = BLUE, // Indigo is a fake color
      	VIOLET,
      };
      
      type bits = enum u8 {
      	FEE = 1 << 0,
      	FIE = 1 << 1,
      	FOE = 1 << 2,
      	FUM = 1 << 3,
      };
      
      export fn main() void = {
      	let color = colors::RED;
      	let flags = bits::FEE | bits::FIE;
      };
  details: |
      One last type class to introduce: **enums**, short for *enumerated
      values*. An enum type is a kind of constrained integer, providing names
      for a specific set of possible values. If omitted, the integer type is
      presumed to be **int**, but you can also specify it explicitly &mdash;
      check out the "bits" type in the sample. Enclosed within the braces, you
      can name each possible value, and optionally assign it a *specific* value.
      If omitted, the values start from zero and increment for each subsequent
      value.

      Though Hare makes *some* attempts to ensure that an enum value only
      contains values valid for its type, these guarantees are somewhat loose.
      This flexibility allows for things like bitfields, but can also be abused
      (deliberately or by mistake) to create unexpected situations. One common
      situation is when reading an enum value from an external source when
      the list of valid values known to each party has become out of sync
      &mdash; Hare won't detect this for you.
- title: Other simple types
  sample: |
      export fn main() void = {
      	let x = 10, y = 20;
      
      	let a: bool = true;	// "true" and "false" are built-in constants
      	let b: bool = false;
      
      	let c = x < y;		// true
      	let d = x <= y;		// true
      	let e = x > y;		// false
      	let f = x >= y;		// false
      	let g = x == y;		// true
      	let h = x != y;		// false
      
      	c && d;	// logical "and": true if both are true
      	c && e;	// false
      
      	c || d;	// logical "or": true if either is true
      	c || e;	// true
      	e || f;	// false
      
      	c ^^ e;	// logical "xor": true if they differ
      	c ^^ d;	// false
      };
  details: |
      Hare also provides a **bool** type, also called a boolean. This type can
      have either the value **true** or **false**. A number of *logical*
      arithmetic operators can have this result; ">" tests if one value is
      greater than another, ">=" greater than or equal to, "==" and "!="
      respectively test for equality and inequality.

      The &&, ||, and ^^ operators compare booleans to each other. Respectively
      these are logical "and", "or", and "xor". Note that the first two of these
      can *short-circuit*, which means the second operand will *not* be
      evaluated if the result can be determined from the first alone. In the
      case of &&, we can assume the result is false if the first operand is
      false, without consulting the second. || can presume the result is true if
      the first result is true. This may seem trite &mdash; but this becomes
      important if some operands can have *side effects*, which do not occur if
      the operator short circuits.

      Another simple type is the **void** type, which you've seen in examples so
      far as part of the "main" function. This type simply refers to the absence
      of a value. It has no size and cannot store anything.

      There *is* one more type class we need to talk about: tagged unions. These
      are one of Hare's flagship features and require a much deeper
      introduction, so we'll be saving those for their own section.
- title: Type aliases
  sample: |
      type my_int = int;
      
      type coords = struct {
      	x: int,
      	y: int,
      };
      
      type permissions = enum u8 {
      	READ = 1 << 0,
      	WRITE = 1 << 1,
      	EXEC = 1 << 2,
      };
      
      type player = struct {
      	location: coords,
      	user: struct {
      		username: str,
      		permissions: uint,
      	},
      };
  details: |
      In addition to each of the built-in type classes we've introduced so far,
      Hare offers user-defined types, or *type aliases*. Unlike, for example, C,
      every type in Hare can be represented without explicitly naming it (enums
      are the exception). It's often useful to name them, however, and you may
      do so by declaring a type alias. The type alias inherits all of the
      semantics of the underlying type.

      Note that declaring a type alias doesn't just create a special word which
      refers to an existing type &mdash; it creates a new, distinct type
      altogether, which has the same semantics as its secondary type. We'll
      explain why this is important later.
- section: Control flow
- title: If statements
  sample: |
      use fmt;
      
      export fn main() void = {
      	let x = 1337;
      
      	if (x > 10) {
      		fmt::println("x > 10");
      	};
      
      	if (x < 10) {
      		fmt::println("x < 10");
      	} else if (x > 20) {
      		fmt::println("x > 20");
      	} else {
      		fmt::println("10 < x < 20");
      	};
      
      	let y = if (x < 10) x else 20;
      
      	let z = if (x > 10) {
      		x += 10;
      		x;
      	} else {
      		fmt::println("Expected x to be greater than 10");
      		abort();
      	};
      };
  details: |
      So far, we've written only linear programs, which execute each statement
      from start to end in order. *Control flow* can be used to direct that flow
      in a non-linear fashion. The first tool of this class we'll examine is the
      **if** expression. In each of these examples, the *condition*, between the
      parenthesis, has a [bool type](#other-simple-types). If the type is true,
      the expression following it &mdash; called the *branch* &mdash; is
      executed.

      This is also our first example of Hare as an *expression-based* language.
      In many languages, the "if" statement has no result value, but in Hare,
      this is not the case. The "y" and "z" variables here are initialized to
      the result of whichever branch is taken. Hare doesn't have a "ternary"
      operator like many other languages do &mdash; try this approach instead.

      Take special note of the "z" variable. In the "y" case, both branches of
      the if statement have the same result type (int). In the "z" case, the
      first branch gives "x" as the result (after adding ten to it), but the
      second branch appears to have no result type (void). In the event that the
      result types of an if statement are non-uniform, a tagged union is
      typically produced, and we'll go into more detail on these later. But,
      this case is different: the second branch *terminates*. Any terminating
      expression, a kind of expression of which `abort()` is a member (also
      `break`, `return`, and others), will not be considered for the result type.

      One other note: the first if statement, with no "else" part &mdash; it
      always has a **void** result type.
- title: For loops
  sample: |
      use fmt;
      
      export fn main() void = {
      	const input = [1, 3, 3, 7];
      	
      	for (let i = 0z; i < len(input); i += 1) {
      		fmt::printfln("input[{}]: {}", i, input[i]);
      	};
      
      	let i = 0z;
      	for (i < len(input); i += 1) {
      		if (input[i] == 3) {
      			break;
      		};
      	};
      	fmt::printfln("index of '3': {}", i);
      
      	for (true) {
      		fmt::println("Hello world!");
      	};
      
      	:outer for (let i = 0z; i < 5; i += 1) {
      		:inner for (let j = 0z; j < 5; j += 1) {
      			fmt::printfln("i: {}, j: {}", i, j);
      			if (j == 3) {
      				continue :outer;
      			};
      		};
      	};
      };
  details: |
      Where if statements provide *branching* logic, for statements provide
      *looping* logic, allowing your control flow to repeat back on itself. Some
      languages have other loop types &mdash; while, do...while, etc &mdash; but
      Hare rolls it all up into a single loop: the for loop.

      The essential parts of a for loop are the *binding*, the *condition*, the
      *afterthought*, and the *body*. The binding allows you to declare a
      variable whose scope is limited to the body of the for loop. The condition
      tells the loop whether or not to repeat on each iteration. The
      afterthought is run after the body completes, and the body is the actual
      code which is repeatedly run.

      You can omit some of these to create different sorts of loops. The
      permissible combinations are:

      <pre>
      for (<em>binding</em>; <em>condition</em>; <em>afterthought</em>) <em>body</em>;
      for (<em>condition</em>; <em>afterthought</em>) <em>body</em>;
      for (<em>condition</em>) <em>body</em>;
      </pre>

      You can terminate a loop early by using a `break` expression anywhere in
      the loop body, and can skip to the next iteration early by using
      `continue`. You can also label the loops &mdash; see the final example
      &mdash; and "break" or "continue" to any loop when working with nested
      loops.
---
