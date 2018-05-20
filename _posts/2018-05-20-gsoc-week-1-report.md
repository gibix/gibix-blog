---
layout: post
title: "Gsoc week 1 report"
tags: [ gsoc, gentoo, rust, report ]
---
Here the report of my first week of code for the Google Summer of Code.

## Cargo-ebuild

The work on [cargo-ebuild](https://github.com/cardoe/cargo-ebuild) has started. The overall design has been redesigned to prepare the projects to receive more features in the next weeks. Some new features like PackageInfo and licences have been ported from [cargo-bitbake](https://github.com/cardoe/cargo-bitbake) by the some author of cargo-ebuild.

### About writing a cli

Writing a cli in rust is a enjoyable experience. I spent some looking at existing projects and how they manage the typical design problems of writing binaries, like arguments parsing, logs, error handling and more. Most of the most interesting insparations comes from the [cli working group](https://github.com/rust-lang-nursery/cli-wg) and testing with two interesting projects: [quicli](https://github.com/killercup/quicli) and [ergo](https://github.com/rust-crates/ergo). They try to make the life easier. Being very simple all-in-one tools that re-export some well-know crates for common needs. Here I summarize some first thoughts about my work.

### Language

- abuse of `impl`
- abuse of `newtype`
- modules are friends

### Errors

I found experimenting with errors a good learning exercise to understand some of the most interesting features of the language. In my case I decided to follow a simple approach, **the cli can panic but lib should never** unless for `std` bad stuff.
But this doesn't prevent me from prototyping and using unwrap/expect a lot! Unwrapping is natural and because is an explicit action is very easy in the future to just grep all the ugly and dangerous stuff done while experimenting and replace them with a better solution.

To summarize:

- `unwrap`, `panic` and `expect` are perfect for a first touch
- play with `failure` and `failure::Error` (instead of explicit use of `error-chain`)
- abuse of `?` operator and `newtype` 
- `and_then`, `map_or_else`, `ok_or_else` and relates are your friend more than you can imagine
- starting from rust 1.26 the main can return `Result`
- in errors like in other subjects of the language rust can seem quite heavy and complex, but because forces the programmer to be explicit is very helpful
- using `Error` is not always the best choice because allocates, but for the most use cases is good. I don't think that writing in a low-level language means in any case to be crazy about performance. Allocating a string is good price to pay for a simple error-handling workflow. If you need more keep playing with error and define yours ;)
- I've never missed exceptions...

Using a strongly statically typed language like rust without getting full control of your errors is a shame.

Some resources:

- The rust [book](https://doc.rust-lang.org/book/second-edition/ch09-00-error-handling.html) and [cookbook](https://rust-lang-nursery.github.io/rust-cookbook/basics.html#handle-errors-correctly-in-main) are (as always) a perfect starting point
- Andrew Gallant wrotes one of the most comprehensive [guide](https://blog.burntsushi.net/rust-error-handling/) about rust's errors.
- Take a look to [failure](https://boats.gitlab.io/failure/ for some very inspiring example)
- Herman J. Radtke wrotes a [guide](https://hermanradtke.com/2016/09/12/rust-using-and_then-and-map-combinators-on-result-type.html) about `and_then` and relates.

### Misc

- The debian team has a good [policy](https://wiki.debian.org/Teams/RustPackaging/Policy) for those who wants to package cargo projects.
- On crates.io there is a good mix of rust common [patterns](https://crates.io/categories/rust-pattern).

## Gentoo fix

I done a bunch of contributions to the gentoo-rust overlay.  During my work I discovered [repoman](https://wiki.gentoo.org/wiki/Repoman) as a tool "used to enforce a minimal level of quality assurance in ebuilds and related metadata added to ebuild repositories".

- add 1.26.0 and multilib [[PR1](https://github.com/gentoo/gentoo-rust/pull/342), [PR2](https://github.com/gentoo/gentoo-rust/pull/345)]
- drop 1.24 [[PR](https://github.com/gentoo/gentoo-rust/pull/343)]
- Repoman lint [[PR](https://github.com/gentoo/gentoo-rust/pull/344)]