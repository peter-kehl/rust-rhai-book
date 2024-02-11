Minimal Build
=============

{{#include ../../links.md}}

Configuration
-------------

In order to compile a _minimal_ build &ndash; i.e. a build optimized for size &ndash; perhaps for
`no-std` embedded targets or for compiling to [WASM], it is essential that the correct linker flags
are used in `Cargo.toml`:

```toml
[profile.release]
lto = "fat"         # turn on Link-Time Optimizations
codegen-units = 1   # trade compile time with maximum optimization
opt-level = "z"     # optimize for size
```


Use `i32` Only
--------------

For embedded systems that must optimize for code size, the architecture is commonly 32-bit.
Use [`only_i32`] to prune away large sections of code implementing functions for other numeric types
(including `i64`).

If, for some reason, 64-bit long integers must be supported, use [`only_i64`] instead of [`only_i32`].


Opt-Out of Features
-------------------

Opt out of as many features as possible, if they are not needed, to reduce code size because,
remember, by default all code is compiled into the final binary since what a script requires cannot
be predicted. If a language feature will never be needed, omitting it is a prudent strategy to
optimize the build for size.

Removing the script [optimizer][script optimization] ([`no_optimize`]) yields a sizable code saving,
at the expense of a less efficient script.

Omitting [arrays] ([`no_index`]) yields the most code-size savings, followed by floating-point support
([`no_float`]), safety checks ([`unchecked`]) and finally [object maps] and [custom types] ([`no_object`]).

Where the usage scenario does not call for loading externally-defined modules, use [`no_module`] to
save some bytes. Disable script-defined functions ([`no_function`]) and possibly closures
([`no_closure`]) when the features are not needed. Both of these have some code size savings but not much.

For embedded scripts that are not expected to cause errors, the [`no_position`] feature can be used
to disable position tracking during parsing. No line number/character position information is kept
for error reporting purposes. This may result in a slightly smaller build due to elimination of code
related to position tracking.


Use a Raw [`Engine`]
--------------------

[`Engine::new_raw`][raw `Engine`] creates a _raw_ engine. A _raw_ engine supports, out of the box,
only a very [restricted set][built-in operators] of basic arithmetic and logical operators.

Selectively include other necessary functionalities by picking specific [packages] to minimize the footprint.

Packages are shared (even across threads via the [`sync`] feature), so they only have to be created once.

In addition, a [`Engine::new_raw`][raw `Engine`] disables the _[strings interner]_, which might
actually increase memory usage if many strings are created in scripts. Therefore, selectively turn
on the [strings interner] via [`Engine::set_max_strings_interned`][options].
