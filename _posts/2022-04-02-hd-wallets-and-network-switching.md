---
layout: post
title: "HD wallets and network switching"
date: 2022-04-02 12:35:33 +0100
tags: ethereum
---

Blockchain 'wallets' are generally just pairs of [public and private keys](https://en.wikipedia.org/wiki/Public-key_cryptography) with some UI wrapped around them.[^1] We take the private key, and use it to derive the public key, which we then use to derive the wallet's address.

What's important is that the process of derivation is very difficult to reverse, in the same way that a hashing function is difficult to reverse: the chance of you guessing the private key correctly at random is about the same as selecting one atom from all the atoms in the universe – and there's no better way than guessing at random.[^2] We can therefore use the wallet address publicly, being able to prove mathematically that we own it, without ever leaking information about the private key we used to generate it.

This works great, until you need more than one wallet. You might be concerned about privacy, or you might want to keep certain types of transactions separated for tax or other organisational reasons. If you have more than one wallet, you need to manage more than one set of private keys, back each key up separately, store each key separately, restore each key separately, etc. This presents a user experience problem: it is inconvenient, and clunky, and pushes a lot of the infosec responsibility onto the user. That might be acceptable for a bunch of nerds or anarcho-libertarians, but isn't going to cut it for the median user.

The agreed-upon solution to these UX problems is Hierarchical Deterministic (HD) wallets, proposed in the Bitcoin BIP-32/44 standards and used by most other chains. This post considers this standard, how we're not meeting it, and why it matters.

The plan, in three sections:

- A short overview of what HD wallets are. Feel free to skip over this if you're familiar with the spec already.
- A discussion of how common wallets are not meeting this standard
- A discussion of why that matters, and what we could do about it.

#### HD Wallets

Hierarchical Deterministic (HD) wallets take the basic derivation mechanism and encode _structure_ into it. We take a master password – a single thing for the user to remember, to back up, etc. – and combine it with a _path_, a string following an a priori agreed-upon schema that allows us to generate multiple private keys from the same master password.

But it needn't actually have much structure at all. You could simply take a master password and append `1`, `2`, `3`, and so on, to generate different wallet addresses. This strategy would generate perfectly usable wallets with no obvious link between them. And since the generation process follows the same general sort of process as it does for the single-key case, the generation process produces hashed values that are similarly difficult to reverse.

We therefore only really need two pieces of information to calculate our wallet address:

- Our master password
- Some sort of _seed_

The master password is the user's responsibility; it's her input, her secret. What seed should we use?

One option is to let the user specify whatever sort of seed she wishes. But this doesn't really solve our problem: instead of multiple private keys, we instead have to deal with a single password plus multiple paths. We've just given ourselves more passwords to remember.

Another is to do what I suggested above: append an incrementing integer to the end of it to generate different wallets. This is equivalent to giving ourselves more passwords, but at least there's some rationale to it: our first wallet has a 1 at the end, our second wallet a 2, etc. It gives us some psychological safety: it means that our wallet is recoverable (assuming we can remember which number we used to generate it, or assuming we don't mind iterating through a few guesses). This approach is fine, as far as it goes, but this is crypto, so, given the opportunity, we should make it more complicated.

A third approach is to develop a _common standard_ for generating our seeds with more variables than just an incrementing number. This way, we can describe a tree structure independent of its values, embedding multiple values with which we might want to generate differing wallets. The benefit to this approach is that we can encode information about the purpose of the wallet into the seed itself, and then recover it later using our knowledge of those purposes without having to remember many arbitrary numbers. The standard gives us the template, and the purposes give us the values of the variables; all we have to do is fill them in. The other benefit to using a common standard is that wallet software can implement the standards too, so you don't need to generate the wallets off-site somewhere.

This standard is called [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) (it was originally a Bitcoin standard), and it presents this exactly this sort of predictable tree structure that we've been discussing. The goal here is minimises user input and maximise the number of wallets that can be generated with a single master password.

The standard calls the seed a _derivation path_, since it's a path in a tree that we append to a master password and use the resulting string to derive a public address. The standard gives derivation paths the following structure:

    m/purpose'/coin'/account'/change/index

And here's the trick: most of these values are knowable by the wallet software, based on what sort of wallet you're using:

- `purpose` is always `44'`.[^3] They gave it a value to allow them to upgrade the standard if they wanted to.
- `coin` varies depending on the crypto network. For instance, `coin = 60'` is Ethereum mainnet, and `coin = 966'` is Polygon.
- `account` gives the wallet a degree of freedom to support multiple user accounts (c.f. to the `/Users/username` directory on your OS)
- `change` will generally be `0`; it refers to whether the wallet should be used externally, or whether it should be use internal to the wallet for Bitcoin-based transaction change reasons. I've read somewhere that Ethereans sometimes use it, though for what I'm not sure.

The only non-guessable input value is `index`, which gives the user a degree of freedom to generate multiple wallets for under the same tree. This parameter is why the user can generate many wallets for a single password: she can keep incrementing `index` to generate more! It's also exactly the same as my much simpler idea discussed previously.

These parameters then get put into the structure, like so:

    m/44'/60'/0'/0/2

The structure then gets combined with the master password (or, more precisely, with a key generated from the master password), and users (or wallets) can vary `coin`, `account` and `index` to generate various wallet addresses.

#### Existing UIs and a Subtle Incompatibility

This isn't a huge, bombshell-dropped discovery, I'll admit it, but I've noticed that most wallets with support for both HD wallets and network switching don't actually implement the BIP-44 correctly, or, at least, there is a tension between the model used for network switching and the model used for wallet generation.

Generally, what happens is:

- Users add a master password (or its equivalent in the form of a mnemonic phrase) from which the wallet derives a single keypair
- As far as I can make out, the 'default wallet' generated through this mechanism still uses the HD standard, it just relies implicitly upon the `m/44'/60'/0'/0/0` derivation path (i.e. "give me external index 0 at account 0 for the Ethereum chain").
- When the user switches between compatible chains – from Mainnet to Arbitrum, for instance – the wallet software uses the same wallet address and private key to sign new transactions. It just switches the RPC endpoint it uses to make the request.

If wallets were to follow the standard correctly, they would be varying the `coin` value when switching networks, generating different wallet addresses for use depending on the network being used. In other words, according to BIP-44 at least, there's no such thing as a 'cross-network address' – and existing wallets ignore this subtle fact entirely.

I've been looking at how various different wallets handle this, and they all seem to do the same thing:

- Metamask's network switcher is [entirely independent](https://metaschool.so/articles/how-to-change-add-new-network-metamask-wallet/) from the wallet list, allowing the user to switch networks on the current wallet, even if that wallet was generated through a derivation path
- MyEtherWallet do the same thing, [switching the network URL](https://github.com/MyEtherWallet/MyEtherWallet/blob/8703b504e8b1166877ac7b2cc2a75e08614bd8a4/src/core/store/wallet/actions.js#L34) used for chain interactions and not (as far as I can see) adjusting the corresponding wallets.
- Similarly, there is nothing in [the WalletConnect spec](https://docs.walletconnect.com/client-api#approve-session-request-connect) preventing this behaviour, meaning that any HD-compatible wallet software using the protocol facilitates wallet-independent network switching

The problem is not so much that nobody's trying to follow the spec. The problem is that the spec is ambiguous with respect to the UI in which it's being implemented. The community therefore has implicitly converged on this non-standard behaviour because of the ostensible UI benefits. This has created an implicit standard incompatible with the original BIP-32/44 proposals.

It gets even more confusing when you notice that there is a third, Ethereum-specific standard, [EIP-601](https://eips.ethereum.org/EIPS/eip-601), designed to modify the BIP-44 standard for Ethereum use cases. From a brief google, I can't see any mentions of 601 that aren't merely links to the spec itself. But this ambiguity – what should happen to the valid wallet list when the user switches networks? – isn't resolved by EIP-601 either.

This ambiguity is born because the BIP-32/44 standards were built around the assumption that the different networks a user might switch between were mutually incompatible. It didn't foresee the rise of EVM-compatible layer 2s, and a range of dapps built to run on several of them concurrently, and therefore the capacity for the user to switch between them easily, in-app.

#### Why this matters, and what to do

Of course, this doesn't seem like a critical problem – there are bigger problems we could be tackling, for sure. Indeed, there's even something comforting about going from Polygon to Ethereum Mainnet and taking your address with you. It's certainly convenient. But this isn't what the BIP-32/44 specs say, and I think there actually are good reasons to obey them more precisely:

1. **It makes it possible to upgrade the spec in the future.** The standard can evolve safely, and those implementing it correctly are able to evolve without having to hack in workarounds for backward compatibility, and keep track of previous fringe behaviours.

2. **It makes interoperability with other wallets easier.** Wallet onboarding and offboarding isn't a light matter; the more activation energy required to move to one wallet from another, or from no wallet at all, the more intimidating crypto as a whole will become to the marginal user. Problems at the tail-end often get publicised more than problems at the mean.

3. **Not doing so undermines one of the main reasons to use HD wallets in the first place:** HD wallets allow you to keep public references to different addresses separated, increasing privacy. A wallet address that comes with you cross-network just makes your transactions that much easier to track.

Fortunately, I don't believe that the UI concessions made by existing wallet implementations need to be locked in. There are some steps that wallets could take today, such as triggering a confirmation model when changing networks, that would enable users to opt-out of the spec. Many users don't knowingly use HD wallets at all; in these cases, the default behaviour could just clear the wallet list and regenerate using the standard specs on network change.

Or, alternatively, we could develop a new, more parsimonious standard to capture the semantics of cross-chain wallets, compatible with the current UI approach. One simple method would be to amend the current spec such that `network = 0` means 'no specific network', allowing cross-chain wallets to be represented in the existing spec. If a network changes while a user is connected with a wallet known to be generated with `network = 0`, the wallet persists.

Either way, this is the exactly the sort of subtle incompatibility that could prove to be an increasing nuisance, compounded by the ongoing growth in usage of layer 2s. Our standards for network switching were designed at a time when the only networks we would switch between were testnets. Today, the UI implications of network switching are a lot more important. And, today, that is incompatible with one of the most useful standards we have for managing multiple wallets.

Multiple wallets, multiple networks, good UX. We don't need to pick only two.

[^1]: The name wallet is therefore a misnomer, since the wallet itself doesn't store anything; it's much closer to a username and password for online banking, than the vault itself.
[^2]:
    Ethereum private keys are 256 bits. Since a bit has two possible states, guessing a 256
    bit sequence correctly at random has a chance of `1/2^256`. There are ~10^78 atoms in the observable universe, which is ~2^260. If you know the Ethereum address of the wallet you're trying to get into it's slightly easier, since wallet addresses are only 160 bits long, but it's still [a very big number](https://www.youtube.com/watch?v=S9JGmA5_unY).

[^3]: The apostrophe in the path tells the key generation algorithm to use the 'hardened' form of the derivation, which puts extra constraints on the derivation such that derived public keys can't be proven to be derived from a given parent public key using only public information. The [details here](https://wiki.trezor.io/Hardened_and_non-hardened_derivation) are a little tricky, and outside the scope of this post.
