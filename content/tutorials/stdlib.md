---
# TODO: This tutorial probably does not work well in the codetutorials template,
# should just be structured normally + table of contents
title: An introduction to the Hare standard library
type: codetutorials
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

  [language introduction tutorial]: /tutorials/introduction/
sections:
- section: Input and output
- title: "Hare's I/O abstraction"
- title: "Hare's filesystem abstraction"
- title: "Host I/O primitives"
- title: "Host filesystem primitives"
- title: "Buffered I/O"
- title: "I/O multiplexing"
- title: "Custom I/O interfaces"
- section: Working with strings
- title: "Formatting text"
- title: "String manipulation"
- title: "Converting to and from strings"
- title: "Efficient string I/O"
- title: "base64, base32, and hex"
- section: More filesystem utilities
- title: "Using user directories"
- title: "Using temporary files and directories"
- title: "Working with paths"
- section: "Handling command line arguments"
- title: "getopt"
- section: Executing other programs
- title: "os::exec basics"
- title: Waiting on children
- title: Setting environment variables
- title: Pipes and file descriptors
- section: Sorted slices
- title: Sorting a slice
- title: Working with sorted slices
- section: Regular expressions
- title: Working with POSIX ERE
- section: Networking
- title: IP addresses
- title: TCP and Unix sockets
- title: UDP support
- title: "net::dial"
- section: Date and time
- title: Basic timekeeping
- title: Working with time zones and chronologies
- title: Calendars
- title: Formatting and parsing
- section: Cryptography
- title: Encrypting and decrypting data
- title: Signing and validation
- title: Key derivation
- title: Low-level cryptographic primitives
---
