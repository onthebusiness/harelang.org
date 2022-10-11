---
title: Notes on third-party tools and Hare
---

This guide shows you how to best use common external tools for working
with Hare programs.

This page is under construction. Patches are welcome!

## Debugging

### gdb

## Profiling

### perf

### valgrind

[valgrind](https://valgrind.org) is a popular tool for examining memory
usage. Link with libc to make valgrind track malloc/free calls:

```
$ hare build -lc -o example example.ha
$ valgrind ./example
```

## Linking Hare programs with C libraries

## More?
