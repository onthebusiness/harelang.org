---
title: Strategies for handling OOMs in Hare programs
date: 2021-04-28
author: Drew DeVault
---

For most programs, an OOM (out-of-memory) scenario is unlikely, and handling
them is annoying. For this reason, the "default" expectation for Hare programs
is that they will abort when the system runs out of memory. Your memory usage is
still your responsibility &mdash; and we provide tools for understanding it (to
be described in a future post) &mdash; but the typical program doesn't need to
worry about running out.

However, Hare is a systems programming language, and some systems do care. One
example of such a situation is a kernel: it's never acceptable to simply crash
the kernel if you run out of memory. So how does someone working in one of these
memory-critical domains use Hare effectively?

Hare provides a few language features to help you out here. First, let's start
with the obvious:

```hare
let x: *int = alloc(1337);          // Crashes on OOM
let x: nullable *int = alloc(1337); // Returns null on OOM
```

If you use `alloc` with a nullable pointer type, then an OOM condition will
return null instead of aborting. In order to use the new pointer, you have to
test for null first:

```hare
let x: nullable *int = alloc(1337); // Returns null on OOM
let x: *int = match (x) {
    x: *int => x,
    null => {
        // Handle OOM here
    },
};
```

Another situation which can cause issues is the use of `append` and `insert`
with slices. The following code will abort on OOM:

```hare
let x: []int = [];
append(x, 1337);    // Abort on OOM!
insert(x[0], 1337); // Abort on OOM!
```

We cannot save you from aborts if you run out of memory when appending to or
inserting values into a slice. But we do offer some tools to let you control
your memory more explicitly, such as *static* slice mutations.

```hare
let buf: [64]int = [0...];
let x = buf[..0];
static append(x, 1337);    // Will never alloc
static insert(x[0], 1337); // Will never alloc
```

This will still abort if you append, in this case, more than 64 items, but it
will never reallocate the buffer. You can use this to control the slice
allocation yourself &mdash; in this case, on the stack. However, you can still
put it on the heap, and deal with allocation failures there:

```hare
let buf: nullable *[64]int = alloc([0...]);
let buf = match (buf) {
    null => // Handle OOM
    buf: *[64]int => buf,
};

// ...

let x = buf[..0];
if (len(x) + 2 >= len(buf)) {
    // Handle OOM
};
static append(x, 1337);
static insert(x[0], 1337);
```

To avoid accidentally writing code which could abort in an OOM scenario, pass
the `-Fstrictoom` flag to harec to raise a compiler error if you use a
non-static append or allocate a non-nullable pointer. If you're writing a
kernel, you're probably providing your own `rt` implementation as well: just
skip `rt::ensure` and you'll also get linking errors when using dynamic slices.
I hope that helps!
