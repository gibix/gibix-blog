---
layout: post
title: "Gsoc week 2 report"
tags: [ gsoc, gentoo, rust, report ]
---

## Cargo-ebuild

The work on cargo-ebuild is keeping the road. I started to add the support for external crates defined via git. This will support all crates definitions like:

```
[dependecies]
soupcrate = { git = "url"}
soupcrate = { git = "url", branch = "supercoolbranch" }
soupcrate = { git = "url", rev = "supercoolbranch" }
soupcrate = { git = "url", version = "supercoolbranch" }

soupcrate = "*"

[soupcrate.patch]
soupcrate = { git = "url" ...}
```

I'm playing with the [cargo.eclass](https://github.com/gentoo/gentoo/blob/master/eclass/cargo.eclass) to fully support git crates, I hope that the next week will be fully working.

### Better cli interface

Having a good cli code design is not difficult in rust here some goods that I come with after last week considerations.

- The `main.rs` should just takes care only of setup logs, stdio, call the core main and process exits. Is not necessary to propagate error context to this point. For this `failure::Error` help for very simple propagation with a message.
- The `lib.rs` with a `run_my_project(cli_args)` function to parse commands (unwrap `Options`) and a different function for every cli subcommand.
- Intense use of `structopt` and standard rust modules really help to keep the project naturally organised. If the cli commands set is well designed should be not necessary to cycling around with a lot of `use::my_module`.

## Gentoo

I spend a little bit of time to add the multilib support to the rust ebuild in [gentoo-rust](https://github.com/gentoo/gentoo-rust). But I emerged it correctly for the first time portage started to complain:

```
 * QA Notice: The following shared libraries lack a SONAME
 * /usr/lib64/libarena-cae17091b76e64d5.so
 * /usr/lib64/libfmt_macros-ede26984756bf551.so
[...]
   usr/lib64/rustlib/x86_64-unknown-linux-gnu/lib/librustc-47b1e7e397747b12.so
   usr/lib64/rustlib/x86_64-unknown-linux-gnu/lib/libstd-d88aca54503564a2.so
[...]
```

So... the [wiki](https://en.wikipedia.org/wiki/Soname)!:

>[...] a soname is a field of data in a shared object file. The soname is a string, which is used as a "logical name" describing the functionality of the object.

There are several issues on the rust project that tracks this problems but I came to [this](https://github.com/rust-lang/rfcs/issues/600) that tries to summarise all the others. Is a very interesting discussion about ABIs and stability. But all this very long chat can be just reduced to the last comment by [@cuvier](https://github.com/cuviper):

> Rust has been committed to a stable language and API since 1.0, but ABI stability has never been claimed. Your code is compatible; your binaries are not. 

This is particularly tricky wen it comes to use cargo tools like clippy, fmt, bindgen. At now if a tool that use the rust stdlib and is not shipped with rustup (like clippy) is necessary to rebuild it on every release.