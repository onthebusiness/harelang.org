---
# vim: ts=2 sw=2 et :
title: An introduction to the Hare programming language
type: codetutorials
summary: |
  This tutorial will introduce you to the Hare programming language. It should
  take an hour or two to complete. If you are already familiar with programming
  in other languages, it might take even less time — feel free to jump around as
  you see fit.

  <div class="alert">
    <strong>Note:</strong> This tutorial is in the process of being rewritten.
    You may want to refer to <a href="/tutorials/old-introduction/">the old
    tutorial</a> to fill in the gaps.
  </div>

  After you complete this tutorial, you should move on to the
  [standard library introduction](/tutorials/stdlib).
sections:
- title: How to get help
  sample: |
      use fmt;
      
      export fn main() void = {
      	fmt::println("Hello world!")!;
      };
  details: |
      We want you to leverage the Hare community to maximize your odds of
      success with Hare. We are a community which wants to help each other build
      the best Hare programs possible, together. Please join our mailing lists,
      chat rooms, and so on, and do not hesitate to ask for help, discuss your
      problems, and offer your knowledge and insights to others.

      Here are some quick links to our [community resources](/community):

      - [hare-users](https://lists.sr.ht/~sircmpwn/hare-users) is a great place
        to ask questions
      - [#hare](https://web.libera.chat/#hare) on irc.libera.chat is a good
        place for IRC users to idle

      Please join us!
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

      <div class="alert">
        <strong>Note:</strong> Many code samples in this tutorial makes use of
        concepts which are introduced later on. If you're unsure about how
        something works, don't worry &mdash; it will be explained later.
      </div>
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
      error occurred. The `!` operator deals with the error case as we described
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
- title: Using const & let to define variables
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
      This sample is designed to illustrate a few common ways to use variables
      in Hare with the **const** and **let** keywords. When declaring a
      variable, you must provide the initial value with an *initializer*, such
      as a constant value like "42" or an arbitrary expression like a function
      call. A variable declared with **const** cannot be modified after it is
      initialized, but you can modify **let** variables with the `=` operator.

      However, though you cannot modify a const variable, you can *re-bind* it
      by creating another variable with the same name. When re-binding a
      variable, you may also change its type: in this example, "source" refers
      to an io::file, a []u8, a str, and a []str, in that order. This is a
      useful pattern for building up a variable from a series of intermediate
      values of various types. This technique is more generally called
      "shadowing" the variable.

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
      to better understand the theory and trade-offs.
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
      (when creating a variable of that type) and re-order them if we so desire.
      The other player, player1, is using an *anonymous*, or un-named, struct
      type.

      player3 is defined with a *tuple* type, which is very similar to a struct,
      but does not name its fields. They are accessed by their ordinal position,
      starting from zero, instead of their names.
- title: Arrays and slices
  sample: |
      use fmt;
      
      export fn main() void = {
      	let x: [_]int = [1, 3, 3, 7];
      	assert(len(x) == 4);	// len() built-in
      	assert(x[3] == 7);	// 0-indexed
      	x[3] = 8;		// assignment
      	assert(x[3] == 8);
      
      	let y: [1024]int = [1, 3, 3, 7, 42...];	// Fill remainder with 42
      
      	printvals(y[..4]);
      	printvals(y[2..8]);
      };
      
      fn printvals(in: []int) void = {
      	fmt::printfln("input: {} integers", len(in))!;
      	for (let i = 0z; i < len(in); i += 1) {
      		fmt::printfln("in[{}]: {}", i, in[i])!;
      	};
      	fmt::println()!;
      };
  details: |
      We'll introduce **array** and **slice** types next. Arrays store a
      specific (determined at compile-time) number of ordered values of a
      uniform subtype; slices store an arbitrary (determined at runtime) number
      of ordered values of a uniform type.

      An array may be declared with a specific length and subtype (e.g.
      "[5]int"), or may infer the length from context using an underscore (e.g.
      "[_]int"). A slice type leaves the length unwritten: "[]int". Values of a
      slice or array type can be *indexed* to obtain the value of one of their
      objects, numbered from zero, with the "[<em>index</em>]" operator as shown
      in the samples. The length (number of items) of an array or slice may be
      obtained with the "len" built-in.

      Like all values, arrays must be initialized when they are declared. This
      can be awkward for large arrays like "y". In such cases, the `...`
      operator is often useful: it assigns all remaining values to the last
      value. In this example, most of "y" is initialized to "42".
- title: Arrays and slices, continued
  sample: |
      use fmt;
      
      export fn main() void = {
      	let x: [_]int = [1, 3, 3, 7];
      	assert(len(x) == 4);	// len() built-in
      	assert(x[3] == 7);	// 0-indexed
      	x[3] = 8;		// assignment
      	assert(x[3] == 8);
      
      	let y: [1024]int = [1, 3, 3, 7, 42...];	// Fill remainder with 42
      
      	printvals(y[..4]);
      	printvals(y[2..8]);
      };
      
      fn printvals(in: []int) void = {
      	fmt::printfln("input: {} integers", len(in))!;
      	for (let i = 0z; i < len(in); i += 1) {
      		fmt::printfln("in[{}]: {}", i, in[i])!;
      	};
      	fmt::println()!;
      };
  details: |
      A *slicing expression* is used to "slice" arrays and slices with the with
      ".." operator. This creates a new slice which references a subset of the
      source object, such that "y[2..5]" will produce a slice whose 0th value is
      the 2nd value of "x" with a length of 5 - 2 = 3. Slicing does not copy the
      underlying data, so modifying the items in a slice will modify the
      underlying array.

      Accesses to arrays and slices are *bounds checked*, which means that
      accessing a value beyond the end of their valid objects will cause your
      program to abort.

      ```hare
      let x = [1, 2, 3];
      x[4]; // Abort!
      ```

      It is occasionally useful (but risky!) to skip the bounds check, for
      interop with code written in other languages, or in carefully reviewed,
      performance-critical code. To disable bounds checking, use `*` in place of
      the array length:

      ```hare
      let x: [*]int = [1, 2, 3];
      x[4]; // Undefined behavior!
      ```
- title: Using default values
  sample: |
      TODO
  details: |
      TODO
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
      to seek alternative strategies when allocating large objects&nbsp;&mdash;
      which we'll address momentarily.

      As a practical demonstration of stack allocation, this sample also
      computes and prints the SHA-256 hash of its source code. A common pattern
      in Hare is for a constructor like sha256::sha256 to return an object on
      the stack, which you can then pass into other functions by reference using
      `&`. This is convenient for many objects which can be cleaned up by simply
      discarding their state on the stack, but other kinds of objects (such as
      file handles) require additional steps.
  # TODO: introduce nullable pointers here
- title: Dynamic memory allocation & defer
  sample: |
      use fmt;
      use io;
      use os;
      
      export fn main() void = {
      	// Allocation basics
      	let x: *int = alloc(42);
      	fmt::printfln(" x: {}", x)!;
      	fmt::printfln("*x: {}", *x)!;
      	free(x);
      
      	// Applied example
      	const file = os::open(os::args[1])!;
      	defer io::close(file);
      
      	let buffer: *[65535]u8 = alloc([0...]);
      	defer free(buffer);
      
      	const n = io::read(file, buffer)! as size;
      	io::write(os::stdout, buffer[..n])!;
      };
  details: |
      Another allocation strategy in Hare is *heap* or *dynamic* allocation of
      resources. A simple sample program is provided here, which opens the file
      referred to by its first command line argument, reads up to 64 KiB from
      it, and writes it to the standard output. You can run it on itself like
      so: `hare run main.ha main.ha`.

      To allocate an object on the heap, use the **alloc** keyword along with an
      initializer in parenthesis. The runtime will request the necessary memory
      from the operating system, initialize it to the value you provide here,
      and return a pointer to this value. The first "fmt" call in this example
      prints the location (or *address*) of the allocated memory, and the second
      call prints the value which was placed there.

      Unlike stack-allocated resources, which clean themselves up when the
      function exits, heap-allocated resources must be "freed" by the caller
      using the **free** keyword. Another concept shown here is the use of
      **defer** to *defer* the execution of an expression to the end of the
      current scope (that is, the code bound by `{` and `}`). This is useful for
      causing the code which cleans up an object to be written near the code
      which creates an object.

      There are other kinds of resources that have to be cleaned up when you're
      done using them, such as open files. **defer** is useful for these cases,
      too, and in this code sample we use it to call "io::close" to clean up the
      file opened by "os::open".

      <div class="alert">
        <strong>Note:</strong> Failing to free allocated memory causes a
        <em>memory leak</em>, which is a bug in your program. Failing to close
        files is another kind of leak. Several of the earlier samples have
        memory leaks &mdash; can you identify and fix them?
      </div>

      TODO: Update me when handling allocation failures is mandatory
- title: Static allocation
  sample: |
      use fmt;
      use io;
      use os;
      
      let items: [4]int = [1, 3, 3, 7];
      
      export fn main() void = {
      	// Example A: Static globals
      	printitems();
      	items[3] = 1;
      	items[0] = 7;
      	printitems();
      
      	// Example B: Static locals
      	fmt::println(increment())!;
      	fmt::println(increment())!;
      	fmt::println(increment())!;
      
      	// Example C: Applied static allocation
      	const file = os::open(os::args[1])!;
      	static let buffer: [65535]u8 = [0...];
      	const n = io::read(file, buffer)! as size;
      	io::write(os::stdout, buffer[..n])!;
      };
      
      fn printitems() void = {
      	fmt::println(items[0], items[1], items[2], items[3])!;
      };
      
      fn increment() int = {
      	static let x: int = 41;
      	x += 1;
      	return x;
      };
  details: |
      The third major approach to allocation in Hare is *static* allocation.
      This approach involves creating a single object (a *singleton*) for the
      whole program to use. This never has to be cleaned up because it never
      goes away. The most apparent drawback of this approach is that you have to
      allocate all static resources in advance, when you write your program, and
      cannot allocate more or fewer static resources at runtime as needs demand
      them. This can increase the footprint of your program on RAM (and on disk,
      in some cases), and there are special constraints imposed on the
      initializers for static variables.

      There are two ways to use static allocation in hare: static locals and
      static globals. The first kind demonstrated in our example is a static
      global, which is defined using **const** or **let** in a similar manner to
      local variables (*local* here meaning *local to a function*). If we
      declare them with **let**, we can modify them.

      A static local is also declared with **const** or **let**, with the
      addition of the **static** keyword. Note that, after being initially set
      to 41, the value of x remains consistent across repeated calls to
      "increment". Unlike a stack-allocated local, which is allocated space in
      that specific *function call*'s stack frame, static locals are allocated
      to the function itself.

      A practical example of static allocation is shown in the third part of
      this program, which, like the previous example, reads up to 64 KiB from
      the file specified by the first command-line argument, and writes it to
      stdout. In this case, however, instead of dynamically allocating a 64 KiB
      buffer, we statically allocate the buffer. We do not have to clean it up
      and the allocation cannot fail because the system is out of memory.

      <div class="alert">
        <strong>Note:</strong> Static variables must be initialized to a value
        which can be computed at compile-time. You cannot, for example,
        initialize a static variable based on the user's input at runtime. You
        can initialize it to a sensible default value and modify it at runtime
        instead.
      </div>
- title: Thinking in terms of ownership
  sample: |
      use fmt;
      use io;
      use os;
      use strings;
      
      export fn main() void = {
      	const file = os::open("main.ha")!;        // Opens file
      	defer io::close(file);
      	const buffer = io::drain(file)!;          // Allocates buffer
      	defer free(buffer);
      	const string = strings::fromutf8(buffer); // Borrows buffer
      	fmt::print(string)!;
      };
  details: |
      Hare uses *manual memory management*, which means that you, the
      programmer, are responsible for planning for and allocating the memory
      your program uses. Some functions in the standard library, and elsewhere,
      have consequences for memory management that you should be aware of. These
      are addressed in the documentation for these functions using standardized
      language of *ownership*, *transfers* and *assumption* of ownership, and
      *borrowing* resources.

      Examine the documentation for the functions in this sample code with
      commands like "haredoc io::drain". This function allocates a buffer to
      store the results into, and returns that buffer to the caller (you), who
      *assumes ownership* over the object. You are responsible for destroying
      this object when you are done with it, freeing resources like memory to be
      used elsewhere.

      Many functions *borrow* resources to make use of them without taking
      responsibility for them. "strings::fromutf8" is an example of this, as we
      can learn from its documentation. The return value, "string", does not
      need to be (and should not be) freed, and will become invalid when the
      buffer it's borrowed from is freed.

      It is important to read the documentation for the functions you use to
      understand the ownership semantics they require, and to plan for this in
      your program. The standard library is extensively documented and "haredoc"
      makes it easy to access this information.

      <div class="alert">
        <strong>Note:</strong> We use terms like "borrow" and "ownership" to
        reason about memory, but this is not enforced at the language level. The
        compiler does not prevent double-free or use-after-free bugs. Hare does
        have other safety features, however, which will be addressed later on.
      </div>
- section: Handling errors
- title: A few words about error handling
  sample: |
      use errors;
      use fmt;
      use fs;
      use fs::{flags};
      use io;
      use os;
      use strings;
      
      export fn main() void = {
      	const path = os::args[1];
      	const oflags = flags::WRONLY | flags::TRUNC;
      
      	const file = match (os::create(path, 0o644, oflags)) {
      	case let file: io::file =>
      		yield file;
      	case errors::noaccess =>
      		fmt::fatal("Error opening {}: Access denied.", path);
      	case let err: fs::error =>
      		fmt::fatal("Error opening {}: {}", path, fs::strerror(err));
      	};
      	defer io::close(file);
      
      	const buf = strings::toutf8("Hello world!\n");
      	match (io::write(file, buf)) {
      	case let err: io::error =>
      		fmt::fatal("Error writing file: {}", io::strerror(err));
      	case let z: size =>
      		assert(z == len(buf), "Unexpected short write");
      	};
      };
  details: |
      Many operations can fail. For instance, writing a file could fail if the
      disk is full, or a network connection could fail if the Ethernet cable is
      unplugged. In Hare, it is mandatory to consider these cases. In prior
      examples, we have used the `!` operator, which causes the program to crash
      when an error occurs. Let's explore some more effective ways of dealing
      with errors.

      This program uses os::create to create a file (named after the first
      command line argument) and writes "Hello world!\n" to that file with
      io::write. Both of these functions can fail for a variety of reasons. You
      can try out some failure cases like so:

      ```
      $ hare run main.ha example.txt      # should work
      $ mkdir test && chmod -w test       # make a directory we cannot write to
      $ hare run main.ha test/example.txt # os::create fails
      $ hare run main.ha /dev/full        # io::write fails
      ```

      Error handling is very important in Hare, so we're going to go over this
      sample in detail in the next few sections.
- title: Handling errors with match
  sample: |
      use errors;
      use fmt;
      use fs;
      use fs::{flags};
      use io;
      use os;
      use strings;
      
      export fn main() void = {
      	const path = os::args[1];
      	const oflags = flags::WRONLY | flags::TRUNC;
      
      	const file = match (os::create(path, 0o644, oflags)) {
      	case let file: io::file =>
      		yield file;
      	case errors::noaccess =>
      		fmt::fatal("Error opening {}: Access denied.", path);
      	case let err: fs::error =>
      		fmt::fatal("Error opening {}: {}", path, fs::strerror(err));
      	};
      	defer io::close(file);
      
      	const buf = strings::toutf8("Hello world!\n");
      	match (io::write(file, buf)) {
      	case let err: io::error =>
      		fmt::fatal("Error writing file: {}", io::strerror(err));
      	case let z: size =>
      		assert(z == len(buf), "Unexpected short write");
      	};
      };
  details: |
      Let's look at the signature for os::create with haredoc:

      ```hare
      fn create(str, fs::mode, fs::flags...) (io::file | fs::error);
      ```

      This function can return one of two possible values: an "io::file", if the
      file was successfully opened, or "fs::error" if not. You can look up both
      of these types with haredoc if you like, and your attention is especially
      drawn to "fs::error". This type is a *tagged union* which represents each
      of the errors that can be caused during file operations.

      One way of handling these errors is with `!`, which you already know how
      to use. A more elegant way is to use **match**. A match expression takes
      an expression between its parenthesis which can return one of several
      types from a tagged union, and each **case** handles one of these types.
      The first case in this example is the successful path, which we'll talk
      about more momentarily. The second case handles a specific error:
      "errors::noaccess", and the third case handles any other errors.

      The **case let** syntax is used to bind the value for each case to a
      variable. In the first branch, this value uses the **yield** keyword to
      "yield" it to the parent expression, which causes it to be "returned", in
      a manner of speaking, from the match expression. This assigns the desired
      "io::file" value to the file variable.

      The third case binds "fs::error" to an "err" variable, then passes it into
      "fs::strerror" to convert it to a string that can be presented to the
      user in an error message. This is a standard pattern in Hare: most modules
      will provide a "strerror" function which stringifies all of the errors
      which can be caused by that module.
- title: Propagating errors
  sample: |
      use errors;
      use fmt;
      use fs;
      use fs::{flags};
      use io;
      use os;
      use strings;
      
      export fn main() void = {
      	const path = os::args[1];
      	match (writehello(path)) {
      	case void =>
      		yield;
      	case let err: fs::error =>
      		fmt::fatal("Error writing {}: {}", path, fs::strerror(err));
      	case let err: io::error =>
      		fmt::fatal("Error writing {}: {}", path, io::strerror(err));
      	};
      };
      
      fn writehello(path: str) (fs::error | io::error | void) = {
      	const oflags = flags::WRONLY | flags::TRUNC;
      	const file = os::create(path, 0o644, oflags)?;
      	defer io::close(file);
      	const buf = strings::toutf8("Hello world!\n");
      	io::write(file, buf)?;
      };
  details: |
      It is cumbersome to use match to enumerate every possible failure for
      every function that might fail. To make it easier to deal with errors, the
      `?` operator is generally useful. The purpose of this operator is to check
      for errors and, if found, return them to a higher call frame, or if not,
      proceed normally.

      To use this functionality, it is necessary to establish some error
      handling code somewhere in the program. In this sample, the "main"
      function is responsible for all error handling. In more complex programs,
      you may handle various kinds of errors at different levels throughout the
      program. An HTTP server, for instance, might have some logic to handle
      configuration errors by printing a message and stopping the server, but
      could handle errors related to a specific client by sending them an error
      response or disconnecting them.

      "os::create" and "io::write" together can return either an fs::error or an
      io::error, so the result type for our "writehello" function is a tagged
      union of either "void" (nothing, indicating success), "fs::error", or
      "io::error". We can then use `?` to return these errors immediately and
      extract the useful types from the return values of these functions, and
      handle both cases in "main".
- title: Defining new error types
  sample: |
      use bufio;
      use fmt;
      use io;
      use os;
      use strconv;
      use strings;
      
      export fn main() void = {
      	match (prompt()) {
      	case void =>
      		yield;
      	case let err: error =>
      		fmt::fatal(strerror(err));
      	};
      };
      
      // An invalid number was provided.
      type invalid = !(strconv::invalid | strconv::overflow);
      
      // An error which indicates that [[io::EOF]] was unexpectedly encountered.
      type unexpectedeof = !void;
      
      // Tagged union of all possible errors.
      type error = !(io::error | invalid | unexpectedeof);

      // Converts an error into a user-friendly string
      fn strerror(err: error) str = {
      	match (err) {
      	case invalid =>
      		return "Expected a positive number";
      	case unexpectedeof =>
      		return "Unexpected end of file";
      	case err: io::error =>
      		return io::strerror(err);
      	};
      };
      
      fn prompt() (void | error) = {
      	fmt::println("Please enter a positive number:")!;
      	const num = getnumber()?;
      	fmt::printfln("{} + 2 is {}", num, num + 2)!;
      };
      
      fn getnumber() (uint | error) = {
      	const name = match (bufio::scanline(os::stdin)?) {
      	case io::EOF =>
      		return unexpectedeof;
      	case let buf: []u8 =>
      		yield strings::fromutf8(buf);
      	};
      	defer free(name);
      	return strconv::stou(name)?;
      };
  details: |
      Here we have a somewhat more complex sample in which we prompt the user to
      enter a number and then enumerate all possible error cases, such as
      entering something other than a number or pressing Ctrl+D to close the
      input file without entering anything at all. Try doing these things
      yourself and seeing how the program responds.

      We can define new error types ourselves by using `!` to prefix the type
      declaration. "invalid" is an error type which derives from the errors
      which can be returned from "strconv::stou", and "unexpectedeof" is a
      custom error based on the "void" type. The latter does not store any
      additional state other than the type itself, so it has a size of zero. We
      also define a tagged union containing each of these error types, plus
      io::error, also using ! to indicate that it is an error type.

      We can return our custom error from "getnumber" upon encountering io::EOF
      (which is not ordinarily considered an error) from "bufio::scanline",
      which is propagated through prompt to "main". If an I/O error were to
      occur here, it would be propagated similarly.

      We can use any type as an error type if we wish. Some errors are an int or
      enum type containing an error code, or a struct with additional
      information like a client IP address or a line and column number in a
      file, and so on. We can also provide our own "strerror" functions which
      provide helpful error strings, possibly incorporating information stored
      in the error type.
- title: Handling allocation failure
  sample: |
      TODO
  details: |
      TODO
- title: Testing your code
  sample: |
      fn sort(items: []int) void = {
      	for (true) {
      		let sorted = true;
      		for (let i = 1z; i < len(items); i += 1) {
      			if (items[i - 1] > items[i]) {
      				const x = items[i - 1];
      				items[i - 1] = items[i];
      				items[i] = x;
      				sorted = false;
      			};
      		};
      		if (sorted) {
      			break;
      		};
      	};
      };
      
      @test fn sort() void = {
      	let items = [5, 4, 3, 2, 1];
      	sort(items);
      	for (let i = 1z; i < len(items); i += 1) {
      		assert(items[i - 1] <= items[i], "list is unsorted");
      	};
      };
  details: |
      Hare has first-class support for tests via the `@test` attribute on
      functions. You can run the tests for this sample program by running "hare
      test" in the directory where this file is present.

      Our simple sample here is a [bubble sort] implementation. You may already
      know that this is not a very good sort algorithm &mdash; you will likely
      wish to use the standard library's [sort] module in real-world code.

      [bubble sort]: https://en.wikipedia.org/wiki/Bubble_sort
      [sort]: https://docs.harelang.org/sort

      Note that this code does not have a "main" function &mdash; `hare run`
      will not work here. You can test code which is not strictly speaking a
      "program", such as libraries written in Hare.

      Also take note of the use of the `assert` built-in: given a condition (a
      `bool`), if the condition is false, the program is stopped and the
      message is printed. You can also skip the message if you don't need to be
      specific; the file name and line number will be printed and you can
      generally figure out what went wrong regardless.
- section: Control flow
- title: "if & switch statements"
  sample: |
      use fmt;
      
      type color = enum {
      	RED,
      	ORANGE,
      	YELLOW,
      	GREEN,
      	BLUE,
      	VIOLET,
      };
      
      export fn main() void = {
      	let stock = [
      		(color::RED, 1),
      		(color::BLUE, 6),
      		(color::VIOLET, 1),
      		(color::ORANGE, 4),
      	];
      	fmt::println("Inventory:")!;
      	printstock(stock[0]);
      	printstock(stock[1]);
      	printstock(stock[2]);
      	printstock(stock[3]);
      };
      
      fn printstock(item: (color, int)) void = {
      	const color = item.0, amount = item.1;
      	fmt::printfln("{} paint\t{} liter{}",
      		colorstr(color), amount,
      		if (amount != 1) "s" else "")!;
      };
      
      fn colorstr(c: color) str = {
      	switch (c) {
      	case color::RED =>
      		return "Red";
      	case color::ORANGE =>
      		return "Orange";
      	case color::YELLOW =>
      		return "Yellow";
      	case color::GREEN =>
      		return "Green";
      	case color::BLUE =>
      		return "Blue";
      	case color::VIOLET =>
      		return "Violet";
      	};
      };
  details: |
      We have used these expressions many times without explanation, but now
      we'll dive into control statements in depth. We'll begin with statements
      that select from one or more "branches" of execution: if and switch.

      An "if" statement selects from one or more branches based on the truth of
      a boolean expression, such as `x > 5`. If this statement is true, the
      corresponding branch is executed. Optionally, you may follow the "true"
      branch with the `else` keyword and another branch, which is executed
      should the conditional expression turn out to be false. Hare also supports
      "else if" expressions, which execute their branch if the previous branch
      is false and a second condition of their own is met.

      ```hare
      if (condition 1) {
          // Executes if condition 1 is true
      } else if (condition 2) {
          // Executes if condition 1 is not true and condition 2 is true
      } else if (condition N...) {
          // And so on...
      } else {
          // Executes if all other conditions were false
      };
      ```

      An important note is that if expressions, like most Hare expressions, can
      appear in any expression and can compute a value. The "printstock"
      function from this sample code uses this to determine if "liter" should be
      written in the plural form by testing if the quantity is not one to select
      between an \"s\" suffix and an empty string (\"\").

      Switch expressions provide a more structured approach to branching. Based
      on a single value, the *switch value*, one of several *cases* is selected.
      The *switch value* can be any primitive type, such as ints or rune, as
      well as str and enum values. The "colorstr" function in the sample uses a
      switch expression to select a different branch based on the value of the
      color parameter, then returns a string representing that value.

      Note that switch expressions are required to be *exhaustive*, which means
      that every possible value of the *switch value* should have a
      corresponding branch. If you do not want to handle every possibility
      individually, you can add a "default" branch by omitting the value `case
      =>`. This branch will be executed should no more-specific branch suffice.
      This is also possible with match cases that omit the match type.
- title: Using yield
  sample: |
      use fmt;
      use fs;
      use io;
      use os;
      
      export fn main() void = {
      	const file = match (os::open(os::args[1])) {
      	case let f: io::file =>
      		yield f;
      	case let err: fs::error =>
      		fmt::fatal("Unable to open {}: {}",
      			os::args[1], fs::strerror(err));
      	};
      
      	match (io::copy(os::stdout, file)) {
      	case size =>
      		yield;
      	case let err: io::error =>
      		fmt::fatal("copy: {}", fs::strerror(err));
      	};
      };
  details: |
      In many of the samples so far, we've seen the use of { and } to denote
      blocks of expressions which are evaluated one after another. In many
      languages, these are decorative, but in Hare they have a semantic meaning:
      these introduce new *compound expressions*, and, like most other
      expressions, these expressions can compute a result using the "yield"
      keyword.

      The simple example given here is meant to show some simple usages of
      yield, similar to ones we've seen and left unexplained in previous
      samples. The only clarification we wish to add here is to draw your
      attention to the { and } characters, and clarify that the yield keyword
      causes the nearest compound expression to stop and to have the result of
      that expression set to the value provided to yield. The sample code shown
      here uses this to cause the value of "f" to be assigned to the "file"
      variable for further use outside of the match expression.

      This example keeps things simple, but, in the field, you will probably
      find "yield" useful less frequently in less obvious situations. Getting
      values out of match expressions is likely to be the most common use of
      this feature in your code. But, keep yield in your tool belt and you might
      end up using it to solve other problems every now and then, too.
- title: for loops
  sample: |
      use fmt;
      
      export fn main() void = {
      	const items = [
      		"Hello,",
      		"world!",
      		"Hare",
      		"is",
      		"cool!",
      	];
      	let i = 0z;
      	for (i < len(items); i += 1) {
      		if (items[i] == "Hare") {
      			break;
      		};
      	};
      	fmt::printfln(`"Hare" is at index {}`, i)!;
      };
  details: |
      All loops in Hare are written with the "for" keyword, which allows some
      logic to repeat so long as a condition is met. The syntax is:

      <pre>for (<em>binding</em>; <em>condition</em>; <em>afterthought</em>)</pre>

      The *binding* allows you to declare and initialize a variable which only
      exists within the loop. The *condition* determines when the loop
      terminates &mdash; it's evaluated at the start of each loop iteration, and
      will terminate the loop if false. The *afterthought* runs after each
      iteration, and is a convenient place to update the variables used for the
      loop condition.

      Any of these elements may be omitted, such as the binding in this code
      sample, which allows us to access the loop item (i) outside of the loop.
      A loop which never terminates may be written like so:

      ```hare
      for (true) {
      	// ...
      };
      ```
- title: Flow control
  sample: |
      use fmt;
      
      export fn main() void = {
      	const items = [
      		"Hello,",
      		"world!",
      		"Hare",
      		"is",
      		"cool!",
      	];
      	for (let i = 0z; i < len(items); i += 1) {
      		if (items[i] == "Hare") {
      			continue;
      		};
      		fmt::println(items[i])!;
      	};
      };
  details: |
      We can use the `break` and `continue` keywords to manipulate control flow
      in Hare loops. In the previous sample, we used `break` to terminate the
      loop early when encountering a string called "Hare"; in this sample we
      stop that loop iteration and continue with the next iteration.

      Both of these keywords are "terminating expressions". What does that mean?
- title: Terminating branches
  sample: |
      use fmt;
      
      export fn main() void = {
      	const color = color::BLUE;
      	const name = switch (color) {
      	case color::RED =>
      		yield "red";
      	case color::ORANGE =>
      		yield "orange";
      	case color::YELLOW =>
      		yield "yellow";
      	case color::GREEN =>
      		abort("green is not a creative color");
      	case color::BLUE =>
      		yield "blue";
      	case color::VIOLET =>
      		yield "violet";
      	};
      	fmt::println("your color is", name)!;
      };
      
      type color = enum {
      	RED,
      	ORANGE,
      	YELLOW,
      	GREEN,
      	BLUE,
      	VIOLET,
      };
  details: |
      Some expressions, such as return, break, and continue, as well as
      others like abort() and calling @noreturn functions like [os::exit][0],
      cause the control flow to "terminate" and prevent future expressions in
      that compound expression from executing.

      [0]: https://docs.harelang.org/os#exit

      This becomes more important when you consider that Hare is an
      expression-oriented language. In the sample here, each branch of our
      switch expression provides a value (the name of the color) to serve as the
      result of the switch expression &mdash; except for `color::GREEN`. The use
      of a terminating expression here, abort(), prevents the code from
      continuing after this point, so this branch is not required to provide a
      result and is not considered when determining the switch expression's
      result type.

      We have taken advantage of this behavior many times throughout the
      tutorial. For example, in [Using yield](#using-yield), the branch which
      calls fmt::fatal terminates, allowing us to only provide a value in one
      case.
- section: Types in depth
- title: Promotion and type inference
  sample: |
      TODO
  details: |
      TODO
- title: Casting & type assertions
  sample: |
      use fmt;
      
      export fn main() void = {
      	// Casting between numeric types allows for lossy conversions:
      	fmt::println(13.37f32: int)!;
      
      	// Type assertions let you examine the type of a tagged union:
      	let x: (int | uint) = 42i;
      	assert(x is int);
      	fmt::println(x as int)!;
      
      	let y: nullable *(int | uint) = &x;
      	assert(!(y is null)); // And nullable pointers
      
      	// You can also use casts for pointer arithmetic, ill-advised as that
      	// may be:
      	fmt::printfln("{:x} {:x}", y, y: uintptr + 10)!;
      
      	// Casts can also be used to change pointer types, ill-advised as that
      	// may be:
      	let z = (y: uintptr + size(int): uintptr): *int;
      	assert(*z == 42);
      };
  details: |
      This sample introduces various ways of working with types explicitly in
      Hare. One such example is *casting* types from one to another. One
      use-case for this is given in the first example: converting between
      numeric types. Since the fractional part is lost, this conversion is
      lossy, and therefore must be done explicitly. Converting between signed
      and unsigned types must also be explicit.

      Hare also supports *type tests* and *type assertions* via the "is" and
      "as" keywords, which can be used to work with the selected type of a
      tagged union at runtime. An "is" expression returns a bool, true if the
      tagged union is set to a value of the given member type. The "as"
      expression is useful for when you *know* that a tagged union has a
      particular type and you want to treat it as that type &mdash; this
      "assertion" is tested at runtime and will cause your program to abort if
      found to be untrue (try swapping "x as int" for "x as uint" to demonstrate
      this).

      Casts are tested at compile time to ensure that the desired conversion is
      generally *possible*, but no attempt is made to test that it is
      *advisable*. Thus, casting can be used to override Hare's type system when
      the programmer believes that they know better, and this comes with risks.
      In the simplest case, converting an f32 to an int might lead to some bugs
      because you wanted to round instead of truncate. A more dangerous example
      is shown here as well: pointer arithmetic and conversions between pointer
      types. Using casts, you can instruct Hare to treat some memory as if it
      were a given type, regardless of if it actually is or not. Use with
      cation.
- title: User-defined types
  sample: |
      type index = size;
      type offs = size;
      
      export fn main() void = {
      	let z: (index | offs) = 1337: offs;
      	assert(z is offs);
      };
  details: |
      You have already seen user-defined types several times throughout this
      tutorial, defining types like player coordinates, enums for colors, and so
      on. You have likely already gathered how this works: use the `type`
      keyword to begin a new type declaration.

      We want to illustrate one additional point here: each user-defined type
      creates a distinct type identity in the Hare type system. One reason this
      is important is for tagged unions: even if two types share the same
      underlying representation (such as "index" and "offs" in our sample code),
      they can be considered distinct types in a tagged union.
- title: Pointer types in depth
  # nullable, null value, auto-dereference
  sample: |
      use fmt;
      
      type coords = struct {
      	x: int,
      	y: int,
      };
      
      export fn main() void = {
      	let pos = coords { x = 10, y = 20 };
      	printcoords(null);
      	printcoords(&pos);
      };
      
      fn printcoords(pos: nullable *coords) void = {
      	match (pos) {
      	case null =>
      		fmt::println("(null)")!;
      	case let pos: *coords =>
      		fmt::printfln("({}, {})", pos.x, pos.y)!;
      	};
      };
  details: |
      Pointers in Hare, by default, cannot be "null". In other words, all
      pointers must refer to a valid address. However, it is often useful to
      signal the absence of a value, and we can do this with a *nullable*
      pointer type.

      The "printcoords" function in the sample code accepts an argument of type
      "nullable *coords". We cannot *deference* this type with the `*` or `.`
      operators like we ordinarily can: we must first test if it is valid. One
      way to do this is to match against "null", as shown here.

      Hare also includes a feature called "auto-dereferencing", which allows you
      to do things like using the `.` operator to access struct members via a
      pointer. This allows us to use "pos.x" rather than "(*pos).x". Many other
      features work similarly &mdash; indexing arrays, append et al (covered
      later), and so on. This works for any level of indirection: pointers to
      pointers to pointers to pointers... can also be dereferenced
      automatically.
- title: Struct sub-typing
  sample: |
     use fmt;
     use io;
     use os;
     
     type limitstream = struct {
     	io::stream,
     	source: io::handle,
     	limit: size,
     };
     
     fn limitwriter(in: io::handle, limit: size) limitstream = {
     	return limitstream {
     		writer = &limit_write,
     		source = in,
     		limit = limit,
     		...
     	};
     };
     
     fn limit_write(st: *io::stream, buf: const []u8) (size | io::error) = {
     	const st = st: *limitstream;
     	const buf = if (len(buf) > st.limit) {
     		yield buf[..st.limit];
     	} else {
     		yield buf;
     	};
     	st.limit -= len(buf);
     	return io::write(st.source, buf)?;
     };
     
     export fn main() void = {
     	const limit = limitwriter(os::stdout, 5);
     	fmt::fprintln(&limit, "Hello world!")!;
     };
  details: |
     Pointers to structs enjoy a special feature in Hare: automatic sub-typing.
     Essentially, a pointer to one type can be automatically cast to a pointer
     of another type so long as the second type appears at the beginning of the
     first type. This is useful for implementing abstract interfaces in Hare
     programs.

     This sample program takes advantage of this by implementing a custom
     [io::stream][0] that limits the total amount of data which can be written,
     essentially demonstrating a slimmed down version of the standard library's
     built-in [limitwriter][1]. Running this program only writes the first five
     bytes to stdout: "Hello".

     The pointer to &limit passed to the fmt::fprintln call is of type
     *limitstream, which embeds the io::stream type as the first field.
     Thus, we can safely pass it to a function expecting an *io::stream
     pointer, such as fmt::fprintln. In the "limit_write" function, we cast this
     pointer back to *limitstream so that we can access additional data we need
     to store to implement the limit writer functionality.

     [0]: https://docs.harelang.org/io#stream
     [1]: https://docs.harelang.org/io#limitwriter
- title: Tagged unions in depth
  # TODO: Revisit this code sample once we finish updating parsing et al
  sample: |
      use bufio;
      use hare::ast;
      use hare::lex;
      use hare::parse;
      use hare::types;
      use io;
      use strings;
      
      type signed = (int | i8 | i16 | i32 | i64);
      // equivalent to:
      // type signed = (i64 | (i32 | (i16 | (i8 | int | int | int))));
      type unsigned = (uint | u8 | u16 | u32 | u64);
      type integer = (...unsigned | ...signed);
      type floating = (f32 | f64);
      type numeric = (...integer | ...floating);
      
      type numeric_repr = struct {
      	id: u32,
      	union {
      		_int: int,
      		_i8: i8,
      		_i16: i16,
      		_i32: i32,
      		_i64: i64,
      		// ...
      	},
      };
      
      export fn main() void = {
      	const input = bufio::fixed(strings::toutf8("int"), io::mode::READ);
      	const lexer = lex::init(&input, "<string>");
      	const _type = parse::_type(&lexer)!;
      	defer ast::type_free(_type);
      	const store = types::store(types::x86_64, null, null);
      	defer types::store_free(store);
      	const itype = types::lookup(store, &_type) as const *types::_type;
      
      	const obj: numeric = 1337;
      	const ptr = &obj: *numeric_repr;
      	assert(ptr.id == itype.id);
      	assert(ptr._int == 1337);
      };
  details: |
      Let's take a closer look at tagged unions in Hare. Tagged union types have
      three important properties: they are *commutative*, *associative*, and
      *reductive*. This is illustrated by the commented-out version of the
      "signed" type in the sample code &mdash; the order of types, inclusion of
      the same type more than once, or use of nested tagged unions has no effect
      on the final type.

      An exception to this occurs when using type aliases: a tagged union which
      contains a type alias referring to another tagged union does not reduce,
      unless you use the `...` operator. The "integer" and "numeric" types in
      our sample make use of this behavior. This reduces the size of the final
      type (by not nesting tagged unions), but is undesirable when the
      underlying representations should be distinct &mdash; two error types
      which are type aliases for `!void` would not be distinguishable if you
      used `...` to incorporate them into the same tagged union.

      The rest of the sample is not important to understand, but does illuminate
      some of the internal implementation details of tagged unions. Each type in
      Hare is assigned a unique ID, which is stored as the first field of a
      tagged union type to indicate which type is stored there. Following the
      tag, a union of all of the possible types is stored.

      The Hare standard library provides access to tools for parsing and
      introspecting Hare programs, which the sample makes use of to find the
      type ID of "int" and compare it against the one stored in the tagged
      union. We're not going to go into detail here on how any of this works,
      but feel free to browse the relevant modules with haredoc if you're
      curious.
- section: Working with slices
- title: Growable slices
  sample: |
      TODO
  details: |
      TODO
- title: Static slice operations
  sample: |
      TODO
  details: |
      TODO
- section: Functions in depth
- title: Variadic functions
  sample: |
      TODO
  details: |
      TODO
- title: Function pointers
  sample: |
      TODO
  details: |
      TODO
- title: "@init, @fini, @noreturn"
  sample: |
      TODO
  details: |
      TODO
- section: Modules
---

And that's the Hare programming language! Nice work getting through all of
that. Now would probably be a good time to remind you about our [community
resources](/community):

- The [hare-users](https://lists.sr.ht/~sircmpwn/hare-users) mailing list is a
  great place to ask questions
- The [#hare](https://web.libera.chat/#hare) chat room on irc.libera.chat is a
  good place for IRC users to idle

If you want to read some more real-world Hare code samples, also check out the
[hautils] project, which offers several small and straightforward
implementations of common shell commands like "tee". This is also a great
project to get started with writing some real-world Hare code&nbsp;&mdash; maybe
you'll send us a patch?

[hautils]: https://git.sr.ht/~sircmpwn/hautils

The next tutorial is the [standard library introduction](tutorials/stdlib).
