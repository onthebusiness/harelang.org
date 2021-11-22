---
# vim: ts=8 sw=8 noet :
title: Hare programming style guide
layout: with_toc
summary: |
    There are better things to argue about than how your code looks, so Hare
    has a single canonical programming style which is considered correct.
    We, the designers of the Hare programming language, declare the coding
    style declared herein to be correct, and all others to be incorrect.
---


## A. General conventions

These apply generally to constructs found throughout Hare programs.

1. Hare source files MUST be indented with tabs. The tab size SHOULD be 8
   columns.
2. Lines SHOULD be limited to 80 columns in width, unless it would break up an
   error string, which would prevent grepping for errors.
3. When breaking a long line into several, subsequent lines MUST be indented
   once &mdash; and MUST NOT be aligned vertically to align with features on the
   previous line. If the following line would be indented due to the
   introduction of a new block, the continuation line MUST be indented twice to
   visually distinguish it from the block.
4. (*subjective*) When breaking a long line into several, items SHOULD be
   distributed to achieve "balance", such that if a line were drawn down the
   middle of the expression, an approximately equal number of characters would
   fall to either side.
5. The `;` following the end of an expression MUST be placed on the final line
   of that expression with no space between `;` and the last token of that
   expression.
6. All lines MUST NOT end in a whitespace character (space or tab).

**CORRECT**

```hare
let result = frobnicate_the_frobs(scary_frob,
	sporty_frob, baby_frob, ginger_frob, posh_frob);

if (was_frobbed_correctly(frob_context, FROB_RESULT_SUCCESS,
		FROB_STANDARD_IEEE_7553, result)) {
	return true;
};
```

**INCORRECT**

```hare
let result = frobnicate_the_frobs(scary_frob,
				  sporty_frob,
				  baby_frob,
				  ginger_frob,
				  posh_frob);

if (was_frobbed_correctly(frob_context, FROB_RESULT_SUCCESS,
	FROB_STANDARD_IEEE_7553, result)) {
	return true;
};
```

## B. Source file organization

A Hare module is made up of one or more files in a directory.

1. Hare source files SHOULD be named in `lower_underscore_case`, with the `.ha`
   file extension. Their mimetype is `text/x-hare`.
2. Hare source files may be named with only a tag (e.g. `+linux.ha`) if
   appropriate, but MUST not be named `.ha`.
3. Each Hare source file MUST list its imports, followed by its declarations,
   with one empty line between them. This empty line MUST not be included if
   there are no imports.
4. Use statements MUST be sorted alphabetically.
5. Declarations which require a single line MAY follow one after the other; but
   declarations which require multiple lines MUST be separated by a single
   empty line.

**CORRECT**

```hare
use bar;
use baz;
use foo;

let x: int = 10;
let y: int = 10;

type my_type = struct {
	x: int,
	y: int,
	z: int,
};

fn foobar() void = {
	// ...
};

export fn main() void = {
	// ...
};
```

## C. Function declarations

These rules govern the declaration of Hare functions and function prototypes.

1. The export status, `fn` keyword, name, parameter list, return type, and the
   `=` and `{` tokens, MUST be on the same line if they fit within 80 columns.
2. If these tokens would *not* fit on the same line, the export status, `fn`
   keyword, name, and opening parenthesis of the parameter list MUST be placed
   on the first line; then each parameter placed on subsequent lines, indented
   once; and on their own line, the `)`, return type, `=`, and `{` tokens. In
   this case, the final parameter MUST end with an extra `,` token, unless the
   function is variadic.
3. Prototypes MUST obey the same rules, but will omit the `=` and `{` tokens,
   and MUST place the semi-colon on the final line following the return type.
4. Function bodies MUST be indented once.
5. Functions whose bodies are not an expression list `{ ... }` MAY place their
   bodies on the next line, indented once.

**CORRECT**

```hare
export fn main() void = {
	do_work(1, 2);
};

fn do_work(x: int, y: int) void = {
	// ...
};

fn many_parameters(
	param_one: int,
	param_two: int,
	param_three: int,
	param_four: int,
	param_five: int,
) void = {
	// ...
};

fn many_variadic(
	param_one: int,
	param_two: int,
	param_three: int,
	param_four: int,
	params: int...
) void = {
	// ...
};
```
## D. Type declarations

Rules governing the declarations of types. For details on style for specific
type subclasses, see [Type specifiers](#g-type-specifiers).

1. Spaces MUST be placed between the `type` token, the type name, the `=` token,
   and the type specifier. All of these tokens MUST be on the same line.
2. Type aliases MUST be named in `lower_underscore_case`.

**CORRECT**

```hare
type my_type = int;
```

## E. Constant declarations

Rules governing constant declarations.

1. A space MUST NOT be placed between the constant name and the `:` token. A
   space MUST be placed between the `:` token and the type specifier.
2. Spaces MUST be placed both before and after the `=` token.
3. Constants MUST be named in `UPPER_UNDERSCORE_CASE`.

**CORRECT**

```hare
def MY_CONSTANT: int = 1234;
```

## F. Global declarations

Rules governing global variable declarations. Note that the use of globals is
often undesirable, as may limit your ability to expand upon or compartmentalize
an interface later.

1. A space MUST NOT be placed between the constant name and the `:` token. A
   space MUST be placed between the `:` token and the type specifier.
2. Spaces MUST be placed both before and after the `=` token.

**CORRECT**

```hare
let my_global: int = 1234;
```

## G. Type specifiers

Rules governing the format of types. Not to be confused with the rules governing
[type declarations](#d-type-declarations).

### i. Struct types

1. Structs MUST be defined with a space between the `struct` and `{` tokens.
3. Structs MAY be defined in either a single-line form or a multi-line form.
4. In the multi-line form, a newline MUST follow the `{` token, followed by the
   struct fields indented once, followed by the `}` token without indentation.
   The final field MUST include the optional `,` token in this form.
5. In the single-line form, the `{` token MUST be followed by a space, followed
   by the struct fields (each separated by a space following the `,` token,
   except for the final field, which MUST omit the `,` token), then a space and
   the `}` token.
6. A field name MUST NOT be separated from the `:` token by a space, but a
   space MUST be placed between the `:` token and the field type.
7. Struct fields MAY be grouped by purpose, with each groups separated by a
   single empty line.
8. Within each group of struct fields, fields MAY be alphabetized by name if the
   subsequent impact on the struct's storage representation is not of
   consequence.

**CORRECT**

```hare
struct {
	x: int,
	y: int,
	z: int,

	metadata: struct {
		foo: int,
		bar: int,
		baz: int,
	},
};
```

### ii. Union types

1. Union types are considered equivalent to struct types for matters of
   style, with the `union` token used in place of the `struct` token.

### iii. Array & slice types

1. There MUST NOT be a space the `[` token, length expression (if present), `]`
   token, and member type.
2. The use of arrays is preferred when possible, as the extra indirection of a
   slice type incurs a performance cost.

**CORRECT**

```hare
[]int
[5]int
[2 + 2]int
[*]int
[_]int
```

**INCORRECT**

```hare
[ ]int
[ 10 ]int
[] int
```

### iv. Tagged union types

1. Tagged union types may be specified in either a single-line or multi-line
   style. If the type would fit on a single line within 80 columns, the
   single-line style MUST be used.
2. In the single-line style, there MUST NOT be a space between the `(` or `)`
   token and the member type list.
3. In the single-line style, there MUST be a space between the `|` tokens and
   each member type.
4. (*subjective*) In the multi-line style, the programmer MAY use their
   discretion to distribute the member types to achieve "balance" as described
   by rule A.4.
5. When breaking to a new line, place the `|` on the first line. Place the final
   `)` token on the same line as the final members.
6. (*subjective*) When using a tagged union with many member types, consider
   categorizing them into additional aliases and using the `...` operator to
   unwrap them.

**CORRECT**

```hare
type my_union = (type_a | type_b | type_d | type_e | type_f | type_g);

type my_results = (result_type_a | result_type_b | result_type_c |
	result_type_d | result_type_e);

type my_errors = (error_type_a | error_type_b | error_type_d | error_type_c);

type my_union = (...my_results | ...my_errors);
```

### v. Tuple types

1. Tuple types may be specified in either a single-line or multi-line style. If
   the type would fit on a single line within 80 columns, the single-line style
   must be used.
2. A tuple type MUST NOT place a space after `(` or before `)`.
3. In the short form, a tuple type MUST NOT place a space before each `,`, and
   MUST place a space after each `,`, except for the last, which MUST be
   omitted.
5. In the long form, a tuple type MUST NOT place a space before each `,`, and
   MUST place a space after each `,`, except for the last, which MUST NOT be
   omitted.
4. (*subjective*) In the multi-line style, the programmer MAY use their
   discretion to distribute the member types to achieve "balance" as described
   by rule A.4.
5. When breaking to a new line, place the `,` on the first line. Place the final
   `)` token on the same line as the final members.

**CORRECT**

```hare
type my_tuple = (int, uint);
type my_tuple = (type_a, type_b, type_c,
	type_d, type_e, type_f);
```

### vi. Pointer types

1. Pointer types MUST NOT have a space between the `*` token and the secondary
   type.
2. Nullable pointer types MUST have a space between the `nullable` token and the
   `*` token.
3. Function pointer types MUST NOT have a space between the `fn` token and the
   parameter list.
4. Function pointer types MAY omit the parameter name from each parameter in the
   parameter list.

## H. Values

Rules governing the style for representations of values.

1. When choosing between explicit and hinted types, prefer whichever produces a
   shorter program.

**CORRECT**

```hare
let x = 0z;

let y: [_]u8 = [1, 2, 3];

let z: nullable *size = [&x, null];
```

**INCORRECT**

```hare
let x: size = 0;

let y = [1u8, 2u8, 3u8];

let z = [&x: nullable *size, null: nullable *size];
```

### i. Struct values

1. Struct values MUST be defined with a space between the `struct` and `{`
   tokens, or between the type alias name and the `{` token.
3. Struct values MAY be defined in either a single-line form or a multi-line
   form.
4. In the multi-line form, a newline MUST follow the `{` token, followed by the
   struct fields indented once, followed by the `}` token without indentation.
5. In the single-line form, the `{` token MUST be followed by a space, followed
   by the struct fields (each separated by a space following the `,` token,
   except for the final field, which MUST omit the `,` token), then a space and
   the `}` token.
6. Either all fields MUST be qualified, or all fields MUST NOT be qualified with
   their type.
7. If a field is qualified, there MUST NOT be a space between the field name and
   the `:` token.
8. There MUST be a space before and after the `=` token.

**CORRECT**

```hare
let x = struct {
	x: int = 10,
	y: int = 10,
};

let x = my_struct {
	x = 10,
	y = 10,
};
```

### ii. Array values

1. Array values MAY be defined in either a single-line form or a multi-line
   form.
2. There MUST NOT be a space between the `[` and `]` tokens and the array
   members.
3. In the single-line form, a space MUST follow each `,` token, except for the
   last.
4. In the multi-line form, a `,` must be used after the final token.
5. In the mulit-line form, a newline MUST follow the `[` token, and each
   subsequent line up to but not including the `]` token MUST be indented.
6. In the multi-line form, values may be grouped onto the same line. They must
   be separated by spaces per rule 3.
7. When using array initialization shorthand, the `...` token MUST NOT be
   separated from the last value by a space.

**CORRECT**

```hare
let x = [1, 2, 3, 4, 5];
let x = [
	1,
	2,
	3,
	4,
	5,
];
let x = [
	1, 2, 3, 4, 5,
	1, 2, 3, 4, 5,
	1, 2, 3, 4, 5,
	1, 2, 3, 4, 5,
	1, 2, 3, 4, 5,
];
let x: [10]int = [1, 2, 3, 4, 5...];
```

### iii. Tuple values

1. Tuple values may be specified in either a single-line or multi-line style. If
   the value would fit on a single line within 80 columns, the single-line style
   must be used.
2. A tuple value MUST NOT place a space after `(` or before `)`.
3. In the short form, a tuple value MUST NOT place a space before each `,`, and
   MUST place a space after each `,`, except for the last, which MUST be
   omitted.
5. In the long form, a tuple value MUST NOT place a space before each `,`, and
   MUST place a space after each `,`, except for the last, which MUST NOT be
   omitted.
4. (*subjective*) In the multi-line style, the programmer MAY use their
   discretion to distribute the member value to achieve "balance" as described
   by rule A.4.
5. When breaking to a new line, place the `,` on the first line. Place the final
   `)` token on the same line as the final members.

**CORRECT**

```hare
let x = (1, 2);
let x = (
	1,
	2,
	3,
	4,
	5,
);
let x = (foo(), bar());
let x = (
	foo(),
	bar(),
	1337,
);
```

## I. Variables

1. Variables MUST be named in `lower_underscore_case`.
2. If splitting a long binding list onto multiple lines, each line MUST be
   consistently either at the `=` token or the `,` token. The breaking token
   shall be placed on the first line. An indent MUST precede each continuation
   line.

**CORRECT**

```hare
let x = 10;
let x: int = 10;
let x: int = 10,
	y: int = 20,
	z: int = 30;
let x: int =
	do_work(lots, of, parameters);
```

## J. Arithmetic expressions

1. Spaces MUST be placed before and after binary arithmetic operators
   (e.g. `/`).
2. A space MUST NOT be placed between a unary arithmetic operator (e.g. `!`) and
   its operand.
3. A space MUST NOT be placed between the `(` and `)` operators and the inner
   expression.
4. When breaking a long line at a binary operator, the operator shall be placed
   on the second line.

**CORRECT**

```hare
2 + 4 + 5;
2 + (5 * 10);
-10 * 20;
!foobar;
let x: int = 2
	+ 2
	+ 2
	+ 2;
```

## K. Casts

1. A space MUST NOT be placed between the operand and the `:` token.
2. A space MUST be placed between the `:` token and the type.

**CORRECT**

```hare
let x = y: int;
```

## L. Postfix expressions

Postfix expressions include function calls, array or slice indexing, etc.

1. Postfix operators MUST NOT be separated from their operands by a space.

**CORRECT**

```hare
func(x, y, z);
list[10];
slice[2..4];
size(int);
```

## M. Branching expressions

1. A space MUST NOT be placed between the `(` and `)` operators and the
   branch predicate.
2. If an expression list (i.e. `{}`) is used, a space MUST be placed between
   the `)` and `{` tokens, followed by a newline. Each subsequent line MUST be
   indented, until the `}` token which MUST be aligned with the first token of
   the expression.
3. In `for` loops, each `;` token of the predicate MUST be followed by a space,
   or a newline.

**CORRECT**

```hare
if (x == y) {
	// ...
};

for (let x = 0; x < 10; x += 1) {
	// ...
};

for (x < y) {
	// ...
};

if (do_work(x) == y
		&& do_work(y) == z
		&& do_work(z) == q) {
	// ...
};
```

## N. Match and switch expressions

1. The preferred style is to align match and switch with their subordinate case
   branches on the same column.
2. The body of each case should be indented an additional level. Generally, `=>`
   should be followed by a newline.
3. If a default case (`case =>`) is given, it MUST be the last case.
4. If a case shouldn't perform any action, the body should just be `void`, ie
   `case ... => void;`. In these cases, the body may be on the same line as the
   `case` keyword.
5. (*subjective*) It is preferred to arrange any terminal cases (i.e. those that
   return, continue, break, call `os::exit` or `abort()`, etc) before any
   non-terminal cases. This groups the code which does not terminate closer to
   the code which logically follows it after the match or switch expression.

**CORRECT**

```hare
match (x) {
case foo =>
	// ...
case foobar =>
	// ...
case foobarbaz =>
	// ...
};

let foobarbaz = match (x) {
case foo =>
	// ...
	yield ...;
case foobar =>
	// ...
	yield ...;
case foobaz =>
	// ...
	yield ...;
};

match (x) {
case (foo | bar) =>
	// ...
case foobar =>
	// ...
case foobaz =>
	// ...
};
```

## Appendices

### Informal recommendations for function names

To name a function, first identify its purpose. If you were to describe this
purpose in a sentence, you should be able to identify up to three grammatical
items of importance: the verb, the object, and the subject. The subject is
usually the actor, the object is usually being acted upon, and the verb defines
the action being taken.

When naming a Hare function, use the format "subject_verbobject", where the
subject preceeds the verb and object, separated by an underscore, and the object
directly follows the verb. If you can infer the subject or object from context,
they may be omitted, so "verbobject", "subject_verb", or simply "verb" may be
appropriate names.

In the sentence "Sam goes to the store", "Sam" is the subject, "store" is the
object, and "go" is the verb. The equivalent function name would be
"sam_gostore". If we have additional context, for example if this is in the
"sam" module, we could call it "sam::gostore". Or perhaps the object is given by
a parameter, in which case "go" is sufficient:

```hare
fn sam::go(to: destination) void;
```

Abbreviating terms is acceptable, such as "str" for "string" or "tok" for
"token".

This approach prefers terseness when unambiguous. Here are some real-world
examples:

```hare
fn bufio::scantok(stream: *io::stream, delim: u8) ([]u8 | io::EOF | io::error);

fn io::read(stream: *io::stream, buf: []u8) (size | io::EOF | io::error);

fn lex::init(in: *io::stream, path: str) lexer;
```

Note the arrangement of parameters: the object being acted upon comes first. For
example, if you have "source" and "destination" parameters, "destination" should
be placed first in the parameter list.

### Verbs for allocation strategies

It is useful to communicate the allocation strategy in function names, to lend
readability to the implications for Hare's manual memory management system. The
following conventions are recommended.

For functions which initialize a value and return it, either via allocation or
via the stack, name these functions after the object being initialized. For
example, to initialize a SHA-256 hash, you use `crypto::sha256::sha256()`.
If a more specific verb than "allocate" or "initialize" would be appropriate,
name the function after that verb. For example, "open" or "connect" may be more
appropriate names than "file" or "client".

If a function accepts a pointer to a value as a parameter, and will initialize
that value, use the "init" verb to name the function.

For functions which free resources associated with an object, if the object
itself is freed, use the verb "free". If the object itself is not freed, but
some other state associated with it, use the verb "finish".

### Informal recommendations for documentation

Consult `man haredoc` for technical details regarding documenting Hare
interfaces through comments in the source code.

It is useful to have some linguistic conventions for inline documentation.  The
following guidelines provide such conventions for any programs which wish to be
consistent with the rest of the Hare ecosystem in their approach to API
documentation.

All programmer-facing documentation should be written in English, and all public
(exported) members should be documented. Not documenting an exported member
signals that it is not designed to be used by third-party programs.

Your documentation should be as concise or as long as is necessary. Programmers
reading the reference documentation are usually in a hurry, but they also
appreciate comprehensive explanations. Aim to be as short as possible without
omitting necessary details.

If you can summarize an interface in one sentence, you may omit the period at
the end. If you need several sentences, include a final period after the last
sentence. Regardless of the approach, be consistent with similar members in the
same context; if one enum value can be described in one sentence, include the
final period if the rest of the values use several sentences.

Function documentation should complete the following thought: "This function
[does, will, is used to]...". Examples:

- "Insert a new entry into the list"
- "Parse the next record from the file"

Type documentation should complete the following thought: "This type is..." or
"This type $verbs...". Examples:

- "An error indicating that an invalid sequence was encountered"
- "Indicates that more data is required to finish processing"
- "Stores the state for an XML parser"

Constant documentation should complete the following thought: "This constant
is..."

- "The size, in bytes, of an MD5 digest"
- "The magic string identifying a PNG file"

### Informal recommendations for errors

These recommendations cover programmer-facing errors. User-facing errors are
addressed separately in [Internationalization recommendations for Hare
programs](/i18n).

Each module should provide an `error` type which is a tagged union of all
possible errors which might be returned by functions in that module, and an
`strerror` function which explains the error as a string. These strings should
be written with "Sentence case" and should not end with a period. It should also
be written so that it makes sense when passed to `fmt::fatal("Error: {}",
example::strerror(err))`.

Write programmer-facing error messages in English. This includes the return
value from `strerror`, and also the error messages used in `abort` and `assert`
expressions.
