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
      
      // Asks a user to provide their name.
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
      
      // Asks a user to provide their name.
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

      In Hare, str is a distinct type from []u8. A function is provided,
      `strings::fromutf8`, which converts a UTF-8 byte slice into a str that we
      can return from "askname".

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
      TODO
  details: |
      TODO
- title: More about types
  sample: |
      TODO
  details: |
      TODO
- title: Struct and tuple types
  sample: |
      TODO
  details: |
      TODO
- section: Memory management
- title: Stack allocation & pass by reference
  sample: |
      TODO
  details: |
      TODO
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
