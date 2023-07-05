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

### I/O multiplexing

### Custom I/O streams

### I/O utilities

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
