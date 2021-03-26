---
title: High-level data structures in Hare
date: 2021-03-26
author: Drew DeVault
---

Hare does not support generics, and our approach to data structures is much like
C: <abbr title="do-it-yourself">DIY</abbr>. Hare supports the following basic
data structures:

- Arrays
- Slices
- Tuples
- Structs and unions
- Tagged unions

That's the complete list of aggregate type classes defined by the specification.
You can build arbitrarily complex data structures out of these, but, like C,
Hare puts the ball in your court. For most of your application's day-to-day data
processing needs, these types are sufficient, shunting data from A to B without
much fuss. For most of your data processing, these limitations will not be your
bottleneck, and simpler (but slower) data structures won't have much of an
impact on your program.

But sometimes data structures *are* your bottleneck. Most programs are primarily
dealing with two or three important ideas that their data is modeled around.
For these, it's likely that the application of a higher-level data structure
will provide a meaningful impact to your program. Instead of providing such data
structures in the standard library (or even, through generics, in third-party
libraries), Hare leaves this work to you.

If the absence of a particular data structure truly is your application's
bottleneck, then writing it yourself may ultimately be the better approach.
You'll have to familiarize yourself with the data structures and algorithms that
manipulate them, so you can have an intimate understanding of the processes most
important to your application. You can also tune and tweak them to keep it lean
and mean within your use-case, only making them as complex as your application
calls for.

For example, let's look at the Hare build driver module cache. The build driver
is responsible for collecting input files, resolving their dependencies, and
producing a plan for compiling your program. Because one module could be
included several times by successive transitive dependencies, and we want to
avoid scheduling it to be built several times (that'd be O(n<sup>c</sup>),
yikes), we need a cache. A hash map is suitable for this. So we define the data
structures like so:

```hare
type plan = struct {
	// ...
	modmap: [64][]modcache,
};

type modcache = struct {
	hash: u32,
	task: *task,
	ident: ast::ident,
	version: module::version,
};
```

We keep this as simple as possible. If we had a more sophisticated use-case, we
may want to add dynamic re-hashing, and indeed, such a change would not be too
terribly difficult. 64 buckets is sufficient for this use-case, and waste not,
want not.

The standard library provides `hash::fnv`, which is a good algorithm to use to
produce hashes for our map. We're keying this map based on the module identifier
(`ast::ident`), so we need to write a function which hashes an identifier.

<!-- TODO: This code sample should be updated if/when we update fnv32() to avoid
allocating a hash structure on the heap -->

```hare
fn ident_hash(ident: ast::ident) u32 = {
	let hash = fnv::fnv32();
	defer hash::close(hash);
	for (let i = 0z; i < len(ident); i += 1) {
		hash::write(hash, strings::toutf8(ident[i]));
		hash::write(hash, [0]);
	};
	return fnv::sum32(hash);
};
```

Then, we need some code to fetch items from the map, or insert them if not
already present. For this, we simply take the hash of the identifier modulo the
length of our module cache to identify a bucket to use. Then we iterate over the
slice in that bucket until we find a matching module. If there's no matching
module, we schedule the object for the build and add it to the module cache for
next time.

```hare
fn sched_module(plan: *plan, ident: ast::ident, link: *[]*task) *task = {
	let hash = ident_hash(ident);
	let bucket = &plan.modmap[hash % len(plan.modmap)];
	for (let i = 0z; i < len(*bucket); i += 1) {
		if (bucket[i].hash == hash) {
			return bucket[i].task;
		};
	};
  
	// ...

	let obj = sched_hare_object(plan, ver, ident, depends...);
	append(*bucket, modcache {
		hash = hash,
		task = obj,
		ident = ident,
		version = ver,
	});
	return obj;
};
```

That's all there is to it. Hare doesn't provide us with a generic hash map, but
we were able to build one ourselves in just a few lines of code. A hash map is
one of the simpler data structures we could have shown here, but even for more
complex ones, you'll generally find that it's not too difficult to implement
them yourself in Hare. In exchange for a little more of your time, we can offer
you a simpler language, the experience and understanding that comes with
implementing the data structures yourself, and the advantages of an
implementation tuned precisely to your specific application's needs.
