---
title: Idempotency belongs at the write boundary
description: How to make a retried payment, membership update or agent tool call converge on one durable outcome.
date: 2026-11-26
tag: Distributed systems
kind: blog
published: true
reviewed: false
---
A client sends a request to create an invoice. The database commits the invoice, but the HTTP connection drops before the response reaches the client. From the client's perspective the operation has an unknown outcome. Retrying it as a new operation can create two invoices. Returning an error does not undo the first one.

The same ambiguity exists when a coding agent invokes a tool that publishes a release or writes to an external service. A timeout is not evidence that the operation failed.

## The contract

An idempotency key identifies an intended operation, not an HTTP attempt. The caller creates it before the first attempt and reuses it when retrying. Scope it to the authenticated tenant and operation type; a key must never let one tenant retrieve another tenant's result.

The server stores a fingerprint of the normalized request alongside the key. Reusing a key with a different payload should produce a conflict rather than silently returning the result of an unrelated operation.

The important part is the atomic boundary. Checking for a key, writing a resource and recording the key in three independent operations leaves windows in which concurrent attempts can both succeed.

## A database implementation

For a write that lives entirely in PostgreSQL, use a unique constraint on the operation identity and put the domain mutation and the idempotency record in the same transaction. This is an illustrative schema, not a complete application:

```sql
CREATE TABLE operation_results (
  tenant_id uuid NOT NULL,
  operation text NOT NULL,
  request_key text NOT NULL,
  request_hash text NOT NULL,
  response jsonb,
  PRIMARY KEY (tenant_id, operation, request_key)
);
```

Start the transaction by attempting to insert the key with `ON CONFLICT DO NOTHING`. If the insert succeeds, perform the domain mutation, save the response, then commit. If another transaction already owns the key, read the committed result in a subsequent statement under `READ COMMITTED` isolation; compare the request hash before returning it. At stronger isolation levels, handle serialization failures by restarting the transaction.

A transaction that crashes before commit should leave neither the domain mutation nor its success record. A response lost after commit should be recoverable by key.

## Crossing a service boundary

An HTTP call to a payment provider cannot participate in a local PostgreSQL transaction. Do not hold a database transaction open while waiting for that call and assume atomicity.

Record the intent and an outbox event together, then have a worker deliver the event using a stable provider-side idempotency key. Store the provider's operation ID when available. If the worker crashes after the provider accepts the request but before the local status update, retry with the same key or reconcile against provider state. If the provider has neither deduplication nor a reliable lookup, the ambiguity cannot be eliminated by local code alone.

## Failure tests

| Injected failure | Expected outcome |
|---|---|
| Response lost after commit | Same invoice on retry |
| Two simultaneous requests | One domain mutation |
| Same key, changed payload | Conflict |
| Worker dies after provider acceptance | Reconcile or retry same provider key |
| Key expired before a late retry | Explicitly documented retention behavior |

Retention is part of the contract. Choose a key lifetime based on the maximum plausible retry and reconciliation window, not just a convenient cache TTL. Monitor duplicate-key conflicts, unresolved operations and age of pending intents. Alert on operations that stop progressing.

The useful question in a code review is not "does this endpoint retry?" It is "what durable evidence will the next attempt find after any possible failure?"

## Further reading

- [Amazon Builders’ Library: Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
