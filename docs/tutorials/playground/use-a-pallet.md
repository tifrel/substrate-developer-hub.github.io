---
title: Interact with the Nicks Pallet
---

The Playground makes it easy to launch a Substrate node and connect to it from a front-end. Click
the Substrate icon on the far-left side of the Playground, expand the "Nodes" tab and click the
button to "Compile & start node". Accept the default options (`--dev --ws-external`).

![Run the Node](assets/tutorials/playground/04-run.png)

Once the node is running, use the "Processes" tab to launch "Polkadot Apps".

![Run the Node](assets/tutorials/playground/05-ui.png)

## Use the Nicks Pallet

The "Polkadot Apps" button will launch an instance of the Polkadot-JS Apps UI that is pre-configured
to connect to the Playground node. In this section, this UI will be used to interact with the Nicks
pallet as well as invoke privileged functions with the
[Sudo pallet](https://substrate.dev/rustdocs/v2.0.0/pallet_sudo/index.html), which is included by
default as part of the Node Template. This section also demonstrates how to interpret the different
types of [events](../../knowledgebase/runtime/events) and errors that FRAME pallets may emit.

Use the "Developer" > "Javascript" component and the script provided below to encode the desired
nickname as hex; make sure the nickname is no shorter than the `MinNickLength` and no longer than
the `MaxNickLength` that was configured in the previous step.

```js
const nickname = "Substratio";
console.log(
  nickname
    .split("")
    .map((c) => c.charCodeAt(0).toString(16))
    .join("")
);
```

![Encode Nickname](assets/tutorials/playground/06-encode-name.png)

Next, use the "Developer" > "Extrinsics" component to call
[the `setName` dispatchable](https://substrate.dev/rustdocs/v2.0.0/pallet_nicks/enum.Call.html#variant.set_name)
function from the `nicks` pallet and provide the hex-encoded nickname as the parameter (make sure to
include the leading `0x`). Use Alice or one of the other pre-funded development accounts.

![Set Nickname](assets/tutorials/playground/07-set-name.png)

Click "Submit Transaction" then "Sign and Submit". Once the transaction has been submitted, navigate
to the "Network" > "Explorer" component and review the
[events](https://substrate.dev/rustdocs/v2.0.0/pallet_nicks/enum.RawEvent.html) that were emitted.

![Events](assets/tutorials/playground/08-events.png)

Along with each event, there is a tuple that indicates the block in which the event occurred as well
as the index of the event within the list of events associated with that block. These tuples are
links to pages with detailed information about the block. Click the link next to the
`nicks.NameChanged` event to see more information about the block in which it occurred, including
the `nicks.setName` extrinsic.

![Block Detail](assets/tutorials/playground/09-block.png)

Notice that the Polkadot-JS Apps UI is using the new nickname instead of "Alice" to identify the
account that submitted the extrinsic. Navigate to the "Developer" > "Chain state" component to read
the nickname for the account formerly known as "Alice" from the
[runtime storage](../../knowledgebase/runtime/storage) of the Nicks pallet.

![Read a Name](assets/tutorials/playground/10-name-of-alice.png)

The return type is a tuple that contains two values: the nickname and the amount that was reserved
from Alice's account in order to secure the nickname. Query the Nicks pallet for Bob's nickname;
notice that the `None` value is returned. This is because Bob has not invoked the `setName`
dispatchable and deposited the funds needed to reserve a nickname.

![Read an Empty Name](assets/tutorials/playground/11-name-of-bob.png)

Navigate back to the "Developer" > "Extrinsics" component and use the account formerly known as
"Alice" to invoke
[the `killName` dispatchable](https://substrate.dev/rustdocs/v2.0.0/pallet_nicks/enum.Call.html#variant.kill_name)
function; use Bob's account ID as the function's argument. The `killName` function must be called by
the `ForceOrigin` that was configured with the Nicks pallet's `Trait` interface in the previous
section. You may recall that we configured this to be the FRAME system's `Root` origin. The Node
Template's
[chain specification](https://github.com/substrate-developer-hub/substrate-node-template/blob/v2.0.0/node/src/chain_spec.rs)
file is used to configure the
[Sudo pallet](https://substrate.dev/rustdocs/v2.0.0/pallet_sudo/index.html) to give Alice access to
this origin. However, if the call is invoked directly by the account formerly known as "Alice" (as
opposed to by way of the Sudo pallet), a `BadOrigin` error will be returned.

![`BadOrigin` Error](assets/tutorials/playground/12-bad-origin.png)

An error of this type implies that the extrinsic was successfully _dispatched_ even though it did
not _complete_ successfully. This means that Alice's account was still charged
[fees](../../knowledgebase/runtime/fees) for the dispatch, but there weren't any state changes
executed because the Nicks pallet follows the important
[verify-first-write-last](../../knowledgebase/runtime/storage#verify-first-write-last) pattern.

Polkadot-JS Apps UI makes it easy to use the Sudo pallet to dispatch a call from the `Root` origin.
Navigate to the "Developer" > "Sudo" component and invoke the `killName` dispatchable with the same
parameter as before: Bob's account.

![Sudo Dispatch](assets/tutorials/playground/13-sudo.png)

This time, the result indicates that the dispatch completed successfully. However, navigate back to
the "Network" > "Explorer" component and review the recent events. The Sudo pallet emits a
[`Sudid` event](https://substrate.dev/rustdocs/v2.0.0/pallet_sudo/enum.RawEvent.html#variant.Sudid)
to inform network participants that the `Root` origin dispatched a call. In this case, the "inner"
dispatch failed with a
[`DispatchError`](https://substrate.dev/rustdocs/v2.0.0/sp_runtime/enum.DispatchError.html) (the
Sudo pallet's
[`sudo` function](https://substrate.dev/rustdocs/v2.0.0/pallet_sudo/enum.Call.html#variant.sudo) is
the "outer" dispatch). In particular, this was an instance of
[the `DispatchError::Module` variant](https://substrate.dev/rustdocs/v2.0.0/frame_support/dispatch/enum.DispatchError.html#variant.Module),
which contains two pieces of metadata: an `index` number and an `error` number. The `index` number
relates to the pallet from which the error originated; it corresponds with the _index_ (position) of
the pallet within the `construct_runtime!` macro. The `error` number corresponds with the index of
the relevant variant from that pallet's `Error` enum. When using these numbers to find pallet
errors, remember that the _first_ position corresponds with index _zero_.

![Sudo Dispatch](assets/tutorials/playground/14-error.png)

In the screenshot above, the `index` is `9` (the _tenth_ pallet) and the `error` is `2` (the _third_
error). Depending on the position of the Nicks pallet in the `construct_runtime!` macro, there may
be a different number for `index`. Regardless of the value of `index`, the `error` value will be
`2`, which corresponds to the _third_ variant of the Nick's pallet's `Error` enum,
[the `Unnamed` variant](https://substrate.dev/rustdocs/v2.0.0/pallet_nicks/enum.Error.html#variant.Unnamed).
This shouldn't be a surprise since Bob has not yet reserved a nickname.

Confirm that the Sudo component can be used to invoke the `killName` dispatchable and forcibly clear
the nickname associated with any account that actually has a nickname associated with it, such as
the account formerly known as Alice. Here are some other things to try:

- Add a nickname that is shorter than the `MinNickLength` or longer than the `MaxNickLength` that
  was configured with the Nick's pallet's `Trait` configuration trait.
- Add a nickname for Bob then use Alice's account and the Sudo component to forcibly kill Bob's
  nickname. Then use Bob's account and dispatch the `clearName` function.

## Adding Other FRAME Pallets

This guide specifically demonstrated how to add the Nicks pallet to a runtime, but, each pallet will
be a little different. Have no fear, the
[demonstration Substrate node runtime](https://github.com/paritytech/substrate/blob/v2.0.0/bin/node/runtime/)
includes nearly every pallet in the library of core FRAME pallets and is an excellent reference with
many helpful examples.

In the `Cargo.toml` file of the Substrate node runtime, there are examples of how to import each of
the different pallets, and in the `lib.rs` file there are examples of how to add each pallet to a
runtime. In general, use that as a starting point when adding a pallet to a FRAME runtime.

### Learn More

- Learn how to add a more complex pallet to the Node Template by completing the
  [Add the Contracts Pallet](../add-contracts-pallet) tutorial.
- Complete the [Upgrade a Chain](../upgrade-a-chain) tutorial to learn how Substrate enables
  forkless runtime upgrades and follow steps to perform two upgrades, each of which is performed by
  way of a distinct upgrade mechanism.

### References

- [Nicks pallet docs](https://substrate.dev/rustdocs/v2.0.0/pallet_nicks/index.html)
