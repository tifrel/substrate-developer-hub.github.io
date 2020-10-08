---
title: Configure the Nicks Pallet
---

Every pallet has a component called `Trait` that is used for configuration. This component is a
[Rust "trait"](https://doc.rust-lang.org/book/ch10-02-traits.html); traits in Rust are similar to
interfaces in languages such as C++, Java and Go. FRAME developers must implement this trait for
each pallet they would like to include in a runtime in order to configure that pallet with the
parameters and types that it needs from the outer runtime. For instance,
[the template pallet](https://github.com/substrate-developer-hub/substrate-node-template/blob/master/pallets/template/src/lib.rs)
that is included in the Node Template defines the following `Trait` configuration trait:

**`pallets/template/src/lib.rs`**

```rust
/// Configure the pallet by specifying the parameters and types on which it depends.
pub trait Trait: frame_system::Trait {
    /// Because this pallet emits events, it depends on the runtime's definition of an event.
    type Event: From<Event<Self>> + Into<<Self as frame_system::Trait>::Event>;
}
```

The template pallet is minimal by design, so it only specifies a single dependency from the outer
runtime: the `Event` type that will be used to notify off-chain users of on-chain actions. The
`Trait` configuration trait is an important piece of FRAME's compositional design - it allows
runtime developers to configure their pallets by specifying the _interfaces_ on which they depend as
opposed to the _concrete types_. The types specified by the Rust traits like `Trait` are referred to
as
"[associated types](https://doc.rust-lang.org/stable/rust-by-example/generics/assoc_items/types.html)".
In addition to supporting runtime composition, the `Trait` configuration trait also allows FRAME
developers to configure pallets with hard-coded constant values. These constant values may be used
to specify the cost of exercising a pallet's capabilities or even limit the resources that a pallet
can consume - both of these use cases will be demonstrated when adding the Nicks pallet to the Node
Template's runtime.

Take a look at the
[`pallet_nicks::Trait` documentation](https://substrate.dev/rustdocs/v2.0.0/pallet_nicks/trait.Trait.html)
or the definition of the trait itself in
[the source code](https://github.com/paritytech/substrate/blob/v2.0.0/frame/nicks/src/lib.rs) of the
Nicks pallet. The source code below has been annotated with new comments that expand on those
already included in the documentation:

**[`frame/balances/src/lib.rs`](https://github.com/paritytech/substrate/blob/v2.0.0-rc6/frame/balances/src/lib.rs)**

```rust
pub trait Trait: frame_system::Trait {
    // The runtime must supply this pallet with an Event type that satisfies the pallet's requirements.
    type Event: From<Event<Self>> + Into<<Self as frame_system::Trait>::Event>;

    // The currency type that will be used to place deposits on nicks.
    // It must implement ReservableCurrency.
    // https://substrate.dev/rustdocs/v2.0.0/frame_support/traits/trait.ReservableCurrency.html
    type Currency: ReservableCurrency<Self::AccountId>;

    // The amount required to reserve a nick.
    type ReservationFee: Get<BalanceOf<Self>>;

    // A callback that will be invoked when a deposit is forfeited.
    type Slashed: OnUnbalanced<NegativeImbalanceOf<Self>>;

    // Origins are used to identify network participants and control access.
    // This is used to identify the pallet's admin.
    // https://substrate.dev/docs/en/knowledgebase/runtime/origin
    type ForceOrigin: EnsureOrigin<Self::Origin>;

    // This parameter is used to configure a nick's minimum length.
    type MinLength: Get<usize>;

    // This parameter is used to configure a nick's maximum length.
    // https://substrate.dev/docs/en/knowledgebase/runtime/storage#create-bounds
    type MaxLength: Get<usize>;
}
```

Refer to the implementation of the Balances pallet's `Trait` configuration trait in
`runtime/src/lib.rs`; this will be used as an example to help understand how to implement the
`Trait` configuration trait for any pallet, including the Nicks pallet. The Balances pallet's
`Trait` implementation consists of two parts: a `parameter_types!` block, where constant values are
defined, and an `impl` block, where the types and values defined by the `Trait` interface are
configured. This code block has also been annotated with additional comments:

**`runtime/src/lib.rs`**

```rust
parameter_types! {
    // The u128 constant value 500 is aliased to a type named ExistentialDeposit.
    pub const ExistentialDeposit: u128 = 500;
    // A heuristic that is used for weight estimation.
    pub const MaxLocks: u32 = 50;
}

impl pallet_balances::Trait for Runtime {
    // The previously defined parameter_type is used as a configuration parameter.
    type MaxLocks = MaxLocks;

    // The "Balance" that appears after the equal sign is an alias for the u128 type.
    type Balance = Balance;

    // The empty value, (), is used to specify a no-op callback function.
    type DustRemoval = ();

    // The previously defined parameter_type is used as a configuration parameter.
    type ExistentialDeposit = ExistentialDeposit;

    // The FRAME runtime system is used to track the accounts that hold balances.
    type AccountStore = System;

    // No weight information is supplied to the Balances pallet by the Node Template's runtime.
    type WeightInfo = ();

    // The ubiquitous event type.
    type Event = Event;
}
```

The `impl balances::Trait` block allows runtime developers that are including the Balances pallet in
their runtime to configure the types and parameters that are specified by the Balances pallet
`Trait` configuration trait. For example, the `impl` block above configures the Balances pallet to
use the `u128` type to track balances. In fact, it is possible to use any unsigned integer type that
is at least 32-bits in size; this is because
[the `Balance` type](https://substrate.dev/rustdocs/v2.0.0/pallet_balances/trait.Trait.html#associatedtype.Balance)
for the Balances pallet `Trait` configuration trait is "bounded" by
[the `AtLeast32BitUnsigned` trait](https://substrate.dev/rustdocs/v2.0.0/sp_arithmetic/traits/trait.AtLeast32BitUnsigned.html).

Implement the `Trait` configuration trait for the Nicks pallet. Add the following code to
`runtime/src/lib.rs`:

```rust
parameter_types! {
    // Choose a fee that incentivizes desireable behavior.
    pub const NickReservationFee: u128 = 100;
    pub const MinNickLength: usize = 8;
    // Maximum bounds on storage are important to secure the chain.
    pub const MaxNickLength: usize = 32;
}

impl pallet_nicks::Trait for Runtime {
    // The Balances pallet implements the ReservableCurrency trait.
    // https://substrate.dev/rustdocs/v2.0.0/pallet_balances/index.html#implementations-2
    type Currency = pallet_balances::Module<Runtime>;

    // Use the NickReservationFee from the parameter_types block.
    type ReservationFee = NickReservationFee;

    // No action is taken when deposits are forfeited.
    type Slashed = ();

    // Configure the FRAME System Root origin as the Nick pallet admin.
    // https://substrate.dev/rustdocs/v2.0.0/frame_system/enum.RawOrigin.html#variant.Root
    type ForceOrigin = frame_system::EnsureRoot<AccountId>;

    // Use the MinNickLength from the parameter_types block.
    type MinLength = MinNickLength;

    // Use the MaxNickLength from the parameter_types block.
    type MaxLength = MaxNickLength;

    // The ubiquitous event type.
    type Event = Event;
}
```

![Configure a Pallet](assets/tutorials/playground/03-configure.png)

### Adding Nicks to the `construct_runtime!` Macro

Next, add the Nicks pallet to the `construct_runtime!` macro. This requires enumerating the types
that the pallet exposes. The complete list of these types can be found in the
[`construct_runtime!` macro documentation](https://substrate.dev/rustdocs/v2.0.0/frame_support/macro.construct_runtime.html).

The Nicks pallet exposes the following types:

- **Storage**, by way of the `decl_storage!` macro.
- **Event**s, by way of the `decl_event!` macro. In the case of the Nicks pallet, the `Event`
  keyword is parameterized with respect to a type, `T`; this is because at least one of the events
  defined by the Nicks pallet depends on a type that is configured with the `Trait` configuration
  trait.
- **Call**able functions, by way of the dispatchable functions defined in the `decl_module!` macro.
- The **Module** type, by way of the `decl_module!` macro.

With this in mind, add the Nicks pallet to the `construct_runtime!` macro as follows:

**`runtime/src/lib.rs`**

```rust
construct_runtime!(
    pub enum Runtime where
        Block = Block,
        NodeBlock = opaque::Block,
        UncheckedExtrinsic = UncheckedExtrinsic
    {
        /* --snip-- */

        /*** Add This Line ***/
        Nicks: pallet_nicks::{Module, Call, Storage, Event<T>},
    }
);
```

Note that not all pallets will expose all of these runtime types, and some may expose more! Always
look at the documentation or source code of a pallet to determine which of these types need to be
expose. The Substrate codebase
[ships with a node](https://github.com/paritytech/substrate/tree/v2.0.0/bin/node) that includes many
of the core FRAME pallets in its
[runtime](https://github.com/paritytech/substrate/blob/v2.0.0/bin/node/runtime/src/lib.rs) - it is
an excellent resource that provides relatively straightforward examples of how many of the core
FRAME pallets are added to a runtime.
