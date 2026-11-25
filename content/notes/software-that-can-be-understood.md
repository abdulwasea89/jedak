---
title: Software that can still be understood
description: An essay on the relationship between faster code generation and the cost of understanding a system.
date: 2026-11-26
tag: Software engineering
kind: essay
published: true
reviewed: false
---
There is a familiar experience in software development: a feature takes an afternoon to implement, but explaining how it behaves takes much longer. The implementation contains decisions about data, permissions, failure and compatibility that were easy to make individually. Taken together, they become a system that someone must understand.

Code generation changes the relative cost of these activities. Producing another implementation is often cheaper than it used to be. Reading and validating the implementation remains work. In a shared codebase, that work is distributed among reviewers, maintainers and operators who may never have seen the original prompt.

## The cost that moves downstream

A change is not finished when the code appears. A reviewer must decide whether it belongs in the system. An operator must diagnose it when it fails. A future developer must distinguish deliberate decisions from accidental complexity.

Generated code can help all three people, particularly when it follows established conventions and makes behavior explicit. It can also burden them. The difference has less to do with whether a model wrote the code than whether the change fits the existing architecture and exposes its assumptions.

A new abstraction may eliminate ten lines today while introducing another concept that every maintainer must remember. The tradeoff can be worthwhile, but the reduction in line count is not evidence of that by itself.

## The interfaces that matter

One useful unit of design is the boundary between parts of a system. A boundary can be described by what crosses it, what it promises and what happens when the promise cannot be kept. Those descriptions remain relevant as implementation techniques change.

A payment interface that clearly defines an unknown outcome is more valuable than a polished wrapper around the happy path. A memory interface that distinguishes retrieved information from verified evidence helps a coding agent decide when it should act. A queue interface with explicit deadlines makes overload behavior visible.

These are not decorative details. They make the system legible to the people who inherit it.

## A different measure of progress

It is tempting to count generated features, completed tickets or lines of code. Those measures are convenient because production is visible. Understanding is harder to count. Yet a system that requires its original author to interpret every failure has accumulated a constraint on future work.

A review can ask whether a change reduces the number of assumptions needed to use the system correctly. Can the operator identify the responsible component? Can a maintainer change one part without silently breaking another? Can a test demonstrate the behavior without simulating the entire product?

None of these questions requires resisting automation. They require treating generated code as a contribution to a long-lived artifact rather than the final answer to an isolated prompt.

The opportunity is to spend some of the time saved in implementation on making the software easier to understand. That is a choice about how to use the increased capacity, not a property the tools supply automatically.
