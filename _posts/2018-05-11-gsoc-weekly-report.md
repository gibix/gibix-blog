---
layout: post
title: "Gsoc week -1 report"
tags: [ gsoc, gentoo, rust, report ]
---
# My activities of the week:

This has been an 1/2 week of work. I spent most of the time to test rust long-running builds in order to optimize the work for the next week.

## Environment

Made some docker images with:

- `--cap-add=SYS_PTRACE` to correctly build in the container when runned. Unfortunatly this cannot be set into the image building.
- `FEATURES="buildpkg"` to easily reuse packages across containers
- `FEATURES="-sandbox -usersandbox"` to correctly build in the container when the rust build is into the `docker build`

## Misc

- I summerised my proposal and this week research in a [post]({% post_url 2018-05-09-gsoc-proposal %}).
- Made a presentation of the gsoc project at the local rust meetup.
- Pushed the new [rust stable](https://github.com/gentoo/gentoo-rust/pull/336) to the gentoo-rust overlay with multilib support.

## Cargo ebuild

Inspected the [metadeps](https://github.com/joshtriplett/metadeps) for integration in `cargo-ebuild`. I decided to drop it and use only [pkg-config](https://crates.io/crates/pkg-config) as a base. `Metadeps` is based on `pkg-config` and can be rewritten into `cargo-ebuild`.

Here follows some cli features that I want to write into `cargo-ebuild` in the next weeks:

### Build

`build` will manage the creation of the ebuild file from a cargo project or directly from crates.io

- `cargo-ebuild build`
- `cargo-ebuild build--diff`
- `cargo-ebuild build--params [args]`
- `cargo-ebuild build merge`

### Overlay

The overlay management will make the life easier to the maintainers.

- `cargo-ebuild update [overlay]`
- `cargo-ebuild list [overlay]`
- `cargo-ebuild status [overlay]`

Build, Overlay and more will be exposed as indipendent crates.