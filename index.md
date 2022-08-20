---
title: LoFiRe
subtitle: Local-First Repositories for Asynchronous Collaboration over Community Overlay Networks
author: "by [P2Pcollab](https://p2pcollab.net)"
---

# Design overview

[Conflict-free replicated data types](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) (CRDTs)
enable asynchronous, conflict-free collaboration on shared data repositories,
and make eventual consistency possible among a set of replicas.

Authenticated CRDT operations disseminated over pub/sub to subscribers
form a tamper-proof log, which is stored in mergeable data repositories
and replicated to subscribers with access control on the allowed operations.

This enables decentralized collaboration on data repositories
without relying on a centralized server for coordination,
and allows resource-constrained mobile and IoT devices on edge networks
to participate in the network.

## Authorization and access control

Authorization is based on public-key cryptography,
where the repository owner can grant access rights to members based on their public key.
Each operation is signed and encrypted by its author
and disseminated to all replicas subscribed to the repository.
Before a replica can merge an operation,
it needs to verify that its causal dependencies are merged already
and that the author is allowed to perform the operation
according to the CRDT access control rules defined by the repository owners.

## Immutable objects

Next to mutable objects, data repositories also need to store immutable objects
using a content-addressed object store that stores encrypted chunked objects in the repository.
These objects are referenced from the mutable store.


# Protocol design & specifications

- [LoFiRe: Local-First Repositories for Asynchronous Collaboration over Community Overlay Networks](design/lofire.html)

# Repositories

- [Protocol design & specifications](https://gitlab.com/p2pcollab/lofire)
- [Rust implementation](https://gitlab.com/p2pcollab/lofire-rs)

# See also

- [P2Pcollab](https://p2pcollab.net)
