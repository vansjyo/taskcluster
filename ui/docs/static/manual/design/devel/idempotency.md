---
filename: design/devel/idempotency.md
title: Idempotency
order: 30
---

All Taskcluster API operations should be idempotent. This means that repeating
a request has no effect, as long as it occurs within a reasonable time.

For example, calling `queue.createTask` twice with exactly the same task
creates only one task. On the other hand, calling the method twice with the
same task ID but different task definitions is an error.

Ensuring idempotency requires some careful design, but is critical for an HTTP
API, as clients may re-try a successful request if the connection fails before
that success is communicated back to the client.

In particular:

 * Identifiers should always be specified by the client, not generated by
   the API endpoint.  Slug IDs provide a simple way to ensure
   (probabilistically) unique IDs without any central coordination.

 * Creation operations should explicitly handle the case where the given
   identifier is already present in the backend storage, comparing the rest of
   the object and only returning an error if the two differ.

 * Update operations, too, should treat updates that make no changes as a
   success condition. For example, it is not an error to resolve an
   already-resolved error, as long as the resolution reason is the same.

 * Over long durations, idempotency is not required. For example,
   updating a client's scopes after it has expired may fail.