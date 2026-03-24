---
layout: default
title: AI-Assisted Formal Methods - A Step Forward
---

# AI-Assisted Formal Methods: A Step Forward

Over the holidays, I wrote a [Two-Phase Commit specification](https://github.com/plusplusoneplusplus/breadthseek/tree/main/formal-methods/simple-2pc) in TLA+. I was able to get from a problem statement to a working spec in about an hour. It took another few hours to convert it into executable, verified Rust using [Verus](https://github.com/verus-lang/verus).

Tasks that used to take me days seemed to take just an afternoon. This post documents a workflow I've been experimenting with — a process that might help lower the barrier for problems requiring formal specification.

## Barriers to Formal Methods

I've written over two dozen TLA+ specifications over past years — before AI assistance became viable. Every single time, I found myself fighting three distinct battles:

1. **The Syntax:** I have to pull up the tutorial and cheatsheet just to remember the syntax. How do you concatenate two sequences? What's the right way to express "eventually always"? When do you use `\cup` vs `\union` vs `UNION`? How do you filter a set with a predicate — is it `{x \in S : P(x)}` or `{x : x \in S /\ P(x)}`? What's the difference between `\in` and `:>`? How do function domains work with `DOMAIN` vs `[S -> T]`?
2. **Iterating the Model:** Writing the initial spec is only the beginning. As you discover edge cases or refine your understanding of the system, you have to rewrite and refactor the model. A small conceptual change often requires cascading updates across multiple actions and invariants, making iteration slow and error-prone.
3. **Debugging Easy Counterexamples:** When the TLC model checker finds a violation, it spits out an error trace. For complex systems, even simple bugs can produce deep, hard-to-read traces. You spend hours manually stepping through state transitions just to realize you missed a trivial condition in one of your actions.

The concepts themselves are straightforward. Distributed systems engineers think in terms of invariants and state transitions daily. But translating that intuition into TLA+, iterating on the design, and debugging the inevitable mistakes creates enough friction that many of us skip formal verification entirely — even for critical modules where we know we probably shouldn't.

Even a year ago, when DeepSeek R1 came out, I thought AI might finally help with this. I tried using it to write a [Raft specification](https://github.com/plusplusoneplusplus/pondhouse-v2/commits/main/spec/tla). The model seemed to understand Raft conceptually — leader election, log replication, the core invariants. But it kept making syntax errors. Wrong operators. Malformed temporal formulas. Missing parentheses in set comprehensions. I spent more time debugging the generated syntax than I would have spent writing it myself. It felt close, but not quite there.

## A More Promising Experience

My recent experience was noticeably different.

I started with a [natural language problem statement](https://github.com/plusplusoneplusplus/breadthseek/blob/main/formal-methods/simple-2pc/statement.md) — the coordinator must rename a key (A → A') across multiple KV stores atomically, handling coordinator crashes, store failures, and network issues. Standard 2PC with WAL-backed recovery.

Using Opus 4.5, the workflow became:
1. Write the problem statement in natural language
2. Define the invariants and safety properties
3. Generate the TLA+ specification
4. Run TLC model checker
5. When TLC finds a counterexample, feed it back to the AI — it often explains the violation clearly, and I decide whether to fix the spec or refine the problem statement

After a few iterations, the spec worked. The key difference from my previous attempts: the AI produced significantly fewer syntax errors. The iteration loop stayed mostly at the semantic level — refining what the spec should express, rather than fighting with how to express it.

## An Approach: Regenerate Instead of Patching

Here's what made this workflow helpful for me: when the model checker found issues, I often didn't patch the spec directly.

Normally, fixing a TLA+ spec can be painful. You change one thing and might break something elsewhere. The interdependencies can be subtle, and debugging formal specs requires holding a lot of the state space in your head.

Instead, I tried going back to the problem statement. I updated the requirements, clarified the invariants, and then asked the AI to regenerate the entire spec from scratch.

This sounds wasteful, but because generation is relatively fast, a complete regeneration only takes seconds. The cost of iteration dropped significantly.

This changes the trade-offs. When generating a draft is cheap, you can afford to throw it away and restart. This sometimes helps escape the local minima that can plague incremental patches.

## Why This Might Matter

Formal methods have always been valuable for critical code — consensus protocols, transaction managers, lock-free data structures. But the activation energy has historically been high. Learning the tools takes time, and writing specs can take days. As a result, many teams never start.

Now, the barrier to entry feels a bit lower.

If you can describe your protocol in natural language — the state, the transitions, the invariants — you might be able to get a working TLA+ spec much faster than before. If TLC finds a bug, you can refine the description and regenerate.

This makes me wonder if we might start seeing more formal methods in standard practice for distributed systems:

- **Protocol design**: Model checking before implementing, to find edge cases early.
- **Critical modules**: Applying formal treatment to stateful components with non-trivial invariants, not just consensus protocols.
- **Code review**: Having a spec to reference during reviews might become more common.

## Looking Forward

A year ago, AI assistance for formal methods felt like a nice idea that didn't quite work for me. Today, it feels much more viable.

These barriers — syntax, slow iteration, and tedious debugging — feel much lower now. This lets us focus more on the interesting part: thinking clearly about invariants, understanding the state space, and reasoning about failure modes.

That's the work we should be doing anyway. It's an exciting time to be exploring formal methods.
