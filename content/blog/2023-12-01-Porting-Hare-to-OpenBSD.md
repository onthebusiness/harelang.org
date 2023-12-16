---
title: Porting Hare to OpenBSD
author: Lorenz (xha)
date: 2023-12-01
---

*OpenBSD support is the first new platform added to Hare since it was made
available to the public in April 2022, initially supporting Linux and FreeBSD.
Hare contributor Lorenz writes for the Hare blog today about how the port was
done.*

I was always very interested in OpenBSD and a few months ago, I decided to give
it a try. I've quickly fallen in love with it! There is, however, a big problem:
Hare does not fully support OpenBSD! So, I decided to port it and I am happy to
announce that my work was merged yesterday and OpenBSD is now fully supported by
Hare. Let me show you some of the tricky stuff that was involved in the port.

## Hare's two porting targets

Porting Hare involves porting two targets: the [compiler][0] and the [standard
library][1].

About a year ago, Dr. Brian Robert Callahan [ported the Hare compiler][2],
harec, to OpenBSD. The compiler, however, is not meant to be invoked directly
by the user; it transforms Hare code into an intermediate representation which
can be passed to the backend, QBE, but it is not a complete tool for building
Hare programs.

Hare programs are meant to be compiled using the Hare build driver. The build
driver then invokes harec, QBE, the assembler and linker for you. The build
driver uses the standard library, so you get it for "free" when porting the
stdlib to a new platform.

[0]: https://git.sr.ht/~sircmpwn/harec
[1]: https://git.sr.ht/~sircmpwn/hare
[2]: https://briancallahan.net/blog/20220427.html

## First step was already done

When I started my journey of porting the stdlib, of course I first searched the
mailing lists to see if someone already did the work. And as it turns out,
Lennart had [already ported the runtime][3] (low-level parts like syscalls) to
OpenBSD so i could use that as a starting point.

The first problem was, that well, it wouldn't work on my machine. As it turns
out, a CPU feature called Indirect Branch Tracking (IBT) was just enforced by
default in the latest OpenBSD development branch and I had a supported CPU.

In a Nutshell, IBT works by expecting endbr64 instructions everywhere you jump.
If you try to jump somewhere that doesn't point to a endbr64 instruction, your
program will get killed by the operating system. This prevents ROP-based
attacks, but is not supported by QBE.[^1]

[^1]: There is a similar feature for aarch64 processors called BTI, which is also
      enforced by default on OpenBSD.

We can tell the linker to create a `PT_OPENBSD_NOBTCFI` segment in the binary by
adding `-z nobtcfi` to the command line arguments. This will signal the kernel
that IBT should not be enforced. After this, Lennart's patch run's fine for me.
However, there is a problem...

[3]: https://lists.sr.ht/~sircmpwn/hare-dev/patches/42418

## OpenBSD doesn't like direct system calls

There is a special syscall on OpenBSD called [msyscall(2)][4] which restricts
from where system calls can be invoked. It can only be called once. If you try
to invoke syscalls from outside the given range, your program will get killed.
This is not a problem with statically linked programs (which is the default on
Linux and FreeBSD). However, the OpenBSD dynamic linker invokes this syscall
during startup:

```C
elf_object_t *
_dl_tryload_shlib(const char *libname, int type, int flags, int nodelete)
{
	[...]
	/* Request permission for system calls in libc.so's text segment */
	if (soname != NULL && !_dl_traceld &&
	    _dl_strncmp(soname, "libc.so.", 8) == 0) {
		if (_dl_msyscall(exec_start, exec_size) == -1)
			_dl_printf("msyscall %lx %lx error\n",
			    exec_start, exec_size);
        }
        [...]
}
```

As you can probably tell, this will call msyscall(2) when it finds the libc,
after which direct syscalls from Hare are no longer possible. As we don't want
to prevent people from linking with libc, and given OpenBSD's long-term plans to
reduce user-level access to syscalls, the best solution is to always link with
libc and use it for system calls. This unfortunately means that I had to re-do a
lot of Lennart's work.

To my surprise, it is really easy to implement libc syscall wrappers in Hare.
For example, here is the read(2) system call:

```hare
@symbol("read") fn libc_read(d: int, buf: *opaque, nbytes: size) size;

export fn read(fd: int, buf: *opaque, count: size) (size | errno) = {
        let res = libc_read(fd, buf, count);
        if (res == -1) {
                return *__errno(): errno;
        };
        return res;
};
```

The example is pretty straightforward: we forward-declare the `read` symbol from
libc. We then export the "read" function (ultimately assigned the symbol
`rt.read`, adding the rt namespace, in the final executable), which will run the
libc function and checks for errors. As a user of the stdlib you can then just
call rt::read() if you need a low-level interface.

Depending on libc has some other benefits, too, like being able to use the
OpenBSD allocator which is generally much more secure and feature-complete than
the Hare standard library's allocator.

With syscalls addressed, there is still one issue before we can run Hare
executables on OpenBSD...

[4]: https://man.openbsd.org/msyscall.2

## Linker scripts are complicated and undocumented

Hare uses linker scripts for placing symbols in different sections. For example,
there is a `.init_array` section which contains all functions that are going to
be ran when the binary is started. This is mostly used by the stdlib to setup
stuff before the main() function is going to be ran. The problem is that linker
scripts also override everything else, and on OpenBSD that means that some
special sections are missing which results in the kernel not being able to load
the executable. You can run the binary without linker scripts, however, this is
going to quickly segfault because stuff like os::getenv() is setup by the init
functions.

After days of debugging linker scripts, I gave up and looked into alternative
ways of running early Hare initialization: when you are linking with libc, you
also get something called crt0 which is basically setting up the environment
before the actual program is ran. As it turns out, crt0 also has a concept of
init functions which we can use in Hare.

Hare also has some special requirements for sourcing runtime information using
the System-V ABI. Since \_start is provided by libc in this configuration, we
can achieve this by putting the function that initializes the environment in the
`.preinit_array` section, and place normal init functions into the `.init_array`
section (generated by harec). In practice, the solution looks like this:

rt/+openbsd/start.s
```assembly
.section ".preinit_array"
.balign 8
.init.initfunc.0:
	.quad preinit_hare+0
```

rt/+openbsd/start.ha
```hare
export @symbol("preinit_hare") fn preinit_hare(
	c_argc: int,
	c_argv: *[*]*u8,
	c_envp: *[*]nullable *u8
) void = {
	argc = c_argc: size;
	argv = c_argv;
	envp = c_envp;
};
```

With this, we can get Hare working without using a linker script! Hare also
depends on linker scripts to populate a similar, Hare-specific array for tests
-- I was able to come up with a simple linker script that doesn't override
everything but is able to add the test_array:

rt/+openbsd/hare+test.sc
```lc
SECTIONS {
	.test_array : {
		PROVIDE(__test_array_start	= .);
		KEEP(*(.test_array*))
		PROVIDE(__test_array_end	= .);
	}
} INSERT AFTER .bss; /* .bss was choosen arbitrarily. */
```

## Bonus: ASLR works

On OpenBSD, since we are not using linker scripts, Hare code can get randomized
and ASLR is fully working. This is currently not the case on both Linux and
FreeBSD. I am happy that I could get it working and it's really great for
security overall.

## Closing thoughts

Although there is still some stuff that needs to be done, like the unveil(2),
pledge(2) and kqueue(2) system calls, most parts of the stdlib are implemented
and all tests are passing.

Overall, it was a lot of fun working on OpenBSD support for Hare, and I am
really happy that it got merged now. Hare is a rather simple language and that
makes it really fun for me to hack on. I am planning to write some exciting
projects for OpenBSD in Hare and continue to maintain and improve Hare in the
future.
