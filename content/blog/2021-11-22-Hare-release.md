---
title: 'Using "hare release" to ship Hare software'
author: Drew DeVault
date: 2021-11-22
---

The "hare" command now includes a new sub-command, "hare release", which can be
used to manage releases for software written in Hare. This tool checks for
common mistakes and establishes common packaging standards for use throughout
the Hare ecosystem, making releases easy, consistent, and [difficult to screw
up][0]. Let me explain how it works, and how to use it for your own releases.

**Note**: We've now gotten rid of hare release. This blog post is still here for
historical purposes, but it no longer reflects the current state of Hare.

[0]: https://drewdevault.com/2019/10/12/how-to-fuck-up-releases.html

First of all, this tool is totally optional. If you have a release process you
already like, or your project's release situation demands special attention, you
can skip this tool entirely. But, if you choose to use it, you can ship the
first version of your software by running `hare release 1.0.0`. If you want to
ship a pre-release, something like `hare release 0.0.1` may be more appropriate.

This process automatically runs a number of sanity checks to avoid common
mistakes, such as:

- Do you have any uncommitted changes?
- Do you have the correct branch checked out?
- Is your local branch behind the upstream branch?
- Does the test suite pass?

You may have also noticed that we prompted the user to generate a new SSH key.
If you already have an SSH key, this step will be skipped. This SSH key is used
to sign a generated .tar.gz file of the release, and store it in git as a "git
note", attached to the git tag. We chose to use SSH keys because they are more
ubiquitous among contemporary developers than PGP, which is also more complex
and difficult to use.

If you wish to verify this signature, create a "principals" file containing a
list of all users who are allowed to sign releases, based on the contents of
~/.ssh/id_ed25519.pub (or whatever key is appropriate). In my case, I can create
a file like this:

```
sir@cmpwn.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKRe0pYxNubFS3VryJzuIH6qSW248SOo/rSFHDVs1Ux1
```

I'm using my git committer email here, as well as the contents of my public key
file. I could also add several more keys, if there are multiple release managers
on this project. I can publish this file on our project website, or in the git
repository, or just let others write it themselves based on my automatically
published SSH keys at
[meta.sr.ht/~sircmpwn.keys](https://meta.sr.ht/~sircmpwn.keys) or
[github.com/ddevault.keys](https://github.com/ddevault.keys).

Once I have a principals file, I can extract the git note and compare it with
the archive:

```
$ GIT_NOTES_REF=refs/notes/signatures/tar.gz git notes show 1.0.0 > /tmp/sig
$ git archive --format=tar.gz --prefix=example-project-1.0.0/ 1.0.0 \
  | ssh-keygen -Y verify -n file -s /tmp/sig -f ~/.ssh/principals -I sir@cmpwn.com
Good "file" signature for sir@cmpwn.com with ED25519 key SHA256:5TPjHnA7eOseMBFSxylGAwg9KaTdofMEM5jLZfF+xs8
```

This approach to creating tag signatures is natively supported by git.sr.ht, and
I hope by other forges soon as well. When you push your tags, a link to download
the corresponding signature will appear in the release notes.

Speaking of release notes, what happens when it's time to release something
other than the initial version? Hare encourages the use of [semantic
versioning](https://semver.org), and the hare release tool provides automatic
version increments for major, minor, and patch releases. In short:

- If you make breaking changes which require user intervention, use `hare release major`
- If you add new features in a backwards-compatible manner, use `hare release minor`
- If you make backwards-compatible bug fixes, use `hare release patch`

hare release then uses [git-shortlog(1)][shortlog] to pre-generate release
notes for you. If you've been using [good commit discipline][discipline], you
might not have to edit them at all!

[shortlog]: https://git-scm.com/docs/git-shortlog
[discipline]: https://drewdevault.com/2019/02/25/Using-git-with-discipline.html

In the future, we hope to be able to analyze the difference between versions and
automatically detect if you have made breaking changes, and choose the
appropriate version increment for you. A release might be as simple as running
hare release with no arguments, glancing over the generated release notes,
punching in your key's password, and running `git push --follow-tags` to ship.

It is my hope that access to this tool will make it easier for Hare users to
manage their releases in a consistent manner which behaves uniformly across the
Hare ecosystem, creating a supply chain with a predictable approach and a
culture of ubiquitous cryptographic signatures to verify the integrity of Hare
software.

Today, this is based on git and SSH, which were selected in response to the
dominant trends in software development. However, the command line interface is
not tied down to these tool choices. I hope that proponents of other version
control systems will help us generalize hare release to support their
software, and to expand to support other approaches to signatures as well. In
addition to providing a uniform approach to release management across the Hare
ecosystem, we are positioned to ship upgrades which make improvements across the
same wide breadth of software in our ecosystem. Per our steadfast commitment to
stability and reliability, large changes along these lines will be scarce, but
rest assured that we are not painted into a corner by choosing these tools.

I hope you find this tool useful for your projects!
