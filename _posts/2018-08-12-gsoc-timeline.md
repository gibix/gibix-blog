---
layout: post
title: "Gsoc timeline"
tags: [ gsoc, gentoo, rust, timeline ]
---

## Week 1

The report of the first week can be found
[here](http://localhost:4000/2018/05/20/gsoc-week-1-report.html).

### Objectives

- [add 1.26.0 and multilib](https://github.com/gentoo/gentoo-rust/pull/342)
- [drop 1.24](https://github.com/gentoo/gentoo-rust/pull/343)
- [Repoman lint](https://github.com/gentoo/gentoo-rust/pull/344)
- [speedup download with .xz and changed libdir](https://github.com/gentoo/gentoo-rust/pull/345)


## Week 2

The report of the second week can be found
[here](http://localhost:4000/2018/05/25/gsoc-week-2-report.html).

### Objectives

- [cargo-ebuild](https://github.com/cardoe/cargo-ebuild/pull/7)

## Week 3

In this week I attended to the RustFest in Paris and I spent some time to
discuss ideas with the amazing rust community. As part of this I
contributed to some project in the community.

### Objectives

- [update to portage 2.3.38](https://github.com/gentoo/gentoo-rust/pull/350)
- [Fix stable](https://github.com/gentoo/gentoo-rust/pull/349)
- [multilib without soname](https://github.com/gentoo/gentoo-rust/pull/352)

## Week 4

I come up with a new design for cargo-ebuild. The big new deal is to drop
cargo as dependency in order to increase significantly compilation time and
binary size.
In order to get the work done I contributed to `¢argo_metadata`.

### Objectives

- [handle cargo features flags](https://github.com/oli-obk/cargo_metadata/pull/36)
- [fix #4746 and add test](https://github.com/rust-lang/cargo/pull/5599/commits)

## Week 5

In this period the work was a bit slower because, as agreed, I spend some
time on my university exams. Besides that the discussion around
`cargo-ebuild` has starved a bit because different point of view of the
contributors.

## Week 6

The work on `cargo-ebuild` has been completely rewritten with a new design.

### Objectives

- [Cli ergonomics](https://github.com/cardoe/cargo-ebuild/pull/9)

## Week 7

Another approach has been adopted on `cargo-ebuild` in agreement with the
maintainer in order to integrate more features.

### Objectives

- [Consistent cli](https://github.com/cardoe/cargo-ebuild/pull/14)

## Week 8

The work on multi-target support in gentoo has started with a first draft
submitted to the [gentoo-rust](https://github.com/gentoo/gentoo-rust).

### Objectives

- [rust eclass for multi-target](https://github.com/gentoo/gentoo-rust/pull/362)

## Week 9

Keep working on gentoo to get full support in rust and cargo eclass. The
work involved also rust and cargo ebuild and slotting system.

### Objectives

- [rust eclass for multi-target](https://github.com/gentoo/gentoo-rust/pull/362)


## Week 10

Because the work on portage is almost finished I spend some time to cleanup
the code and add the support into `eselect-rust`. Now features-as-use-flags
is working correctly with the patched version of `cargo-ebuild` and cargo
eclass.

### Objectives

- eselect
- [add support to cargo binaries](https://github.com/jauhien/eselect-rust)

## Week 11

The work done until now has been submitted to gentoo tree and in developers
mailing list. Thanks to reviews and discussions some fix has been
integrated in the patch.

### Objectives

- [eclass/{rust, rust-utils, cargo}: add support for rust
  multi-target](https://github.com/gentoo/gentoo/pull/9388)
- [gentoo-dev](https://archives.gentoo.org/gentoo-dev/message/7622fa005908d8c10e8f496a85ff8895)

## Week 12

Final discussions and fix around the gentoo main patch. I realised a
write-up of my experience of writing a gentoo eclass.

### Objectives

- [eclass/{rust, rust-utils, cargo}: add support for rust
  multi-target](https://github.com/gentoo/gentoo/pull/9388)
- [Journey into gentoo eclass]({% post_url 2018-08-11-journey-into-gentoo-eclass %})
