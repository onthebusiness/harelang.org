---
title: Scaling the Hare community
author: Drew DeVault
date: 2023-08-28
---

I started the Hare project on December 27th, 2019, with a small commit
introducing a lexer and a yacc parser for prototyping the grammar. In the nearly
four years since then, a community of over one hundred contributors has grown
around the project, including some eight co-maintainers working together to
incorporate dozens of patches every week.

Over the past several weeks, I have been working with the other maintainers and
project leaders to discuss possible reforms to the community structure and
workflow to reduce bottlenecks and streamline the work, to reduce frustration
with slow patch approval and make it easier for people to work autonomously --
and to reduce the, uh, "Drew bottleneck".

As Hare is organized with a mailing-list-based workflow for preparing,
reviewing, and applying patches, we have drawn inspiration from how the largest
mail-based project -- Linux -- scaled up to its massive size. I'm fond of the
Linux model because it is both effective and simple, so it can be applied to a
medium-size project and scale all the way up to the largest free software
project in the world.

To this end, the changes are focused on:

- Empowering maintainers to work independently
- Facilitating better communication throughout the community
- Encouraging a more distributed approach to development

The reforms have been under development since April and should be completed in
the next few weeks, then tested and refined over time. Let's sum up each of the
efforts to achieve this.

## MAINTAINERS

We have introduced a [MAINTAINERS] file to the Hare repository, as well as a
[maintainer policy] which outlines the rights, responsibilities, and procedures
that these maintainers employ. This file is adapted from a similar approach in
Linux and allows us to formally dole out responsibilities over subsystems in the
codebase. The areas of responsibility include things like "AArch64 support"
(maintained by Willow Barraco), "Date/time support" (Byron Torres), and so on,
as well as a top-level list of global maintainers: myself, Ember Sawady,
Sebastian, and Bor Grošelj Simić.

[MAINTAINERS]: https://git.sr.ht/~sircmpwn/hare/tree/master/item/MAINTAINERS
[maintainer policy]: https://git.sr.ht/~sircmpwn/hare/tree/master/item/docs/maintainers.md

We have also standardized the way that maintainers work on their patch queues so
that we can communicate to each other more effectively the status of ongoing
code reviews and patch approvals.

These changes:

1. Empower notable contributors to oversee their subsystems
2. More clearly assign code review responsibilities across subsystems
3. Reduce the Drew bottleneck

Each maintainer has push access and can review, approve, and apply patches
independently.

## Distributing and autonomizing Hare development

Hare maintains centralized resources for governing the upstream project,
including our primary mailing lists, bug trackers, and source code repositories.
However, larger initiatives within the project can often benefit from access to
dedicated infrastructure, allowing them to organize independently without being
bottlenecked by the constraints of upstream.

Today, there is already a significant degree of autonomous work being done on
Hare, but it's organized informally and has some accessibility problems. Many
Hare maintainers have their own personal Hare trees and organize large efforts
(like the module3 work or linear types research) alone, perhaps looping in one
or two other contributors on an informal basis to participate in this work. We
like this approach but want to encourage it to happen with improved
organization, communication, and efficiency.

We want to encourage participants to self-organize independently of upstream. To
keep these efforts well-organized and accessible, we have established a [TREES]
file to keep track of trees maintained independently of upstream. This includes
for now a list of maintainer's personal trees, where they stage work on their
various subsystems, to make these efforts more visible to the community. It will
be expanded in the future in two directions: permanent and project trees.

[TREES]: https://git.sr.ht/~sircmpwn/hare/tree/master/item/TREES

A permanent tree is a tree maintained in the long-term for a specific purpose,
for instance the maintenance of a platform port. For example, Willow, the
aarch64 maintainer, might establish a permanent hare-arm tree to organize work
associated with ARM ports. This could include an ARM-specific bug tracker and
mailing lists, as well as a source tree. Other candidates include cryptography,
date/time, and other long-term, large-scale efforts within the Hare community,
where work is conducted independently and regularly sent to Hare upstream in
batches.

This is analogous to the Linux model, where something like hare-arm corresponds
to linux-arm, and other examples of permanent trees include linux-drm,
linux-btrfs, and so on.

Project trees have a medium-term lifecycle, and are started to organize work
around a specific initiative in the Hare project. Two efforts which have been
ongoing under this framework in an informal model include the module3 rewrite
and research into linear types. These larger, research-heavy projects can
benefit from more organized collaborative resources and visibility, to be
dismantled once merged upstream.

## Establishing an RFC process

Changes to the language are implemented through a process of community
consensus. The unclear nature of the consensus process and issues with
communication tools throughout the community has occasionally led to cases where
the status of that consensus was unclear, leading contributors to expend work on
projects which, when presented for broader consensus, require frustrating
retooling and rewrites to be accepted.

We are establishing an RFC process to address these issues. The RFC process is
designed to be a tool rather than a bottleneck. It is not required for all
changes. The author of a change can opt-in to the RFC process, for changes of
any size, if they believe that a structured conversation around the proposed
change would be useful to their work. A reviewer or maintainer can also request
that the RFC process is invoked for a change if they believe it would be useful.
Some guidelines for determining the requirements for a change to be discussed as
an RFC have been prepared, but it will ultimately be left to the discretion of
the participants.

Changes to be discussed within the RFC framework will follow the process
outlined in the [RFC process document][rfc], wherein proposals are prepared and
submitted to the new hare-rfc mailing list for discussion.

[rfc]: https://git.sr.ht/~sircmpwn/hare/tree/master/item/docs/rfc.md

## Defining the BDFL role

One of the problems we're working on fixing is my status as the "Drew
bottleneck". Work to decentralize development and establish more maintainers
goes a long way towards addressing this, but it's also helpful to figure out
exactly what my role as a BDFL (Benevolent Dictator for Life) actually means for
the project.

I have written [a personal note][bdfl] to the repository in docs/bdfl.md which
elaborates on these topics. I view the role of BDFL as a service to the
community, rather than something that is imposed on the project, and I possess
unique tools in this role that the community can call upon when necessary. This
includes matters such as overseeing fiduciary responsibilities, setting the
long-term vision and planning for the project, and assigning rights and
responsibilities to other participants.

[bdfl]: https://git.sr.ht/~sircmpwn/hare/tree/master/item/docs/bdfl.md

This document also clarifies how best to make use of my resources such that they
can be leveraged efficiently when needed without introducing too many
bottlenecks, such as good ways to draw my attention to specific issues that
require BDFL feedback or manage administrative tasks like updating DNS records
and ACLs on SourceHut.

## Social spaces and the code of conduct

Hare worked without a code of conduct for a long time; even when working alone
and at my discretion as the sole moderator I am generally competent at keeping
jerks out of the community and making people feel welcome. However, as the
community grows, moderation is a burden which is more efficient when shared with
the rest of the team. The community shapes its own culture, and empowering
participants to steer the discussions and assist in moderation is an important
part of placing the culture in the hands of the community.

For these reasons, and to more clearly signal our values and expectations to new
community members, we introduced a [Code of Conduct][coc] and a committee of
three trusted community members to oversee conflict resolution:

[coc]: https://harelang.org/conduct/

- Drew DeVault
- Ember Sawady
- Vlad-Stefan Harbuz

We are also expanding the role of community spaces to include some social space
in addition to working space. One of the foundational values of the Hare
community is professionalism: the working spaces are kept on-topic and
professional in tone, though they don't enforce any kind of overt formality.
However, the community is also a fun place to be and many friendships have
formed here -- that's something to be encouraged!

We noticed a few groups of Hare contributors forming friendships and taking
their social activities outside of the Hare spaces -- respecting the on-topic
rules and professional attitude of these spaces. But, if you get a group of
excited Hare contributors in a room they're eventually going to talk about Hare,
and this started leading to cases where a few Hare contributors would talk
outside of the Hare community about their ideas, form a consensus in their
insular group, then get put out when this consensus does not translate to the
broader community.

To improve matters, we have established the #hare-soc IRC channel as a lighter
social space for the Hare community on the Libera Chat IRC network, where we
hope that these friendships and connections can be encouraged, fun is allowed,
and we can keep the community communicating better together without affecting
the professional and on-topic attitude of the working spaces. The expanded role
of moderators and the new code of conduct is anticipated to provide the
necessary tools to maintain a more social space in the Hare community where the
standards can be loosened while maintaining a respectful and inclusive
environment.
