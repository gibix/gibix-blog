---
layout: post
title: "Make Rust a first class citizen in Gentoo"
tags: [ gsoc, gentoo, rust ]
---

Here follow an abstract of my gsoc proposal and after it some further considerations.

# Proposal

The project aims at making Rust a first class citizen in Gentoo. Currently rust programming language has a mature development ecosystem. Tools like cargo, rustup and crates.io become stable in the last year and can be well integrated in gentoo. I will spend my google summer of code to make rust a first class citizen in gentoo with better support to ebuild generation from a cargo project directly through portage and overlays. Also the cross-compilation and multi-architecture support will be implemented to be fully integrated in gentoo.

## Current workflow

Cargo-ebuild is currently used to generate ebuild from a cargo project, it doesn’t support all cargo features and has to inspect a local source of the project. The cargo eclass in gentoo is well implemented, but it lacks of some important capabilities like cross-compilation and multi-target compilation. Metadeps is currently used by many rust project to keep track of external dependencies, on one side this helps to keep track of dependency in the cargo project, but lacks of integration in a build environment like portage. Portage cannot know about the dependency of a project and, for example, could drop or update it to an unsupported version. This behaviour leads to unsafe dependency resolution.

## Proposed workflow

The proposal will implement full rust support into portage, gentoo overlays and eselect. The gentoo experience will be transparent to rust exposing all the most appreciated features of rust in the well-know gentoo workflow. The workflow design of exposing cargo features to gentoo follows also the rust-roadmap decision to maintain cargo as the main core of the rust build system. All the design could be fully automated in order to simplify the maintenance work. Will be implemented an organic support to dependency, that currently are completely manual (via metadeps), exposing a more comfortable way to inspect a dependency directly into portage.

## Side effect

The work on cargo-ebuild is intend to define the pipeline package manager -> metadata -> ebuild -> overlay -> test -> upstream. All of the component of the pipeline will be implemented as a module and except from the first will be implemented in an language-independent design. Therefore any module will be available for other gentoo’s project.

# Some thoughts

During the last period I looked around in the rust community discussion. It was a pleasent experience to discover how much the debates around the language ecosystem is very rich! Particularly [cli working group](https://github.com/rust-lang-nursery/cli-wg) has tracked some interesting discussion about how rust binary projects are [packed](https://github.com/rust-lang-nursery/cli-wg/issues/8) and [distributed](https://github.com/rust-lang-nursery/cli-wg/issues/20).

There is also [cargo-bundle](https://github.com/burtonageo/cargo-bundle) an alpha project that goes in the same direction of the cli-wg. A wrapper around distribution/os bundles in order to cross-distribute binary projects.

On the side of the cross-compilation support at the rust all hands event there have been several discussion around cargo and rustup and how the they should be reviewed in the current year. [Here](https://blog.rust-lang.org/2018/04/06/all-hands.html#cargo) a general summary of the cargo team and [here](http://aturon.github.io/2018/04/06/rustup-xargo/) some thoughts by Aaron Turon about rustup and xargo.

To summarize the most important point...

**What's coming in rust:**

- Convergence of packaging/distributing tools in a maintained ecosystem.
- Some of the important rustup features could migrate in cargo.
- Many projects like carnix, cargo-deb, cargo-arch, etc are becoming quite stable.

**What does it means for cargo-ebuild and other distribution-specific tools:**

- Should be easy for projects like cargo-bundle to integrate cargo-ebuild like a lib.
- Rustup should no longer be fundamental in a distribution's package build.

**What is missing (and I want to add at least in cargo-ebuild):**

- A scriptable integration with the distribution package system. (aka overlay/repository maintenance)
- Easy way to inspect external dependencies.
- Support directly projects from crates.io

That's it! In the next days I will post more details about the working progress. I would be very happy to hear about needs and opinion from gentoo/rust/distributions users/devs.
