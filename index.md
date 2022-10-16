---
title: LoFiRe
subtitle: Local-First Repositories for Collaborative Decentralized Applications
author: "[P2Pcollab](https://p2pcollab.net)"
header-image: img/cover.png
header-image-class: invert
---

## About

LoFiRe is a decentralized, local-first [@local-first], end-to-end encrypted
data repository and application platform
with public key authentication, access control, and change validation,
based on Conflict-free Replicated Data Types (CRDTs) [@cmrdts],
and a two-tier peer-to-peer (P2P) network
that offers end-to-end encrypted object storage
and asynchronous publish-subscribe (pub/sub) notifications.

LoFiRe has the following properties:

Data ownership and portability
: Users own their data and have a local copy.

Self-Sovereign Identities
: Users control their identities, and have the choice to use their already existing public key identities or create new identities for each repository.

End-to-end encryption
: Data in the repository is stored end-to-end encrypted.

Privacy
: Minimize the amount of user data and metadata exposed to intermediaries.

Permissions & Access control
: Fine-grained permissions for write access to the repository.

Tamperproof
: Once a transaction is stored in a branch, it cannot be removed.

Asynchronicity
: Allow collaboration between users,
  even if they are not online at the same time or work offline.

Controlled data locality
: Each repository is replicated within a private network composed only of community member's devices and their authorized replicas.

Multiple devices per user
: Data is available and synchronized on multiple user devices.

## Applications

LoFiRe can be used by local-first decentralized applications
that use [Conflict-free Replicated Data Types](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) (CRDTs) as their data model.
It is designed to support collaboration within communities and organizations.
It does not depend on any specific blockchain, does not have a token, and it can synchronize its membership from external sources.

Communities and organizations (including Decentralized Autonomous Organizations, DAOs)
can use it to support secure and authenticated interaction of their members
through wikis, knowledge bases, structured discussions and decision making tools.

It supports Open Knowledge and Decentralized Science (DeSci) by providing infrastructure
for collaboration around decentralized data repositories and knowledge bases
that can be used to collaborate on research projects,
as well as to share and publish research results and artifacts.

Here's a selection of applications LoFiRe can be used for
that we intend to develop once we're ready with the underlying infrastructure:

- Decentralized wikis & knowledge bases with semantic graph data models
- Project collaboration and publishing tools
- Structured discussion and decision making tools
- Personal information management tools
- Local-first search & discovery of relevant information and repositories

## Introduction

LoFiRe is a decentralized, collaborative data repository
with authentication, access control, and change validation.
It is built on local-first data storage, synchronization,
and change notification protocols
that aim to protect privacy by minimizing metadata exposed to intermediaries.
It enables local-first, asynchronous collaboration and data storage within communities
while respecting privacy and maintaining data ownership,
and provides foundations for developing local-first decentralized applications
and community overlay protocols.

Community members use [local-first software](https://www.inkandswitch.com/local-first/)
to collaborate around a partition-tolerant, permissioned, tamper-proof data repository
that contains [Directed Acyclic Graphs](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG)
of causally related transactions with operations on
[Conflict-free Replicated Data Types](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) (CRDTs).
In other words, it is a permissioned, DAG-structured distributed ledger, or blockchain, with partially ordered CRDT transactions.
CRDTs require only a partial order on transactions, there's no need to determine a total order using a consensus protocol,
which makes the protocol efficient and light-weight.

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

## Design overview

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

### Authorization and access control

Authorization is based on public-key cryptography,
where the repository owner can grant access rights to members based on their public key.
Each operation is signed and encrypted by its author
and disseminated to all replicas subscribed to the repository.
Before a replica can merge an operation,
it needs to verify that its causal dependencies are merged already
and that the author is allowed to perform the operation
according to the CRDT access control rules defined by the repository owners.

### Immutable objects

Next to mutable objects, data repositories also store immutable objects
using a content-addressed object store that stores encrypted chunked objects in the repository.
These objects are referenced from the mutable store.

## Protocol design & specifications

- [LoFiRe: Local-First Repositories for Asynchronous Collaboration over Community Overlay Networks](design/lofire.md)

## Repositories

- [Protocol design & specifications](https://github.com/p2pcollab/lofire)
- [Rust implementation](https://github.com/p2pcollab/lofire-rs)

## Contact

- Email: [#@p2pcollab.net](mailto:#@p2pcollab.net)
- IRC: [#p2pcollab@irc.oftc.net](ircs://irc.oftc.net:6697/p2pcollab)
- XMPP: [p2pcollab@chat.disroot.org](xmpp:p2pcollab@chat.disroot.org)
- Matrix: [#p2pcollab:asra.gr](https://matrix.to/#/#p2pcollab:asra.gr)
- Twitter: [@p2pcollab](https://twitter.com/p2pcollab)

## Donate

[We accept donations](https://p2pcollab.net/donate) in order to be able to continue research & development of local-first protocols and applications.

## See also

- [P2Pcollab](https://p2pcollab.net)
- [Cover image](https://tg-x.net/lsys/#?i=30&r=L%20%3A%20S%0AS%20%3A%20F%2B%3E%5BF-Y%5BS%5D%5DF%29G%0AY%20%3A--%5B%7CF-F-FY%5D%0AG%3A%20FGY%5B%2BF%5D%2BY&p.size=9,0.0001&p.angle=-3769.0402,0.042717&offsets=0,0,0&s.size=8.8,7.5&s.angle=7.6,4&l=0.218&c=black,white,cyan,#e8cc00,#007272,#ff4c00&play=0&anim=return%20%7B%0A%20angle%3A%20t%2F50%2C%0A%20angleG%3A%20t%2F50%2C%0A%20size%3A%20null%2C%0A%20sizeG%3A%20null%2C%0A%20offsetX%3A%20null%2C%0A%20offsetY%3A%20null%2C%0A%20rotation%3A%20null%0A%20%7D&name=pollenate) --
  [L-system](https://en.wikipedia.org/wiki/L-system) rules
