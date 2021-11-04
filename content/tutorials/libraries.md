---
title: Using and writing Hare libraries
summary: |
  This tutorial will introduce you to the Hare approach to dependency
  management, explain the various ways to use libraries for your project, and
  how to package your own libraries for re-use.
---

TODO: Docs for making new libraries

## Using Hare libraries

Hare libraries live in the same namespace as the standard library, and to some
extent, libraries are encouraged to extend the standard library where it makes
sense to. For example, a library may add a new network protocol by adding a new
namespace in `net::`, such as `net::irc`. Additionally, third-party modules can
entirely replace standard library modules with their own (though they are
expected to provide a compatible API). Some parts of the standard library are
deliberately simplified, such as `format::xml`, and "shadowing" them with a more
feature-complete third-party library is a useful pattern.

When you use an import statement, such as `use os::exec`, Hare will search the
"HAREPATH" for candidates. You can view the default HAREPATH for your system
with the `hare version -v` command, or override it by defining a HAREPATH
environment variable with a colon (":") separated list of search paths. The
default search path is generally, in order of preference:

1. `.` (the current working directory)
1. `./vendor` (vendored modules)
1. `/usr/local/src/hare/stdlib` (local standard library)
1. `/usr/local/src/hare/third-party` (local third-party modules)
1. `/usr/src/hare/stdlib` (distribution standard library)
1. `/usr/src/hare/third-party` (distribution modules)

Hare modules are distributed in source form and can be found in these
directories. Importing `os::exec` will check `./os/exec/`, then
`./vendor/os/exec`, and so on. The full algorithm for module resolution is
described by hare(1) man page.

You are strongly advised to add dependencies with caution. Hare is not an
ecosystem where every little piece of functionality can be added with a separate
library. Dependencies are important, and you should evaluate each one carefully
to make sure that it is suitable for your needs, that you can trust the
maintainers, and that you can understand it should you need to debug it. Each
dependency is a liability as well as an asset. A good Hare program will
typically have fewer than 10 dependencies &mdash; often much fewer.

## Library installation

Hare does not have a package manager, by design. There are four ways to install
a Hare package, each for different use-cases:

1. System-wide (`/usr/src` or equivalent) with your distribution's package manager
2. System-wide (`/usr/local/src` or equivalent) with `make install`
3. Vendored into your project (`./vendor`) with `git subtree`

Generally, you should prefer to install the system package if one is available.
Installing into `/usr/local` is a good alternative if not, but it requires admin
access. You may install packages without special permissions by installing them
in `~/.local/src`. Additionally, under certain circumstances, it may be
desirable to "vendor" the package into your project's repository.

### Distribution packages

The recommended naming convention for Hare packages is "hare-$pkgname", e.g.
"hare-irc", but your distribution's conventions may vary. Locate the appropriate
package and install it according to your package manager's usual means. Reach
out to the maintainer of your package distribution for help if you have issues
with these packages.

### make install

To install a library to your entire system without your distribution's package
manager (*not recommended*), obtain the module source code (e.g. via git clone)
and run `make install` as root from the source directory. You can remove it by
running `make uninstall` as root.

### Vendoring modules

Modules can also be copied directly into your project. One good reason to do
this is if you need to edit the module to apply a patch which has not been
accepted (or is not acceptable) upstream. This is done by copying the module
into a directory in `./vendor`. Many Hare modules use git, so [git subtree][0]
is recommended for this purpose. The following command will install the
[hare-irc][1] module into your vendor directory:

[0]: https://manpages.debian.org/testing/git-man/git-subtree.1.en.html
[1]: https://git.sr.ht/~sircmpwn/hare-irc

```
git subtree -P vendor/hare-irc/ add https://git.sr.ht/~sircmpwn/hare-irc master
```

You can update it later like so:

```
git subtree -P vendor/hare-irc/ pull https://git.sr.ht/~sircmpwn/hare-irc master
```

You can also vendor modules from the standard library by simply copying them:

```
mkdir -p vendor/os/exec/
cp -R /usr/src/hare/stdlib/os/exec/* vendor/os/exec/
```

**You are strongly advised to read the license of vendored projects**. The Hare
standard library, for example, uses the Mozilla Public License, which *requires*
you to release any changes you make to standard library files the same Mozilla
Public License. Other licenses may have similar requirements. Read them!
