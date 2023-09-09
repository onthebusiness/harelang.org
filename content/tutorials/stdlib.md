---
title: An introduction to the Hare standard library
type: tutorials
summary: |
  This tutorial introduces you to the Hare standard library. It assumes
  familiarity with most of the fundamental language concepts, which you can
  learn from the [language introduction tutorial]. You are also encouraged to
  make liberal use of the standard library's reference documentation, which is
  available in your terminal via the "haredoc" tool, or available online at
  [docs.harelang.org](https://docs.harelang.org).

  We will not cover the entire standard library in this tutorial, but we will
  introduce you to the most important parts of the standard library, and give
  you an idea of its general design and use.

  <div class="alert">
    <strong>Note:</strong> As you can tell, much of this tutorial remains to be
    written! However, comprehensive reference documentation is available for the
    standard library. You can browse it in your terminal via the "haredoc"
    command, or online at
    <a href="https://docs.harelang.org">docs.harelang.org</a>.
    If you have any questions, do not hestitate to
    <a href="/community">connect with the Hare community</a>.
  </div>

  [language introduction tutorial]: /tutorials/introduction/
---

## Input and output

The Hare standard library offers support for file I/O, to access the host
filesystem, read and write files, and work with pipes. A userspace I/O
abstraction is provided that allows the user to wrap I/O sources in various
processing tools to deal with compression, encryption, hashing, and so on.

### Hare's I/O abstraction

The essential resource for I/O in Hare is [io::handle]. This type is a tagged
union which can store either a native file handle, [io::file] (i.e. a Unix file
descriptor), which is backed by the host operating system and provides access to
files, network sockets, and other resources; or an [io::stream], which
implements I/O operations in userspace.

Resources for creating an io::file are varied throughout the standard library,
for instance [os::open] creates an io::file by creating or opening a file on the
host filesystem, while [net::tcp::connect] opens a TCP connection and returns
the file descriptor associated with the socket.

Additionally, various implementations of [io::stream] are provided for various
purposes. Buffered I/O is implemented via [bufio], for example. Other examples
include the [hash] module, and modules which implement hashes (such as
[hash::fnv] or [crypto::sha256]), which extend the I/O abstraction for improved
performance and additional tasks which may be accomplished with I/O operations.

[io::handle]: https://docs.harelang.org/io#handle
[io::file]: https://docs.harelang.org/io#file
[io::stream]: https://docs.harelang.org/io#stream
[os::open]: https://docs.harelang.org/os#open
[net::tcp::connect]: https://docs.harelang.org/net/tcp#connect
[bufio]: https://docs.harelang.org/bufio
[hash]: https://docs.harelang.org/hash
[hash::fnv]: https://docs.harelang.org/hash/fnv
[crypto::sha256]: https://docs.harelang.org/crypto/sha256

```hare
use crypto::sha256;
use encoding::hex;
use fmt;
use fs;
use hash;
use io;
use os;

export fn main() void = {
	if (len(os::args) != 2) {
		fmt::fatalf("Usage: {} <input>", os::args[0]);
	};

	const path = os::args[1];
	const file = match (os::open(path)) {
	case let file: io::file =>
		yield file;
	case let err: fs::error =>
		fmt::fatalf("Error opening {}: {}",
			path, fs::strerror(err));
	};
	defer io::close(file)!;

	const hash = sha256::sha256();
	io::copy(&hash, file)!;

	let sum: [sha256::SIZE]u8 = [0...];
	hash::sum(&hash, sum);

	fmt::println(hex::encodestr(sum))!;
};
```

*This program computes the sha256 hash of a file using various I/O features from
the Hare standard library.*

### Performing I/O operations

Support for standard file operations is provided by the standard library, such
as:

* [io::read]: read from a file
* [io::write]: write to a file
* [io::close]: close a file
* [io::seek]: change the position in a file
* [io::copy]: efficiently copy all data between files

[io::read]: https://docs.harelang.org/io#read
[io::write]: https://docs.harelang.org/io#write
[io::seek]: https://docs.harelang.org/io#seek
[io::copy]: https://docs.harelang.org/io#copy
[io::close]: https://docs.harelang.org/io#close

Generally speaking, I/O takes the form of obtaining an I/O object (such as by
[opening][os::open] a file) and performing read or write operations against that
resource. Read operations have three outcomes:

```hare
// Reads up to len(buf) bytes from a [[handle]] into the given buffer, returning
// the number of bytes read.
fn read(
        h: handle,
        buf: []u8,
) (size | EOF | error);
```

Reads may return the number of bytes read (which may be *less* than the size of
the provided buffer), an end-of-file condition (indicating there is no further
data to read; this is not considered an error), or an error.

Write operations have two outcomes; writing some number of bytes and returning
that number (which also may be less than the amount provided in the buffer), or
an error.

Unless otherwise documented by the interface that provides an [io::handle], the
caller should *close* the resource once they are finished with it via
[io::close]. In the case of io::files, this operation generally closes the
underlying file descriptor and the host operating system will clean up state
associated with the file. In the case of io::streams, the close operation will
perform domain-specific clean-up associated with the stream, such as freeing any
memory that was allocated for the stream's operation. Closing a file can fail,
but generally only in the case of programmer error, thus the use of `!` to
assert errors on io::close is common (as seen in the sample above).

Most systems are resource-limited in how many files they can have open at once,
failing to close files and manage file lifetimes properly will exhaust this
resource and lead to program failure. If appropriate to your use-case, it is
recommended to `defer io::close(object)!` shortly after creating the resource.

Additional support is provided for [io::file] objects to integrate with the host
operating system, such as the use of [io::mmap] for memory-mapped I/O, or
[io::readv] et al for vectored I/O. Additional support for system features that
utilize file descriptors makes use of io::file throughout the standard library.

[io::mmap]: https://docs.harelang.org/io#mmap
[io::readv]: https://docs.harelang.org/io#readv

### Access to the host filesystem

Access to the host filesystem is provided by the [os] module, through functions
like [os::open]. os::open accepts a set of [flags][fs::flag] to tune the
operation, some of which are not implemented by all platforms, such as
flag::APPEND to open files in append mode. [os::create] is also provided to
create new files and accepts an additional parameter to specify the desired file
mode; [fs::mode] is provided to make the construction of the desired mode, and
interpretation of file modes, easier.

[os]: https://docs.harelang.org/os
[os::create]: https://docs.harelang.org/os#create
[fs::flag]: https://docs.harelang.org/fs::flag
[fs::mode]: https://docs.harelang.org/fs::flag

Manipulating the filesystem itself is also supported:

* [os::chmod], [os::chown]: change access and ownership information for a file
* [os::iter]: efficiently iterate over entries in a directory
* [os::mkdir]: create a new directory
* [os::move]: move a file from one location to another
* [os::rename]: rename a file
* [os::stat]: retrieve details about a file, such as ownership and type

[os::chmod]: https://docs.harelang.org/os#chmod
[os::chown]: https://docs.harelang.org/os#chown
[os::iter]: https://docs.harelang.org/os#iter
[os::mkdir]: https://docs.harelang.org/os#mkdir
[os::move]: https://docs.harelang.org/os#move
[os::rename]: https://docs.harelang.org/os#rename
[os::stat]: https://docs.harelang.org/os#stat

These functions are all designed to be portable, but access to Unix-specific
functionality is also provided by this module, including:

* The use of directory file descriptors
* Manipulating symbolic and hard links
* Working with block device, character device, and FIFO nodes

The use of Unix-style non-blocking I/O is supported, though generally advised
against in favor of I/O multiplexing (described below). To use Unix-style
non-blocking I/O, open a file with fs::flag::NONBLOCK and test for
[errors::again] to detect I/O operations that would otherwise have blocked.

[errors::again]: https://docs.harelang.org/errors#again

### Buffered I/O

Many I/O operations are most efficient when performed in bulk, by reading or
writing large amounts of data at a time. However, it is often much more
convenient to write your code with many small reads or writes rather than
buffering it yourself to perform I/O in large batches.

The "[bufio]" module aims to make the process of buffering I/O operations into
fewer, larger operations easier. This module also includes a number of tools for
working with buffers and I/O efficiently, for example efficiently scanning lines
of text from an input.

[bufio::init] accepts an arbitrary I/O handle, as well as a buffer for reads and
a buffer for writes (one can omit either for a read-only or write-only stream),
and then will batch reads and writes using these buffers. The following example
illustrates its use and the performance advantages of the approach; comment out
the "bufio::init" line to see the difference in performance.

```hare
use bufio;
use fmt;
use fs;
use io;
use os;
use time;

export fn main() void = {
	const input = match (os::open(os::args[1])) {
	case let file: io::file =>
		yield file;
	case let err: fs::error =>
		fmt::fatalf("Error opening {}: {}",
			os::args[1], fs::strerror(err));
	};
	defer io::close(input)!;

	// Create a buffered stream
	let rdbuf: [os::BUFSZ]u8 = [0...];
	let input = &bufio::init(input, rdbuf, []);

	const start = time::now(time::clock::MONOTONIC);

	// Read entire file one byte at a time
	let buf: [1]u8 = [0];
	for (!(io::read(input, buf)! is io::EOF)) void;

	const stop = time::now(time::clock::MONOTONIC);
	const elapsed = time::diff(start, stop);
	const sec = elapsed / time::SECOND;
	const nsec = elapsed % time::SECOND;
	fmt::printfln("Took {}.{:09}s to read file", sec, nsec)!;
};
```

The standard file descriptors [os::stdin] and [os::stdout] are buffered. You can
access the underlying files via [os::stdin_file] and [os::stdout_file]. stderr
is unbuffered.

[bufio]: https://docs.harelang.org/bufio
[bufio::init]: https://docs.harelang.org/bufio#init
[os::stdin]: https://docs.harelang.org/os#stdin
[os::stdout]: https://docs.harelang.org/os#stdout
[os::stdin_file]: https://docs.harelang.org/os#stdin_file
[os::stdout_file]: https://docs.harelang.org/os#stdout_file

#### Token scanners

The bufio module also provides a "scanner", which scans an input stream for
certain kinds of tokens, such as new lines, and internally manages a buffer to
batch smaller reads into fewer I/O operations. You can create a new scanner with
[bufio::newscanner], then use the various scanner functions, such as:

* [bufio::scan_byte]: reads a single byte from the input
* [bufio::scan_rune]: reads a single UTF-8 rune from the input
* [bufio::scan_bytes]: reads a number of bytes up to a provided delimiter
* [bufio::scan_string]: reads a UTF-8 string up to a provided delimiter
* [bufio::scan_line]: reads a UTF-8 string up the next newline

[bufio::newscanner]: https://docs.harelang.org/bufio#newscanner
[bufio::newscanner_static]: https://docs.harelang.org/bufio#newscanner_static
[bufio::scan_byte]: https://docs.harelang.org/bufio#scan_byte
[bufio::scan_rune]: https://docs.harelang.org/bufio#scan_rune
[bufio::scan_bytes]: https://docs.harelang.org/bufio#scan_bytes
[bufio::scan_string]: https://docs.harelang.org/bufio#scan_string
[bufio::scan_line]: https://docs.harelang.org/bufio#scan_line
[bufio::scan_buffer]: https://docs.harelang.org/bufio#scan_buffer

The scanner can be configured to allocate and resize its own internal buffers,
up to a limit specified in the [bufio::newscanner] call, or you can supply your
own fixed-size buffer with [bufio::newscanner_static].

Note that the scanner reads data from the underlying source *ahead* of the last
value returned from each of the `scan_*` calls, so if you abandon the scanner
and resume reading directly from the underlying file, you will miss any data
which was read ahead. To mitigate this, you can access the read-ahead buffer via
[bufio::scan_buffer].

Here is an example program which efficiently reads lines of text from a file and
numbers them:

```hare
use bufio;
use fmt;
use fs;
use io;
use os;
use types;

export fn main() void = {
	const input = match (os::open(os::args[1])) {
	case let file: io::file =>
		yield file;
	case let err: fs::error =>
		fmt::fatalf("Error opening {}: {}",
			os::args[1], fs::strerror(err));
	};
	defer io::close(input)!;

	const scan = bufio::newscanner(input, types::SIZE_MAX);
	for (let i = 1u; true; i += 1) {
		const line = match (bufio::scan_line(&scan)!) {
		case io::EOF =>
			break;
		case let line: const str =>
			yield line;
		};
		fmt::printfln("{}\t{}", i, line)!;
	};
};
```

### Memory I/O

It is often useful to use I/O operations to work with buffers of data. For
instance, one might wish to prepare a `[]u8` or a `str` for some operation by
using [io::write], [fmt::fprintf], etc. You may have a buffer of data from some
source as a `[]u8` and wish to pass it to functions which use I/O semantics;
imagine you have a tarball in a memory buffer and wish to process it with
[format::tar]. The [memio] module is designed to facilitate these use-cases.

The two main entry points to this module are [memio::dynamic] and
[memio::fixed], which respectively create an [io::stream] which performs reads
and writes against an internally managed, dynamically allocated buffer and a
user-managed fixed-length buffer. One can obtain the buffer as a `[]u8` with
[memio::buffer] or as a string with [memio::string].

[fmt::fprintf]: https://docs.harelang.org/fmt#fprintf
[format::tar]: https://docs.harelang.org/format/ini
[memio]: https://docs.harelang.org/memio
[memio::fixed]: https://docs.harelang.org/memio#fixed
[memio::dynamic]: https://docs.harelang.org/memio#dynamic
[memio::buffer]: https://docs.harelang.org/memio#buffer
[memio::string]: https://docs.harelang.org/memio#string

```hare
use fmt;
use io;
use memio;

export fn main() void = {
	const sink = &memio::dynamic();
	defer io::close(sink)!; // Frees the underlying buffer

	const username = "Drew";
	fmt::fprint(sink, "Hello, ")!;
	fmt::fprint(sink, username)!;
	fmt::fprint(sink, "!")!;

	fmt::println(memio::string(sink)!)!;
};
```

memio does not hold any dynamically allocated state aside from the buffer
itself, so you can skip [io::close] and use [memio::buffer] to claim ownership
of the buffer (freeing it yourself later with `free()`) without leaking memory.

### I/O multiplexing

If you have several sources of I/O to read from or write to, you may wish to
know which operations can be performed without blocking. Programmers familiar
with NONBLOCK usage on Unix systems can take advantage of it as described in [an
earlier section](#access-to-the-host-filesystem), but the recommended approach
on Unix uses [unix::poll], which is a wrapper around the portable
[poll](https://pubs.opengroup.org/onlinepubs/9699919799/functions/poll.html)
syscall. Be aware that this module only works with [io::file], rather than
[io::stream].

[unix::poll]: https://docs.harelang.org/unix/poll

If you are familiar with the syscall you will already have a generally strong
understanding of the usage of the Hare module. Otherwise, examples of unix::poll
usage will be covered in the networking section later in this tutorial.

For more complex use-cases (those covered by non-portable tools such as epoll(2)
on Linux or kqueue on \*BSD), see the extended library [hare-ev] project, which
provides more comprehensive event loop support.

[hare-ev]: https://git.sr.ht/~sircmpwn/hare-ev

### Custom I/O streams

It is often useful to create custom implementations of the I/O abstraction,
writing I/O objects that provide your own implementations of read, write, etc,
which can be passed into any function that expects an I/O object. [io::stream]
is provided for this use, which is used to implement many userspace I/O
operations throughout the standard library, but which can also be used in your
own code to implement custom streams.

One must define the implementation using an [io::vtable], filling in whichever
I/O operations you wish to support, then place an io::stream (initialized as a
pointer to this table) at the start of an object to create a custom stream. You
can fill in the remainder of the object with your custom state.

A simple illustrative example of such a stream is provided by the standard
library's [io::limitreader], which only allows a user-defined number of bytes to
be read from an underlying source of input. The implementation is concise:

[io::vtable]: https://docs.harelang.org/io#vtable
[io::limitreader]: https://docs.harelang.org/io#vtable
[io::limitwriter]: https://docs.harelang.org/io#writer

```hare
export type limitstream = struct {
	vtable: stream,
	source: handle,
	limit: size,
};

const limit_vtable_reader: vtable = vtable {
	reader = &limit_read,
	...
};

// Create an overlay stream that only allows a limited amount of bytes to be
// read from the underlying stream. This stream does not need to be closed, and
// closing it does not close the underlying stream. Reading any data beyond the
// given limit causes the reader to return [[EOF]].
export fn limitreader(source: handle, limit: size) limitstream = {
	return limitstream {
		vtable = &limit_vtable_reader,
		source = source,
		limit = limit,
	};
};

fn limit_read(s: *stream, buf: []u8) (size | EOF | error) = {
	let stream = s: *limitstream;
	if (stream.limit == 0) {
		return EOF;
	};
	if (len(buf) > stream.limit) {
		buf = buf[..stream.limit];
	};
	match (read(stream.source, buf)) {
	case EOF =>
		return EOF;
	case let z: size =>
		stream.limit -= z;
		return z;
	};
};
```

### I/O utilities

[io::limitreader] is an example of a simple I/O utility provided by the standard
library to facilitate common I/O usage scenarios. [io::limitwriter] is similar.
Additional useful utilities provided include:

* [io::tee]: a stream which copies reads and writes from an underling source to a secondary handle
* [io::empty]: a stream which always reads EOF and discards writes
* [io::zero]: a stream which always reads zeroes and discards writes
* [io::drain]: reads an entire I/O object into a []u8 slice
* [io::readall] & [io::writeall]: reads or writes an entire buffer, without underreads

[io::tee]: https://docs.harelang.org/io#tee
[io::empty]: https://docs.harelang.org/io#empty
[io::zero]: https://docs.harelang.org/io#zero
[io::drain]: https://docs.harelang.org/io#drain
[io::readall]: https://docs.harelang.org/io#readall
[io::writeall]: https://docs.harelang.org/io#writeall

## Working with strings

### Formatting text

### String manipulation

### Converting to and from strings

### Efficient string I/O

### base64, base32, and hex

## More filesystem utilities

### Using user directories

### Using temporary files and directories

### Working with paths

The [path](https://docs.harelang.org/path) module provides utilities for
normalizing and modifying filesystem paths. The paradigm of this module
is centered around the [path::buffer](https://docs.harelang.org/path), which
represents a normalized path. A path buffer can be converted to or from a
string at any time, but the advantage of using a buffer is that it is
mutable, and none of the functions in the path module will ever perform
heap allocation (unlike many string manipulations). Additionally, the path
buffer will ensure that the right path separators for the system are used, and
that the buffer does not exceed the system's maximum path length.

Most effective use of the path module involves creating one buffer and
passing around a pointer to that buffer, only converting it to a string when a
string is required. This prevents excessive copying and re-normalizing of the
buffer.

Below is a simple recursive filetree traversal program.

```hare
use fmt;
use fs;
use os;
use path;

fn walk(buf: *path::buffer) void = {
	let iter = os::iter(path::string(buf))!;
	defer os::finish(iter);
	for (true) match (fs::next(iter)) {
	case void => break;
	case let d: fs::dirent =>
		if (d.name == "." || d.name == "..") continue;
		path::push(buf, d.name)!;
		fmt::println(path::string(buf))!;
		if (fs::isdir(d.ftype)) walk(buf);
		path::pop(buf);
	};
};

export fn main() void = {
	const root = if (len(os::args) <= 1) "." else os::args[1];
	let buf = path::init(root)!;
	walk(&buf);
};
```

## Handling command line arguments

### getopt

## Executing other programs

### os::exec basics

### Waiting on children

### Setting environment variables

### Pipes and file descriptors

## Sorted slices

### Sorting a slice

### Working with sorted slices

## Regular expressions

### Working with POSIX ERE

## Networking

### IP addresses

### TCP and Unix sockets

### UDP support

### net::dial

## Date and time

### Basic timekeeping

### Working with time zones and chronologies

### Calendars

### Formatting and parsing

## Cryptography

### Encrypting and decrypting data

### Signing and validation

### Key derivation

### Low-level cryptographic primitives
