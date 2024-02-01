---
title: New debugging features coming to Hare
author: Drew DeVault
date: 2024-02-01
---

We just merged a series of patches for Hare that adds a number of nice features
for debugging Hare programs. I want to introduce these features to you, and
share a bit about what's going on that makes them possible. Let's look at some
kinds of problems which invoke these new debugging features.

Let's start with a simple case: your program has an assertion failure. You get a
pretty backtrace now:[^col]

[^col]: Yep, the columns are wrong. Working on it.

```
$ hare run main.ha 
Abort: How does math work
/home/sircmpwn/sources/hare/main.ha:2:43 main+0x56 [0x802e7cd]
|       assert(1 == 0, "How does math work");
                                         ^
/home/sircmpwn/sources/hare/rt/+linux/start.ha:13:13 rt::start_ha+0xe [0x8003901]
|       main();
           ^
/home/sircmpwn/sources/hare/rt/+linux/platformstart-libc.ha:8:17 rt::start_linux+0x3a [0x8003ae8]
|       start_ha();
               ^
Aborted
```

We also detect a variety of other situations and print more useful information
and a nice backtrace. Here's another example: stack overflow.

```
$ hare run main.ha 
Stack overflow (address not mapped to object) at address 0x7ffe4c920ff8
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6]
| 	main();
           ^
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
	(523572 additional frames omitted)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/main.ha:2:13 main+0x9 [0x802e6f6] (already shown)
/home/sircmpwn/sources/hare/rt/+linux/start.ha:13:13 rt::start_ha+0xe [0x8003901]
| 	main();
           ^
/home/sircmpwn/sources/hare/rt/+linux/platformstart-libc.ha:8:17 rt::start_linux+0x3a [0x8003ae8]
| 	start_ha();
               ^
Aborted
```

We also have signal handlers for a number of other situations, including
segfaults, arithmetic exceptions, bus errors, and more, which provide a similar
backtrace and add useful details about the specific nature of the problem. For
instance, consider the following (exceptionally stupid) Hare code:

```hare
export fn main() void = {
	let x: *int = &main: *int;
	*x = 1337;
};
```

This causes a segfault because "main" is executable, but not writable. We now
print a handy error message when you run this program, which distinguishes it
from other kinds of segfaults like a null pointer dereference:

> Illegal pointer access (invalid permissions for mapped object) at address 0x802e6fd

These features also help you make better use of Ember's improved memory
allocator, which checks for a number of common memory errors and aborts should
they occur. Consider the double free bug:

```hare
export fn main() void = {
	let x: *int = alloc(42);
	free(x);
	free(x);
};
```

Previously, running this program gave the following error:

```
Abort: rt/malloc.ha:185:15: tried to get metadata for already-freed pointer (double free?)
```

This is better than blindly thrashing the heap and moving on, but it doesn't
give you much information about the cause of the error. If you wanted more,
you'd have to dump core and fire up gdb. It's much easier to see where the error
comes from now:

```
Abort: tried to get metadata for already-freed pointer (double free?)
/home/sircmpwn/sources/hare/rt/malloc.ha:185:18 rt::getmeta+0x7f [0x80015ec]
|       assert(m.sz & 0b1 == 0,
                ^
/home/sircmpwn/sources/hare/rt/malloc.ha:133:32 rt::free+0x3c [0x8001753]
|                 yield getmeta(p);
                              ^
/home/sircmpwn/sources/hare/main.ha:4:15 main+0x4f [0x802e73c]
|       free(x);
             ^
/home/sircmpwn/sources/hare/rt/+linux/start.ha:13:13 rt::start_ha+0xe [0x8003901]
|       main();
           ^
/home/sircmpwn/sources/hare/rt/+linux/platformstart-libc.ha:8:17 rt::start_linux+0x3a [0x8003ae8]
|       start_ha();
               ^
Aborted
```

Some of the tools used to provide this information, such as walking call frames,
are available for users in the new debug module. In the future, we'll expand the
runtime debugging support in a few interesting ways:

- Rigging the allocator more closely into debug
- Recording stack traces on malloc/free and providing valgrind-like histories
  for each pointer
- Switching to an "emergency" mode which ensures that the debug support code can
  proceed in error cases, such as by evicting file descriptors and setting up a
  new heap
- Expanding DWARF support for a more complete debugging experience
- Runtime configuration of debug information via environment variables; for
  instance you might configure the segfault handler to dump /proc/self/maps to
  stderr

Update your local copy of Hare to get access to all of these goodies!

## Under the hood

So, how does all of this work? Let's take a look at a few key areas:

1. Hooking up internal error sources
2. Hooking up terminating signals
3. Walking call frames
4. Mapping addresses with symbol names
5. Mapping addresses with file names & line numbers

### Hooking into abort()

This one is the easiest. When you use abort() in Hare, it generates a function
call rt::abort. Same if you use assert() and the condition is false -- it ends
up in rt::abort. This also happens on a number of internal conditions that cause
an abort, such as an array-out-of-bounds problem. rt::abort looks like this:

```hare
export @symbol("rt.abort") fn _abort(
	path: *str,
	line: u64,
	col: u64,
	msg: str,
) never = {
	platform_abort(path, line, col, msg);
};

// See harec:include/gen.h
const reasons: [_]str = [
	"slice or array access out of bounds",			// 0
	"type assertion failed",				// 1
	"out of memory",					// 2
	"static insert/append exceeds slice capacity",		// 3
	"execution reached unreachable code (compiler bug)",	// 4
	"slice allocation capacity smaller than initializer",	// 5
	"assertion failed",					// 6
	"error occurred",					// 7
];

export fn abort_fixed(path: *str, line: u64, col: u64, i: u64) void = {
	platform_abort(path, line, col, reasons[i]);
};
```

The "platform\_abort" function is defined by your target (e.g. Linux, OpenBSD,
whatever) and does some platform-specific functionality to process the abort,
i.e. write details to stderr and raise SIGABRT.

This was easy to make pluggable. Here's the new version:

```hare
// Signature for abort handler function.
export type abort_handler = fn(
	path: *str,
	line: u64,
	col: u64,
	msg: str,
) never;

let handle_abort: *abort_handler = &platform_abort;

// Sets a new global runtime abort handler.
export fn onabort(handler: *abort_handler) void = {
	handle_abort = handler;
};

export @symbol("rt.abort") fn _abort(
	path: *str,
	line: u64,
	col: u64,
	msg: str,
) never = {
	handle_abort(path, line, col, msg);
};
```

Now debug can register its own abort handler when it's linked into a program:

```hare
@init fn init_abort() void = {
	rt::onabort(&debug_abort);
};

// Note: take care not to get into an abort loop when working on this code
fn debug_abort(
	path: *str,
	line: u64,
	col: u64,
	msg: str,
) never = {
	// we'll get to this part later
};
```

You can also see one of the unique constraints of the new debug module here:
it's quite important that it never raises an abort during the abort handler. To
this extent we have no assertions -- be it assert(), error assertions (!), the
"as" operator, etc -- without carefully checking that they will not cause an
abort loop.

## Handling signals

The next kind of case we need to hook into is so-called "terminating" signals,
the most obvious of which is SIGTERM -- but we aren't handling that particular
signal. Instead, we're interested in signals that crash the program:

- SIGBUS: bus error (bad memory access)
- SIGFPE: arithmetic exception (e.g. divide by zero)
- SIGILL: illegal instruction (e.g. using ldgdt from user mode on x86\_64)
- SIGSEGV: the infamous segfault

We can set up a signal handler to handle all of these cases much like any other
signal. However, there is one case to consider carefully: what if a SIGSEGV
occurs because of a stack overflow? We will not be able to run the signal
handler in this case. Unix has an answer: [sigaltstack][0]. This allows the
kernel to switch the stack pointer over to an alternate stack when executing a
signal handler, which neatly eliminates this issue. We start by setting up a
static 64 KiB memory reservation in the debug module, then use it to set up an
alternate stack and our signal handler:

[0]: https://pubs.opengroup.org/onlinepubs/9699919799/functions/sigaltstack.html

```hare
// altstack.s
let altstack: [ALTSTACK_SIZE]uintptr;

// 16 KiB, sync with altstack.s
def ALTSTACK_SIZE: size = 16384;

@init fn init_overflow() void = {
	rt::sigaltstack(&rt::stack_t {
		ss_sp = &altstack,
		ss_flags = 0,
		ss_size = ALTSTACK_SIZE,
	}, null)!;
	signal::handle(sig::SEGV, &signal_handler, signal::flag::ONSTACK);
	signal::handle(sig::FPE, &signal_handler, signal::flag::ONSTACK);
	signal::handle(sig::BUS, &signal_handler, signal::flag::ONSTACK);
	signal::handle(sig::ILL, &signal_handler, signal::flag::ONSTACK);
};

fn signal_handler(sig: sig, info: *signal::siginfo, uctx: *opaque) void = {
	// ...
};
```

Good -- now we'll safely end up here when the program would otherwise crash.

Next, we're going to do something that few readers will have ever done in their
signal handler: we're going to open up this mysterious ucontext\_t parameter.
This parameter is opaque per POSIX but it's a (well hidden) part of the stable
userspace ABI on all Unix platform we support, and it contains the state --
registers -- of our program when the signal occurred. Since it will naturally be
platform-specific, the signal handler relegates the work to unpack it to
platform-specific code elsewhere in debug. We're going to extract the
instruction pointer, stack pointer, and stack frame.

```hare
fn signal_handler(sig: sig, info: *signal::siginfo, uctx: *opaque) void = {
	const ip = uctx_ip(uctx);
	const sp = uctx_sp(uctx);
	let frame = uctx_frame(uctx);
	// ...
```

This data structure varies both by platform and CPU architecture, so let's
narrow in and just look at the structure for x86\_64 Linux:

```hare
export type ucontext = struct {
	uc_flags: u64,
	uc_link: *ucontext,
	uc_stack: stack_t,
	uc_mcontext: sigcontext,
	uc_sigmask: sigset,
};

export type sigcontext = struct {
	r8: u64,
	r9: u64,
	r10: u64,
	r11: u64,
	r12: u64,
	r13: u64,
	r14: u64,
	r15: u64,
	di: u64,
	si: u64,
	bp: u64,
	bx: u64,
	dx: u64,
	ax: u64,
	cx: u64,
	sp: u64,
	ip: u64,
	flags: u64,
	cs: u16,
	gs: u16,
	fs: u16,
	ss: u16,
	err: u64,
	trapno: u64,
	oldmask: u64,
	cr2: u64,
	fpstate: u64,
	reserved1: [8]u64,
};
```

Mysteries revealed! rip is the instruction pointer, rsp is the stack pointer,
and bp is the frame pointer -- we'll cover frame pointers later. The
platform-specific helper functions just grab these for us. The rest of the
signal handler here is pretty straightforward:

```hare
	switch (sig) {
	case sig::SEGV =>
		const is_overflow = addr & ~0xFFFF == sp & ~0xFFFF;
		fmt::errorfln("{} ({}) at address 0x{:x}",
			if (is_overflow) "Stack overflow"
			else "Illegal pointer access",
			errcode_str(sig, info.code), addr): void;
	case sig::BUS =>
		fmt::errorfln("Bus error ({}) at address 0x{:x}",
			errcode_str(sig, info.code), addr): void;
	case sig::FPE =>
		// addr is the location of the faulting instruction, construct
		// an additional synethetic stack frame
		let copy = frame; frame = mkframe(&copy, addr);
		fmt::errorfln("Arithmetic exception ({})",
			errcode_str(sig, info.code)): void;
	case => void;
	};

	const self = match (image::self()) {
	case let img: image::image =>
		yield img;
	case => halt();
	};
	defer image::close(&self);

	backtrace(&self, frame);

	halt();
```

One or two points of note here: we distinguish a stack overflow from other
segfaults by looking to see if the faulting address (via siginfo) is close to
the stack. Arithmetic exceptions put the faulting address in this field, so we
construct a new stackframe to pass along to the backtrace to include the error
site.

## Walking call frames

The next point of interest is the code to examine the call stack and print a
backtrace. With Hare, this is relatively simple: we use frame pointers. The way
this works is by constructing a linked list of call frames on the stack, and
storing the head in the frame pointer register (%rbp on x86\_64). Each Hare
function call begins with the following code to achieve this (on x86\_64):

```hare
.globl main
main:
	pushq %rbp
	movq %rsp, %rbp
	// ...
```

Other languages (e.g. C) don't use the frame pointer, freeing it up for bigger
and better things (like storing locals or temporaries). On such platforms
unwinding the stack is *much* more complicated; Hare keeps it simple and so we
can implement stack unwinding fairly easily:

```hare
fn getfp() *stackframe;

// Details for a stack frame. Contents are architecture-specific.
export type stackframe = struct {
	fp: nullable *stackframe,
	ip: uintptr,
};

// Returns the caller's stack frame. Call [[next]] to walk the stack.
export fn walk() stackframe = *getfp();

// Returns the next stack frame walking the stack.
export fn next(frame: stackframe) (stackframe | void) = {
	match (frame.fp) {
	case null =>
		return;
	case let next: *stackframe =>
		if (next.fp == null) {
			return;
		};
		return *next;
	};
};

// Return the program counter address for the given stack frame.
export fn frame_pc(frame: stackframe) uintptr = frame.ip;
```

The getfp function is implemented in a separate assembly file:

```
.global debug.getfp
debug.getfp:
	endbr64
	movq %rbp,%rax
	ret
```

Equipped with this support code, walking stack frames is easy:

```hare
	const frame = debug::walk();
	for (true) {
		fmt::printfln("pc: {:x}", debug::frame_pc(frame))!;

		match (debug::next(frame)) {
		case let next: debug::stackframe =>
			frame = next;
		case void =>
			break;
		};
	};
```

...at least, if all you want is the program counter for return address (and
technically the value of %rsp, if you want it). If you want to get the symbol
names and line numbers associated with each address, stay tuned...

## Mapping addresses to symbol names

So far in our journey to backtraces we've been able to get a list of program
addresses walking back the stack, but turning numeric addresses into something
more useful requires a bit more work. We have one place we can look to: the
symbol table.

We can use objdump to examine the symbol table of a Hare executable with
`objdump -t`, which on a simple program looks similar to the following:

```
main:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 583b400c29ea82026e8e4ab879319afd5eb2437ed7f7831b75c631bdeafccd18.o
0000000080000000 l       .data	0000000000000000 strdata.15
0000000080000018 l       .data	0000000000000000 strconst.14
0000000080000030 l       .data	0000000000000000 strdata.27
0000000080000048 l       .data	0000000000000000 strconst.26
0000000080000060 l       .data	0000000000000000 strdata.35
0000000080000070 l       .data	0000000000000000 strconst.34
0000000080000088 l       .data	0000000000000000 strdata.43
0000000080000098 l       .data	0000000000000000 strconst.42
00000000800000b0 l       .data	0000000000000000 strdata.55
00000000800000c0 l       .data	0000000000000000 strconst.54
000000000800000f l     F .text	00000000000001a0 rt.u64tos
00000000800007e8 l       .bss	0000000000000000 static.299
0000000008000610 l     F .text	00000000000000c9 finifunc.4
00000000800004b8 l       .data	0000000000000000 rt.heap
0000000008000da4 l     F .text	000000000000003a rt.bin_getsize
00000000080006d9 l     F .text	00000000000000d6 rt.checkpoison
0000000008000cb5 l     F .text	0000000000000072 rt.meta_next
...
```

This tells us the address and size of all of the symbols in the program. If we
know that an error occurred at address 0x800010f, we can see that this would land
us in rt::u64tos based on this symbol table -- let's see how we can get this
information at runtime to provide a better backtrace.

Hare executables are based on the [ELF] format. This may differ from platform to
platform, but for the time being all of the supported Hare platforms use ELF, so
we'll focus on that. To start things off, we find the currently running
executable file (on Linux, we check /proc/self/exe, on other systems the
approach differs) and open it up. We could read(2) our way through it, but the
following steps require a great deal of random access so it makes more sense to
just mmap it.

[ELF]: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format

To get at the symbol table and translate addresses to symbol names, we need to
do the following steps:

1. Parse the ELF header and find the section string table
2. Find the .symtab and .strtab sections (symbol & string tables)
3. Enumerate symbols until we find one which contains our address of interest
4. Look up the symbol name in the string table

The data structures for ELF have been available in [format::elf] in the standard
library for some time now -- I made good use of it writing program loaders for
[Ares OS]. The types of interest today are:

[format::elf]: https://docs.harelang.org/format/elf
[Ares OS]: https://ares-os.org/

```hare
// ELF header for ELF64
export type header64 = struct {
	// ELF identification
	e_ident: [EI_NIDENT]u8,
	// Object file type
	e_type: elf_type,
	// Machine type
	e_machine: elf_machine,
	// Object file version ([EV_CURRENT])
	e_version: u32,
	// Entry point address
	e_entry: u64,
	// Program header offset
	e_phoff: u64,
	// Section header offset
	e_shoff: u64,
	// Processor-specific flags
	e_flags: u32,
	// ELF header size
	e_ehsize: u16,
	// Size of program header entry
	e_phentsize: u16,
	// Number of program header entries
	e_phnum: u16,
	// Size of section header entry
	e_shentsize: u16,
	// Number of section header entries
	e_shnum: u16,
	// Section name string table index, or [shn::UNDEF]
	e_shstrndx: u16,
};

// Section header for ELF64
export type section64 = struct {
	// Section name
	sh_name: u32,
	// Section type
	sh_type: u32,
	// Section attributes
	sh_flags: u64,
	// Virtual address in memory
	sh_addr: u64,
	// Offset in file
	sh_offset: u64,
	// Size of section
	sh_size: u64,
	// Link to other section
	sh_link: u32,
	// Miscellaenous information
	sh_info: u32,
	// Address alignment boundary
	sh_addralign: u64,
	// Size of entries, if section has table
	sh_entsize: u64,
};

// Symbol table entry
export type sym64 = struct {
	// Symbol name offset
	st_name: u32,
	// Type and binding attributes
	st_info: u8,
	// Reserved
	st_other: u8,
	// Section table index
	st_shndx: u16,
	// Symbol value
	st_value: u64,
	// Size of object
	st_size: u64,
};
```

Once we've mmapped it we have a pointer to \*elf::header64 to get us started. I
started a new module called debug::image and whipped up a basic data structure
to stick our references and state into:

```hare
export type image = struct {
	fd: io::file,
	data: []u8,
	header: *elf::header64,
	shstrtab: nullable *elf::section64,
};
```

We just have the file, a slice of the whole mmap'd ELF file, and a pointer to
the ELF header, simple as pie.[^1] We can get this data structure ready to go as
follows:

[^1]: There is some extra caching functionality present here which I've omitted
    from this blog post for brevity, but it's not much.

```hare
// Opens an [[io::file]] as a program image.
export fn open(
	file: io::file,
) (image | io::error) = {
	const orig = io::tell(file)?;
	io::seek(file, 0, io::whence::END)?;
	const length = io::tell(file)?: size;
	io::seek(file, orig, io::whence::SET)?;

	const base = io::mmap(null, length,
		io::prot::READ,
		io::mflag::PRIVATE,
		file, 0z)?;

	const data = (base: *[*]u8)[..length];
	const head = base: *elf::header64;

	let shstrtab: nullable *elf::section64 = null;
	if (head.e_shstrndx != shn::UNDEF) {
		const shoffs = head.e_shoff + head.e_shstrndx * head.e_shentsize;
		shstrtab = &data[shoffs]: *elf::section64;
	};

	return image {
		fd = file,
		data = data,
		header = head,
		shstrtab = shstrtab,
		...
	};
};
```

The start is about what you'd expect from my earlier description, some simple
mmap stuff, but we also make a small side trip to grab the shstrtab section
upfront -- this is necessary to map names to elf::section64 pointers later.
Speaking of which:

```hare
// Returns a program section by name. Returns null if there is no such section,
// or if the section names are not available in this image (e.g. because it was
// stripped).
export fn section_byname(
	image: *image,
	name: str,
) nullable *elf::section64 = {
	const head = image.header;
	for (let i = 0u16; i < head.e_shnum; i += 1) {
		const shoffs = head.e_shoff + i * head.e_shentsize;
		const sec = &image.data[shoffs]: *elf::section64;
		if (sec.sh_type == sht::NULL) {
			continue;
		};

		const cand = section_name(image, sec);
		if (cand == name) {
			return sec;
		};
	};
};
```

The "section\_name" function is of interest. The way that strings work in ELF is
that they are relegated to separate sections (.strtab or .shstrtab) and each
reference to a string stores an offset relative to the appropriate section. For
section names, we grab the name from the shstrtab section we cached earlier:[^2]

[^2]: Quick side note, for the detail-oriented readers: section\_validate is a
    small helper function which just asserts that the section we're looking at
    actually comes from the image it belongs to.

```hare
// Returns the name of this [[elf::section64]], returning "" if the section
// names are not available in this image (i.e. it has been stripped).
export fn section_name(
	image: *image,
	sec: *elf::section64,
) const str = {
	section_validate(image, sec);

	const shtab = match (image.shstrtab) {
	case let sec: *elf::section64 =>
		yield sec;
	case null =>
		return "";
	};

	const offs = shtab.sh_offset + sec.sh_name;
	return c::tostr(&image.data[offs]: *const c::char)!;
};
```

Another important function for working with sections we've gotta peek at before
moving on is section\_data, which returns a slice containing the actual data for
this section, and section\_reader, which is similar but returns an [io::stream].

[io::stream]: https://docs.harelang.org/io#stream

```hare
// Returns a slice of the data contained with a given section.
export fn section_data(image: *image, sec: *elf::section64) []u8 = {
	section_validate(image, sec);
	return image.data[sec.sh_offset..sec.sh_offset+sec.sh_size];
};

// Returns a [[memio::fixed]] reader for the given section.
export fn section_reader(image: *image, sec: *elf::section64) memio::stream = {
	const data = section_data(image, sec);
	return memio::fixed(data);
};
```

This gives us all of the tools we need to work with ELF sections. In order to
get symbol data, we'll actually have to peel those sections open and look at the
data inside. This is quite straightforward now:

```hare
// Returns the symbol that occupies a given address.
export fn symbol_byaddr(
	image: *image::image,
	addr: uintptr,
) (elf::sym64 | io::error | void) = {
	const addr = addr: u64;
	const symtab = match (image::section_byname(image, ".symtab")) {
	case let sec: *elf::section64 =>
		yield sec;
	case null =>
		return;
	};

	const data = image::section_data(image, symtab);
	const entsz = symtab.sh_entsize: size;
	const nsym = len(data) / entsz;
	for (let i = 0z; i < nsym; i += 1) {
		const sym = &data[i * entsz]: *elf::sym64;
		const min = sym.st_value;
		const max = sym.st_value + sym.st_size;
		if (min <= addr && addr < max) {
			return *sym;
		};
	};
};
```

That's all there is to it -- given an address and an ELF image, we can figure
out which elf::sym64 is associated with an address. To get that symbol's name,
we have to hop over to .strtab, similarly to the code for section names:

```hare
// Returns the name of the given symbol, or void if the executable was stripped.
export fn symbol_name(
	image: *image::image,
	sym: *elf::sym64,
) (const str | io::error | void) = {
	const strtab = match (image::section_byname(image, ".strtab")) {
	case let sec: *elf::section64 =>
		yield sec;
	case null =>
		return;
	};
	const data = image::section_data(image, strtab);
	return c::tostr(&data[sym.st_name]: *const c::char)!;
};
```

Tying this API all together, we can use the following program to print a
backtrace with symbol names:

```hare
use debug;
use debug::image;
use format::elf;
use fmt;

export fn main() void = {
	const self = image::self()!;
	defer image::close(&self);

	const frame = debug::walk();
	for (true) {
		const pc = debug::frame_pc(frame);
		const sym = debug::symbol_byaddr(&self, pc) as elf::sym64;
		const name = debug::symbol_name(&self, &sym) as const str;
		const offs = pc: u64 - sym.st_value;
		fmt::printfln("0x{:x}: {}+0x{:x}", pc, name, offs)!;

		match (debug::next(frame)) {
		case let next: debug::stackframe =>
			frame = next;
		case void =>
			break;
		};
	};

};
```

```
$ hare run main.ha 
0x802e9a2: main+0x189
0x8003901: rt.start_ha+0xe
0x8003ae8: rt.start_linux+0x3a
```

Great! This is good progress, and now we can at least identify the functions
that are implicated in our call stack. The most utility, however, will come from
mapping these addresses to file names and line numbers -- and this is by far the
most difficult part of our journey.

## Mapping addresses to line numbers

Now we come to the difficult part. Line numbers are hidden away in some extra
information in the executable: the [DWARF debugging information][dwarf]. The
[specification][dwarf spec] describing this data is a real doozy -- nearly 500
pages. It contains heaps of data, everything you would need to debug a program:
line numbers, but also type information, call frame details, and more. The
information we're interested in (line numbers) is encoded in a very complex
format which is only reachable after unwrapping several levels of data encodings
to find it. The process of mapping an address to a line number looks something
like this:

[dwarf]: https://dwarfstd.org/
[dwarf spec]: https://dwarfstd.org/doc/DWARF5.pdf

1. Find the debug information entry (DIE) associated with an address by looking
   it up in the .debug\_aranges section.
2. Find the DIE in .debug\_info and parse the header to find the abbreviations
   table associated with it.
3. Find this abbreviations table in .debug\_abbrev and parse the format there to
   discover the structure of the DIE.
4. Parse the DIE and find the stmt\_list attribute associated with the
   appropriate compile unit tag.
5. Find this statement list in .debug\_line.

That's the process for *finding* the line number data. Actually decoding it
involves setting up a *line number state machine* and running the virtual
machine over the instructions that appear in this section, using an instruction
set which is defined *at runtime* by the header of the line number program. The
instructions look something like this, decoded with objdump:

```
 Line Number Statements:
  [0x0000017a]  Set File Name to entry 2 in the File Name Table
  [0x0000017c]  Set column to 36
  [0x0000017e]  Extended opcode 2: set Address to 0x800001f
  [0x00000189]  Special opcode 8: advance Address by 0 to 0x800001f and Line by 3 to 4
  [0x0000018a]  Set column to 19
  [0x0000018c]  Special opcode 6: advance Address by 0 to 0x800001f and Line by 1 to 5 (view 1)
  [0x0000018d]  Set column to 14
  [0x0000018f]  Special opcode 6: advance Address by 0 to 0x800001f and Line by 1 to 6 (view 2)
  [0x00000190]  Set column to 20
  [0x00000192]  Copy (view 3)
  ...
```

The reason this format is so complex is that it has to be optimized for size
above almost any other concern. The simplest approach to mapping addresses to
line numbers is simply to store the file name, line number, and column
associated with each address directly in the object file -- but this would take
up a lot of space, likely more than the program itself does. This virtual
machine approach exists to shrink the line number data way down.

In order to get started, we have to write some support code to parse the
primitive data structures used by the DWARF specification. These are mostly
integers of various sizes, including a heavily used variable-length integer
format. I'll spare you the details and show you the API that I came up with for
it:

```hare
// Creates a new DWARF table reader.
//
// If "read_length" is true, this function will read the length from the start
// of the table. Returns [[io::EOF]] immediately if there is insufficient data
// available in the provided I/O handle.
//
// The reader will return [[io::underread]] if the DWARF table is truncated.
fn new_table_reader(
	in: *memio::stream,
	read_length: bool,
) (table_reader | io::EOF | io::error);

// Returns true if the reader is in an EOF state.
fn read_iseof(rd: *table_reader) bool;

// Aligns the head of the reader with the nearest aligned address.
fn read_align(rd: *table_reader, alignment: size) (void | io::error);

// Returns the current location of the reader from the start of the section.
fn read_tell(rd: *table_reader) size;

// Reads a signed byte.
fn read_sbyte(rd: *table_reader) (i8 | io::error);

// Reads an unsigned byte.
fn read_ubyte(rd: *table_reader) (u8 | io::error);

// Reads an unsigned 16-bit value.
fn read_uhalf(rd: *table_reader) (u16 | io::error);

// Reads an unsigned 32-bit value.
fn read_uword(rd: *table_reader) (u32 | io::error);

// Reads an unsigned 64-bit value.
fn read_ulong(rd: *table_reader) (u64 | io::error);

// Reads a "section offset", either 32- or 64-bits depending on the reader
// state.
fn read_secword(rd: *table_reader) (u64 | io::error);

// Reads an unsigned variable-length integer.
fn read_uleb128(rd: *table_reader) (u64 | io::error);

// Reads a signed variable-length integer.
fn read_sleb128(rd: *table_reader) (i64 | io::error);

// Borrowed from underlying source
fn read_slice(rd: *table_reader, amt: size) ([]u8 | io::error);

// Borrowed from underlying source
fn read_string(rd: *table_reader) (const str | io::error);
```

Some of the DWARF data structures are simple and relatively boring, such as
.debug\_aranges (just a series of addresses and file offsets). Let's start with
.debug\_info. Each .debug\_info section begins with a header which, among other
things, tells us what abbreviation table to use to decode it. Here's an example
header:

```
  Compilation Unit @ offset 0x11dc:
   Length:        0x2a8 (32-bit)
   Version:       3
   Abbrev Offset: 0x26
   Pointer Size:  8
```

There's a lot more information in this section than this header alone, but
before we move on we have to fetch the abbreviations at offset 0x26 from the
top of .debug\_abbrev. That table looks like this:

```
  Number TAG (0x26)
   1      DW_TAG_compile_unit    [has children]
    DW_AT_stmt_list    DW_FORM_data4
    DW_AT_ranges       DW_FORM_data4
    DW_AT_name         DW_FORM_strp
    DW_AT_comp_dir     DW_FORM_strp
    DW_AT_producer     DW_FORM_strp
    DW_AT_language     DW_FORM_data2
    DW_AT value: 0     DW_FORM value: 0
   2      DW_TAG_subprogram    [no children]
    DW_AT_name         DW_FORM_strp
    DW_AT_external     DW_FORM_flag
    DW_AT_type         DW_FORM_ref_udata
    DW_AT_low_pc       DW_FORM_addr
    DW_AT_high_pc      DW_FORM_addr
    DW_AT value: 0     DW_FORM value: 0
   3      DW_TAG_unspecified_type    [no children]
    DW_AT value: 0     DW_FORM value: 0
```

This information tells us the format of the DIE we want to parse. Each DIE is a
series of tags, and attributes of those tags, which forms a tree. The
debug\_abbrev section maps abbreviations (small integers) to known constants
assigned to various tags and attributes; for instance the first entry here maps
the ID 1 to DW\_TAG\_compile\_unit, which is assigned 0x11 by the specification.
Following each tag is a description of the format of the data that tag
represents, namely the list of attributes and the form(at) with which those
attributes are encoded. For instance, DW\_AT\_stmt\_list is encoded as
DW\_FORM\_data4, which means that if we encounter a "1" in the DIE, it's a
compile unit which is followed by a 4-byte value representing the statement
list associated with that compile unit, which is then followed by several other
fields and the next tag (a child of the compile unit).

Armed with this information, we can return to the debug\_info section and decode
the DIE associated with the address of interest. It looks like this:

```
  Compilation Unit @ offset 0x11dc:
   Length:        0x2a8 (32-bit)
   Version:       3
   Abbrev Offset: 0x26
   Pointer Size:  8
 <0><11e7>: Abbrev Number: 1 (DW_TAG_compile_unit)
    <11e8>   DW_AT_stmt_list   : 0x4870
    <11ec>   DW_AT_ranges      : 0xbf0
    <11f0>   DW_AT_name        : (indirect string, offset: 0): <unknown>
    <11f4>   DW_AT_comp_dir    : (indirect string, offset: 0xa): /home/sircmpwn/sources/hare
    <11f8>   DW_AT_producer    : (indirect string, offset: 0x26): GNU AS 2.41
    <11fc>   DW_AT_language    : 32769  (MIPS assembler)
 <1><11fe>: Abbrev Number: 2 (DW_TAG_subprogram)
    <11ff>   DW_AT_name        : (indirect string, offset: 0x929): bytes.zero
    <1203>   DW_AT_external    : 1
    <1204>   DW_AT_type        : <0x1486>
    <1206>   DW_AT_low_pc      : 0
    <120e>   DW_AT_high_pc     : 0
 <1><1216>: Abbrev Number: 2 (DW_TAG_subprogram)
    <1217>   DW_AT_name        : (indirect string, offset: 0x934): bytes.max_suf
    <121b>   DW_AT_external    : 0
    <121c>   DW_AT_type        : <0x1486>
    <121e>   DW_AT_low_pc      : 0x8006354
    <1226>   DW_AT_high_pc     : 0x80064ae
    ...
```

This is the debug information for the bytes module in the standard library.
The compile\_unit at the top associates all of these child tags with the same
module, and each subprogram describes a single function. We can see here that
the bytes::max\_suf function is assigned to addresses 0x8006354 through
0x80064ae. More importantly to our interests, we can also see that this whole
compile unit is associated with a statement list at offset 0x4870 in the
.debug\_line section. Finally, we're getting somewhere.

The .debug\_line entry in question opens with the following header:

```
  Offset:                      0x4870
  Length:                      3847
  DWARF Version:               3
  Prologue Length:             142
  Minimum Instruction Length:  1
  Initial value of 'is_stmt':  1
  Line Base:                   -5
  Line Range:                  14
  Opcode Base:                 13

 Opcodes:
  Opcode 1 has 0 args
  Opcode 2 has 1 arg
  Opcode 3 has 1 arg
  Opcode 4 has 1 arg
  Opcode 5 has 1 arg
  Opcode 6 has 0 args
  Opcode 7 has 0 args
  Opcode 8 has 0 args
  Opcode 9 has 1 arg
  Opcode 10 has 0 args
  Opcode 11 has 0 args
  Opcode 12 has 1 arg

 The Directory Table (offset 0x488b):
  1     bytes

 The File Name Table (offset 0x4892):
  Entry Dir     Time    Size    Name
  1     0       0       0       <unknown>
  2     1       0       0       zero.ha
  3     1       0       0       two_way.ha
  4     1       0       0       trim.ha
  5     1       0       0       tokenize.ha
  6     1       0       0       reverse.ha
  7     1       0       0       index.ha
  8     1       0       0       equal.ha
  9     1       0       0       contains.ha
```

This header defines a few inputs from which an instruction set is derived; some
using "standard" opcodes (defined by the DWARF spec and described in the opcode
list for backwards compatibility), as well as setting some parameters from which
a file-specific instruction set can be derived. It also sets up some constants
to initialize the state machine. All of this state ends up in these data
structures in Hare:

```hare
// Boolean flags for the line number state machine
export type line_flag = enum uint {
	NONE		= 0,
	IS_STMT		= 1 << 0,
	BASIC_BLOCK	= 1 << 1,
	END_SEQUENCE	= 1 << 2,
	PROLOGUE_END	= 1 << 3,
	EPILOGUE_BEGIN	= 1 << 4,
};

// Line number program state
export type line_state = struct {
	vm_loc: u64,
	addr: uintptr,
	op_index: uint,
	file: uint,
	line: uint,
	column: uint,
	flags: line_flag,
	isa: uint,
	discriminator: uint,
};

// A file with associated line numbers.
export type line_file = struct {
	name: str,
	dir: u64,
	mtime: u64,
	length: u64,
};

// Header information for a .debug_line program.
export type line_header = struct {
	min_instr_length: u8,
	max_ops_per_instr: u8,
	default_isstmt: bool,
	line_base: i8,
	line_range: u8,
	opcode_base: u8,
	opcode_lengths: []u8,
	dirs: []str,
	files: []line_file,
};
```

After parsing this header, we can execute the virtual machine. The details of
the instruction set are not worth getting into, but if you're curious here's a
small snippet:

```hare
// Step the line number state machine. Returns the current line_state on a copy
// or end-of-sequence instruction, [[io::EOF]] at the end of the file, or void
// otherwise.
export fn line_step(
	prog: *line_program,
) (line_state | void | io::EOF | io::error) = {
	let state = &prog.state;
	if (read_iseof(prog.rd)) {
		return io::EOF;
	};
	state.vm_loc = read_tell(prog.rd);

	const opcode = read_ubyte(prog.rd)?;
	if (opcode == 0) {
		// Extended opcode
		const length = read_uleb128(prog.rd)?;
		const opcode = read_ubyte(prog.rd)?;
		switch (opcode) {
		case DW_LNE_end_sequence =>
			let copy = *state;
			line_prog_reset(prog);
			return copy;
		case DW_LNE_set_address =>
			state.addr = read_ulong(prog.rd)?: uintptr;
		// ...
	} else if (opcode < prog.head.opcode_base) {
		// Special opcode
		switch (opcode) {
		case DW_LNS_copy =>
			let copy = *state;
			state.discriminator = 0;
			state.flags &= ~(
				line_flag::BASIC_BLOCK |
				line_flag::PROLOGUE_END |
				line_flag::EPILOGUE_BEGIN);
			return copy;
		case DW_LNS_advance_pc =>
			const op_adv = read_uleb128(prog.rd)?;
			state.addr += (prog.head.min_instr_length * op_adv): uintptr;
		case DW_LNS_advance_line =>
			const line = state.line: i64;
			const offs = read_sleb128(prog.rd)?;
			line += offs;
			state.line = line: uint;
		case DW_LNS_set_file =>
			state.file = read_uleb128(prog.rd)?: uint;
		// ...
	} else {
		const opcode = opcode - prog.head.opcode_base;
		const op_adv = opcode / prog.head.line_range;
		state.addr += (prog.head.min_instr_length * op_adv): uintptr;
		let line = state.line: int;
		line += prog.head.line_base: int +
			opcode: int % prog.head.line_range: int;
		state.line = line: uint;
	};
```

Essentially, this function steps the virtual machine, executing a single
instruction which can update the file name, advance the program counter,
increase or decrease the line and column numbers, and so on. Each time it
encounters a COPY instruction, we commit the state of the machine as a single
decoded line number entry. The following function helps to run the machine for
as long as it takes between COPY instructions, returning the current state when
it occurs:

```hare
// Runs the line number state machine until the next COPY instruction.
export fn line_next(prog: *line_program) (line_state | io::EOF | io::error) = {
	for (true) {
		match (line_step(prog)?) {
		case let state: line_state =>
			return state;
		case io::EOF =>
			return io::EOF;
		case void => continue;
		};
	};
};
```

Tying all of this together, we can now produce the dwarf::addr\_to\_line
function, a "simple" function which accepts a program image and an address and
tells you the file name, line number, and column number associated with that
address. Here it is, in all of its glory:

```hare
// Determines the file path, line number, and column number of a given address
// in the program image. Returns void if unknown. The return value is statically
// allocated.
export fn addr_to_line(
	image: *image::image,
	addr: uintptr,
) ((const str, uint, uint) | void | io::error) = {
	const dinfo_offs = match (arange_lookup(image, addr)) {
	case let offs: u64 =>
		yield offs;
	case =>
		return;
	};
	const dinfo = match (read_debug_info(image, dinfo_offs)?) {
	case let rd: debug_info_reader =>
		yield rd;
	case =>
		return;
	};
	defer debug_info_finish(&dinfo);

	let comp_dir = "";
	let stmt_list = 0u64, found = false;
	for (!found) {
		const entry = match (debug_info_next(&dinfo)) {
		case io::EOF =>
			return;
		case let ent: entry =>
			yield ent;
		};
		defer entry_finish(&entry);

		if (entry.tag != DW_TAG_compile_unit) {
			continue;
		};

		for (let i = 0z; i < len(entry.fields); i += 1) {
			const field = &entry.fields[i];
			switch (field.attr) {
			case DW_AT_stmt_list =>
				stmt_list = field.constant;
				found = true;
			case DW_AT_comp_dir =>
				comp_dir = field.string;
			case => yield;
			};
		};
	};

	const prog = match (exec_line_program(image, stmt_list)) {
	case let prog: line_program =>
		yield prog;
	case =>
		return;
	};
	defer line_program_finish(&prog);

	let last = line_state { ... };
	for (true) {
		const state = match (line_next(&prog)?) {
		case let state: line_state =>
			yield state;
		case io::EOF =>
			break;
		};
		defer last = state;

		if (state.file == 1) {
			continue;
		};
		if (state.addr < addr) {
			continue;
		};

		// If this is the first state we've seen, use it
		if (last.vm_loc != 0) {
			state = last;
		};

		if (state.file == 0) {
			return;
		};

		const file = &prog.head.files[state.file - 1];
		static let path = path::buffer { ... };

		path::set(&path)!;

		if (!path::abs(file.name)) {
			let dir = "";
			if (file.dir != 0) {
				dir = prog.head.dirs[file.dir - 1];
				if (!path::abs(dir) && comp_dir != "") {
					path::set(&path, comp_dir, dir)!;
				} else {
					path::set(&path, dir)!;
				};
			} else if (comp_dir != "") {
				path::set(&path, comp_dir)!;
			};
		};

		path::push(&path, file.name)!;
		return (path::string(&path), state.line, state.column);
	};
};
```

Finally, making use of this function, we can extend our sample program with line
numbers:

```hare
use debug;
use debug::dwarf;
use debug::image;
use format::elf;
use fmt;

export fn main() void = {
	const self = image::self()!;
	defer image::close(&self);

	const frame = debug::walk();
	for (true) {
		const pc = debug::frame_pc(frame);
		const sym = debug::symbol_byaddr(&self, pc) as elf::sym64;
		const name = debug::symbol_name(&self, &sym) as const str;
		const offs = pc: u64 - sym.st_value;

		match (dwarf::addr_to_line(&self, pc)) {
		case let tuple: (const str, uint, uint) =>
			const (file, line, col) = tuple;
			fmt::printfln("{}:{}:{}: 0x{:x}: {}+0x{:x}",
				file, line, col, pc, name, offs)!;
		case void =>
			fmt::printfln("0x{:x}: {}+0x{:x}", pc, name, offs)!;
		};

		match (debug::next(frame)) {
		case let next: debug::stackframe =>
			frame = next;
		case void =>
			break;
		};
	};
};
```

```
$ hare run main.ha 
/home/sircmpwn/sources/hare/main.ha:11:34: 0x802e9c4: main+0x1ab
/home/sircmpwn/sources/hare/rt/+linux/start.ha:13:13: 0x8003901: rt.start_ha+0xe
/home/sircmpwn/sources/hare/rt/+linux/platformstart-libc.ha:8:17: 0x8003ae8: rt.start_linux+0x3a
```

Phew! That was a lot of work. The debug:: module does this with a little extra
pizazz -- it also prints the context of that line from the source file, trims
exceptionally long backtraces (e.g. from stack overflows), and has much more
error handling. I hope you understand how it all works now!

## That's a wrap!

So, with these changes we have started building some more sophisticated tools
for debugging Hare programs, hopefully making it easier for new and experienced
users alike to work on Hare code. These features are enabled by default -- you
can make a release build with hare build -R to slim your binary for production.
Enjoy!
