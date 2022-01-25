---
# vim: ts=2 sw=2 et :
title: An introduction to the Hare programming language
type: codetutorials
summary: |
  This tutorial will introduce you to the Hare programming language. It
  should take an hour or two to complete. If you are already familiar with
  programming in other languages, it might take even less time — feel free to
  jump around as you see fit.

  After you complete this tutorial, you should move on to the
  [standard library introduction](/tutorials/stdlib).
sections:
- title: Getting started
  sample: |
      use fmt;
      
      export fn main() void = {
      	fmt::println("Hello world!")!;
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
- title: Breaking down "Hello world!"
  sample: |
      use fmt;
      
      export fn main() void = {
      	fmt::println("Hello world!")!;
      };
  details: |
      Let's briefly break down what's happening in this program, then go into
      more detail throughout the remainder of the tutorial.

      The first line is an *import* which pulls in the [fmt] module from the
      standard library &mdash; click that link to view the documentation for
      this module. The purpose of this module is formatting text, and one of the
      functions it provides is [println], which formats some text and prints it
      to the program's output along with a new line.

      [fmt]: https://docs.harelang.org/fmt
      [println]: https://docs.harelang.org/fmt#println
      
      The program's *entry point* is "main", which executes after the program's
      environment is initialized. The shape of this function is defined by the
      Hare standard: it accepts no parameters and returns no values (this is
      what `void` means), and it must be "exported" so that the Hare runtime can
      use it. The only thing that this function does is call the "println"
      function from the fmt module, printing our "Hello world!" message to the
      program's standard output.
- title: Error handling in "Hello world!"
  sample: |
      use fmt;
      
      export fn main() void = {
      	fmt::println("Hello world!")!;
      };
  details: |
      Note the `!` operator which follows the function call: this is the error
      assertion operator. It is possible for writing to the standard output to
      fail, and the programmer is required to address this scenario. Try
      removing it and compiling this program again &mdash; the compiler will
      complain. The simplest approach to error handling, though usually
      incorrect, is to use this operator to promise the compiler that the error
      will never occur. The compiler will hold you to this: if an error does
      occur, the program will crash. You can see this in action by simulating an
      error; on Linux you can do this like so:

      ```
      $ hare run main.ha >/dev/full
      ```

      Crashing is generally thought to be more desirable than allowing the
      program to continue under conditions which were thought to be impossible,
      as this often leads to security vulnerabilities or other undesirable
      behavior. We will go over better ways to deal with errors later on in the
      tutorial, but for now it's okay to use `!` to make the compiler happy.

      Next, we'll dive a bit deeper with a slightly more complex example.
- section: Hare basics
- title: Functions & parameters
  sample: |
      use bufio;
      use fmt;
      use os;
      use strings;
      
      export fn main() void = {
      	const user = askname();
      	greet(user);
      };
      
      // Asks the user to provide their name.
      fn askname() str = {
      	fmt::println("Hello! Please enter your name:")!;
      	const name = bufio::scanline(os::stdin)! as []u8;
      	return strings::fromutf8(name);
      };
      
      // Greets a user by name.
      fn greet(user: str) void = fmt::printfln("Hello, {}!", user)!;
  details: |
      The sample program shown here is a bit more complex, and we're going to
      examine it in detail over the next few headings. This program prints a
      greeting, then prompts the user to enter their name, then greets the user
      by name. There's a lot to unpack here, but we'll start by looking at the
      use of functions.

      A function is the basic unit of executable code in a Hare program, which
      defines a step or series of steps for the program to perform. Functions
      can "call" each other to deputize various tasks, such as relying on "fmt"
      to format text for printing.

      Functions may accept a list of parameters by placing them between the `()`
      tokens when declaring the function. "greet" accepts a "str" (string)
      parameter, which it names "user". Functions can also *return* values to
      the function which calls them, such as "askname", which returns a "str".
      Our main function makes use of the return value by storing it in a
      *variable* and *passing* it to "greet".
- title: A closer look at bufio::scanline
  sample: |
      use bufio;
      use fmt;
      use os;
      use strings;
      
      export fn main() void = {
      	const user = askname();
      	greet(user);
      };
      
      // Asks the user to provide their name.
      fn askname() str = {
      	fmt::println("Hello! Please enter your name:")!;
      	const name = bufio::scanline(os::stdin)! as []u8;
      	return strings::fromutf8(name);
      };
      
      // Greets a user by name.
      fn greet(user: str) void = fmt::printfln("Hello, {}!", user)!;
  details: |
      Like "askname", many standard library functions can return values.
      bufio::scanline is interesting example of this. We can read the
      documentation for this function in our terminal by running the "haredoc
      bufio::scanline" command &mdash; try this now.

      This function's signature (the combination of its parameters and return
      type) is a bit more complex in that it can return one of several kinds, or
      *types*, of values, forming what's referred to as a *tagged union*. The one
      we're interested in is []u8, which is a "slice" of bytes containing the
      line we want to read. It can also return io::EOF, which indicates that
      the "end of file" was reached, or io::error, which indicates that an I/O
      error occured. The `!` operator deals with the error case as we described
      before. Encountering the end of the file, however, is not considered an
      error.

      To address this, we use the `as` operator to interpret the value as if it
      were a []u8 &mdash; this is called a type assertion. Like other
      assertions, the compiler will check our work and, if proven wrong, will
      cause the program to crash. You can simulate this case by pressing Ctrl+D
      in your terminal emulator instead of entering your name.

      A key take-away from this should be the "haredoc" tool, which you should
      use liberally as you work with Hare. Try to use it to learn about the
      other standard library functions we're using in this sample, such as
      strings::fromutf8. You can also use it to browse the modules themselves
      &mdash; try "haredoc fmt" or "haredoc strings". Lastly, you can use it to
      browse your *own* documentation: try "haredoc askname" and "haredoc
      greet".

      TODO: https://todo.sr.ht/~sircmpwn/hare/555
- title: "Variables: const & let"
  sample: |
      use fmt;
      use io;
      use os;
      use strings;
      
      export fn main() void = {
      	// Example A
      	const source = os::open("main.ha")!;
      	const source = io::drain(source)!;
      	const source = strings::fromutf8(source);
      	const source = strings::split(source, "\n");
      	first3(source);
      
      	// Example B
      	let i: int = 1337, j: int = 42;
      	fmt::printfln("{} + {} = {}", i, j, i + j)!;
      	j = i;
      	fmt::printfln("{} + {} = {}", i, j, i + j)!;
      };
      
      fn first3(lines: []str) void = {
      	fmt::println("The first three lines of main.ha are:")!;
      	fmt::println(lines[0])!;
      	fmt::println(lines[1])!;
      	fmt::println(lines[2])!;
      };
  details: |
      This sample is designed to illustrate a few ideas regarding the use of
      variables in Hare. Like the parameters we've used in earlier examples, we
      can create *local variables* which store values that we can use and
      (sometimes) modify. Variables are *bound* with the **const** and **let**
      keywords. A variable declared with const cannot be assigned to, and a
      variable declared with let may be assigned to. All variables must be
      *initialized* when they are declared.

      Though you cannot modify a const variable, you can *re-bind* it by
      creating another variable with the same name. A point of note is that when
      you re-bind a variable like this you can change it to a different type: in
      this example "source" refers to an io::file, a []u8, a str, and a []str,
      in that order. This is a useful pattern for building up a variable from a
      series of intermediate values of various types.

      <div class="alert">
        <strong>Tip:</strong> Re-binding variables is a special case of the more
        general concept of <em>shadowing</em>, which you may be familiar with
        from other languages.
      </div>

      Take note of the syntax as well. Each of the bindings in Example A use
      *type inference*, in which the variable automatically assumes the type of
      the right-hand side of the statement. The 'let' examples demonstrate the
      use of an explicit type (**int**). In some cases it may be necessary or
      helpful to state the type explicitly, this can be done at your discretion
      or when the compiler asks you to.
- title: More about types
  sample: |
      export fn main() void = {
      	// Numeric types can be declared explicitly:
      	let
      		a: int = 10,	// Signed integer
      		b: uint = 10,	// Unsigned integer
      		c: u8 = 10,	// Unsigned 8-bit integer
      		d: f32 = 13.37;	// 32-bit floating point number
      
      	// Or inferred from a suffix:
      	let
      		a = 10i,	// Signed integer
      		b = 10u,	// Unsigned integer
      		c = 10u8,	// Unsigned 8-bit integer
      		d = 13.37f32;	// 32-bit floating point number
      
      	// Some other types:
      	let
      		a: str = "hi",			// String
      		b: (int, int) = (42, 24),	// Tuple
      		c: struct {
      			x: int,
      			y: int,
      		} = struct {
      			x: int = 10,
      			y: int = 20,
      		},				// Struct
      		d: [4]int = [1, 2, 3, 4],	// Array
      		e: []int = [1, 2, 3, 4];	// Slice
      };
  details: |
      Here is a small (non-comprehensive) sample of some other types supported
      by Hare. Hare supports signed and unsigned integers of various sizes
      (signed numbers are able to store negative values), as well as 32- and
      64-bit floating point types (which can store a fractional component). The
      exact type can be inferred from context, as in the first set of examples,
      or specified with an appropriate suffix.

      <p class="alert"><strong>Note</strong>: 
      If you are not already familiar with binary floating point arithmetic, you
      may be surprised when arithmetic using f32 and f64 types gives unexpected
      results. Programmers unfamiliar with the subject are encouraged to read
      the <a
        href="https://en.wikipedia.org/wiki/Floating-point_arithmetic"
      >Wikipedia page on Floating-point arithmetic</a>
      to better understand the theory and tradeoffs.
      </p>

      Hare also supports a number of composite types, a few examples of which
      are shown here. We'll go into more detail on how these work later. There
      are also a number of more specialized types that are not shown here, such
      as **size** and **uintptr**. Some of these are introduced later in this
      tutorial, while others are not useful outside of specialized situations
      and are left for you to discover when you need them.
- title: Struct and tuple types
  sample: |
      use fmt;
      
      type coords = struct { x: int, y: int };
      
      export fn main() void = {
      	let player1 = struct {
      		x: int = 10,
      		y: int = 20,
      	};
      	let player2 = coords {
      		y = 10,
      		x = 20,
      	};
      	let player3: (int, int) = (42, 24);
      	fmt::printfln("Player 1: ({}, {})", player1.x, player1.y)!;
      	fmt::printfln("Player 2: ({}, {})", player2.x, player2.y)!;
      	fmt::printfln("Player 3: ({}, {})", player3.0, player3.1)!;
      };
  details: |
      Structs are one of the composite types supported by Hare. These define a
      *structured* value which is made up of other values in a certain order. In
      this example, we show two ways of using structs: first manually, and then
      by using a user-defined type, for player1 and player2 respectively. We can
      access the *fields* of the struct by using the `.` operator.

      Note the "type coords" declaration above the main function: this defines a
      type named "coords". This can be used for any type, not just structs. This
      type is equivalent to the hand-written struct used for the player1
      variable, but by giving it a name we can skip the types for each *field*
      and re-order them if we so desire. The other player, player1, is using an
      *anonymous*, or un-named, struct type.

      player3 is defined with a *tuple* type, which is very similar to a struct,
      but does not name its fields. They are accessed by their ordinal position,
      starting from zero, instead of their names.
- section: Memory management
- title: Stack allocation & pass by reference
  sample: |
      use crypto::sha256;
      use encoding::hex;
      use hash;
      use io;
      use os;
      use fmt;
      
      export fn main() void = {
      	// Pointer basics
      	let i = 10;
      	fmt::println(i)!;
      	increment(&i);
      	fmt::println(i)!;
      
      	// Applied usage
      	const hash = sha256::sha256();
      	const file = os::open("main.ha")!;
      	io::copy(&hash, file)!;
      
      	let sum: [sha256::SIZE]u8 = [0...];
      	hash::sum(&hash, sum);
      	hex::encode(os::stdout, sum)!;
      	fmt::println()!;
      };
      
      fn increment(ptr: *int) void = {
      	*ptr = *ptr + 1;
      };
  details: |
      This sample demonstrates the use of *pointers* and *stack allocation*,
      the latter of which is an important design pattern in Hare. First it
      passes a copy of the "i" variable to the "increment" function by
      *reference*, allowing the increment function to modify i from outside of
      "main" by using the `*` operator.

      When you call a function in Hare (or when the runtime calls "main" for
      you), a *stack frame* is allocated to store all of the function's
      variables and parameters. This is automatically cleaned up when you return
      from the function, which makes it useful for getting rid of resources when
      you're done using them. However, it's important not to allow a reference
      to any stack-allocated variables to persist after the function ends.
      Additionally, there is a limited amount of stack space, so it's often wise
      to seek alternative strategies when allocating large objects &mdash; we'll
      address that momentarily.

      As a practical demonstration of stack allocation, this sample also
      computes and prints the SHA-256 hash of its source code. A common pattern
      in Hare is for a constructor like sha256::sha256 to return an object on
      the stack, which you can then pass into other functions by reference using
      `&`. This is convenient for many objects which can be cleaned up by simply
      discarding their state on the stack, but other kinds of objects (such as
      file handles) require additional steps.
- title: Heap allocation & defer
  sample: |
      TODO
  details: |
      TODO
- title: Static allocation
  sample: |
      TODO
  details: |
      TODO
- title: Cleaning up other kinds of resources
  sample: |
      TODO
  details: |
      TODO
- title: Thinking in terms of ownership
  sample: |
      TODO
  details: |
      TODO
- section: Handling errors
- title: Working with match
  sample: |
      TODO
  details: |
      TODO
- title: Propagating errors
  sample: |
      TODO
  details: |
      TODO
- title: Defining new error types
  sample: |
      TODO
  details: |
      TODO
- title: Testing your code
  sample: |
      TODO
  details: |
      TODO
- section: Control flow
- title: "if & switch statements"
  sample: |
      TODO
  details: |
      TODO
- title: Using yield
  sample: |
      TODO
  details: |
      TODO
- title: Terminating branches
  sample: |
      TODO
  details: |
      TODO
- title: for loops
  sample: |
      TODO
  details: |
      TODO
- section: Working with slices
- title: Array types
  sample: |
      TODO
  details: |
      TODO
- title: Slicing arrays
  sample: |
      TODO
  details: |
      TODO
- title: Dynamically allocated slices
  sample: |
      TODO
  details: |
      TODO
- section: Types in depth
- title: Promotion and type inference
  sample: |
      TODO
  details: |
      TODO
- title: Casting & type assertions
  sample: |
      TODO
  details: |
      TODO
- title: More on tagged unions
  sample: |
      TODO
  details: |
      TODO
- title: More pointer types
  sample: |
      TODO
  details: |
      TODO
- title: Structs, unions, and C compatibility
  sample: |
      TODO
  details: |
      TODO
- section: Modules
---

And that's the Hare programming language! Nice work getting through all of
that. Now would probably be a good time to introduce you to some [community
resources](/community):

- The [hare-users](https://lists.sr.ht/~sircmpwn/hare-users) mailing list is a
  great place to ask questions
- The [#hare](https://web.libera.chat/#hare) chat room on irc.libera.chat is a
  good place for IRC users to idle
- The [stdlib documentation](#TODO) can fill you in on a lot of details about
  the APIs we saw here and others you encounter on your journey

The next tutorial is the [standard library introduction](tutorials/stdlib).
