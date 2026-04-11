---
applyTo: "**"
description: "Core engineering principles and decision-making heuristics. Applies to all engineering work — design, code, review, architecture, and communication."
---

# Engineering Principles

*KISS — Keep It Simple, Stupid*
Complexity is the enemy of reliability. Solve the present problem. No more.

*YAGNI — You Aren't Gonna Need It*
Do not build for imagined futures. Defer decisions until you have real information.

*DRY — Don't Repeat Yourself*
Every piece of knowledge has one authoritative representation. Duplication is a future bug waiting to diverge. But a little repetition is cheaper than the wrong abstraction.

*SRP — Single Responsibility*
A module has one reason to change. When something does too much, it becomes untestable, unreusable, and impossible to reason about.

*Composition over Inheritance*
Prefer small, composable pieces over deep class hierarchies. Inheritance creates coupling. Composition creates flexibility.

*Design for Deletion*
Make dependencies explicit, avoid global state, avoid hidden side effects. A feature that cannot be removed cleanly was never truly owned.

*Names are the Most Important Design Decision*
A good name eliminates the need for a comment. Name variables for what they are, functions for what they do. Never abbreviate. Never lie.

*Comments Explain Why, Not What*
The code says what. Write comments only to explain intent, trade-offs, and non-obvious constraints.

*Fail Fast*
Validate early. Surface errors loudly. Never let invalid state propagate silently.

*The Boy Scout Rule*
Leave the code better than you found it.

*Test Behavior, Not Implementation*
Tests are documentation that runs. Verify what a system does, not how. Testing internals creates fragile tests that break on every refactor.

*Measure Before Optimizing — Knuth*
First correct. Then clear. Then fast — only if measurement proves it necessary.

*Design for Observability*
A system you cannot observe is a system you cannot operate. Logs, metrics, and traces are part of the design, not afterthoughts.

*Conway's Law*
Systems reflect the communication structure of the teams that build them.

*Entropy is Inevitable*
Every system accumulates complexity without deliberate effort to reduce it. Refactoring is not optional — it is how quality is sustained.

*Humility*
You will be wrong. Hold your designs lightly, welcome review, and be wrong quickly so you can be right sooner.
