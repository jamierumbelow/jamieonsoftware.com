---
layout: post
title: "Wallets As Identity"
date: 2022-06-27 10:14:21 +0100
tags: ethereum
---

_This is a very rough first draft of a chapter from my upcoming book on Product Engineering in Ethereum. Any and all feedback welcome!_

Technologists use metaphors to make **skeuomorphic links**, bridging old paradigms to new. The 'desktop' and 'trash can' and even the keyboard is a form of skeuomorphic link, giving users of a new platform enough conceptual and aesthetic resources to make sense of it, to feel familiar, connected to it. Computers and interfaces are physical, and rely on all the same instinctual responses, the same clusters of predispositions and responses attached to entities in the physical world. So new technologies can co-opt these instincts by creating these skeuomorphic links.

Perhaps the most common example of skueomorphich linkage in crypto is the metaphor of the 'wallet'. This post argues that it is one of the most misleading.

A quick diversion on the structure of metaphors: metaphors behave by implying certain things about the subject based on the object. Good metaphors are good when the subject does indeed resemble the object in those ways. Good metaphors also have the nice property of illuminating the subject of the metaphor in ways that the reader doesn't expect. A good metaphor can help reveal the shapes and contours of the subject by making us consider the subject from a new angle. 'All the world's a stage', Jacques soliliquises in **As You Like It**, and the entrances and exits and performance that this equivalence conjures up help illuminate the human condition, revealing it to be a form of entrapment, confinement to a pre-determined script.

What, then, does the metaphor of 'wallet' imply? It suggests a few features:

1. A wallet holds things

2. A wallet holds a specific sort of thing, namely financial things (such as credit cards and cash)

3. A wallet is a physical item, and plays the various roles that physical items do, such as signalling style, wealth, and status

Crypto wallets – what we'll call 'accounts' from now on to avoid confusion – have none of these features. Accounts don't hold things: accounts represent identity. Accounts aren't limited to interacting with financial products: they interact with anything representable on the blockchain. Accounts, of course, aren't physical items: they are an abstraction over an individual on a ledger.

It's worth stopping at this point and asking why any of this matters. Isn't it just a shallow, semantic dispute? No. Because as well as being illuminating about the subject, a metaphor can often **constrain** our understanding of the subject, especially if it becomes the primary mechanism through which the subject is known.

This has happened with the wallet metaphor in crypto, which is why I believe it's especially important to pay attention to what wallets actually are, how the metaphor shapes our thinking, and how it limits our thinking too. All the world is a stage, maybe, but if it is we still have plenty of freedom to improvise.

### Accounts don't hold things

An account is a pair of public and private keys. A private key is used to generate a public key, and the public key is used to generate an address. So when you see account addresses such as `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` (one of Vitalik's accounts), what you're really seeing is the result of passing Vitalik's private key through the set of cryptographic functions that produce these addresses.

The details of how this works are interesting, but out of the scope of this post. All we need to know for now is that this process is **deterministic**, in the sense that it returns the same value for the same input, no matter how often you run it, and **irreversible**, in the sense that it's very very hard to get the private key from the wallet address.[^1]

So if you know the private key then you can generate the public key, but you can't (easily) go in the other direction. This is one of the reasons why it's so important to keep your private key private: your private key is the one piece of information somebody needs to get access to your account. Fortunately, most users don't have to worry about the content of the private key at all: they can use wallet software that holds the private key in a secure way. For product engineers, you'll always need to keep in mind that the private key is sacred.

How does this link to the wallet metaphor?

Well, I lied a little: accounts do 'hold' something. They hold ether, the native token of the Ethereum blockchain. But this is by communal assent: a group of people, incentivised to follow the same set of rules, collectively agree that a certain amount of ether is owned by a specific account, and another amount of ether is owned by a different account. It's the collective assent that underwrites the ownership claim, not the account itself.

If you've got cash in your wallet, you actually do have cash in your wallet. A portion of the 'state' of the money-system is in your pocket. If you've got ether in your account, you've got a claim on the group for the amount of that ether. It's a guarantee, secured by cryptography and incentives, that when you decide you want to do something with that ether we'll all agree that you can.

So I'm also telling the truth: accounts don't really hold that much at all, and the notion of 'holding' your ether and your tokens is a euphemism. Your account is the mechanism of a record of ownership. The state is not in your pocket, it's 'over there'.

This might seem a little opaque, so let's consider some code. One of the clearest examples of how accounts don't actually hold anything is the ERC-20 token standard.

This standard is the core primitive in DeFi, the basic lego brick upon which we build this cathedral. It offers a consistent interface for smart contracts that behave as a record of ledger. Contracts have to choose to implement it – that's why it's a standard, not a rule – but those that do share the same methods.

We can distil the essence of ERC-20 down to only two function signatures:

    function balanceOf(address _owner) public view returns (uint256 balance);
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success);

(The actual standard contains more than these two methods, but the rest are for convenience, expedience, and extensions to the functionality, rather than being central to the model of how tokens are represented by contracts that implement it.)

These two functions tell us everything we need to know about a piece of state. What is that piece of state? It's a mapping from address to the number of tokens owned by that address:

    mapping(address => uint256) public balanceOf;

`balanceOf` is a map – otherwise known as a dictionary, or an associative array – from an owner to the total amount owned. The total amount owned of what? The token that the contract represents.

If we want to find out how much of the token we own, we call `balanceOf`, passing our address. If we want to send our tokens to somebody else, we call `transferFrom`, passing our address, the recipient's address, and the amount we wish to send. The token contract does the work in updating the list it holds.

Think about this carefully: **a token is just a list of who owns the token**, plus a bit of metadata (such as the token's name and symbol).

Your account doesn't 'hold' anything. It doesn't know anything about the token, per se. It just gives you a way of identifying yourself as the owner of some amount of the token. And a token is just a piece of code that understands how to update this `balanceOf` map.

This is a very simple and powerful idea, because the token contract is able to stipulate its own rules and policies around who can transfer, who can be transferred, and what the balance is.

You could stipulate that only a specific whitelist of addresses should be able to receive the token, for instance, restricting ownership of the token to a known set of individuals. You could exclude certain addresses. You could require a certain amount of ether in exchange for the token, or even a certain amount of another token in exchange for the token. You could even **change the amount transferred when somebody transfers it**, deducting a fee. In other words, you can implement a monetary policy, controlling how the token enters the market, how it leaves it, and how it flows around between participants.

### Accounts aren't limited

Since the token contract is the mechanism for recording who owns what, and stipulating the rules that govern that ownership, your account isn't limited to holding financial instruments like cash or credit cards. Tokens can be used to represent basically anything.

Some tokens represent non-fungible assets, things that can't be exchanged for another asset just like it. You will have heard of NFTs, and NFTs are token contracts much like the ERC-20 contract we just discussed. The biggest difference between ERC-20 and ERC-721 – the basic standard for representing NFTs – is that the parameters of the `transferFrom` function are slightly different:

    function transferFrom(address from, address to, uint256 id) public virtual;

An ERC-721 contract introduces the notion of an `id`, and requires that the pair `(owner, id)` is globally unique. So a specific token – that described by the contract and the specific ID the contract defines – is owned by one and only one person.

NFTs are often used for digital art or collectibles, but they can be used to represent ownership of lots more. In time, they may be used to represent ownership of real-world assets: houses, cars, paintings, bottles of wine.

NFTs can also be used for interesting, crypto-native use cases: Uniswap v3, for instance, mints an NFT that represents a deposit to a specific pool within a specific price range. Each deposit has its own custom configuration, and so the pool mints an NFT that represents your specific deposit with your specific settings.

Fungible tokens, too, can be used to represent many more types of things than just those that map to financial value. They can represent governance rights, as they do with the `TRIBE` token, which grants the right to vote on proposals of the Tribe DAO. (At the time of writing, most governance tokens grant voting rights on a quantity basis: if you have more tokens, your vote represents more of the. But this isn't the only model for governance design made possible by the ERC-20 token standard). They can also represent other sorts of values, such as carbon credits, or reputation, or points accumulated in a game.

The point is that the 'wallet' metaphor suggests a constraint on the sorts of things that can be 'held'. Since accounts don't 'hold' things, the design space for what sorts of ownership and participation claims they can represent is much, much bigger.

### Accounts aren't physical items

This might seem like a silly point to make, but the fact that accounts aren't physical items runs a little deeper than the obvious. Accounts aren't physical items: they are abstractions over an individual, a face you can show to the blockchain.

And this means that your accounts aren't tied to one place. If you know your private key, you can use your account in whatever interface you want.

This means that **different interfaces can display different aspects of the same account**. Some interfaces – CHECK: such as what? – put an emphasis on NFT collectibles, a kind of gallery. Others might focus on the purely financial aspects of the account, its deposits and loans and investments. Perhaps an interface could be built around governance, showing the proposals and votes and engagements with protocols. Perhaps another could be built around whatever gaming tokens the account has claims to. And so on.

When you start to consider these sorts of product implications of the fact that accounts aren't physical items, it makes you realise that the design space for account interfaces is co-extensive – overlapping with – the design space for tokens themselves.

It also pushes the sorts of signalling properties we mentioned earlier elsewhere. Perhaps people do signal with their **wallet software** – i.e. their account interfaces – a little, using a newer or more feature-complete piece of software to indicate the sort of user they are, the sorts of things they are interested in. But the account itself is a very lightweight thing.

And because it's lightweight, it can be transported, as we've discussed. It can also be 'hosted' on-chain, which opens up the possibility of smart contract wallets and multi-signature wallets.

### Accounts are identity

So what is an account, then, if not a wallet?

My answer: an account is an **identity**. It is a way of informing other participants that you are who you say you are. It is a username and password. It is a mechanism to authenticate yourself.

If accounts are identity, then it means you can separate out your identities depending on different contexts of usage. You might use an account to trade, another account to represent your investments, a third to hold your claims to any artwork or collectibles. You might use accounts for different platforms with different risk profiles, or with different privacy considerations. You can separate out your identities in whatever way you choose, isolating your public, on-chain behaviour into different categories. The ownership of a private key gives you full control over what aspect of yourself you wish to show to the world.

Products built on Ethereum need to understand this multiplicity of identities, and implement support for it in a deep way. We'll see how this works more concretely in other posts. But for now, consider that **your identity goes with you wherever you take it**. And where your identity goes, so do your claims of ownership.

This, combined with the standardisation of interfaces such as ERC-20 and ERC-721, increases the power of the interfaces we can build substantially. To take a simple example: a tool for listing whatever ERC-20-compatible tokens are 'held' by an account will work with **any token that meets the requirements of that interface**. It's as if PayPal could support all currencies from day one, with no extra development work.

The wallet metaphor is generally broken, then, and can severely constrain your imagination vis-a-vis what accounts are capable of. For sure, there are ways in which an account can seem like a wallet: when you lose your account, just like your wallet, you lose the money in it; to that extent, the metaphor is true. But it's true for different reasons. If you lose a physical wallet you've actually lost the cash inside it. If you lose a crypto wallet, you've lost your ability to prove you are the person who owns the cash.

We are likely to continue using the term 'wallet', since that's what the community uses, and there's not much marginal benefit in changing this sort of heavily-entrenched meme. I use 'wallet' and 'account' interchangeably. But I hope this chapter has given you a sense of how accounts actually work, and why the design space for them is an awful lot bigger than a piece of folded leather that you keep in your pocket.

[^1]: How hard? There's no better way than guessing at random. Ethereum private keys are 256 bits. Since a bit has two possible states, guessing a 256 bit sequence correctly at random has a chance of 1/2^256. There are ~10^78 atoms in the observable universe, which is roughly 2^260. Account addresses are only 160 bits long, which means that there are multiple possible private keys, but 2^160 is still a very big number, equivalent to a little bit less than the total number of atoms on Earth.
