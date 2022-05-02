---
title: Will Hare replace C? Or Rust? Or Zig? Or anything else?
author: Drew DeVault
date: 2022-05-02
---

Nope.

If the short answer leaves you wanting, keep reading. I must admit that this
sentence from the [Hare announcement] may have caused some confusion:

[Hare announcement]: https://harelang.org/blog/2022-04-25-announcing-hare/

> Hare is most similar to C, and almost all programs written in C can also be
> written in Hare.

This is especially true in the context of the rising and competing compliment of
"C replacement" languages, of which some are explicit about their goals to
replace C, such as Rust. There is a movement to rewrite existing C code in Rust
and by such means take over the world, mwahahaha. Hare is not interested in
taking over the world, and I do not recommend that programmers start rewriting
their C programs in Hare.

Hare aims to be successful within its niche for the programmers that find its
ideas compelling, and nothing further. If you are using and enjoying C, C++,
Rust, Zig, or any other language, and don't find Hare's ideas all that
interesting, then I encourage you to keep using these languages.

We designed Hare to be similar to C, and useful everywhere C is useful, because
we want to be able to write the kinds of programs we like writing in C with
Hare. Many of us are still C programmers, especially given that C is planned to
be a part of the bootstrap path for Hare indefinitely. We don't need to
"replace" C to achieve our goals, however. Even in the best case "replace C"
scenario, new code is written in Hare while code written in C is slowly made
obsolete by the march of time &mdash; ideally being actively maintained until
it's no longer needed.

In spite of its strengths, Hare is ruled out for many use-cases. It lacks
multithreading in the standard library, only supports free software platforms,
and while new ports are easy, they still have to be written in the first place
&mdash; it will likely be decades before Hare starts to approach C's catalogue
of supported targets. I also generally agree that Rust is probably the better
choice for high-stakes use-cases such as life-critical software, or else
leveraging other techniques like formal verification or independent audits.

Part of our work in developing Hare is laying the groundwork for a
collaborative, productive, healthy community that people want to work in,
which is something that I think some other languages have done a poor job of. I
strongly advise members of the growing Hare community to avoid forming a "Hare
Evangelism Strike Force" that goes out to the rest of the ecosystem and makes a
case for rewriting things in Hare.

I was pretty frustrated to see the "Hare is a C replacement" mantra repeated in
the media despite issuing no such claims. I am even more frustrated with the
moral crusaders from languages like Rust, one of whom went as far as to suggest
that I should personally be criminally prosecuted if some downstream Hare
software has a use-after-free bug. My goal is not to force anyone who doesn't
like Hare to use it, or issue judgements upon projects which choose another
language. In return, I will be pleased if members of other language communities
refrain from flaming *too* much on Hare.

A healthy ecosystem is a diverse ecosystem, and I hope Hare contributes to
diversifying it a little bit more. If you like your current language, do your
part for that diversity by continuing to use it.

That said, if you do find Hare interesting, and want to learn more about it,
please check out the [language introduction], and join us for the upcoming [Hare
talk] at Techinc in Amsterdam, in person or virtually. I'm looking forward to
working together on our shared goals for software 🙂

[language introduction]: https://harelang.org/tutorials/introduction/
[Hare talk]: https://drewdevault.com/talks/hare.html

---

This post (hopefully) addresses one of two major issues that emerged from the
discussions following the Hare announcement. I will do another post which
addresses the other question: memory safety.
