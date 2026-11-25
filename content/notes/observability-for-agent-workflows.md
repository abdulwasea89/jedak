---
title: Observing the work, not just the model
description: Instrument an agent workflow so that an operator can explain latency, failures, cost and incomplete execution.
date: 2026-11-26
tag: Observability
kind: blog
published: true
reviewed: false
---
An agent completes a task in 47 seconds. The final response looks correct, but the operator cannot tell whether the agent spent most of that time waiting for inference, retrying a tool or reading a large repository. A token counter alone cannot answer the question.

Instrument the workflow as a sequence of operations. A trace should connect a user request to model calls, tool attempts, verification and the final outcome. Record which steps were retried and which were blocked by policy or deadlines.

## A minimal event model

Use a stable workflow ID and distinct attempt IDs. Record parent relationships, timestamps, operation names, outcomes and resource consumption. Avoid recording complete prompts and tool arguments by default; they can contain source code, credentials and personal information.

```json
{
  "workflow_id": "example-workflow",
  "step": "verify_patch",
  "attempt": 2,
  "duration_ms": 840,
  "outcome": "failed",
  "reason": "stale_revision"
}
```

The numbers are illustrative. In a real system, define the event schema and its retention policy before instrumenting individual components. Use monotonic clocks for local durations where possible, and wall-clock timestamps for cross-process correlation.

## Separate service health from task success

HTTP success does not imply task completion. A model API can return HTTP 200 with an unusable tool call; a tool can execute successfully but leave the desired postcondition false. Track at least three layers:

| Layer | Example measurements |
|---|---|
| Infrastructure | Latency, traffic, errors, saturation |
| Workflow | Completion rate, time to verified outcome, retries |
| Quality | Failed postconditions, incorrect edits, human corrections |

Use low-cardinality dimensions for metrics. A repository path or workflow ID is useful in a trace but can explode time-series cardinality when used as a metric label. Keep identifiers in sampled traces or structured logs instead.

## Cost needs an attribution model

Record input and output token counts separately, alongside provider-reported usage where available. Estimated cost should carry the pricing version and distinguish cache usage if the provider bills it differently. Separate user-facing estimates from reconciliation against actual billing records.

The useful measure is often cost per successful, verified task. A workflow that uses fewer tokens per attempt can still be more expensive if it retries more often. Report failures and partial completions alongside averages so that they are not hidden by a success-only denominator.

## Make the trace actionable

An operator investigating a failed patch should be able to identify the last verified repository revision, the tool attempt that changed state and the first failing check. A trace that contains only model messages forces the operator to reconstruct the execution by reading prose.

Start with a small number of structured events. Add detail when an actual debugging question cannot be answered. Test trace propagation across workers and make sure telemetry failures do not prevent critical business operations unless the operation specifically requires an audit record to proceed.

## Further reading

- [Google SRE: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
