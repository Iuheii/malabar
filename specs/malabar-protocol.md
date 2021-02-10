# Malabar Protocol Specification

Author: Sawyer Herbst

Version: 0.2.0

- 0\. [Preface](#0.-preface)
  - 0.1\. [Key Words](#0.1.-key-words)
  - 0.2\. [Scope](#0.2.-scope)
  - 0.3\. [Out of Scope](#0.3.-out-of-scope)
  - 0.4\. [Terminology](#0.4.-terminology)
- 1\. [Introduction](#1.-introduction)
  - 1.1\. [Background](#1.1.-background)
  - 1.2\. [Requirements](#1.2.-requirements)
    - 1.2.1\. [Decentralized](#1.2.1.-decentralized)
    - 1.2.2\. [Self-Sustaining](#1.2.2.-self-sustaining)
    - 1.2.3\. [Low-Latency](#1.2.3.-low-latency)
    - 1.2.4\. [Scalable](#1.2.4.-scalable)
- 2\. [Network](#2.-network)
  - 2.1\. [Libp2p](#2.1.-libp2p)
  - 2.2\. [Transports](#2.2.-transports)
  - 2.3\. [Ambient Discovery](#2.3.-ambient-discovery)
  - 2.4\. [Active Discovery](#2.4.-active-discovery)
  - 2.5\. [Peer Scoring](#2.5.-peer-scoring)
    - 2.5.1\. [Foreign Peer Scoring](#2.5.1.-foreign-peer-scoring)
    - 2.5.2\. [Direct Peer Scoring](#2.5.2-direct-peer-scoring)
  - 2.6\. [Proof-of-Entry](#2.6.-proof-of-entry)
  - 2.7\. [Gas](#2.7.-gas)
- 3\. [Transactions](#3.-transactions)
  - 3.1\. [Message ID](#3.1.-message-id)
  - 3.2\. [Routing Communication](#3.2.-routing-communication)
  - 3.3\. [ACK Communication](#3.3.-ack-communication)
  - 3.4\. [Payload Communication](#3.4.-payload-communication)
  - 3.5. [Fee Paying](#3.5.-fee-paying)
- 4\. [Binary Serialization](#4.-binary-serialization)
  - 4.1\. [Transport Node](#4.1.-transport-node)
  - 4.2\. [Routing Communication](#4.2.-routing-communication)
  - 4.3\. [ACK Communication](#4.3.-ack-communication)
  - 4.4\. [Payload Communication](#4.4.-payload-communication)
- 5\. [Future Work](#5.-future-work)
  - 5.1\. [Sharding](#5.1.-sharding)
- 6\. [Changelog](#6.-changelog)

# 0. Preface

This document specifies the Malabar protocol, a decentralized, low-latency communication protocol for use in the Ethereum ecosystem.

## 0.1. Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as
described in [BCP 14](https://tools.ietf.org/html/bcp14)
[[RFC2119]](https://tools.ietf.org/html/rfc2119) [[RFC8174]](https://tools.ietf.org/html/rfc8174)
when, and only when, they appear in all capitals, as shown here.

## 0.2. Scope

This document is the specification of the Malabar protcol. It describes how nodes discover peers,
connect, and send and transport messages. A developer should be able to implement a Malabar
protocol-compliant client by reading this specification alone.

## 0.3. Out of Scope

This document does not explain the workings of the currency used to pay fees on the Malabar network.

## 0.4. Terminology

The following terms are used throughout this specification

| Term               | Definition                                                   |
| ------------------ | ------------------------------------------------------------ |
| Node               | Any client that is running and supports the Malabar protocol |
| Network            | A group of interconnected nodes supporting the Malabar protocol |
| Peer               | A node that is directly connected to a given node            |
| Communication      | Any communication defined by the Malabar protocol that is made between two nodes |
| Transaction        | A combination of communications made between nodes in order to send a message from one Ethereum address to another |
| Sender             | The node that initiates the transaction to send a message    |
| Recipient          | The node that the sender wishes to send the message to       |
| Transport node     | Any node other than the sender or reciever that engages in the transportation of a message |
| Peer-to-peer (p2p) | Communication directly between peers; not through a centralized service |

## 0.5. Versioning

The Malabar protocol uses [semantic versioning 2.0.0](https://semver.org/) for versioning. A summary of changes to this specification for each version is described in [section 6](#6.-changelog).

# 1. Introduction

Malabar is a low-latency communication protocol for use with (but not limited to) Ethereum
decentralized applications. Peers in the Malabar network send and recieve messages to/from their
Ethereuem addresses, while concealing their IP addresses. In order to incentive peers to participate
in the network, and reduce spam/DDoS risks, the senders of messages pay a fee to each of the peers
that contributes to the successful transportation of the message to the recipient.

## 1.1. Background

Countless decentralized applications rely on real time, low cost communication between users. Right
now there are three main ways to achieve this. The first is to use a centralized service to
facilitate communication between users. This works fine for some apps, but it means that the
successful functioning of the app is under the control of the developer, and possibly the
government. The second option is to design and implement a p2p network solely for your application.
This is simply not possible for most developers, and provides a lesser level of security. The third
and final option is to use an existing general-purpose decentralized messaging protocol.

Existing decentralized messaging protocols exist, but are flawed in ways that make them unusable in
production dApps. Perhaps the most notable decentralized messenging protocol is Whisper. Whisper is
a part of the Ethereum protocol suite and is included in Geth. In its current state, Whisper is not
incentived and has to be manually enabled in Geth, and as such the protocol is more esoteric than
practical. In addition, its lack of network sharding means that it is not scalable to a large
message volume. Other protocols do exist (see: Waku), but are lacking many features that make them
not feasible for production dApps.

The Malabar protocol intends to fix many of these issues and provide a better alternative to
existing options.

## 1.2. Requirements

The Malabar protocol is designed to meet the following requirements:

### 1.2.1. Decentralized

Nodes in the network interact with each other directly (or through other nodes). Unlike the
tradition client/server model, no centralized services need to be maintained for the Malabar network
to function properly. As such, no one entity can shut down the Malabar network via authority (for
example, censorship by a government).

### 1.2.2. Dark

Nodes send and receive messages through their Ethereum addresses. Despite this, the IP address of a
node should not be associable to a matching Ethereum address.

### 1.2.3. Self-Sustaining

Nodes are incentivized to participate in the transportation of messages throughout the network. The
network can run indefinitely without the maintenance of Malabar or a third party.

### 1.2.4. Low-Latency

The time it takes for a message to arrive at the recipient node should be short enouph to allow for
real-time communications. This ideally would be under a second.

### 1.2.5. Scalable

The Malabar network must be able to scale to allow for the sending of a large amount of concurrent
messages.

# 2. Network

This section describes how nodes discover and connect to each other, forming a network of nodes.
This section also introduces the underlying wire protocol, Libp2p.

## 2.1. Libp2p

Libp2p is the underlying wire protocol used by the Malabar protocol to handle peer-to-peer (p2p)
communication between nodes in the network. Libp2p works with many underlying transports, including
TCP, UDP, WebRTC, and WebSocket. In addition, Libp2p provides implementations for various technology
that is useful in a p2p network (such as [Kademlia](https://en.wikipedia.org/wiki/Kademlia)).

## 2.2. Transports

A node MUST support TCP and WebSocket as the underlying transports. The reason WebSocket must be
supported is because browsers can't make p2p requests over TCP. In addition, a node MAY support any
number of additional transports.

## 2.3. Ambient Discovery

Ambient discovery is how a client discovers peers without being connected to the network. A client
wishing to connect to the network uses ambient discovery methods in order to discover a peer that is
a connected to the network. The client then uses active discovery methods to discover additional
peers.

Although nodes MAY choose any ambient discovery method, it is recomended to use bootstrap nodes.
Bootstrapping is simply the process of connecting to one or more known nodes that are part of the
network. Libp2p develops a module that provides bootstrapping functionality.

## 2.4. Active Discovery

Active discovery is how a client discovers peers while already connected to the network. While
ambient discovery is used to initially connect to the network, active discovery is how the client
discovers additional peers while connected to the network.

The active discovery method used by the Malabar protocol is Kademlia DHT random walking. Random
walking is the process of querying the DHT at random until a the number of connections is greater
than or equal to 20.

## 2.5. Peer Scoring

The Malabar protocol must be resilient against misbehaving and non-conformant nodes in the network.
One of the main ways the Malabar protocol combats this is through peer scoring. There are two kinds
of peer scoring: foreign peer scoring, and direct peer scoring. Foreign peer scoring is a scoring
system that assigns scores to different Ethereum addresses. Direct peer scoring is a scoring system
that assigns scores to the IP addresses of directly connected peers. Note that peer scores are not
propagated throughout the network. Also note that nodes MAY blacklist peers for any reason at any time.

### 2.5.1. Foreign Peer Scoring

Foreign peer scoring is designed to assign scores to the Ethereum addresses of nodes that may or not
be directly connected. Because Ethereum addresses are not associable to IP addresses in the Malabar
protocol, this method of peer scoring does not work for scoring misbehaving directly connected
nodes. Foreign peer scoring works as follows:

1. The node SHOULD keep a permanent list of all nodes (identified by their Ethereum addresses) it
   has interacted with and their "scores" (starting at 0)
2. If a node behaves in an unexpected way in contrary to the Malabar protocol, the node's score
   SHOULD be incremented by a set number. The amount the score should be incremented is different
   for different infractions, and are listed below.
3. If at anytime the score of a peer exceeds or is equal to 100 the node SHOULD ignore all future
   interactions with the misbehaving peer

The possible infractions and their corresponding scores are listed below:

| Infraction             | Score Increase |
| ---------------------- | -------------- |
| Invalid proof-of-entry | 100            |

### 2.5.2. Direct Peer Scoring

Direct peer scoring is designed to assign scores to the IP addresses of nodes that are directly
connected. Direct peer scoring works as follows:

1. The node SHOULD keep a permanent list of all nodes (identified by their IP addresses) it has
   connected to and their "scores" (starting at 0)
2. If a node behaves in an unexpected way in contrary to the Malabar protocol, the node's score
   SHOULD be incremented by a set number. The amount the score should be incremented is different
   for different infractions, and are listed below.
3. If at anytime the score of a peer exceeds or is equal to 100 the node SHOULD disconnect from the
   misbehaving node and refuse all future connection attempts

The possible infractions and their corresponding scores are listed below:

| Infraction | Score Increase |
| ---------- | -------------- |
|            |                |

## 2.6. Proof-of-Entry

The peer scoring mechanism addresses the issue of non-compliant and misbehaving nodes, but does not
address the situation where a misbehaving node can simply use a new Ethereum address upon being
disconnected from. Proof-of-entry (or PoE) provides a solution to this problem.

Before sending a message for the first time, a node MUST solve a PoE challenge specific to the
Ethereum address they wish to connect as. This challenge needs to only be solved once per Ethereum
address and will always be valid. The PoE algorithm is essentially the same as a proof-of-work (PoW)
algorithm, and is performed as follows:

1. A 64-bit unsigned integer, the "nonce", is given the value of 0
2. A 256-bit hash of the concatenation of the ascii representation of "malabar" (to defend against
   potential nonce-reuse attacks), Ethereum address (20 bytes), and the nonce (8 bytes) is computed
   using sha256
3. If the hash is greater than 2^220 then discard the hash and return to step two, incrementing the
   nonce by one. Otherwise, this 256-bit hash is the solved PoE

Verifying the PoE is essentially the same as computing the PoE, except that you know the nonce
beforehand. The process, given a 64-bit unsigned integer nonce and Ethereum address, is as follows:

1. Calculate the 256-bit hash of the concatenation of the Ethereum address (20 bytes) and the nonce
   (8 bytes) using sha256
2. Verify that this hash is equal to the hash you are verifying

## 2.7. Gas

The concept of "gas" is to define a maximum fee that a node is willing to pay for certain
interactions with the network. Gas is represented as the amount of the smallest denomination of the
currency that the node is willing to pay.

# 3. Transactions

This section describes the steps that the sender, recipient, and transport nodes must undertake in
order to send a message through the Malabar protocol.

## 3.1. Message ID

Each transaction uses a message ID that is used in each of the communications. The message ID is a hash of the message, and as such, can also be used as a checksum. The message ID is the SHA-256 hash of the concatenation of the `To` address, `From` address, the `Time`, and the `Body` of the message.

## 3.2. Routing Communication

The first step in a transaction is the routing communication. The purpose of the routing communication is to determine the cheapest route to the recipient without sending the entire payload of the message. The process is as follows:

1. The sender assembles the routing communication using the format described in [section 4.2](#4.2.-routing-communication). Note that the list of transport nodes will be empty upon sending, and as such will take up 0 bytes. After assembling the routing communication, the sender sends the routing communication to each of it's peers using the protocol identifier `/malabar/route/0.1.0`.
2. Any node, upon recieving a routing communication:
   1. MUST ignore the communication if they have already seen a routing communication for the same message ID
   2. SHOULD ignore the communication if they have blacklisted the sender or recipient
   3. MAY add themselves to the transport node list on the routing communication
   4. MUST decrement the TTL by 1
   5. MUST forward the message to each of their peers if the TTL is greater than 0, and MUST NOT otherwise
   6. If the node is the recipient of the message, the node MUST respond with an [ACK communication](#3.3.-ack-communication)

## 3.3. ACK Communication

Upon the recipient receiving the routing communication, it MUST respond to the sender with an ACK communication. The process is as follows:

1. The recipient assembles the ACK communication using the format described in [section 4.3](#4.3.-ack-communication). The list of transport nodes MUST be copied directly from the routing communication.
2. The recipient MUST send the ACK communication to the node it recieved the routing communication from and MUST NOT send it to other nodes
3. Any node, upon receiving an ACK communication:
   1. MUST ignore the communication if they have not previously received a routing communication with the same message ID (or are the sender for the message ID)
   2. If the node is the sender of the message, the node MUST respond with the [payload communication](#3.4.-payload-communication)
   3. If not, the node MUST send the ACK communication to the node it recieved the routing communication from and MUST NOT send it to other nodes

## 3.4. Payload Communication

The sender, after receiving the ACK communication from the recipient, sends the payload communication to the recipient in the following way:

1. The sender assembles the payload communication using the format described in [section 4.4](#4.4.-payload-communication)
2. The sender MUST send the communication to the node from which it received the ACK communication from
3. Any node, upon receiving a payload communication:
   1. If the node is the recipient, then the node SHOULD verify the signature signed by the sender
   2. If not, MUST forward the communication to the node from which it received the ACK communication from and MUST NOT send it to other nodes

## 3.5. Fees

After the sender receives the ACK communication it MUST follow the process described in [section 3.3](#3.3.-ack-communication). In addition, the sender MUST pay the transaction fees to each transport node involved in the succesful transfer of the message. For each transport node and gas fee on the transport node list on the ACK communication, the sender MUST make the appropriate payment using a verifyiable method. If the total gas used is greater than the maximum gas then the sender MAY not pay for the fees.

If the sender nodes does not make the necessary payments to the transport nodes (assuming that the used gas is less than or equal to the maximum gas), the transport may choose to handle the misbehaving sender node in any way it sees fit. It is RECOMENDED that if the sender does not make the necessary fee payments to a given transport node within a set amount of time, the node increments the foreign peer score of the sender by 100.

# 4. Binary Serialization

This section describes how objects in the Malabar protocol are serialized and deserialized into
their binary representations. Malabar uses network (big-endian) byte order in its binary serialization.

## 4.1. Transport Node

The transport node object holds the Ethereum address and the gas used of a transport node engaged in the transportation of a message.

| Name     | Size (bytes) | Description                                                  |
| -------- | ------------ | ------------------------------------------------------------ |
| Address  | 20           | Ethereum address                                             |
| Gas Used | 32           | How much gas was used by the node to help in the successful transportation of the message |

## 4.2. Routing Communication

| Name            | Size (bytes)      | Description                                                  |
| --------------- | ----------------- | ------------------------------------------------------------ |
| Message ID      | 32                |                                                              |
| To              | 20                | Ethereum address of the recipient                            |
| From            | 20                | Ethereum address of the sender                               |
| PoE             | 32                |                                                              |
| PoE Nonce       | 8                 | Nonce used to compute the valid PoE                          |
| Time            | 8                 | Current time in seconds since the Unix epoch. Apart from providing a timestamp, this is useful so that two identical messages have different IDs. |
| Size            | 4                 | Size of the body of the message, in bytes                    |
| TTL             | 2                 | Time to live                                                 |
| Transport Nodes | 52 bytes per node | Concatenation of the binary representations of each transportation node |

## 4.3. ACK Communication

| Name            | Size (bytes)      | Description                                                  |
| --------------- | ----------------- | ------------------------------------------------------------ |
| Message ID      | 32                |                                                              |
| Transport Nodes | 52 bytes per node | Concatenation of the binary representations of each transportation node |

## 4.4. Payload Communication

| Name       | Size (bytes) | Description                                             |
| ---------- | ------------ | ------------------------------------------------------- |
| Message ID | 32           |                                                         |
| Signature  | 72           | ECDSA signature of the message ID. Signed by the sender |
| Body       | any size     | The arbitrary body of the message                       |

# 5. Future Work

This section describes potential future work that would improve the Malabar protocol.

## 5.1. Sharding

The current version of this specification assumes that there is no network subdivision. This means
that a single message transaction would result in requests being made to every node in the network.
This is not suitable to scale to a large number of concurrent transactions. A potential fix to this
issue is "sharding". Sharding is where the network is subdivided such that there are many
sub-networks, or "shards", as part of the Malabar network.

# 6. Changelog

- Version 0.2.0
  - Initial specification
