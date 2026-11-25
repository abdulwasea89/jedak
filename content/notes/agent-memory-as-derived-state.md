---
title: Agent memory is derived state
description: Represent the evidence behind a coding agent’s assumptions and invalidate them when the repository changes.
date: 2026-11-26
tag: Agent systems
kind: blog
published: true
reviewed: false
---
A software engineering agent remembers that a function accepts a particular argument. During a later session, another developer changes the function signature. The memory is still retrievable, relevant and confidently worded. None of those qualities make it correct.

This is a derived-state problem. A memory entry about a repository is a claim based on observations, not a permanent property of the repository. The evidence and the conditions under which the claim was made matter as much as the claim itself.

## Store the dependency, not just the conclusion

Consider an agent that has inspected `billing.py` at revision A, observed a function signature and concluded that a caller can pass a specific keyword. Three things exist: the observed source, the derived claim and a proposed action that relies on the claim.

A useful memory representation keeps these separate:

```json
{
  "claim": "create_invoice accepts customer_id",
  "evidence": ["billing.py:sha256:example"],
  "depends_on": ["function-signature:billing.create_invoice"],
  "observed_at_revision": "example-revision",
  "status": "valid-under-recorded-evidence"
}
```

The values here are placeholders. In a real system, evidence references should resolve to captured observations, and identities must survive ordinary renames or be conservatively invalidated when they do not.

## Invalidation is transitive

Suppose claim B depends on claim A, and a planned action depends on B. Invalidating A but leaving B marked valid is not enough. A dependency index can traverse affected descendants, marking them stale until they are verified again.

A practical implementation can begin with file content hashes and explicit claim-to-evidence edges. On a changed hash, mark affected claims stale and traverse the reverse dependency graph. Use a visited set to bound work when dependencies contain cycles. More expressive justification models can represent alternative evidence paths, but those paths must be checked rather than assumed independent.

Distinguish stale from false. A changed file may still support the original conclusion. Staleness means the old justification is no longer sufficient; it does not establish that the claim itself is wrong.

## Verification under a budget

Re-reading a large repository after every tool call is expensive. Prioritize re-verification around actions that are about to be executed and the potential consequences of a wrong assumption. A claim used to format a comment can have a different verification budget from a claim used to alter billing code.

For external state, a repository hash may prove nothing. A live database, an API permission or a remote configuration may change without a Git commit. Use environment-specific observations, freshness constraints and preconditions. If the relevant state cannot be observed, represent that uncertainty explicitly and gate high-impact actions.

## Evaluate with counterfactual mutations

Construct paired environments from the same repository. In one, change only a dependency relevant to a stored claim. In the other, make an unrelated change. Measure whether the agent invalidates the affected claim, preserves useful unaffected memory and avoids an action whose preconditions are no longer established.

Useful measures include stale-claim detection, false invalidation, verification cost and unsafe-action rate. Report the distribution across mutation classes. A benchmark composed entirely of obvious file edits will not expose hidden dependencies or partially observable external state.

This is a design direction, not a claim that the proposed system is empirically validated. The difficult problem is capturing the dependencies that were never recorded in the first place.

## Related research

- [Causally Sound Memory Invalidation for Long-Horizon Software Engineering Agents](../research/causally-sound-memory-invalidation.html)
