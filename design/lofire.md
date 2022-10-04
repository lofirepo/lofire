---
title: LoFiRe
subtitle: Local-First Repositories for Asynchronous Collaboration over Community Overlay Networks
author: TG x Thoth
date: July 2022 -- Initial design proposal & protocol specification
bibliography: design/lofire.bib
---

# Introduction

LoFiRe is a decentralized, collaborative data repository
with authentication, access control, and change validation.
It is built on local-first data storage, synchronization,
and change notification protocols
that aim to protect privacy by minimizing metadata exposed to intermediaries.
It enables local-first, asynchronous collaboration and data storage within communities
while respecting privacy and maintaining data ownership,
and provides foundations for developing local-first decentralized applications
and community overlay protocols.

Community members use local-first software [@local-first]
to collaborate around a partition-tolerant, permissioned, tamper-proof data repository
that contains Directed Acyclic Graphs (DAG) of causally related transactions
with operations on Conflict-free Replicated Data Types (CRDTs) [@cmrdts].
In other words, it is a permissioned, DAG-structured distributed ledger, or blockchain, with partially ordered CRDT transactions.
CRDTs require only a partial order on transactions,
there's no need to determine a total order using a consensus protocol,
which makes the protocol light-weight and low on resource use.

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

In the following we describe the repository and network components,
and the application protocol for interacting with the system.

## Requirements

We set the following requirements to guide our design:

Local first
: Store data locally, use local networks when available,
  and allow working even offline.c

Decentralized
: No centralized services are required to access and modify data.

Asynchronicity & partition tolerance
: Allow collaboration between users
  even if they are not online at the same time.

Identity & data ownership
: Users should have full control over their data and identities.

Causality
: Changes have causal relationships and are delivered in causal order.

Authenticity
: Changes are signed by their author.

Tamperproof
: Once a transaction is stored in a branch, it cannot be removed,
  except by forking the branch.

Access control
: Rules that specify which users can perform what operations on which part of the data.

Strong eventual consistency
: Replicas reach the same state after receiving the same set of changes.

Privacy
: Respect user privacy and minimize the amount of user data and metadata
  exposed to intermediaries.

End-to-end security
: Only the intended recipients should be able
  to read data stored and transmitted in the network.

Ephemerality
: Data can have expiration date after which it should be deleted from all replicas.

Multiple devices per user
: Allow users to use multiple devices, sync data, and share an identity among them.

Local-first software [@local-first] stores data locally,
allows offline data access and modification,
and uses the network for data synchronization with local and remote nodes.
Conflict-free Replicated Data Types (CRDTs)
allow concurrent changes and conflict-free merges of a shared data structure.

Local-first networking allows local collaboration
on edge networks among local nodes on the LAN or other kind of proximity network,
while also allows remote collaboration over the internet when needed.

Data locality ensures control over data location and distribution,
such that data shared in a repository remains private
and only distributed to authorized peers.
Data should not be dispersed over the network to arbitrary nodes, even in encrypted form,
and nodes should only store and forward data they are interested in,
providing incentives for participating in the network,
and limiting damages in the event of an encryption key or algorithm compromise.

End-to-end security ensures only the intended recipients can decrypt data,
i.e. intermediary nodes used for routing and storage cannot see the contents of communications,
which is only visible to end-users who hold encryption keys on their personal devices.

Identity and data ownership entails affording users full control over their data and identities,
by using cryptographic identities and allowing them to store local copies of their data and private keys.
This is not possible in centralized systems that use centralized identifiers such as user names or phone numbers,
neither in federated communication services that use DNS-based identities tied to a provider
and keep user data on the provider's servers with no easy way for users to acquire a full copy of their data,
or to migrate their data and identities to another service provider.

Asynchronicity is a common requirement for communication,
and it's provided by both centralized (e.g. CryptPad, Nextcloud)
and federated services (e.g. SMTP, XMPP, Matrix).
However P2P systems often fail to offer this functionality
and only provide synchronous means of communication
or implement asynchronicity by storing data at arbitrary nodes in the network
that violates the data locality requirement (e.g. libp2p, Holochain, GNUnet).

## Related work

Vegvisir [@vegvisir] proposes a partition-tolerant permissioned DAG blockchain
with a similar structure to ours, but it is not encrypted, has a single branch,
and it is synchronized via a higher latency pull-based gossip protocol
rather than lower latency push-based pub/sub.

Automerge [@automerge] uses operation-based CRDTs and a Bloom-filter-based DAG synchronization algorithm [@bec][@becblog] and provides a JSON data model,
however it is missing encryption and a permission system, and uses a synchronous P2P model.
Their DAG synchronization algorithm is used in our design for branch synchronization.
Its CRDT state machine and JSON data model are complementary to the design presented here, and can be used in the application layer.


Hyper Hyper Space [@hhs] uses immutable append-only Merkle-DAGs together with operation-based CRDTs,
and a gossip-based P2P synchronization protocol for synchronization of JSON objects,
with a permission system and cryptographic identities.
The network is a single-tier P2P system with browsers and headless nodes
synchronizing data through a gossip protocol.
Storing private user data in browsers with complex codebases prone to security vulnerabilities is a security risk by itself, since nodes store data unencrypted, which also exposes user data to third-party service providers of headless nodes, which is necessary for asynchronous communication when participants are not online at the same time.
Permission revocations are not final,
i.e. new operations with past dependencies can show up that have to be undone by an admin,
which becomes problematic especially with multiple admins with diverging histories.

In contrast, LoFiRe focuses on privacy, security, and composability.
It stores user data end-to-end encrypted, supports object expiry and deletion,
branch forking, and snapshots for compacting operations,
as well as a quorum-based operation acknowledgement mechanism.
Permission revocations are implemented via forking, and are final, and support read access revocation, too.
The system is composable: transactions are entirely application defined,
including the data format and CRDT types used,
and thus can be used together with application-defined CRDT types and libraries.
Similarly, the immutable object store supports arbitrary binary data, it is not restricted to JSON.
Each repository uses a two-tier P2P network with low-latency asynchronous pub/sub,
which saves resources on end-user devices and offers them location privacy.

Radicle [@radicle] is a decentralized code collaboration network with Git support that uses public keys for identifying repositories and users,
and employs a gossip protocol for repository replication. As it is intended for code collaboration, it relies on manual merges and does not use CRDTs
that would allow generic collaboration for other use cases, not just code.

Distributed Mutable Containers (DMC) [@dmc] stores CRDT operations in an encrypted immutable object store, ERIS [@eris].
Operations do not have any causal dependencies among each other, thus partial ordering of operations is not possible,
in absence of which nodes can only rely on unsynchronized absolute timestamps when trying to establish an ordering, which cannot be trusted.
Also, there's no guarantee that if a user receives an operation then all users will receive it, either because of network issues or a censorship attempt.

ERIS [@eris] is a content-addressed convergent encryption scheme
that uses similar cryptographic primitives,
however, in our design block size is defined by the application layer,
and padding is only added in the transport protocol and on-disk storage,
it is not part of content-addressed objects,
making the protocol more efficient in terms of bandwidth and disk usage.
Another difference in our design is that storage nodes can traverse (but not decrypt)
the Merkle tree to be able to return an entire subtree in response to a request,
and set an expiry time for an entire tree while keeping data deduplicated.

LoCaPS [@locaps] is a localized causal pub/sub protocol
that uses LoCaMu [@locamu], a localized reliable causal multicast protocol.
We use a pub/sub subscription and event routing protocol inspired by LoCaPS,
but applied to topic-based pub/sub instead of content-based,
only forwarding authenticated events received from publishers.

# Protocol design

## Data repositories

A data repository contains branches, each with a Directed Acyclic Graph of commits
linked by causal dependencies among them. See [@fig:repo] for an example.

<div class="fullwidth">
![Repository R with branches X, Y, Z, commits R*, X*, Y*, acks of X3, Y3, and data D*. Branch Y was forked from X3, and Y3 merges X6. *deps* are solid lines, *acks* are dashed, *refs* are dotted.](lofire-repo.svg){#fig:repo .invert}
</div>

The repository definition is the root of trust for the repository,
and contains a list of published branches.
Similarly, the branch definition is the root of trust for the commits in a branch.
It specifies the permissions and validation rules that every replica has to follow when applying commits for a given branch.

Each commit refers to a list of earlier commits it depends on,
and it is signed by its author, that we also refer to as the publisher.
CRDT operations are grouped in a transaction, which is then placed inside a commit.
Edge nodes of users validate all incoming commits according to the permissions and validation rules set in the branch definition,
and apply the changes to their local replica of the repository only if they are valid.
Valid commits are acknowledged by a quorum of publishers in subsequent commits.
The causal dependencies between commits form a DAG that enables partial ordering of operations,
and provide strong eventual consistency guarantees of valid commits in a branch.

Public keys are used to identify repositories, branches, and members,
which allows the owner of a repository to grant permissions for adding & removing branches,
and the owner of a branch to grant permissions for publishing commits.

The repository structure and storage objects are specified in the [Data structures](#data-structures) section.

### Data storage

Data is chunked to max. 1 MB chunks and stored encrypted in a Merkle tree.
Merkle tree nodes use convergent encryption with a convergence secret
that allows deduplication inside the repository,
while defending against confirmation attacks of stored objects.

Both commits of a branch and files outside branches are stored in immutable objects.
An `Object` contains either a leaf node of the Merkle tree with an encrypted data chunk,
or an internal node with an unencrypted list of `ObjectID` references to child nodes in the Merkle tree, and an encrypted list of  keys for the referenced children.
The unencrypted object header at the root of the Merkle tree further contains
an optional list of dependencies of the object, which is used to list dependencies and acknowledgements of commit objects to allow efficient DAG synchronization,
and an optional expiry time when replicas should delete the object and its children recursively, enabling ephemeral object storage.

Objects are stored in a content-addressed immutable object store
where the object ID (`ObjectId`) is the BLAKE3 hash of the entire serialized object.
In order to be able to decrypt and read an object's content, the user needs not only the ObjectId (to retrieve it) but also the object's key. These are combined in an `ObjectRef`, which enables its holder to retrieve and read the referenced object.
To the contrary, sharing only the ID of an object does not provide a read capability for the recipient.

Implementations should employ an additional layer of encryption and padding
before storing objects on disk in order to provide metadata-protection for data at rest,
i.e. to hide stored object IDs and object sizes.
Objects can be packed together into larger storage blocks before applying padding
in order to reduce storage space requirements.

### Branches & Commits

The repository is organized in branches, which consist of a DAG of commits that contain transactions of CRDT operations.
Commits are stored chunked in encrypted objects, the same way as files.
The commit body is stored in a separate object, in order to be able to reference it directly, and to allow deduplication of commit bodies.

The first `Commit` in a branch contains the `Branch` definition that specifies the commit validation rules and permissions for the branch,
as well as the public key for the pub/sub topic used for publishing commits of the branch.
A member list defines the public keys of members and their permissions for publishing commits in the branch.
Each `Commit` references other commits it directly depends on (via `deps`)
and acknowledges non-dependent branch heads (via `acks`)
known by the publisher in order to reduce branching in the DAG,
and may reference files it depends on (via `refs`).
The referenced dependencies (`deps`) and acknowledgements (`acks`) in a commit implies that the publisher has validated those commits and all their dependencies recursively.
The object IDs (but not the keys) of `deps` & `acks` of the `Commit` are listed in the unencrypted `Object` header in order to allow storage nodes to traverse branches for efficient synchronization, without letting them to decrypt and read the content of the objects.

To ensure durability of commits,
a quorum of acknowledgements is needed before a commit can be considered valid.
The quorum is specified in the branch definition,
and is calculated as the number of commits depending on, or acknowledging the commit either directly or indirectly.
Acknowledgements can be either implicit (dependencies and acknowledgements listed in a subsequent commit) or explicit (a `CommitAck` is sent, with a list of dependencies).
Explicit acknowledgements are only needed if a commit hasn't been acknowledged implicitly within a certain time frame from reception,
which is specified in the branch definition (`ackDelay`).

The end of a branch is marked with an `EndOfBranch` commit, after which no more commits are accepted, except the acknowledgements of this last commit.
This commit can reference a fork where the branch continues.
This allows for changing the branch definition, migration to a new data schema, history truncation by making a new snapshot that does not depend on any previous operations (compaction), changing permissions, and removing access from past members by excluding those members from the newly encrypted branch definition.

A forked branch can either depend on the previous branch with the heads listed in  its `deps`, or on a `Snapshot` listed in its `refs`.
A `Snapshot` contains the data structures resulting from applying all the operations contained in the commits reachable from the heads the snapshot refers to.
A `Snapshot` can also be part of a `Commit`
for the purpose of exporting the current branch state but not for compaction,
in which case other publishers validate it via `acks`,
but no other commits would depend on it.

The branch definition also allows adding application-specific metadata about each member and about the branch itself.
This enables applications to specify roles and permissions for members (who can change which part of the data structure),
define custom validation rules for commits in the branch (or reference an executable validation code that runs in a sandboxed environment, such as WebAssembly),
and reference an application that provides a user interface to display and interact with the data.

The branch definition cannot be changed after it has been commited,
except for adding new members and adding new permissions for an existing member,
along with an adjusted `quorum` and `ackDelay`, via the `AddMember` commit.
For any other changes the branch needs to be forked.
After forking, the new branch can either depend on the commits of the old branch,
or the branch owner can make a snapshot and compact the commits of the previous branch
and thus remove dependencies on earlier commits,
after which the commits of the previous branch can be garbage collected.

The root branch is identified by the public key of the repository,
and has a pub/sub topic assigned that corresponds to the overlay ID.
The root branch contains the `Repository` definition, which references the branches available in the repository.
Branches can be added and removed via the `AddBranch` and `RemoveBranch` commit types.
After a branch is removed, its commits can be garbage collected,
unless it was replaced with a new branch that depends on commits from the removed branch.
When forking the root branch, the fork reference must be present unencrypted
in the `EndOfBranch` commit, and the forked branch must depend on the previous branch, since the root branch serves as the entry point to the repository, and applications need to be able to find the latest fork of it.
It is not possible to publish commits containing transactions in the root branch, as the root branch is only used for managing the other branches.
A branch can remain private if it is never added to the root branch.

### Deletion of data

Data deletion is possible by setting an expiry time for the storage objects.
The expiry field and child node references are not encrypted in order to allow storage nodes to perform garbage collection without being able to decrypt object contents.
In case of expiring commits, in order to keep the DAG intact,
the commit object itself never expires, only the object in the commit body,
and all commits depending on an expiring commit must expire
at the same time as, or earlier than the one they're depending on.

Data that has been removed by a commit remains in the branch, since all commits are kept in the branch.
Permanently deleting data from a branch is possible by making a snapshot,
compacting the operations from all the commits in the branch,
then ending the current branch with an `EndOfBranch` commit,
and creating a forked branch that depends only on the snapshot.
After this the old branch can be removed from the published list of branches in the root branch,
and its commits can be deleted at the expiry time set in the `EndOfBranch` commit.

### User repositories

Each user has a repository shared among their devices
that stores the addresses and encryption keys for repositories, branches, and user identities,
as well as brokers to connect to for each repository.
This allows a user to share configuration, data and identities across their different devices.

To access this repository, the Argon2 key-derivation function is used
to derive the encryption key from a password.
This key is used to encrypt a `RepoKeys` data structure
that contains the private key for the user repository.

## Network architecture

![Core and edge networks of a two-tier community overlay network](lofire-net.svg){#fig:net .invert}

The network is organized as an independent overlay network for each community,
composed of community members' edge and core nodes (we use the terms nodes and peers interchangeably).
See [@fig:net] for an example.

A data repository is associated with each community overlay network,
that is used by community members to share and collaborate on data.
Each community is responsible for their own networking and storage,
as opposed to storing data and relaying messages via non-interested nodes,
as it is the case when a global overlay is used.
Relying on community members' nodes instead of arbitrary non-interested and untrusted nodes
increases efficiency, reliability, privacy, and security of the network,
and establishes incentives for node participation,
since nodes only contribute resources towards the communities they participate in.

The network architecture follows a two-tier peer-to-peer design
that consists of local edge networks composed of end-user devices,
and a core overlay network composed of brokers.
The core network facilitates communication among remote nodes in different edge networks,
and enables asynchronous communication via store-and-forward message brokers
that provide routing and storage services.
Each user is responsible for acquiring access to one or more core nodes of their choice,
which can be self-hosted or offered by service providers.
Each core node provides services only to the communities their users are part of.

The two-tier design also enables location privacy and reduces load on end-user devices,
since they only connect to their brokers of choice in the core network
and do not establish direct connections with remote nodes.
This allows mobile devices to participate in the network
without quickly draining their batteries,
since they only have to maintain a single connection to a message broker,
and they're not responsible for routing or storing messages for other nodes,
while they can still participate in P2P protocols on edge networks on an on-demand basis.

The overlay messages are specified in the [Data structures](#data-structures) section.

### Global overlay

For community overlays that want to be discoverable by ID, we employ a global overlay among core nodes, which consists of a trust-aware peer sampling protocol and a privacy-preserving interest clustering protocol based on community overlay membership, as described in [@upsycle].
The global overlay is optional. In the absence of it, peers rely on an explicit list of community overlay peers in order to join the overlay for the first time (see `OverlayJoin` and `RepoLink`).
Peers store the list of connected peers for each community overlay, including the ones later discovered, and use it to rejoin the overlay the next time.

### Community overlay networks

Each community overlay hosts a data repository
and provides content-addressed object storage
and pub/sub event dissemination services for their members.
The pub/sub service is used for publishing commits in branches,
and the object store allows synchronization between replicas.

The overlay network is identified by a BLAKE3 keyed hash over the repository public key,
using a BLAKE3-derived key from the repository public key & secret.
This helps to avoid correlation of overlays across edge networks.

Messages in the overlay (`OverlayMessage`) are first padded then encrypted using ChaCha20
with a key derived from the repository public key & secret and a per-peer random session ID,
such that only peers in the overlay can decrypt them.
Message padding provides additional metadata protection
against observing object sizes transmitted over the network.

Peers establish TLS or QUIC connections among each other when connecting over IP.
However, the system does not depend on IP,
and can function over other transports as well,
such as overlay networks that provide public key routing.

### Overlay construction & maintenance

A peer first establishes a connection following
the reception of a `RepoLink` message,
which contains the repository public key and secret,
and the network addresses of one or more overlay peers to connect to.

Peers in the overlay discover each other via peer advertisement messages (`PeerAdvert`) that are sent periodically
and follow random walks across the overlay with limited hops.
Each node strives to reduce the number of connections
by preferring to connect to peers with overlapping pub/sub topic subscriptions
(interest clustering).

The `PeerAdvert` message contains a Bloom filter of a peer's topic subscriptions
for the overlay it is sent to.
This allows peers to find others with a similar subscription set,
making pub/sub event dissemination in the overlay more efficient
by reducing the number of connections required for a peer
to cover all of its topic subscriptions,
and by reducing the number of forwarding hops necessary from a publisher to subscribers.

Interest clustering also works across different overlays:
when nodes receive multiple `PeerAdvert` messages from the same peer in different overlays,
the subscriptions from all overlays are considered together in the interest clustering algorithm.

### Publish-subscribe protocol

Pub/sub is used for sending low-latency push notifications
about changes in a branch
to subscribers of the corresponding pub/sub topic.
Each commit and its referenced files are published via pub/sub.

The pub/sub protocol should satisfy the following requirements:

- Causal delivery :: Events should be delivered to the application in causal order.
- Reliability :: All published events should be delivered exactly once to subscribers (by detecting and retransmitting missed events and storing delivered events)
- Fault tolerance :: The system should tolerate and recover from faults in the event forwarding path (by establishing redundant paths)

An overview and detailed analysis of causal pub/sub can be found in [@locaps].

A pub/sub topic is identified by a public key.
Each published event must be signed with the corresponding private key
in order to be routed by the pub/sub broker network.
Events without a valid signature get dropped.
This is necessary to enforce publishing permissions,
to eliminate unsolicited events sent to a topic,
and to avoid amplification attacks by unauthorized publishers
that may result in denial of service.

Branches are mapped to pub/sub topics by the `Branch` definition.
The topic private key is shared among all publishers,
who publish each new commit as a signed event in the pub/sub topic.
This is necessary in order to restrict the publishing of events to authorized publishers only,
and at the same time hide the publisher's public key identities from the pub/sub brokers.

For each commit, change notifications are sent to the appropriate pub/sub topic
as a `Change` inside a signed `Event` message.
Each `Change` contains an encrypted object,
which is either a chunk of a commit,
or a chunk of a file that a commit references.
In case of the first chunk of a commit,
it also contains the encryption key for the commit object,
encrypted with a key derived from the branch public key & secret,
and the publisher's public key.
Brokers cannot decrypt this object, only subscribers,
who need to look up the branch secret and the publisher's public key
matching the publisher hash in the branch definition.

The `Event` is signed by the topic's private key,
and contains a publisher hash
that is a BLAKE3 keyed hash over the commit author's public key,
the publisher's event sequence number used as encryption nonce and to detect missed events.

A pub/sub broker when subscribing to a topic first issues a `BranchHeadsReq`
request to its upstream peers in the topic,
then uses the synchronization protocol to synchronize the DAG for the branch
based on the object IDs of the heads.
Pub/sub brokers store and forward events for their subscribers,
and recover missing events when they notice a missing sequence number
via an `EventReq` request sent to upstream peers in the pub/sub topic.

We use a pub/sub subscription and event routing protocol inspired by LoCaPS [@locaps].
Here follows a description of the base protocol without optimizations for subscription coverage and interest clustering that we leave for future work.

Publishers flood topic advertisements (`TopicAdvert`) to the network creating subscription routing table entries.
A subscription request is forwarded from a subscriber to all publishers
along subscription routing table entries,
and creates an event routing table entry at each broker on the path.
In response, each publisher sends an acknowledgement (`SubAck`) to all subscribers in an `Event`.

Events are forwarded from publishers to subscribers along event routing table entries.
The subscription algorithm can establish multiple event routing paths
when a subscriber sends subscription requests to multiple peers.

### Branch synchronization

Replicas perform branch synchronization using a DAG synchronization protocol described in [@bec][@becblog],
by exchanging branch heads and a Bloom filter of known commits since the last synchronization with the given peer (`BranchSyncReq` & `BranchSyncRes`).

### Object requests

Objects requests either follow the reverse path of a pub/sub topic from a subscriber to publishers (`ObjectReqTopic`), or a random walk (`ObjectReqRandom`).
The response (`ObjectResponse`) contains one or more objects,
and are sent either directly to the requesting node, or along the reverse path of the request.

Requests along a pub/sub topic are effective
when a publisher refers to an object in a commit
that some subscribers want to fetch,
which is likely hosted by the publisher and its broker.
The response gets cached on the way back to the requestor,
in order to serve subsequent requests from other subscribers.

The other method involves issuing multiple requests along random walks across the overlay, and routing the response back to the requestor.

### Broker protocol

The broker protocol is a client-server protocol
that provides an interface to the message broker running on a core or edge node.
The broker provides access to its authorized clients to the pub/sub network and object store.
Applications use it to connect to their local edge node,
while edge nodes use it to connect to a core node,
and perform local-first processing on application requests:
if the local can answer the application request it does so,
otherwise forwards the query to its configured broker in the core network,
and/or sends the query to the edge network.
In case a local node is not available,
applications can talk directly to a broker in the core network.

The broker protocol allows applications to request brokers to
join & leave an overlay (`OverlayJoin` & `OverlayLeave`) ,
subscribe to & unsubscribe from topics (`TopicSubReq` & `TopicUnsubReq`),
publish & receive events (`Event`),
synchronize branches (`BranchHeadsReq` & `BranchSynchReq`),
as well as interact with the object store to:
download & upload objects (`ObjectGet` & `ObjectPut`),
pin objects for permanent storage (`ObjectPin`),
copy objects with a different expiry time before sharing (`ObjectCopy`),
and delete objects from storage (`ObjectDel`).

### External requests

Sharing data from a repository for external clients not part of the overlay and not members of the repository
is possible by including a Message Authentication Code (MAC) in the request (`ExtRequest`).
For this purpose we use a BLAKE3 keyed hash over the request content,
keyed with the repository public key and secret.
This functionality is optional and can be enabled
by setting the `allowExtRequests` flag in the `Repository` definition.

Object requests from external clients (`ExtObjectReq`)
contain a list of object IDs to request,
and a flag whether or not to include all children dependencies of the object,
which allows cloning a branch at a specific commit,
and an expiry time of the request after which it becomes invalid
and overlay peers won't serve the request anymore.

Branch synchronization for external clients is provided
via `ExtBranchHeadsReq` and `ExtBranchSync`
that work the same way as the similarly named requests
in the application protocol.

# Future work

As future work we intend to:

- Add a name system that allows repositories to define named references to branches, commits, and files,
  as well as allow users to define named references to repositories they subscribe to.
- Specify a URI scheme that allows references to repositories, branches, commits, and files, either by ID or name.
- Add external membership & authentication mechanisms that synchronize the repository membership with an external source
- Describe a publishing workflow, starting from collaboration in a repository that results in published immutable artefacts.
- Define collaborative editing and publishing of semantic graph data models.
- Improve and optimize the pub/sub protocol to leverage subscription coverage and interest clustering.
- Specify LAN protocols based on IP multicast for peer discovery and pub/sub.
- Specify local-first search & discovery protocols that find information in locally stored data, in the community overlays, and in the global network.
- Define additional synchronization methods, including synchronizing from a repository on the local file system (e.g. on a USB drive)

# Data structures

In this section we document the data structures for repository objects and overlay messages
as BARE [@bare] message schema.
BARE is a compact binary encoding that does not encode schema information.
An external schema is used instead that provides a human and machine-readable specification,
which is the source of code generation for various programming language targets.
Data structures are versioned in order to allow schema evolution
with forwards and backwards compatibility of messages.

```ruby
!include design/lofire.bare
```

# Acknowledgements

Thanks to Niko Bonnieure for feedback on a draft of this document.

# References
