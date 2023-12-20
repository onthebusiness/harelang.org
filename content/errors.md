---
# vim: ts=2 sw=2 tw=80 et :
title: Error message details
type: codetutorials
summary: |
  This page summarizes conditions from which various errors can arise when
  working on Hare projects.

  Each error includes a description of the problem, a sample of the kind of code
  which can create such a scenario, and suggested options for resolving the
  situation.
sections:
- section: Runtime assertions
- title: Slice or array out of bounds
  id: fa-0
  sample: |
      export fn main() void = {
      	let x: [3]int = [1, 2, 3];
      	x[0]; // okay
      	x[1]; // okay
      	x[2]; // okay
      	x[3]; // abort
      
      	x[0..2];  // okay
      	x[0..10]; // abort
      };
  details: |
    The "slice or array out of bounds" message is a **runtime assertion** which
    occurs when accessing an index of a slice or array which falls outside of
    its defined length. This may occur when using an indexing expression
    (`x[4]`), a slicing expression (`x[2..4]`), a slice assignment expression
    <br />(`x[..] = y[..]`), or an insert or delete expression.

    Note that when using an insert expression, it is legal to insert a value at
    the end of a slice, such that the index equals `len(slice)`. This index is
    not valid in any other kind of expression. The first index which will cause
    this runtime assertion to occur when using an insert expression is
    <br />`len(slice) + 1`.

    This message indicates that the programmer has made a design error.

    **Recommended solutions**

    - Test that your index is less than the length of the object before
      accessing a value at that index:

      ```hare
      let x: [3]int = [0...];
      let ix = 10z;
      if (ix < len(x)) {
              x[ix]; // okay
      };
      ```
    - Ensure that you are computing indices correctly. For example, look for
      [off-by-one errors](https://en.wikipedia.org/wiki/Off-by-one_error).
      Note that slices and arrays in Hare are zero-indexed, so the first item is
      at `0` and the last item is at `len(x) - 1`.

    *Further reading: [Arrays and slices](https://harelang.org/tutorials/introduction/#arrays-and-slices),
    [Working with slices](https://harelang.org/tutorials/introduction/#working-with-slices)*
- title: Type assertion failed
  id: fa-1
  sample: |
      export fn main() void = {
      	let x: (int | uint) = 10i;  // value is of type int
      	x as int;                   // okay
      	x as uint;                  // abort
      };
  details: |
    The "type assertion failed" message is a **runtime assertion** which occurs
    when the `as` operator is used on a tagged union value which is storing a
    value of a type which differs from the right-hand side of the `as`
    operation.

    This message indicates that the programmer has made a design error.

    **Recommended solutions**

    - Determine why the value was of a different type than the programmer
      expected at this point.
    - Replace the `as` expression with a match expression which handles more
      possible cases.

      ```hare
      // Before:
      let x: (int | uint) = 10i;
      x as uint; // Will fail if x is an int

      // After:
      let x: (int | uint) = 10i;
      match (x) {
      case let x: int =>
              // x is an int
      case let x: uint =>
              // x is a uint
      };
      ```

    *Further reading: [Casting & type assertions](https://harelang.org/tutorials/introduction/#casting--type-assertions)*
- title: Out of memory
  id: fa-2
  sample: |
      export fn main() void = {
      	let x: *int = alloc(42);         // might fail
      	let x: []int = alloc([1, 2, 3]); // might fail
      	append(x, 42);                   // might fail
      	insert(x[0], 42);                // might fail
      };
  details: |
    The "out of memory" message is a **runtime assertion** which occurs when a
    memory allocation fails without being handled by the programmer.

    **Recommended solutions**

    - If using the `alloc` expression, use a *nullable* pointer type. In the
      event of an allocation failure, the value will be null and this runtime
      assertion will not occur.

    <div class="alert">
      <strong>Note:</strong> Improving support for managing out-of-memory
      conditions in Hare programs is a design area planned for future
      improvement.
    </div>
- title: Static insert/append exceeds slice capacity
  id: fa-3
  sample: |
      export fn main() void = {
      	let x: [4]int = [0...];
      	let y: []int = x[..0];
      	static append(x, 1);
      	static append(x, 2);
      	static append(x, 3);
      	static append(x, 4);
      	static append(x, 5);  // abort
      };
  details: |
    The "static insert/append exceeds slice capacity" message is a **runtime
    assertion** which occurs when a `static append` or `static insert` operation
    would exceed the capacity of the slice. When using static insert and append,
    the programmer must ensure that their program will not exceed the amount of
    space allocated for the underlying slice.

    This message indicates that the programmer has made a design error.

    **Recommended solutions**

    - Test that the capacity is sufficient before performing an insert or
      append, and handle the case where there is insufficient space.
    - Increase the size of the underlying array.
    - If better suited to your use-case, switch to dynamically allocated slices
      (i.e. drop the "static" keyword).

    *Further reading: [Static slice operations](https://harelang.org/tutorials/introduction/#static-slice-operations)*
- title: Execution reached unreachable code
  id: fa-4
  sample: |
      export fn main() void = {
      	let x: (u8 | u16 | u32 | u64) = 0u64;
      	match (x) {
      	case u8 => // ...
      	case u16 => // ...
      	case u32 => // ...
      	};
      };
  details: |
    The "execution reached unreachable code" message is a **runtime assertion**
    which occurs when the program reaches an area in the generated code which
    was thought to be impossible by the compiler designers. This is a safeguard
    put in place by the compiler to avoid introducing vulnerabilities or
    unexpected behavior if such situations were to arise anyway.

    The most common cause of this error is that the programmer failed to
    consider all possible cases when writing a match expression. Currently the
    Hare compiler does not perform exhaustivity analysis on match expressions at
    compile-time; this error can be raised at runtime instead.

    This can also occur in situations where a variable used in match or switch
    expression has an illegal value. If you cast an integer to an enum type, the
    language does not test that the final value is a valid value for that
    enumeration; switching against this value may lead to this error. Similarly,
    if you create an invalid tagged union (through pointer manipulation, for
    example), this error may occur when using a match expression. It may also
    occur in similar edge cases, such as in the case of stack or heap
    corruption. Most of these cases are rare, unless your code is doing
    something unusual the most likely explanation is a non-exhaustive match
    expression or an invalid enum cast.

    Otherwise, this message indicates that a bug is present in the Hare
    compiler. Please report it to the [hare-users] mailing list.

    [hare-users]: https://lists.sr.ht/~sircmpwn/hare-users

    **Recommended solutions**

    - Double check that your match expression handles all possible cases
    - Check that the objects being matched or switched against have sane values
    - Otherwise, report the issue upstream
- title: Slice allocation capacity smaller than initializer
  id: fa-5
  sample: |
      export fn main() void = {
      	let capacity = 3z;
      	let x: []int = alloc([1, 2, 3, 4, 5], capacity); // abort
      };
  details: |
    The "slice allocation capacity smaller than initializer" message is a
    **runtime assertion** which occurs when allocating a slice with both an
    initializer and a desired capacity using an `alloc` expression if the
    provided initializer has a greater length than the desired capacity.

    This message indicates that the programmer has made a design error.

    **Recommended solutions**

    - Ensure that the initializer has no excess elements
    - Increase the requested capacity
- title: Error occurred
  id: fa-7
  sample: |
      use fmt;
      
      // Use `hare run main.ha >/dev/full` to simulate this error
      export fn main() void = {
        fmt::println("Hello world!")!; // abort
      };
  details: |
    The "error occurred" message is a **runtime assertion** which occurs when
    the error assertion operator (`!`) is used and an error occurred at runtime.

    This message indicates that the programmer has made a design error.

    **Recommended solutions**

    - Determine why an error occurred where one was not expected
    - Use a match expression to handle the possible error cases explicitly
    - Use the error propagation operator (`?`) and update the return type to
      pass the error to the function which called the relevant code

    <div class="alert">
      <strong>Tip:</strong> Using haredoc to look up the function which
      produced the error is a good way to quickly find out what possible
      outcomes you might need to handle.
    </div>

    *Further reading: [Handling errors](https://harelang.org/tutorials/introduction/#handling-errors)*
---
