---
description: Learn about convenience libraries.
---

# Convenience libraries

Various convenience libraries exist for dapp developers.
Some of them simplify the creation of specific user interface elements, some entirely manage the
user account onboarding, and others give you a variety of methods for interacting with smart
contracts, for a variety of API preferences (for example, promises, callbacks, and strong types).

The [MetaMask Ethereum provider API](wallet-api.md#ethereum-provider-api) is lightweight, and wraps
[Ethereum JSON-RPC](wallet-api.md#json-rpc-api) formatted messages, which is why
some developers use a convenience library for interacting with the provider, such as
[Ethers](https://www.npmjs.com/package/ethers) or [web3.js](https://www.npmjs.com/package/web3).
You can refer to those tools' documentation to use them.

You can use [MetaMask SDK](/sdk), which provides a reliable, secure, and
seamless connection from your dapp to MetaMask.
It onboards users smoothly from multiple dapp platforms using the MetaMask extension or
mobile app, and your dapp can call any [Wallet API](wallet-api.md) methods with the SDK installed.
