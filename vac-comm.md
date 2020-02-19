# [WIP] vac.comm aka waku/next

*This document is an early work in progress, and largely consists of notes that will be used to fill out a more thorough document such as [[WIP] Dagger: A Distributed Storage Network](https://hackmd.io/CDg3GXyTSbSmBQjL93hooA?both) and/or [vacp2p.Communication - System Design & Technical Architecture](https://docs.google.com/document/d/1OkltbPr9jF1cx9O38evw-VarfKQ5L2LOsadCvV7xpZw/edit?ts=5e29c00d&pli=1#).*

vac.comm provides private and decentralized communication, and is part of the Vac communication/storage/compute umbrella.

## Motivation 

Unlike many other secure messaging applications, our goal is not to have a tightly coupled set of protocols, nor is it to reinvent the wheel. Instead, we aim to provide options at each layer in the stack, and build on the shoulders of giants, putting a premimum on interoperability. It’s similar in philosophy to projects such as [libp2p](https://libp2p.io) or [Substrate](https://www.parity.io/substrate/) in that regard. Each choice comes with different trade-offs, and these look different for different applications.

The protocols we work on are pure p2p, and aim to minimize centralization. This too is in opposition to many initiatives in the secure messaging space.

Outside of traditional notions of secure messaging, such as ensuring end to end encryption, forward secrecy, avoiding MITM-attacks, etc, we are also concerned with two other forms of secure messaging. We call these private messaging and censorship-resistance. Private messaging means viewing privacy as a security property, with all that entails. Censorship resistance ties into being p2p, but also in terms of allowing for transports and overlays that can’t easily be censored by port blocking, traffic analysis, and similar.

### Challenges

The challenges we experience with private and decentralized communication directly stem from fact that we are building a protocol which is not centralized. This property, adds quite a few complexities to the protocol which we are working on resolving.

**Scalability** is a large challenge we face. When a centralized messaging protocol has problems handling its load, we simply add a new server. This is easier said than done with a decentralized protocol, adding a new node does not guarantee increase of throughput.

**Spam-resistance** is another, in a centralized application spam-resistance is easy to handle, we can simply throttle one user and be done with it. This however is not the case with decentralized messaging protocols. Additionally, currently whisper uses proof-of-work to reduce spam, which does not actually prevent spam as we need to lower the PoW so far for it to be mobile friendly that it is easy to spam the network yet again.

In order to add a large amount of participants actively supporting the network, we need to somehow incentivize those users.

**Resource restricted devices** are our main user base. Nodes are mostly offline, which is not an issue for a centralized messaging protocol where all messages are stored on a server to later be downloaded by a client in order to read. How do we guarantee that messages reach their destination even if it is mostly offline?

## System design, technical architecture and rationale

These are rough notes of things we currently know and don't know. They should be reformulated as a more coherent narrative.

**Relation between storage and communication:**
- We'll use the same building blocks (Nim, libp2p and many its more basic protocols, etc)
- We don't want to force a solution that fits for communication but not storage, and vice versa
- That said, we'll continuously look and strive for overlap between the two when it comes to routing, incentivization, etc

**On message routing:**
- Current Whisper-based routing fundamentally doesn't scale
- For structured approaches, Kademlia for full nodes is the leading candidate
- For unstructured approaches, gossipsub and episub are worth looking into more as complementary approaches

**On Kademlia routing:**
- This provides point to point and point to neighborhood capabilities
- Neighborhood routing is provided for with partial addressing a la PSS, and needs more experimental research to confirm feasibility
- As a fallback, an unstructured gossip mechanism can be used for neighborhood messaging

**On classical vs forwarding Kademlia:**
- Trade-off between classical and forwarding Kademlia: what's faster, opening new connections or re-using them?
- Classical Kademlia better studied, and also more tractable when it comes to who is doing the work
- We can likely provide both as options to keep flexibility, e.g. for heavy TCP forwarding might make more sense

**On incentivization:**
- Structured and more direct approaches like classical Kademlia more tractable in terms of who does what
- Incentivization in an unstructured routing setup a la gossip currently unproven
- More ideas need to be explored and falsified aggressively in this area, e.g. see Block.Science effort, Swarm postage stamps, etc

**On adaptive nodes:**
- Nodes with limited battery and short connection windows won't relay messages, and will instead likely rely on an emergent service network
- In this emergent service network, lighter nodes can have a simple cryptoeconomic game with fuller nodes in a simple request/response manner
- For semi-powered nodes, they can seamlessly join the DHT (e.g.) and provide some form of routing/storage

**On security, spam protection and availability:**
- S/Kademlia can be used as anextension with some form of registration cost
- Spam protection mechanisms needs further experimentation, e.g. stake based priority queue, postage stamps, zkSnarks rate limiting
- Node uptime assumptions and replication factors are fairly well-studied in Kademlia and can be leveraged statistically

**On privacy:**
- Transport privacy will be developed as a modular approach since not all use cases require it, and there are fundamental performance limitations
- With pseudoanonymity, spam protection and E2EE a lot of privacy guarantees are achieved over current state of things
- Offloading a lot of research to Nym et al, can also be complemented by making it easy to run over existing mixnet/Tor-like stacks, a la Briar

## Timeline and milestones

### February - March 2020

**Ongoing:**

1. Write an experimental Kademlia implementation for nim-libp2p, with the goal of
  - increasing tacit knowledge of Kademlia/libp2p/Nim
  - acting as a base layer for upcoming applied research work

2. Do research in conjunction with Dagger priorities as outlined below

**Outstanding:**

1. Get consensus on strategy, direction for most immediate users (Vac meetup, Dagger and Status Core)

2. Create a detailed timeline for implementing the project as a deliverable

**Overlap / in conjunction with Dagger priorities:**

1. Research existing projects such as:
        Swarm
        IPFS/Filecoin
        BitTorrent

2. Research relevant academic literature

3. Validating existing economic models as well as exploring our own

### March - June 2020

As an initial goal, we will focus on building an MVP with a limited subset of features such as:

- Send messages to a single point
- Send messages to a neighborhood
- Find nodes and content

This will require a few components in place:

**Completed:**

- p2p networking stack
    we have an initial implementation of libp2p in Nim, this should be enough to start development, but it will be improved and augmented with new features on an ongoing basis

**Outstanding:**

- A wire protocol to
    Coordinate with other nodes
    Send and receive messages to one and multiple nodes

- Feasibility study of neighborhood routing

- Accounting and incentivization simulation for adaptive nodes

- PoC for offline/storage interface

- Node capabilities and discovery beyond DNS
