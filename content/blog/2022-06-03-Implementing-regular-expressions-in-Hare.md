---
title: Implementing regular expressions in Hare
author: Vlad-Stefan Harbuz
date: 2022-06-03
---

Regular expressions are one of those tools that we kind of take for granted.
They're really powerful and useful in so many situations, but most people don't
quite understand how they work under the hood. I'm Vlad, the developer who
implemented Hare's regular expression engine, and I thought I'd show you why
regular expressions aren't as scary as they might seem at first sight. In this
post, I'd like to take a look at how Hare's implementation works, as well as how
an end-user would go about using it. Let's get started.

## Which flavour of regex?

The first thing I had to decide when implementing regular expressions was which
features are appropriate for Hare and which are not. As you may know, there is
a basic "core" of regular expression features which are generally defined by the
POSIX standard, such as the `*+?` operators. Each implementation (Perl, Python
etc.) adds various different features of its own on top, such as backreferences
and so on.

Because Hare is a language focused on simplicity, and because I wanted to
implement something easy to understand and extend, I decided that sticking to
POSIX's Extended Regular Expressions would be the best option. The name is a bit
misleading, as there isn't anything really "extended" about them in practical
terms. Rather, this is the "core" set of regex features I was referring to.

In simple terms, this means we support the following:

* `.` for any character
* character classes with `[abc]`, `[^a-c]`, `[[:digit:]]` etc.
* the `*`, `+`, `?` operators
* the `^` and `$` anchors
* subexpressions with `()`
* alternation with `(a|b)`
* repetitions with `a{3,5}`

Supporting all combinations of these features gives us a fully functional
regular expression engine! We don't support any other vendor extensions, such as
backreferences or the like, even though enhancements are definitely welcome in
the extended standard library as well as third party packages. If you need a
more detailed explanation, check the [POSIX specification][erespec], which is
also summarized in this [wikibooks][erewiki] page.

## Representing regular expressions

Since we're writing a regex engine, we need to represent the
expressions in some kind of form. Because these expressions are made up of a
sequence of operations, it makes a lot of sense to represent them as [finite
state automata][fsm]. "Automata" are something we can execute, and "finite
state" means that this execution is made up of a finite set of steps.

Let's look at a simple example that should definitely clear things up for you.
Here's the finite state automaton for the expression `/(ab)+c/`. I've put
slashes around regexes in this post for clarity.

```
/(ab)+c/

   a     b     c
s1 ─► s2 ─► s3 ─► end
      ▲     │
      └─────┘
         a
```

`s1`, `s2` and `s3` are different states the automaton can be in. We start in
`s1`. Our only option is to move to `s2` by reading an `"a"`, which we signify
with a labelled arrow. From `s2`, we can only read a `"b"`. Afterwards, we can
either continue to read an `"ab"` sequence, or eventually just read a `"c"` and
terminate the match. If you work through this yourself, you can see that this
automaton will easily match a string such as `"abababc"`.

I won't go into the details of how one converts a regular expression to an
automaton. If you'd like to read up about this topic in more detail, [Russ Cox's
articles][swtch] are the place to do so, as the approach I'm describing here is
heavily based on his work. However, I've tried to simplify the strategies he
describes in order to make them as simple as possible to work with, which is one
of the reasons the entire `regex.ha` implementation is only around 800 lines for
a fully-functional POSIX-compatible engine.

## Checking regular expressions

Now that we have represented our regex in some way, we need to decide how we're
going to test whether a certain string matches it, for example whether `"abbb"`
matches `/(abab|abbb)/`. First of all, let's look at the representation of this
latter expression.

```
/(abab|abbb)/

              a     b     a     b
       ┌─► s1 ─► s2 ─► s3 ─► s4 ──┐
start ─┤                          ├─► end
       └─► s5 ─► s6 ─► s7 ─► s8 ──┘
              a     b     b     b
```

You'll notice that there are two possible branches at the start. This makes our
automaton an NFA, or a [nondeterministic finite automaton][nfa]. All this
basically means is that there are certain points in this automaton where you
don't really know which path you should take, such as our start branch, hence
the "nondeterministic" part.

A simple way of checking would be to try each branch, one at a time. We could
test the top branch, by going `start -> s1 -> s2 -> s3 -> s4`. Once we see that
that branch doesn't match, we could _backtrack_ and try the bottom match, by
going `start -> s5 -> s6 -> s7 -> s8` and seeing that we get a match.

This backtracking-based approach works, but it comes with a problem. The number
of branches we need to test can exponentially increase with the size of the
regex in certain situations, which means that testing all the branches would
potentially take forever. This is unfortunately what happens in many regular
expression engines that have this problem, such as those in Perl, Python and
most other languages.

There's no need to despair, though, as there's a better way. We can simply test
all branches _at the same time_. This means we simply need to take each
character of the string we're testing, and step forward through our branches by
the appropriate amount. Here's what it would look like for the expression
`/(abab|abbb)/` and the test string `"abbb"`. Try to follow along with the above
diagram.

* We begin in states `s1` and `s5` simultaneously
* We read `"a"` from the test string and are able to advance to `s2` on the top
  branch and `s6` on the bottom branch
* We read `"b"` from the test string and are able to advance to `s3` on the top
  branch and `s7` on the bottom branch
* We read `"b"` from the test string and _are unable to advance further_ on the
  top branch, but advance to `s8` on the bottom branch
* We only have the bottom branch left, we read `"b"` and advance to the end

This approach means we only need to do an amount of tests that's directly
proportional to the size of the string we're testing against. There's no
possibility for exponential growth, and so our algorithm is said to guarantee to
match in linear time. It's also not that different to implement. Big win!
Incidentally, this is the approach used in grep and awk.

## The virtual machine approach

You might be wondering how this simultaneous testing approach is actually
implemented in the code. There's another small trick that helps us out a lot
here, and that's implementing each part of the regex as an "instruction",
treating the whole regex as a virtual machine.

To avoid going into too many details, here's a quick example to show you what I
mean. Let's look at how we would compile the expression `/ab*/`.

```
0: literal(a)
1: branch(4)
2: literal(b)
3: jump(1)
4: match
```

You can see we've compiled the regex into a set of instructions. Here's the
meaning of each instruction:

* `literal(x)`: Eat character `"x"` and move on
* `branch(n)`: Create a new branch starting from instruction `n`, which executes
  _in parallel_ with the original branch
* `jump(n)`: Go to instruction `n`
* `match`: Declare we've successfully matched the string

The branch part is important. After running `1: branch(4)` above, we will have
_two execution threads_ instead of one, running simultaneously.[^1] This means
that the next step will see us running one thread at the next instruction
(`2:`), and another parallel thread at the instruction we branched to (`4:`).

[^1]: These "threads" are switched cooperatively &mdash; they aren't implemented with threads at the operating system level.

Therefore, if you were to run the string `"a"` through this "program", the
successful thread would execute in this order: `0 → 1 → 4`. On the other hand,
the successful thread matching string "abb" would execute as
`0 → 1 → 2 → 3 → 1 → 2 → 3 → 1 → 4`.

There are other details and optimisations, such as merging two threads into one
when they reach the same instruction, but that's the general idea. So what does
this approach help us with? Well, two things.

Firstly, it allows us to keep track of as many branches as we want, by allowing
us to launch multiple _execution threads_ that run in parallel. Each thread only
needs to keep track of what would normally be called its "program counter",
which basically just means "which instruction is going to be executed next".

Secondly, the instruction-based approach means that, whenever we want to add a
new feature to our regex engine, we just add a new instruction! For example,
there's a `repeat` instruction in `regex.ha` that implements the `a{n,m}`
repetition operator.

And honestly, that's basically it. You should be well-equipped to read and
understand [regex.ha][regex.ha] now, if you were so inclined. Next, let's
fast-forward to the completed implementation and look at how to actually use the
finished product in Hare.

## How to use regex:: in Hare

The most important functions in [regex::][docs] are:

* `fn compile(expr: str) (regex | error)` compiles a string into a `regex`
* `fn find(re: *regex, string: str) result` returns the first match for the
  regex in the given string
* `fn findall(re: *regex, string: str) []result` returns all matches for the
  regex in the given string
* `fn test(re: *regex, string: str) bool` returns whether or not the given
  string matches the regex
* `fn replace(re: *regex, string: str, targetstr: str) (str | error)` returns a
  string with all non-overlapping matches of a regex replaced with `targetstr`,
  supporting escape sequences for specifying captures
* `fn rawreplace(re: *regex, string: str, targetstr: str) str` returns a string
  with all non-overlapping matches of a regex replaced with `targetstr`

The only thing that might not be fully clear is: what's a `capture`? It's
extremely straightforward to understand with an example.

**Tip**: It's helpful to use ``` `raw strings` ``` for specifying regular
expressions, so that you can freely type backslashes without them being
interpreted as escape sequences.

```
const re = regex::compile(`hello(there)`)!;
const captures = regex::find(&re, "hellothere") as []regex::capture;
```

This will give you two captures:

```
capture 0:
	content = "hellothere"
	start = 0
	end = 10
capture 1:
	content = "there"
	start = 5
	end = 10
```

This makes things obvious: the first capture is always the full match, while
subsequent captures are the submatches created by `(capture groups)`.

Here's a more complete example, which shows how to use the `findall()` function
to find all matches, as well as how to manage memory and handle errors.

```
const matches = regex::findall(&re, "Hello Hare, hello Hare.");
if (len(matches) > 0) {
	defer regex::free_matches(matches);
	// matches[0]: All captures for the first match.
	// matches[0][0]: The full matching string for the first match.
	// matches[0][1...]: A capture for every capture group in the
	//     first match.
	for (let i = 0z; i < len(matches); i += 1) {
		fmt::printfln("{} ({}, {})", matches[i][0].content,
			matches[i][0].start,
			matches[i][0].end)!;
	};
};
```

The thing to keep in mind here is that `capture`s need to be freed, and
`regex::` provides convenience functions such as `free_matches()` to help you do
so.

## Closing remarks

With that, we've looked at the most important aspects of regular expressions in
Hare. Building this regex engine was a really rewarding project for me, and I
hope you got something interesting out of this post.

If you'd like to have a chat or have any questions, you can find me [on
IRC][community] or send me an email at vlad@vladh.net. Take care and have fun
programming!

[erewiki]: https://en.wikibooks.org/wiki/Regular_Expressions/POSIX-Extended_Regular_Expressions
[erespec]: https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap09.html#tag_09_04
[fsm]: https://en.wikipedia.org/wiki/Finite-state_machine
[nfa]: https://en.wikipedia.org/wiki/Nondeterministic_finite_automaton
[regex.ha]: https://git.sr.ht/~sircmpwn/hare/tree/master/item/regex/regex.ha
[docs]: https://docs.harelang.org/regex
[swtch]: https://swtch.com/~rsc/regexp/regexp1.html
[community]: https://harelang.org/community/
