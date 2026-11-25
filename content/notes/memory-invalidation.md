---
title: When should an agent forget what it knows?
description: A short research overview on causal memory invalidation and reliable long-horizon software engineering.
date: 2026-10
tag: Research
published: false
---
Persistent memory can make an engineering agent more useful, but information retrieved from a previous session is not automatically valid in the current repository state.

## The problem

An observation about a file, dependency or test may become stale after a change. Other conclusions can depend on that observation, creating a chain of assumptions that needs to be checked.

## A causal approach

CAVI proposes representing justifications with provenance and dependencies, propagating invalidation through those relationships, and allocating verification effort under uncertainty.

The distinction between observation validity, justification validity and action admissibility is important: an old observation can remain true while the reasoning that relied on it is no longer sufficient for a new action.

## Further reading

- [Causally Sound Memory Invalidation for Long-Horizon Software Engineering Agents](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7644118)
