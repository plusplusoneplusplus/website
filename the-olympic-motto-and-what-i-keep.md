---
layout: default
title: "The Olympic Motto and What I Keep"
---

AI is climbing quickly. Every model generation is noticeably better than the last, and the work we can hand over keeps growing. It's easy to read this as a story about machines, but I think it's just as much a story about people: as AI climbs, we need to climb with it. What follows are some reflections from the past few months, borrowing the Olympic motto: faster, higher, stronger.

## Faster: let the loop run

The most obvious way to climb with AI is simply to get faster. Not everything speeds up equally, though: the less supervision a change needs, the faster it can move. So the easy wins are the low-risk stuff, personal software and tooling that nobody else has to review, maintain, or inherit. The math is different now too. When a working tool costs an afternoon instead of a sprint, a lot of the recurring toil I used to just live with is suddenly worth automating away with a tool built for exactly that job.

### Loops, inner and outer

My default way of vibe coding has settled into a four-phase inner loop: **(1) Get grilled** — the agent interviews me to resolve ambiguity up front (goal, edge cases, non-goals) instead of discovering it mid-implementation. **(2) Draft a goal spec** with explicit acceptance criteria; this is the contract everything downstream is judged against. **(3) Implement each AC** one at a time, keeping review chunks small and verifiable. **(4) Final check** — an end-to-end pass against the full spec catches interactions between ACs, producing a gap list and one more fix round.

The pattern concentrates my attention on what matters — the spec — and lets the cheap part run with minimal supervision. The value is in turning ambiguity into steps; execution is a commodity, a lesson that will come back in the next section.

A fast inner loop alone isn't enough; iteration speed decays as complexity piles up. That's where the outer loop comes in. Feature work goes through the full loop above. Refactoring is lighter: periodically the agent picks its own topics, I quickly review, and it fixes things — mostly unsupervised, keeping the codebase clean at almost no cost to my attention.

![Nested loops diagram](/assets/ralph_transformer_style_nested_loops.svg)

### Keep the context, discard the run

Another thing I noticed in practice: in a single chat, planning and execution interleave, mixing discussion with tool output and retries, and the context fills up until quality drops or a compaction wipes it all. So I started splitting the two.

The **context chat** is never archived; it holds the accumulated context and focuses only on finding the right plan. When the plan is ready, it spawns an **execution chat**, a fresh session whose only context is the prompt it was handed. Because the context chat has everything, that prompt carries exactly the right background, constraints, and steps, far more complete than anything I'd write by hand. For the run itself, a new chat beats a sub-agent: it's easy to observe and steer mid-run, and easy to continue later, since a follow-up fix just picks up where it left off while a finished sub-agent is sealed. Once the change lands, the chat is discarded. The pattern shines on multi-step work. When a change decomposes into step one, step two, and step three, each step gets its own execution chat spawned from the same context chat, so every step starts clean while the plan and progress stay in one place, and launching the next step costs a single prompt. [CoC](https://github.com/plusplusoneplusplus/shortcuts) now supports launching a new chat from the current chat, so the handoff is built in.

![Context chat tree](/assets/context_chat_tree.svg)

## Higher: intelligence is moving up the stack

Fable 5 is back, and I've had the chance to play with it a bit more. It's more powerful than Opus 4.8, and the improvement shows up most on ambiguous, open-ended tasks. Each model generation seems to take over a higher level of the work: execution went first, and Fable 5 signals that design is next.

I gave both models a simple challenge: figure out what it would take to get [RSL](https://github.com/Azure/RSL) running on Linux. On the design side, Fable 5 clearly won. It found more compatibility issues and gave more detailed solutions, including the lock-free data structures that rely on Windows events, which Opus 4.8 missed entirely, and it proposed building a baseline from the existing test suites first, while Opus jumped straight into the change phases. On the execution end, there's no clear difference: given a plan with clear steps, both models follow it competently. Execution is mostly solved at this point if the plan is good; the real difference between models is now in planning, where ambiguity has to be turned into concrete steps, the same split the loops above are built around.

So what does "more intelligent" actually mean here? [Wikipedia defines intelligence](https://en.wikipedia.org/wiki/Intelligence) as `abstraction, logic, understanding, self-awareness, learning, emotional knowledge, reasoning, planning, creativity, critical thinking, and problem-solving`. The upgrade from Opus 4.8 to Fable 5 makes several of these directly visible on ambiguous tasks: abstraction to frame an ill-defined problem, critical thinking to spot the shaky assumptions, planning to turn a vague goal into steps. As models climb, people have to climb too. For me that means doing less of the repeated, well-understood work, and spending the time on what the model still can't do well: rethinking, from first principles, how things should be done. The model executes well within the frame it's given; observing the work, finding the real problems, and questioning the frame are still on us. And much of that frame, designed when execution was slow and expensive, now deserves a fresh look.

## Stronger: the safety net and the head

Faster and higher don't hold up on their own. AI-written changes still break production, complexity still piles up, and a few months later someone still has to explain why the code looks the way it does. That's what stronger is about: a safety net around the work, and the head doing the work.

### A stronger safety net

The shallow net is human. Slow down where it's critical, keep real human review in the loop, write down why changes were made while it's still fresh, and think the invariants through carefully. This doesn't contradict getting faster; it's the same gradient from the other end: critical changes need the most supervision, so they move the slowest. Code arriving faster than I can comprehend just becomes the bottleneck for cleanup and review, and the cleanup is real: style and taste remain weak spots even for the latest models like GPT-5.6 Sol, and redundant code keeps getting generated (if you're interested, I suggest reading [this post on minimal editing](https://nrehiew.github.io/blog/minimal_editing/)). For minor changes I've moved back toward hand-writing with auto-completion, since an agent is overkill there.

A few practices keep review from becoming the choke point:

- **Split large changes** into a stack of small, self-contained diffs; each reviews far faster than one big drop.
- **Write a clear description** while the context is still fresh, covering what changed and why, with a diagram for structural changes so the reviewer isn't reconstructing the architecture in their head.
- **Automate the nits** with pre-submission hooks that format, lint, and clean up the known issues, so review time goes to substance instead of style.

The deeper net is mechanical: formal methods and test cases. Tests that encode the invariants, specs that outlive any chat session, verification that doesn't depend on someone reading carefully. AI has also made this net much cheaper to build, since the agent can write most of it: derive test cases from the spec, generate property checks for the invariants, and keep the suite growing alongside the code. The human net only works when someone is paying attention; the mechanical one holds either way. Critical pieces should rest on both.

### A stronger head

AI doesn't replace everything, but it has quietly taken the part where engineers learned the most: debugging, investigating, tracing a failure back to its root. The better the loop gets, the less of that I ever see. The agent hits the error, fixes it, retries, and hands me a green result. The failures still happen; they just never pass through me anymore, and that walk was where the real experience came from.

So where does experience come from now? My answer so far: it has to become deliberate. Learning used to come free with the work; now the work finishes without it. I've started protecting learning time the way I'd protect time for testing or design, and treating "I shipped it" and "I understand it" as two different deliverables.

In practice, I pay for it in small ways. Sometimes I read the agent's failed attempts instead of only the final diff, and ask it why the first approach didn't work. I keep some debugging for myself even when the agent would be faster. I still hand-write code now and then, because the muscle memory of shaping a change is part of what keeps review sharp. And I let AI speed up the learning itself: a diagram of the architecture before I touch it, the state machine rendered instead of traced in my head.

## Final thoughts

Here's what nags at me. Scaling has run on two supplies: the internet's worth of text, and the deep engineering judgment labs like Anthropic and OpenAI hire for, earned the slow way before agents existed. The first is mostly consumed, and the second stops replenishing if the next generation never walks the failures themselves. Where does new experience come from?

Partly from us, I think. Delivery and growth both matter, but growth probably matters more now: delivery keeps getting easier, while growth still takes deliberate time, and it's the part that compounds.