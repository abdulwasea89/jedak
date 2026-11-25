---
title: The execution contract for an AI agent
description: Separate model proposals from authority, durable tool execution and verification when an agent can modify real systems.
date: 2026-11-26
tag: Agent infrastructure
kind: blog
published: true
reviewed: false
---
A model produces a tool call to modify a file. The arguments look valid, the tool name exists and the model explains why the edit is necessary. That establishes an intention. It does not establish that the operation is authorized, safe to repeat or based on current state.

For agents that can modify real systems, the execution layer should be a separate component with an explicit contract. The model proposes; the execution layer decides whether the request can proceed under the application's rules.

## Split proposal from execution

A proposal should identify the requested operation, typed arguments, relevant resource and expected preconditions. The execution layer attaches trusted identity and policy context independently of the model output.

For a repository edit, a precondition might be an expected content hash. For a deployment, it might be an approved artifact digest and an authorized target environment. Neither should be inferred from natural-language claims in a model response.

```text
proposal -> argument validation -> policy check
         -> precondition check -> durable intent
         -> tool execution -> result verification
```

Some checks must be enforced by the resource itself. A precondition checked before a write does not protect against a concurrent change between the check and the write. Use compare-and-swap, conditional requests or a transaction where the resource supports them.

## Treat tools as distributed operations

The model may see a timeout even after the tool has completed. Give mutating calls stable operation IDs and define what evidence is returned for success, failure and unknown outcome. A verified postcondition is stronger than a free-form success message.

The state machine can distinguish proposed, denied, accepted, executing, succeeded, failed and unknown. A recovery process inspects durable records and reconciles unknown outcomes. Do not let a model retry an ambiguous destructive operation simply because it believes the first call failed.

## Authority does not travel with text

A repository file, issue body or webpage can contain text asking an agent to run another command. That text is untrusted input, even when it resembles an instruction. Authorization comes from the user and policy engine, not from data fetched by a tool.

Constrain tools by resource scope and capability. Read-only analysis rarely needs credentials that can publish a release. Credentials available to a worker should be short-lived and limited to its task where practical. Require an explicit approval for operations whose consequences exceed the granted scope.

## Evaluate the boundary

Test stale file hashes, malformed arguments, path traversal, concurrent edits, lost responses and untrusted tool output containing fake instructions. A useful integration test inspects actual state after the operation, not only the model's explanation.

Observe rejected proposals separately from tool failures. A rejected proposal can indicate that the boundary worked correctly. Record enough context to diagnose the decision without leaking secrets into logs.

The design goal is not to make every model decision deterministic. It is to make the conditions under which a nondeterministic proposal becomes a real operation explicit, testable and recoverable.
