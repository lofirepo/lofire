---
title: LoFiRe
subtitle: Local-First Repositories for Asynchronous Collaboration over Community Overlay Networks
author: "by [P2Pcollab](https://p2pcollab.net)"
---

# About

LoFiRe is a decentralized, collaborative data repository
with authentication, access control, and change validation.
It is built on local-first data storage, synchronization, and change notification protocols
that aim to protect privacy by minimizing metadata exposed to intermediaries.
It enables local-first, asynchronous, collaboration and data storage within communities
while respecting privacy and maintaining data ownership,
and provides foundations for developing
decentralized, local-first applications, data stores, and protocols
(including semantic graph data models, and local-first search & discovery protocols).

Community members use [local-first software](https://www.inkandswitch.com/local-first/)
to collaborate around a partition-tolerant permissioned tamper-proof data repository
that contains [Directed Acyclic Graphs](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG)
of causally related transactions with operations on
[Conflict-free Replicated Data Types](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) (CRDT).
The DAG encodes a partial order of transactions through causality relations,
and together with a reliable, causal publish-subscribe (pub/sub) protocol for change notifications
and a DAG synchronization protocol,
it provides strong eventual consistency of replicas,
with persistence of transactions through a lightweight, quorum-based acknowledgement mechanism.

Each repository is synchronized within a private community overlay network
that offers immutable block storage, data synchronization,
and asynchronous publish-subscribe change notification services.

The two-tier network architecture consists of a stable core network and ephemeral edge networks.
On edge networks, edge nodes synchronize locally and directly between each other,
while when communicating with remote participants, they connect to a core node
that stores and forwards encrypted objects and change notifications for them,
thus acting as a pub/sub broker and object store for the edge nodes.

The system is composed of the following components:

Repository
: Data structures, encryption, permissions, authentication and access control.

Network
: Data synchronization, publish-subscribe change notification.

Applications
: CRDT state machine & change validation.


# Design overview

Conflict-free replicated data types enable asynchronous,
conflict-free collaboration on shared data repositories,
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

Next to mutable objects, data repositories also store immutable objects
using a content-addressed object store that stores encrypted chunked objects in the repository.
These objects are referenced from the mutable store.


# Protocol design & specifications

- [LoFiRe: Local-First Repositories for Asynchronous Collaboration over Community Overlay Networks](design/lofire.md)

# Repositories

- [Protocol design & specifications](https://gitlab.com/p2pcollab/lofire)
- [Rust implementation](https://gitlab.com/p2pcollab/lofire-rs)

# See also

- [P2Pcollab](https://p2pcollab.net)
