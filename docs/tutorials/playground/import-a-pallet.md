---
title: Import a Pallet
---

In this tutorial, the Node Template will be modified to include the
[Nicks pallet](https://substrate.dev/rustdocs/v2.0.0-rc6/pallet_nicks/index.html), which allows
blockchain users to pay a deposit to reserve a nickname and associate it with an account they
control.

## Importing a Pallet Crate

Substrate, FRAME and the Node Template are all written in
[the Rust programming language](https://www.rust-lang.org/), which uses a package manager and build
system called "[Cargo](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html)". The first step to
adding a new pallet to a FRAME runtime is to add that pallet as a dependency of the Cargo project
for the runtime. Cargo uses
[a file named `Cargo.toml`](https://doc.rust-lang.org/cargo/reference/manifest.html) to define a
project's manifest, including its dependencies. Open the Node Template's `runtime/Cargo.toml` file.

![Import a Pallet](assets/tutorials/playground/02-import.png)

The `Cargo.toml` file lists all of the project's dependencies; each of these dependencies is a
module of Rust code, which Cargo refers to as a "crate". For example, the Node Template's runtime
depends on the crate for the
[Balances pallet](https://substrate.dev/rustdocs/v2.0.0-rc6/pallet_balances/index.html), which is
referred to three times in the `Cargo.toml` file for the Node Template's runtime's:

**`runtime/Cargo.toml`**

```TOML
#--snip--

pallet-aura = { default-features = false, version = '2.0.0' }
pallet-balances = { default-features = false, version = '2.0.0' }  # <-- included as dependency here
pallet-grandpa = { default-features = false, version = '2.0.0' }

#--snip--

[features]
default = ['std']
runtime-benchmarks = [
    'hex-literal',
    #--snip--
    'pallet-balances/runtime-benchmarks',  # <-- mentioned here
    'pallet-timestamp/runtime-benchmarks',
    'sp-runtime/runtime-benchmarks',
]
std = [
    'codec/std',
    'serde',
    'frame-executive/std',
    'frame-support/std',
    'frame-system/std',
    'frame-system-rpc-runtime-api/std',
    'pallet-aura/std',
    'pallet-balances/std', # <-- mentioned here
    #--snip--
]
```

The next section will explain the purpose of the `default-features = false` statement that is used
when importing a pallet as well the `features` element of the `Cargo.toml` file.

### Crate Features

Cargo uses the concept of "[features](https://doc.rust-lang.org/cargo/reference/features.html)" to
encapsulate sets of user-facing capabilities or optional dependencies. Cargo features are used in
FRAME runtime development in order to allow a single codebase to support the creation of both native
and [WebAssembly (Wasm)](https://webassembly.org/) runtime binaries. Wasm is "a binary instruction
format for a stack-based virtual machine", which means that Wasm is a way for developers to express
their programs as portable executables (binaries).

The use of portable Wasm runtime binaries is one of Substrate's defining characteristics. It allows
the runtime executable itself to be expressed in a way that can become a part of a blockchain's
evolving state; it also means that the definition of the runtime is subject to the cryptographic
consensus mechanisms that ensure the security of the blockchain network. The fact that the runtime
can be updated and distributed in a way that does not require network participants to trust one
another enables one of Substrate's most innovative features: forkless runtime upgrades, which are
covered in [the next tutorial](../upgrade-a-chain).

[Rust supports compilation to Wasm](https://rustwasm.github.io/docs/book/introduction.html), but
only under
[certain constraints](https://rustwasm.github.io/docs/book/reference/which-crates-work-with-wasm.html),
one of the most important of which is the exclusion of
[the `std` library](https://doc.rust-lang.org/std/); Rust makes it easy to accomplish this with
[the `#![no_std]` attribute](https://doc.rust-lang.org/1.7.0/book/no-stdlib.html). The Balances
pallet is configured to use the `#![no_std]` attribute when it is compiled _without_ the `std`
feature flag:

**[`frame/balances/src/lib.rs`](https://github.com/paritytech/substrate/blob/v2.0.0-rc6/frame/balances/src/lib.rs)**

```rust
#![cfg_attr(not(feature = "std"), no_std)]
```

The `features` element of the runtime's `Cargo.toml` file configures the `std` feature flag as a
member of the set of default features, which means that any dependency that specifies
`default-features = false`, including the Balances pallet, will be compiled _without_ the `std`
feature flag. This configuration makes it possible to compile FRAME pallets like the Balances pallet
to Wasm binaries. The `features` element of the `Cargo.toml` file also configures two explicit
feature flags: `runtime-benchmarks`, which is used for
[benchmarking](https://github.com/paritytech/substrate/tree/v2.0.0/frame/benchmarking), and `std`,
which is used for compiling native (as opposed to Wasm) binaries.

### Importing the Nicks Pallet Crate

Importing the Nicks pallet is almost the same as importing the Balances pallet, but since the Nicks
pallet does not include any benchmarking, it will not be included in the runtime's benchmarking
flag. Update the runtime's `Cargo.toml` file as follows:

**`runtime/Cargo.toml`**

```TOML
#--snip--
pallet-grandpa = { default-features = false, version = '2.0.0' }
pallet-nicks = { default-features = false, version = '2.0.0' }  # <-- add this
pallet-randomness-collective-flip = { default-features = false, version = '2.0.0' }

#--snip--

[features]
default = ['std']
runtime-benchmarks = [
    #--snip--
]
std = [
    #--snip--
    'pallet-grandpa/std',
    'pallet-nicks/std',  # <-- add this
    'pallet-randomness-collective-flip/std',
    #--snip--
]
```

Before moving on, check that the new dependencies resolve correctly. Press `` Ctrl + `  `` to open
the Playground's integrated terminal, then execute the following command:

```bash
cargo check
```

Substrate Playground uses Docker images that include build artifacts in order to decrease initial
build times. Build artifacts are included for the following Cargo commands:

- `cargo build --release`
- `cargo check`
