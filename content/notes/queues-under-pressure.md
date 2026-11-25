---
title: A queue is not a capacity plan
description: Design admission control, retry budgets and backpressure before connecting autonomous workers to production services.
date: 2026-11-26
tag: Production engineering
kind: blog
published: true
reviewed: false
---
A background queue is often introduced to make a slow operation asynchronous. It improves the request path, but it does not increase the capacity of the database, model provider or external API that eventually executes the work.

Consider a service accepting 60 jobs per second while its workers can complete 40 per second. Under those assumed steady rates, the backlog grows by 20 jobs per second: 1,200 per minute. Adding workers only helps if the bottleneck has spare capacity. If the bottleneck is the model provider's quota, more workers can create more failures instead of more throughput.

## Start with a deadline

Give each job a creation time, a deadline and an operation identity. A notification that is useful for five minutes should not be processed hours later merely because a queue retained it.

Workers check the deadline before expensive work and pass the remaining time budget to downstream calls. Keep deadlines distinct from per-attempt timeouts. Three ten-second attempts do not fit into a job with eight seconds remaining.

Separate work by resource class if a single job can monopolize a worker. A lightweight notification should not wait behind dozens of long-running code-analysis jobs. That does not require a different queue for every feature; it requires isolation wherever workloads contend for scarce capacity.

## Retries consume capacity

Retries are load. Treat them as part of the traffic model rather than an exception outside it. Use exponential backoff with jitter for transient failures, cap the number of attempts and honor provider retry instructions when they are available. Do not retry invalid credentials or malformed input without a change in the underlying state.

A worker may receive the same message more than once. Make its side effects idempotent and acknowledge only after the durable outcome is recorded. When an operation cannot safely be repeated, move it into an explicit reconciliation path rather than letting a generic retry policy decide.

```text
receive job
  if deadline passed: record expired, acknowledge
  acquire capacity for the relevant dependency
  attempt with remaining deadline and stable operation key
  if confirmed success: persist outcome, acknowledge
  if retryable: schedule bounded retry, acknowledge when durable
  otherwise: persist terminal failure or reconciliation task
```

The scheduling and acknowledgment operations in this sketch need the durability guarantees of the chosen broker. A crash between those steps must not silently lose the job.

## Observe age, not just depth

Ten thousand queued jobs could represent seconds or hours of work. Measure age of the oldest pending job, completion rate, enqueue rate, retry rate and dependency saturation. Inspect latency distributions by workload rather than a single global average.

Set an admission policy before exhaustion. Options include rejecting optional jobs, delaying non-interactive work, applying tenant-level quotas or returning a retryable overload response. The response should reflect what the caller can safely do next.

## Validate the overload behavior

Run a controlled load test that exceeds downstream capacity. Confirm that memory remains bounded, critical jobs still make progress and queue age eventually recovers after traffic falls. Simulate a dependency outage and observe whether retries amplify traffic after recovery.

A queue that accepts every request is easy to demonstrate. A queue that declines work predictably when it cannot meet its contract is a production system.

## Further reading

- [Google SRE: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Amazon Builders’ Library: Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
