---
title: Introduction
---

As [the first tutorial](../create-your-first-substrate-chain) demonstrated, the
[Substrate Developer Hub Node Template](https://github.com/substrate-developer-hub/substrate-node-template)
is a working blockchain node that is designed to be a flexible starting point for a new
Substrate-based blockchain. The Node Template is comprised of
[a number of components](../../index#architecture), including a
[runtime](../../knowledgebase/getting-started/glossary#runtime) that is constructed using the
[FRAME](../../knowledgebase/runtime/frame) system for runtime development. FRAME takes a
compositional approach to runtime development, which means that a custom runtime is created by
[composing modules](../../knowledgebase/runtime/macros#construct_runtime), known in FRAME as
"pallets". The Substrate codebase ships with
[dozens of battle-tested pallets](https://github.com/paritytech/substrate/tree/v2.0.0-rc6/frame) and
the goal of this tutorial is to explain and demonstrate the steps that are taken when adding a
simple pre-built pallet to a FRAME runtime. Subsequent tutorials will explain how to
[write a new FRAME pallet](../build-a-dapp) in order to encapsulate custom logic.

In addition to covering the basics of adding a pallet to FRAME runtime, this tutorial will
demonstrate how to use the [Polkadot-JS Apps UI](https://github.com/polkadot-js/apps) to interact
with a Substrate node. The UI will be used to invoke extrinsics, explore the blockchain network,
query its storage, invoke privileged functions and troubleshoot errors.

If you have problems with this tutorial, the Substrate community is full of helpful resources. We
maintain an active
[#SubstrateTechnical chat room](https://app.element.io/#/room/!HzySYSaIhtyWrwiwEV:matrix.org) and
monitor the
[`substrate` tag on Stack Overflow](https://stackoverflow.com/questions/tagged/substrate). You can
also use the [`subport` GitHub repository](https://github.com/paritytech/subport/issues/new) to
create an Issue.

## Substrate Playground

This tutorial will use the [Substrate Playground](https://playground.substrate.dev/), an online IDE
that makes it easier to build with Substrate by abstracting system dependencies and providing a
pre-built Node Template. The Playground uses GitHub authentication, so users are required to have
[a GitHub account](https://github.com/join). Login to the Playground, make sure that "Node template"
is selected, and click "CREATE".

![Launch Playground](assets/tutorials/playground/01-launch.png)
