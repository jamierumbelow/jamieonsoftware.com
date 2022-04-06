---
layout: post
title: "Notes on Solidity Fridays Episode with Transmissions11"
date: 2022-04-06 10:38:55 -0700
tags: ethereum
---

December's [Solidity Fridays with transmissions11](https://www.youtube.com/watch?v=58eKnYR_EyM) is really excellent, and I think a very high-leverage way to learn how good Solidity actually gets written. When the guest is especially good, as with transmissions, I put Solidity Fridays in the same sort of category as [Destroy All Software](https://www.destroyallsoftware.com) – teaching by communicating models and patterns of thought, rather than regurgitating tutorial content.

Anyway, here are my notes in case somebody else finds them useful:

- When emitting events, t11s emits instances of the contract rather than the address directly; the compiler will swap it out for an address anyway, but this approach gives you greater type safety

- Call `CREATE2` to revert if the contract has already been deployed, by adding a salt of the underlying address to the deployment

- Addresses are 20 bytes, salts have to be 32 bytes. So we call `fillLast12Bytes` to add the remaining bytes (provided in Solmate's `bytes32` library)

- The gas cost of `>` is equivalent to `!=` when the comparator is `0`, more expensive otherwise.

- Illustration of his opinionated approach to smart contract development: "the performance of the code for users who aren't stupid matters" - that's why the [[ERC20]] implementation in [[Solmate]] doesn't stop you from transferring into the contract's address. "I'm not raising the cost for everyone else"

- `fdiv` is like division, but accounting for the bases. 'Scale this down by the contract's base'. Multiply the numerator by `baseUnit` and then divide the `numerator * baseUnit` by the denominator. Also checks for overflow on `numerator * baseUnit`, since overflow isn't protected in assembly calls.

- In general, keep external calls all the way at the end of the function – including after any events are emitted – to make reentrancy more difficult.

- More important with eg ERC777 since eg `safeTransferFrom` might allow arbitrary code execution (https://eips.ethereum.org/EIPS/eip-777)

- Removing things from the end of an array is significantly cheaper than from the beginning, since in the latter case you have to move everything over.

- `uint256 currentIndex = withdrawalQueue.length - 1; for (; ; currentIndex--)` is better in this case than initialising the `uint56 i = withdrawalQueue.length - 1` since we're only doing effects (the tx will revert by underflow automatically), so we don't need to check the length. Saves gas.

- Add a `trusted` boolean to the strategies, which is then checked on deposit and withdrawal. Makes it easier for EOAs to manage vaults without having to be wrapped in some other contract. Also makes it possible to disable withdrawal from strategies easily if they're malicious in some way.

- Two reads to the same struct from `getStrategyData[strategy]` has no extra gas cost, since it gets optimised by the compiler into one single `SLOAD`. & makes it clearer to read by a dev where it's coming from.

- Use `unchecked` when you know you won't underflow or overflow, and so can therefore do without the safety. Saves gas.

- `unchecked` isn't leaky - it won't uncheck in nested function calls

- The implementation of Compound's cToken is a little funky, since in a lot of places function calls return an error code rather than revert. So sometimes you need to `require(cToken.blah() == 0)` to ensure it succeeded.
