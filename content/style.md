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

1. Hare source files MUST be indented with tabs. The tab size SHOULD be 8
   columns.
2. Lines SHOULD be limited to 80 columns in width, unless it would break up an
   error string, which would prevent grepping for errors.
3. When breaking a long line into several, subsequent lines MUST be intended
   once &mdash; and MUST NOT be aligned vertically with features from the
   previous line.
4. When breaking a long line into several, items SHOULD be distributed to
   achieve "balance", so that if a line were drawn down the middle of the
   expression, an approximately equal number of characters would fall to either
   side.
5. The `;` following the end of an expression shall be placed on the final line
   of that expression with no space between `;` and the last token of that
   expression.

**CORRECT**

```hare
let result = frobnicate_the_frobs(scary_frob,
	sporty_frob, baby_frob, ginger_frob, posh_frob);

if (was_frobbed_correctly(frob_context, FROB_RESULT_SUCCESS,
		FROB_STANDARD_IEEE_7553, result) {
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
	FROB_STANDARD_IEEE_7553, result) {
	return true;
};
```

## B. Source file organization

1. Hare source files SHOULD be named in `lower_underscore_case`, with the `.ha`
   file extension. Their mimetype is `text/x-hare`.
2. Each Hare source file MUST list its imports, followed by its declarations,
   with one empty line between them.
3. Use statements MUST be sorted alphabetically.
4. Declarations which require a single line MAY follow one after the other; but
   declarations which require multiple lines MUST be separated by a single
   empty line.

**CORRECT**

```hare
use bar;
use baz;
use foo;

let x: int = 10;
let y: int = 10;

def my_type = struct {
	x: int,
	y: int,
	z: int,
};

fn foobar void = {
	/* ... */
};

export fn main void = {
	/* ... */
};
```

## C. Function declarations

1. Function declarations MUST omit the parameter list `()` if empty.
2. The export status, `fn` keyword, name, parameter list, return type, and the
   `=` and `{` tokens, MUST be on the same line if they fit within 80 columns.
3. If these tokens would *not* fit on the same line, the export status, `fn`
   keyword, name, and opening parenthesis of the parameter list MUST be placed
   on the first line, then each parameter placed on subsequent lines, indented
   once, followed by the `)`, return type, `=`, and `{` tokens on the final
   line. In this case, the final parameter shall end with an extra `,` token.
4. Function bodies MUST be indented once.
5. Functions whose bodies are not an expression list `{ ... }` MAY place their
   bodies on the next line, indented once.

**CORRECT**

```hare
export fn main void = {
	do_work(1, 2);
};

fn do_work(x: int, y: int) void = {
	/* ... */
};

fn many_parameters(
	param_one: int,
	param_two: int,
	param_three: int,
	param_four: int,
	param_five: int,
) void = {
	/* ... */
};
```

## D. Type declarations

1. Spaces MUST be placed between the `def` token, the type name, the `=` token,
   and the type specifier. All of these tokens MUST be on the same line.
2. Type aliases MUST be named in `lower_underscore_case`.

**CORRECT**

```hare
def my_type = int;
```

## E. Constant declarations

1. A space MUST NOT be placed between the constant name and the `:` token. A
   space MUST be placed between the `:` token and the type specifier.
2. Spaces MUST be placed both before and after the `=` token.
3. Constants MUST be named in `UPPER_UNDERSCORE_CASE`.

**CORRECT**

```hare
def MY_CONSTANT: int = 1234;
```

## F. Global declarations

1. The use of globals is discouraged.
2. A space MUST NOT be placed between the constant name and the `:` token. A
   space MUST be placed between the `:` token and the type specifier.
3. Spaces MUST be placed both before and after the `=` token.

**CORRECT**

```hare
let my_global: int = 1234;
```

## G. Type specifiers

### i. Struct types

1. Structs MUST be defined with a space between the `struct` and `{` tokens.
3. Structs MAY be defined in either a single-line form or a multi-line form.
4. In the multi-line form, a newline MUST follow the `{` token, followed by the
   struct fields indented once, followed by the `}` token without indentation.
5. In the single-line form, the `{` token MUST be followed by a space, followed
   by the struct fields (each separated by a space following the `,` token,
   except for the final field, which MUST omit the `,` token), then a space and
   the `}` token.
6. A field name MUST NOT be separated from the `:` token by a space, but a
   space MUST be placed between the `:` token and the field type.
7. Struct fields MAY be grouped by purpose, with each groups separated by a
   single empty line.
8. Within each group of struct fields, fields MAY be alphabetized by name if the
   subsequent impact on the internal storage of the struct fields is not of
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

1. Union types shall be considered equivalent to struct types for matters of
   style, with the `union` token used in place of the `struct` token.

### iii. Array & slice types

1. There MUST NOT be a space the `[` token, length expression (if present), `]`
   token, and member type.

**CORRECT**

```hare
[]int
[5]int
[2 + 2]int
[*]int
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
4. In the multi-line style, a newline MUST follow the `(` token. On each
   subsequent line, an indentation MUST be followed by one member type, a space,
   and the `|` token. The last type MAY omit the `|` token.
5. In the multi-line style, member types MAY be grouped. Each group MUST be
   separated by a single empty line. However, this is discouraged for tagged
   unions with fewer than 10 members.

**CORRECT**

```hare
def my_union = (type_a | type_b | type_c);

def my_union = (
	type_a |
	type_b |
	type_d |
	type_e |
	type_f |
	type_g
);

def my_union = (
	result_type_a |
	result_type_b |
	result_type_c |
	result_type_d |
	result_type_e |

	error_type_a |
	error_type_b |
	error_type_d |
	error_type_c
);
```

### v. Pointer types

1. Pointer types MUST NOT have a space between the `*` or `&` token and the
   referent type.
2. Nullable pointer types MUST have a space between the `nullable` token and the
   `*` or `&` token.
3. Function pointer types MUST NOT have a space between the `fn` token and the
   parameter list, if present.
4. Function pointer types MUST omit the parameter list if no parameters are
   specified.
5. Function pointer types MAY omit the parameter name from each parameter in the
   parameter list.

## H. Values

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
4. When breaking a long expression which is the predicate of a conditional
   expression (`if`, `for`, `while`), it MUST be indented *twice*, to
   distinguish the continuation line from the body of the conditional
   expression.

**CORRECT**

```hare
if (x == y) {
	/* ... */
};

for (let x = 0; x < 10; x += 1) {
	/* ... */
};

while (x < y) {
	/* ... */
};

if (do_work(x) == y
		&& do_work(y) == z
		&& do_work(z) == q) {
	/* ... */
};
```
