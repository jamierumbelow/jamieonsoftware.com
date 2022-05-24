---
layout: post
title: "Decentralisation is a trade-off space"
date: 2022-05-24 21:36:22 +0100
tags: ethereum decentralisation
---

When you ask a crypto-sceptic what decentralisation means wrt crypto, they offer these sorts of properties:

1. Many nodes are run across many jurisdictions; computation happens in a way that can't ever be shut down or censored
2. Individuals can access the system for [whatever nefarious purposes](https://twitter.com/SGBarbour/status/1528432799941808130) they wish, without the need for inter alia KYC checks
3. Power is completely diffused rather than concentrated in any one party; therefore, no individual party is responsible, either legally, operationally, morally, or all of the above.

These sorts of conditions actually refer to what I'll call 'maximal decentralisation' – a system which is decentralised in every way possible, up to whatever point of diminishing marginal returns seems appropriate. It is different from decentralisation, which refers to a system that can have more or less participation, more or less accessibility, more or less diffused power.

This might seem like an unimportant semantic distinction, but not all semantic distinctions are meaningless, and some are indeed important. How we define our terms matters, because it limns the shape of what we build and gives us criteria against which we evaluate our success. Moreover, if we disagree over what we mean by a term like 'decentralisation', we end up arguing fruitlessly. We exchange claims and both sides miss the point.

A lot of crypto people also believe, somewhat reflexively, that we should be aiming for maximal decentralisation. The worst crypto-fanatics are just as ideological as the worst crypto-sceptics. This is a problem. People shouldn't be fanatical.

For almost every category, maximalism is bad. It ties you into a priori commitments, which reduce your optionality and cripple your ability to adjust to new facts. Strong beliefs are important, but they should generally be held weakly. It also makes you much more likely to act tribally. Humans are tribal, and 'decentralise everything' can become a rallying cry, a standard around which troops array for battle. It makes technical questions political, and [politics is the mind-killer](https://www.lesswrong.com/posts/9weLK2AJ9JEt2Tt8f/politics-is-the-mind-killer).[^1]

Crypto people shouldn't be fanatical, because computing is the art of the science of trade-offs, and 'decentralisation' gives us a lot more room for manoeuvre than either its critics or fanatics seem to allow. Trade-offs are useful, and understanding the trade-off space is most of the work.

I know that crypto people don't actually want decentralisation per se, because when you ask why decentralisation matters, you get answers like:

> It makes the technology more accessible and more censor-resistant. It gives broader access to a broader set of people, especially those less well-served by existing financial infrastructure.

Which means that their motivations are prior to decentralisation. They want accessibility! They want censorship-resistance, and greater equality of opportunity!

We also want other things. Decentralisation means more competition, and more competition means more innovation. (Diversity is good! Let a thousand flowers bloom!) The ideas being generated and built upon by the crypto community are not commodities: they are meaningfully new contributions to a meaningfully new technology stack; we get more ideas when we have a more competitive space, and we get more competition when people are able to use the technology and gain access to the liquidity needed to prove the new ideas out.

More subtly, another important corollary of decentralisation is that it leads to implicit coordination through standards. And standards make composability possible. So decentralisation is also good because it leads to more composable technologies.

So how then can we think about decentralisation in a more yielding way? How can we make it more supportive of our broader aims – accessibility, censorship-resistance, competition, composability – without holding ourselves hostage to its demands, on the one hand, or giving our opponents a stick to beat us with, on the other?

First, we must realise that decentralisation is a spectrum. It isn't a binary state. Systems can be more or less decentralised, and what we're actually arguing over is how decentralised a given system should be.

Second, we must realise that there are different sorts of components in a web3 system, and decentralisation doesn't just apply to the computation. There are the RPC nodes, the API interface into the computation layer: they can be more or less decentralised. There are frontends, which serve user-friendly interfaces to interact with the contracts via these RPC nodes: they can be more or less decentralised. There is the data itself, the state, which can be more or less decentralised. There is the liquidity, the value locked in the system, which can also be thought of as more or less decentralised. Etc etc etc.

Third, we must realise that how decentralised a given system should be depends on the purpose of that system. A platform that needs to perform a lot of complicated computation can trade off a bit of decentralisation by pulling some computation off-chain.[^2] Many web3 frontends use the semi-decentralised [The Graph](https://thegraph.com/en/) to provide indexing, trading off some centralisation in exchange for faster data retrieval. Almost no web3 frontend uses decentralised alternatives to the substrate of internet technologies on which they sit, DNS and web hosts and friends. This is all okay.

Fourth, we must realise that often just the ability to decentralise means that the goals of decentralisation can be achieved. In other words, centralised parts of a decentralised system don't immediately make the system totally centralised – it's not a binary – and therefore useless vis-a-vis the goals of decentralising. This is one of the reasons why [moxie](http://moxie.org/2022/01/07/web3-first-impressions.html) is wrong: while it's true that there are a few dominant hosts of RPC nodes, such as [Alchemy](https://www.alchemy.com), it is very easy to run nodes, and would not be especially capital intensive to start competitors.[^3] Alchemy won't abuse their power, because if they did it would be extremely easy to compete with them. The fact that the computation layer and its implementation – e.g. the geth codebase – is permissionlessly accessible means that there are downstream pressures on people who use those implementations to not abuse their power. 'Counterfactually decentralised' is a real thing!

Finally, and most importantly, we must realise that we have a choice, as builders and consumers, and that the truest expression of a decentralised system is one in which individuals are given freedom to engage (in the broadest possible sense) with the project (in the broadest possible sense) –– on their own terms. On this count, at least, crypto seems to be doing well, even though aspects of it are indeed centralised.

My point is that crypto-fanatics and -sceptics alike seem to think that the fight is over whether decentralisation is 'necessary', which is about as helpful as asking whether a bridge is strong enough, or a cow is fat enough, or a child is confident enough.

Necessary for what?

[^1]: I hope to write more about the relationship between fact-questions and values-questions in the future. I feel intuitively and reflexively annoyed by the post-modern adage that everything is political, because grounding facts-questions in values removes our one objective standard, our one ability to offer a response beyond 'I don't like it', not to mention all the other socio-psychological effects that make respectful debate impossible and seem to have poisoned the epistemic commons.
[^2]: Ethereum does this itself by not permitting contracts to autonomously call other contracts; mechanisms like [MakerDAO's liquidation process](https://makerdao.world/en/learn/vaults/liquidation/) need to be triggered by off-chain actors incentivised through rewards. There are ways of solving the obvious trust problems, usually through incentives, if you believe they are problems. Much of the time they are not.
[^3]: We know this, since there actually are [a bunch](https://ethereum.org/en/developers/docs/nodes-and-clients/nodes-as-a-service/) [of](https://developers.cloudflare.com/web3/ethereum-gateway/) [competitors](https://infura.io).
