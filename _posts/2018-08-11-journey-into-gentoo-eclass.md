---
layout: post
title: "Journey into gentoo eclass"
tags: [ gsoc, gentoo, rust, report, portage, eclass ]
---

I spent some days writing a portage eclass for gentoo. I want to share my
experience.

# Why

The rust build experience is quite complete and powerful, particularly
thanks to cargo and rustup. While cargo is a well integrated project and
easy to script rustup is not the best choice, mainly because is not
integrated in a package/build system. Portage is wonderful project and is a
pity to keep rust out of it!

Rustup has some features that I want to have into portage:

- toolchain management
- component
- directory sugar

The toolchain support is already in tree with multilib and multitarget
support. The user can easily handle that with a rust-eselect module that
sets the default toolchain and some tools (rustc, rust-gdb, rust-doc,
rust-lldb).

```
gentoo-test ~ # eselect rust list
Available Rust versions:
  [1]   rust-1.26.2
  [2]   rust-1.27.1 *
  [3]   rust-9999
```

Usually rust projects are statically compiled, this is very handy when it
stands to dependencies management, but some projects have differente needs.
For example rustfmt, clippy, bindgen, rls and some others tools, usually
distributed as components in rustup, depend on `libsyntax`.

I dedicated some time to look for a good solution to integrate this pattern
in gentoo, this brought me to an interesting journey in gentoo build
ecosystem.

# Introduction to portage

Portage is a very powerful [ports
collection](https://en.wikipedia.org/wiki/Ports_collection) system that
thanks to its long time history can handle quite every corner needs.  In
this context the requirements are quite straightforward and easily paired
with portage features.

| feature                           | gentoo    |
|-----------------------------------|--------   |
| multiple package version          | slotting  |
| language-specific behaviour       | eclass    |
| consistent configuration system   | eselect   |

# Setup test environment

Portage is very resilient to complex configuration and is very common also
as user to use the [layman](https://wiki.gentoo.org/wiki/Layman) tool to
handle external overlays. Overlays works intuitively one on each other and
portage can looks for a package in every overlay. But because I have to
work on tree and I want to keep my env as clean as possible I decided to
maintain my own fork in a VM to work with. This is not necessary and quite
useless for package development but this project have to touch a lot of
stuff like profile and eclass.

First I have to configure my fork:

```
gentoo-test ~ # echo "[DEFAULT]
main-repo = gentoo

[gentoo]
location = /usr/portage
sync-type = git
sync-uri = https://github.com/gibix/gentoo.git
auto-sync = true
" > /etc/portage/repos.conf/gentoo.conf

gentoo-test ~ # emerge --sync
```

This will take a bit because portage has to rebuild all its cache but now
the environment can be easily keep in sync with the public repository or
also with rsync on the laptop.

# First draft

Let's start to code from the eclass, looking at existing projects I ended
up watching the python eclass and is a rich source of inspiration.

What is the core workflow of the eclass:

- keep track of the available rust implementation
- look to supported implementation in the cargo ebuild
- export some tool for ebuild to execute a work on multiple target

Looking at the requirements the eclass need at least an external variable
that will be filled by the eclass consumer.

```bash
# @ECLASS-VARIABLE: RUST_COMPAT
# @DESCRIPTION:
# This variable contains a list of Rust implementations the package
# supports. It must be set before the `inherit' call. It has to be
# an array.
#
# Example:
# @CODE
# RUST_COMPAT=( rust1_26, rust1_27 )
# @CODE
#
```

The portage syntax is very clear and minimal, but why I need to setup the
variable before the `inherit` function call in the consumer ebuild? This is
because the portage syntax is nothing more than bash with some sugar. Bash
like the most of the scripting languages is a top-down interpreter that
expands functions while are encountered. In particular the inherit function
used in portage calls `source` on each eclass passed as argument. Because
of that every consumer-defined component that needs to be used inside the
eclass must be configured before.

Now the eclass need some way to have a list of the targets that have to be
used. But how the targets will be defined by the user, we need also a way
to setup global defaults for that. I don't want to specify a stable target
for every rust ebuild, could be quite crazy to maintain! Fortunately
portage have some sugar for that! `USE_EXPAND` are some particular use
flags that have a name that can be used as reference for global
configuration or as prefix in local flag. What does it means? Let's have a
look.

First the `USE_EXAPAND` has to be defined in the profile used by portage,
the best place for a generic one like rust is in `base/make.conf`. Now I
can easily add a line to my `make.conf` for test a global configuration:

```
RUST_TARGETS="rust1_27"
```

But how is viewed by our eclass? Is an array of elements like
`rust_targets_rust1_27`. Thats good.

Let's define a variable to keep track of available implementations and
configure it.

```bash
# @FUNCTION: _rust_set_impls
# @INTERNAL
# @DESCRIPTION:
# Check RUST_COMPAT for well-formedness and validity, if RUST_COMPAT
# is not used looks to _RUST_ALL_IMPLS then set two global variables:
#
# - _RUST_SUPPORTED_IMPLS containing valid implementations supported
#   by the ebuild (RUST_COMPAT),
#
# - and _RUST_UNSUPPORTED_IMPLS containing valid implementations that
#   are not supported by the ebuild.
#
_rust_set_impls() {
	local i supp=() unsupp=()

	if ! declare -p RUST_COMPAT &>/dev/null; then
		for i in "${_RUST_ALL_IMPLS[@]}"; do
			supp+=( "${i}" )
		done
	else
		if [[ $(declare -p RUST_COMPAT) != "declare -a"* ]]; then
			die 'RUST_COMPAT must be an array.'
		fi

		for i in "${RUST_COMPAT[@]}"; do
			# trigger validity checks
			_rust_impl_supported "${i}"
		done

		for i in "${_RUST_ALL_IMPLS[@]}"; do
			if has "${i}" "${RUST_COMPAT[@]}"; then
				supp+=( "${i}" )
			else
				unsupp+=( "${i}" )
			fi
		done
	fi

    [...]
}
```

```bash
# @ECLASS-VARIABLE: _RUST_ALL_IMPLS
# @INTERNAL
_RUST_ALL_IMPLS=(
	rust1_25
	rust1_26
	rust1_27
)
readonly _RUST_ALL_IMPLS

# @FUNCTION: _rust_impl_supported
# @USAGE: <impl>
# @INTERNAL
_rust_impl_supported() {
	local impl=${1}

	case "${impl}" in
		"${_RUST_ALL_IMPLS[@]}")
			return 0
			;;
	esac

	return 1
}
```

We reached a good point, all the major functions to support multitarget are
defined, but all this functions are never invoked! So...

```bash
_rust_set_globals() {
	local deps

	_rust_set_impls

	local requse=""
	[ ${#flags[@]} -gt 0 ] && requse="|| ( ${flags[*]} )"

	local flags=( "${_RUST_SUPPORTED_IMPLS[@]/#/rust_targets_}" )

	REQUIRED_USE=${requse}
	IUSE=${flags[*]}
}
_rust_set_globals
unset -f _rust_set_globals
```

The targets is not yet used, but could be at least recognized.

```
gentoo-test ~ # emerge -p rustfmt

These are the packages that would be merged, in order:

Calculating dependencies... done!
[ebuild  N    ~] dev-util/rustfmt-0.10.0  USE="-debug -fetch-crates" RUST_TARGETS="-rust1_27 -rust1_26"


gentoo-test ~ # RUST_TARGETS="rust1_27 rust1_26" emerge -p rustfmt

These are the packages that would be merged, in order:

Calculating dependencies... done!
[ebuild  N    ~] dev-util/rustfmt-0.10.0  USE="-debug -fetch-crates" RUST_TARGETS="rust1_26 rust1_27"
```

Good! Portage is working as expected!

# Add dependency

The situation now is quite defined, the rust eclass can handle the targets
and is quite easy to maintain, the only absolute definition of the target
is in `_RUST_ALL_IMPLS`. But could be quite a pain to maintain all this
dependency in the ebuild, could be better to have the eclass to keep track
of all this stuff for the ebuild.

Again, what are the needs:

- an ebuild that inherit from the rust eclass should automatically resolve
  the toolchain dependencies based on the rust targets.
- an easy way to retrive the rust ebuild in tree from a rust target

First let's look how dependencies are defined in portage. For a complete
reference look at the complete [gentoo
devmanual](https://devmanual.gentoo.org/general-concepts/dependencies/).

In the current context the dependencies are binded to a use flag. Portage
makes available a simple interface to define USE-conditional dependencies:

`(!)USEFLAG? ( DEPS )`

in our case we want something like:

`rust_targets_rust1_27? (=virtual/rust-1.27.2) ...`

For having this work is necessary to generate the deps:

```bash
rust_package_dep() {
	case ${1} in
		rust1_25)
			echo "=virtual/rust-1.25*"
			;;
		rust1_26)
			echo "=virtual/rust-1.26*"
			;;
		rust1_27)
			echo "=virtual/rust-1.27*"
			;;
		*)
			die "Invalid implementation: ${impl}"
	esac
}
```

Now is just necessary to add some lines in the `set_globals` function:

```bash
_rust_set_globals() {
	local deps
    [...]

	for i in "${_RUST_SUPPORTED_IMPLS[@]}"; do
		deps+="rust_targets_${i}? ( $(rust_package_dep ${i}) ) "
	done
    [...]

	RDEPEND=${deps}
    [...]
}
```

Is time to test if the implementation is working correctly. Is simply
necessary to append rust as inherited eclass in a test ebuild. In my case
rustfmt.

```
gentoo-test ~ # RUST_TARGETS="rust1_27 rust1_26" emerge -p rustfmt

These are the packages that would be merged, in order:

Calculating dependencies... done!
[ebuild  N    ~] virtual/rust-1.27.1-r1
[ebuild  N    ~] virtual/rust-1.26.2-r1
[ebuild  N    ~] dev-util/rustfmt-0.10.0  USE="-debug -fetch-crates" RUST_TARGETS="rust1_26 rust1_27"
```

# It's time to build

All the tools are available for enforce multitarget build of a rust
project.

Some breadcrumbs:

- rely on cargo as high level eclass
- cargo should use some high-level function to trigger the build for
  *every* defined target
- cargo must have two different behaviour: one in case of single-target
  (more common) and one for multitarget

Gentoo has a very nice eclass for this!
[Multibuild](https://devmanual.gentoo.org/eclass-reference/multibuild.eclass/).

Let's write this part with a top-down approach, starting from the cargo. In
order to have multibuild work properly is necessary to setup variants and
some environment variables to run the build with the correct toolchain.
This feature will be privided by the cargo eclass, just assume that is
already available.


```bash
_cargo_run_foreach_impl() {

	MULTIBUILD_VARIANTS=${RUST_TARGETS[@]}

	rust_build_foreach_variant "${@}"
}
```

Whit this change is necessary to adapt all the other cargo functions to use
the brand new `foreach_impl()`.

```bash
cargo_src_install() {
	debug-print-function ${FUNCNAME} "$@"

	_cargo_run_foreach_impl cargo_install
}
```

Ok, now all the cargo step in the pipeline are executed within multibuild.
It's time to come back to our loved rust eclass and write
`rust_build_foreach_variant`.

```bash
rust_build_foreach_variant() {
	debug-print-function ${FUNCNAME} "${@}"

	local MULTIBUILD_VARIANTS
	_rust_obtain_impls

	multibuild_foreach_variant _rust_multibuild_wrapper "${@}"
}

_rust_multibuild_wrapper() {
	rust_export ${MULTIBUILD_VARIANT}

	"${@}"
}
```

Multibuild runs a command for each variant in `MULTIBUILD_VARAIANTS`, we are
lucky because we already have `SUPPORTED_IMPLS` filled with the proper
information, but not all the implementations in there must be triggered. Is
necessary to check witch of them are triggered by the configuration.

```bash
_rust_obtain_impls() {
	MULTIBUILD_VARIANTS=()

	local impl
	for impl in "${_RUST_SUPPORTED_IMPLS[@]}"; do
		use "rust_targets_${impl}" && MULTIBUILD_VARIANTS+=( "${impl}" )
	done
}
```

The only missing part now is `export` function that have to setup `RUSTC`
environment variable. This is read by cargo for use a specific compiler.

```bash
rust_export() {
	local impl

	case "${1}" in
		rust*)
			impl=${1/rust/rustc-}
			impl=${impl/_/.}
			shift
			;;
		*)
			die "rust export called without a valid rust implementation"
			;;
	esac
}
```

All done? Not exactly, thinking a bit about the workflow is clear that
there is a big incomplete configuration. If a project is build with more
than a rust implementation all the binaries will be installed in the same
path, with the same name!

Here there are multiple possible solutions from my point of view.

Have all the binaries installed normally in `/usr/bin` with a postfix, for
example `my-binary-rust-1.27` and a `my-binary`symlink to it. But this
could stand to a quit messy situation. Why not have a `bin` directory in
`/usr/lib/rust_${version}`? Let's try this solution.

Because ${version} has to be configured for each implementation we know
were to touch:

```bash
rust_export() {
    [...]
	if [ ${#MULTIBUILD_VARIANTS[*]} -gt 1 ]; then
		export RUST_ROOT="/usr/$(get_libdir)/$(ls /usr/$(get_libdir) | grep ${impl/rustc/rust})"
	else
		export RUST_ROOT="/usr"
	fi

}
```

and finally.

```bash
cargo_install() {
	cargo install -j $(makeopts_jobs) --root="${D}${RUST_ROOT}" \
		$(usex debug --debug "") \
		|| die "cargo install failed"
	rm -f "${D}${RUST_ROOT}.crates.toml"
}
```

# Enforce the work done

Usually if a project is build with multiple target is because has runtime
linking dependency to rustc libraries like `libsyntax` so is important that
if `my-project-binary` is called in that moment there is a proper library
path configured. This is done correctly via `eselect-rust` module.

```
gentoo-test ~ # grep LDPATH /etc/env.d/50rust-*
/etc/env.d/50rust-1.26.2:LDPATH="/usr/lib64/rust-1.26.2"
/etc/env.d/50rust-1.27.1:LDPATH="/usr/lib64/rust-1.27.1"
/etc/env.d/50rust-9999:LDPATH="/usr/lib64/rust-9999"
```

Or can be overridden at runtime with `LD_LIBRARY_PATH`:

```
LD_LIBRARY_PATH=/usr/lib64/rust-1.26.2 rustfmt
```

Furthermore the same system is used by `eselect-rust` to handle rust
versions, can we use the same system without reinventing the wheel?
A clear solution is to have a `*binaries-${target}` file in
`/etc/env.d/rust` for every project that have multiple variants compiled.

The `eselect-rust` patch is quite simple and I will not describe the code
in details. Can be found [here](https://github.com/jauhien/eselect-rust/pull/4).

But let's look at the last fix that we have to bring to the `cargo.eclass`.

```bash
cargo_install() {
    [...]

	if [ ${#MULTIBUILD_VARIANTS[*]} -gt 1 ]; then
		env_file="${PN}-binaries-$(basename ${RUST_ROOT})"

		for binary in ${D}${RUST_ROOT}/bin/*; do
			echo /usr/bin/$(basename $binary) >> "${T}/${env_file}"
		done

		dodir /etc/env.d/rust
		insinto /etc/env.d/rust
		doins "${T}/${env_file}"
	fi
}
```

# Conclusion

This work has been realized as part of my Google summer of code project,
required some time to investigate also alternative solutions that I decided
to drop during the time.

The code can find at this [pull request](https://github.com/gentoo/gentoo/pull/9388).
