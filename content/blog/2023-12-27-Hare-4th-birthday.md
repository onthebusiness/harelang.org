---
title: Happy fourth birthday, Hare!
author: Drew DeVault
date: 2023-12-27
---

Happy birthday, Hare! Today is December 27th, 2023 -- four years after the
initial commit of the original Hare prototype repository. Let's take this
opportunity to celebrate Hare's birthday and look over some ancient history
together, in particular the early prototype implementation of Hare and all of
its good old [early installment weirdness][trope].

[trope]: https://tvtropes.org/pmwiki/pmwiki.php/Main/EarlyInstallmentWeirdness

```
commit d8941a4d35d109030250f8f017d771df5ad69a66
Author: Drew DeVault <sir@cmpwn.com>
Date:   Fri Dec 27 14:07:08 2019 -0700

    Initial commit
```

This commit from the old git repository introduced a few
early pieces of Hare, not all of which survive to this day. It included:

- The GPL license
- A simple lex(1)-based lexer
- A simple yacc(1)-based parser (and the prototype grammar)
- Build system riggings with meson
- And some mocked-up Hare code

Let's check out the example code -- a mocked-up implementation of cat(1) I wrote
to play around with how the language might look and feel, and what kind of
features it might include.

```hare
use getopt;
use io;
use os;

static fn usage = io::fprintf(io::stderr, "usage: cat [-u] [file...]\n");

static fn cat (path: const str) (error | nil) =
{
	let fd: int =
		if (path[0] == '-' && path[1] == '\0') io::stdin
		else match io::open(path, io::O_RDONLY) {
			fd: int => {
				defer io::close(fd);
				fd
			},
			err: error => return err
		};
	let static buf: [io::BUFSIZ]u8;
	while ((let n = read(fd, &buf)) != nil) match n {
		[]u8 => match write(io::stdout, buf) {
			([]u8 | nil) => nil,
			err: error => return err
		},
		err: error => return err
	};
};

fn main (args: const []string) int =
{
	let a_flag: bool, b_val: str;
	def options = ('a' | ('b', str) | str);
	let args = map (let n <- getopt::parse(args, options)) {
		match n as options {
			'a' => {
				a_flag = true;
				continue
			},
			('b', val: str) => {
				b_val = val;
				continue
			},
			arg: str => arg,
			err: error => panic(err),
		}
	};
	while (let n <- args) match cat(arg) { err: error => return err };
	os::SUCCESS
};
```

There's some interesting things going on here! This code is still recognizable
something Hare-like, but any Hare programmer is going to find it quite strange.
Some of the stuff here appear in Hare's present form, and others did not. Some
notable details here include:

- A number of keywords and syntax details changed. We switched from using static
  to make a function local, to making that the default and writing global
  symbols with "export" instead. "let static" for static locals appears in Hare
  today, but the keywords were swapped -- "static let". The match syntax shown
  here also differs considerably from modern Hare, though some of the syntax was
  used for a while in early versions of the compiler.
- Of particular note here is the supposed inclusion of [dependent
  types](https://en.wikipedia.org/wiki/Dependent_type), used for the getopt API
  shown here, which never made it into the language.
- We also see some examples of a mocked-up iterator syntax here and some
  map/reduce functionality, neither of which made it in -- but a similar syntax
  for iterators is [undergoing a heated discussion][iter-rfc] on the RFC mailing
  list right now.

[iter-rfc]: https://lists.sr.ht/~sircmpwn/hare-rfc/%3C6p2uj3pshkxfnh5fmax4iwgha6cketzrpd573a4nrsjngwhhe7%40jqafva65ryu7%3E

There are quite a few other differences from what Hare eventually became -- the
experienced practitioner of Hare should be able to pick out many more. For
comparison, here is an implementation of cat(1) in modern Hare, from [hautils]:

[hautils]: https://git.sr.ht/~sircmpwn/hautils

```hare
use fmt;
use fs;
use getopt;
use io;
use main;
use os;

export fn utilmain() (main::error | void) = {
	const cmd = getopt::parse(os::args,
		('u', "POSIX compatibility, ignored"),
		"[file...]");
	defer getopt::finish(&cmd);

	if (len(cmd.args) == 0) {
		io::copy(os::stdout, os::stdin)?;
		return;
	};

	for (let i = 0z; i < len(cmd.args); i += 1z) {
		const file = open(cmd.args[i]);
		match (io::copy(os::stdout, file)) {
		case size => void;
		case let err: io::error =>
			io::close(file): void;
			return err;
		};
		io::close(file)?;
	};
};

fn open(path: str) io::handle = {
	if (path == "-") {
		return os::stdin;
	};

	match (os::open(path)) {
	case let err: fs::error =>
		fmt::fatalf("Error opening '{}': {}", path, fs::strerror(err));
	case let file: io::file =>
		return file;
	};
};
```

The mock-up from earlier is the earliest *idea* of what Hare might be, but what
about some early code that actually compiled and ran? The earliest Hare program
that successfully compiled and ran is as simple as can be:

```hare
/* Exit with successful status */
export fn main = 0;
```

Hare modules did not work at this point, and there was no runtime or syscall
access, so other early Hare programs mainly landed in the compiler test suite
and simply exited with a zero or nonzero status to indicate that they behaved
correctly. Some early Hare programs were manually bullied into linking with libc
to do some simple "hello world" stuff. However, on April 18th, 2020, the Hare
runtime was born (though it was called "sys" at this point, not rt), and the
first complete Hare program which did not link to libc was possible:

```hare
fn write (fd: int, buf: *void, count: size) size;

export fn main () int =
{
	const string = "Hello world!\n";
	write(1, string: *void, 13);
	0;
};
```

Note that at this point we still did not have modules, hence the need to
forward-declare the write syscall wrapper from libsys. Also note that early Hare
strings were compatible with C strings, and could be cast to void (or char\*)
and passed directly to the write syscall. Also missing at this point was any
kind of convenient build system for Hare, so compiling this program required the
following incantation:

```
$ ./harec compile < main.ha | qbe > main.s
$ as -o main.o main.s
$ ld -o main lib/sys/hart1.o lib/sys/libhare.a main.o
$ ./main
Hello world!
```

Let's skip forward in time again, and look at the state of Hare when we called
it quits on the prototype and began building the source code lineage which
became Hare today. The last commit to the prototype was on September 5th, 2020,
by present-day-maintainer Ember Sawady. At this point, the Hare repository
included:

- 15,000 lines of C (and yacc/lex) code in the compiler
- 3,000 lines of early Hare code in the baby "standard library"

Some weirdness which was still present at this time included:

- Optional parenthesis on function declarations if they had zero arguments
- Standard library bindings for Linux's io\_uring API, for some reason
- A primitive prototype implementation of closures, which had quite serious
  design defects and was later abandoned
- Early slices which were allocated not with the **alloc** built-in as in modern
  Hare, but rather through the standard library's slice::new:

```hare
use sys;
use types;

// Low-level slice allocation function. Subject to change or be removed.
export fn new(length: size, capacity: size, membsize: size) *types::slice = {
	let s = alloc(types::slice {
		capacity = capacity,
		length   = length,
		data     = sys::must_malloc(capacity * membsize): *void,
	});
	return s;
};
```

Honestly, I do not remember how that was supposed to work. Anyway, it was at
this point that we threw out the prototype and started working on the codebase
that would become modern Hare. After this, Hare really started to find its
identity and come together as a language. In total, five people worked on the
prototype:

- Drew DeVault
- Ember Sawady
- Michael Forney
- Preston Carpenter
- Tom Lebreux

The rest is history: today Hare is a rapidly maturing language with about 100
contributors, and it's being used to implement a wide variety of systems
programming projects. Thanks for joining me on this little journey through the
early Hare prototype, and I hope you're all enjoying writing code with the
language Hare was to become a few years later. Happy holidays!

![Photo of Hare's mascot, Harriet, pen on paper](https://l.sr.ht/pUo5.jpg)

<div style="text-align: center">
<em>
The original drawing of Hare's mascot, Harriet, by Louis Taylor
</em>
</div>
