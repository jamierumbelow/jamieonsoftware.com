---
layout: post
title: "AlphaTensor, Taste, and the Scalability of AIs"
date: 2022-10-09 10:26:00 +0100
tags: ai
---

This week's [AlphaTensor](https://www.deepmind.com/blog/discovering-novel-algorithms-with-alphatensor) announcement shows not only that AI is getting really good at doing things, but also that DeepMind are really good at getting AI to do things.

Their approach is a variant of the deep reinforcement-learning-guided Monte Carlo tree search that they have applied so successfully to playing Chess and Go. What they have done, very effectively, is to design a game with the objective of finding the most efficient tensor multiplication algorithm for a matrix of some dimension.

There are a number of important object-level takeaways from the paper:

- The search space for various algorithms and mathematical techniques is enormous
- Existing algorithms and techniques are often not provably the most globally efficient; there is room to improve
- We can improve an algorithm by paying attention to its implementation, as well as its theoretical virtues

But it also teaches us some things about the meta-level, and the benefits of DeepMind's distinctive approach:

- DeepMind's approach is scaleable vertically: the more time and compute they spend on a task, the more they can achieve.
- DeepMind's approach is scaleable horizontally: the more tasks they apply their approach to, the more tasks they can achieve.

I think these points suggest something important about the [scaling hypothesis](https://www.gwern.net/Scaling-hypothesis#scaling-hypothesis).

A weak version of the scaling hypothesis claims that most of what we need to improve AI performance is more compute. All we need to do is scale up the computational and data resources of a sufficiently scalable model, and we'll get much better performance. Give the right model architecture enough data, and enough compute/time, and our problem is solvable (up to the phyical limits, or something like that.)

Before I begin my argument, I'll make a presupposition: we won't get to AGI. My hunch is that AGI is a bit of a chimera, and that progress in AI will consist mostly in the discovery and application of specialist models for narrow-domain problem solving. Some models may generalise to some extent, some may not. But over time the progress of narrow-domain AIs will far outstrip the progress in broad-domain AIs, and the market will reallocate resources accordingly.[^1]

Given this presupposition, we can ask an important question: does the weak scaling hypothesis hold?

In a world in which AI is applied to solving narrow-domain issues, AI researchers have two major tasks:

1. How should they decide which tasks to work on?
2. How should they encode a given problem such that it is tractable for AI to solve?

(1) is hard to decide _a priori_, because it will depend on the path-dependence progress of science (and the supporting AI). Utility-maximising functions might be helpful here: what task has the lowest cost and the highest utility? But a lot will be baked into 'cost' and 'utility', and the various cost bases of tasks will change depending on how the technology evolves, what data is available, etc.

Instead, AI researchers will develop heuristics, industry and academic partnerships, new data sources, etc., and that will help narrow down the range of tasks susceptible to quick resolution. In short, AI researchers will rely on _taste_ and experience to guide their decisions over the medium- and long-term.

(2) is more analysable from first principles. Tasks need to be encoded in such a way as to make them solvable by the model: the model architecture needs to be designed in such a way as to not preclude the task; the input needs to be readable by the model; the set of training data needs to be large and varied enough to prevent overfit, and contain sufficient signal to allow generalisation to inputs outside that dataset.

But most importantly, the game needs to be designed _such that it's the sort of game that the model can play_. This is what DeepMind seem especially good at: they are able to express, e.g., protein folding, or tensor decomposition, in the form of a game that their RL agent can play.

This ability – being able to reorganise a question in the form of a model-appropriate game – doesn't look nearly as susceptible to Moore's Law-style exponential speed-ups. Researchers' insights and abilities – in other terms, researcher productivity – don't scale exponentially.[^2] It takes time and energy and the serendipitousness of a well-functioning research lab to cultivate them. Scaling compute down to an effective cost of zero doesn't help if we're not using these models to attack the right issues (1) in the right way (2). The marginal next unit of compute will exhibit diminishing returns.

But if that's the case, then it's clear that _compute isn't the bottleneck_. The bottleneck, instead, will be humans figuring out which problems to tackle (1) and how to regiment them, encode them, and design games/reward systems to get AI to solve them (2).

DeepMind have the absolute best people on the planet working on this stuff, and it takes them time and huge amounts of resources. So we end up back at _taste_. Researchers need time and experience to develop taste, and the ability to apply heuristics and deep domain knowledge – something much more ineffable and difficult to scale.

This doesn't of course mean that there won't be wonderful progress in many, many domains, brought about by 'merely' scaling the compute available. Even Kuhn is at pains to stress that 'normal science' is progress! But it does mean that the scaling hypothesis doesn't necessarily scale research output at the same rate, at least not without the accompanying good taste.

[^1]: This is just an assumption, and one that changes the following argument significantly if it is false. A world with AGI looks, I think we can all assume, quite a bit different.
[^2]: For some empirical data on this point, [see here](https://www.nature.com/articles/467912a), which suggests that doubling the population of a city increases its productivity per capita by ~15%. In other words, productivity per capita scales linearly, not exponentially.
