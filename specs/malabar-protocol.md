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
- 3\. [Messages](#3.-messages)
  - 3.1\. [Preperations](#3.1.-preperations)
    - 3.1.1\. [Message ID](#3.1.1.-message-id)
    - 3.2.1\. [Digital Signature](#3.1.2.-digital-signature)
  - 3.2\. [Routing Communication](#3.2.-routing-communication)
  - 3.3\. [ACK Communication](#3.3.-ack-communication)
  - 3.4\. [Payload Communication](#3.4.-payload-communication)
  - 3.5. [Fee Paying](#3.5.-fee-paying)
- 4\. [Binary Serialization](#4.-binary-serialization)
  - 4.1\. [Stop](#4.1.-stop)
  - 4.2\. [Routing Communication](#4.2.-routing-communication)
  - 4.3\. [ACK Communication](#4.3.-ack-communication)
  - 4.4\. [Payload Communication](#4.4.-payload-communication)
- 5\. [Future Work](#5.-future-work)
  - 5.1\. [Sharding](#5.1.-sharding)
- 6\. [Changelog](#6.-changelog)

# 0. Preface

This document specifies the Malabar protocol: a decentralized, low-latency communication protocol
for use in the Ethereum ecosystem.

## 0.1. Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as
described in [BCP 14](https://tools.ietf.org/html/bcp14)
[[RFC2119]](https://tools.ietf.org/html/rfc2119) [[RFC8174]](https://tools.ietf.org/html/rfc8174)
when, and only when, they appear in all capitals, as shown here.

## 0.2. Scope

This document is the specification of the Malabar protcol. It describes how peers discover other
peers, connect, send, and transport messages.

## 0.3. Out of Scope

This document does not explain or specify the currency used to pay fees on the Malabar network.

## 0.4. Terminology

The following terms are used throughout this specification

| Term               | Definition                                                                                                               |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| Peer               | A computer that is running the Malabar protocol and connected to the Malabar network                                     |
| Direct Peer        | A peer that is directly connected to a given peer                                                                        |
| Network            | A group of interconnected peers supporting the Malabar protocol                                                          |
| Communication      | Any transmission defined by the Malabar protocol that is made between two peers                                          |
| Message            | An umbrella term describing the data and the communications and actions taken to send that data, in the Malabar network. |
| Sender             | The originating peer of a message                                                                                        |
| Recipient          | The peer that the sender wishes to send the message to                                                                   |
| Transport peer     | Any peer other than the sender or reciever that engages in the transportation of a message                               |
| Peer-to-peer (p2p) | Communication directly between peers; not through a centralized service                                                  |

## 0.5. Versioning

The Malabar protocol uses [semantic versioning 2.0.0](https://semver.org/) for versioning. A summary
of changes to this specification for each version is described in [section 6](#6.-changelog).

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
This is simply not possible for most developers, and likely won't reach the same levels of
reliability and security as a standalone project. The third and final option is to use an existing
general-purpose decentralized messaging protocol.

Existing decentralized messaging protocols exist, but are flawed in ways that make them unusable in
production dApps. Perhaps the most notable decentralized messenging protocol is Whisper. Whisper is
a part of the Ethereum protocol suite and is included in Geth. In its current state, Whisper is not
incentived and has to be manually enabled in Geth, and as such the protocol is not practical. In
addition, its lack of network sharding means that it is not scalable to a large message volume.
Other protocols do exist (see: Waku), but are lacking many features that make them not feasible for
production dApps.

The Malabar protocol intends to fix many of these issues and provide a better alternative to
existing options.

## 1.2. Requirements

The Malabar protocol is designed to meet the following requirements:

### 1.2.1. Decentralized

Peers in the network interact with each other directly (or through other peers). Unlike the
tradition client/server model, no centralized services need to be maintained for the Malabar network
to function properly. As such, no one entity can shut down the Malabar network via authority (for
example, censorship by a government).

### 1.2.2. Dark

Peers send and receive messages through their Ethereum addresses. Despite this, the IP address of a
peer should not be associable to the corresponding Ethereum address.

### 1.2.3. Self-Sustaining

Peers are incentivized to participate in the transportation of messages throughout the network. The
network can run indefinitely without the maintenance of Malabar or a third party.

### 1.2.4. Low-Latency

The time it takes for a message to arrive at the recipient should be short enouph to allow for
real-time communications. This ideally would be under a second.

### 1.2.5. Scalable

The Malabar network must be able to scale to allow for the sending of a large amount of concurrent
messages.

# 2. Network

This section describes how peers discover and connect to each other, forming a network of peers.
This section also introduces the underlying wire protocol, Libp2p.

## 2.1. Libp2p

Libp2p is the underlying wire protocol used by the Malabar protocol to handle peer-to-peer (p2p)
communication between peers in the network. Libp2p works with many underlying transports, including
TCP, UDP, WebRTC, and WebSocket. In addition, Libp2p provides implementations for various technology
that is useful in a p2p network (such as [Kademlia](https://en.wikipedia.org/wiki/Kademlia)).

## 2.2. Transports

A peer MUST support TCP as the underlying transport. In addition, a peer MAY support any number of
additional transports.

## 2.3. Ambient Discovery

Ambient discovery is how a local peer discovers others peers without being connected to the network.
A local peer wishing to connect to the network uses ambient discovery methods in order to discover a
peer that is a connected to the network. The local peer then uses active discovery methods to
discover additional peers.

Although peers MAY choose any ambient discovery method, it is recomended to use bootstrap peers.
Bootstrapping is simply the process of connecting to one or more known peers that are part of the
network. Libp2p maintains a module that provides bootstrapping functionality.

## 2.4. Active Discovery

Active discovery is how a local peer discovers peers while already connected to the network. While
ambient discovery is used to initially connect to the network, active discovery is how the local
peer discovers additional peers while connected to the network.

The active discovery method used by the Malabar protocol is Kademlia DHT random walking. Random
walking is the process of querying the DHT at random until a the number of connections is greater
than or equal to 20.

## 2.5. Peer Scoring

The Malabar protocol must be resilient against misbehaving and non-conformant peers in the network.
One of the main ways the Malabar protocol combats this is through peer scoring. There are two kinds
of peer scoring: foreign peer scoring, and direct peer scoring. Foreign peer scoring is a scoring
system that assigns scores to different Ethereum addresses. Direct peer scoring is a scoring system
that assigns scores to the IP addresses of directly connected peers. Note that peer scores are not
propagated throughout the network. Also note that peers MAY blacklist peers for any reason at any
time.

### 2.5.1. Foreign Peer Scoring

Foreign peer scoring is designed to assign scores to the Ethereum addresses of peers that may or not
be directly connected. Because Ethereum addresses are not associable to IP addresses in the Malabar
protocol, this method of peer scoring does not work for scoring misbehaving directly connected
peers. Foreign peer scoring works as follows:

1. The local peer SHOULD keep a permanent list of all peers (identified by their Ethereum addresses)
   it has interacted with and their "scores" (starting at 0)
2. If a peer behaves in an unexpected way in contrary to the Malabar protocol, the peer's score
   SHOULD be incremented by a set number. The amount the score should be incremented is different
   for different infractions, and are listed below.
3. If at anytime the score of a peer exceeds or is equal to 100 the local peer SHOULD ignore all
   future interactions with the misbehaving peer

The possible infractions and their corresponding scores are listed below:

| Infraction             | Score Increase |
| ---------------------- | -------------- |
| Invalid proof-of-entry | 100            |

### 2.5.2. Direct Peer Scoring

Direct peer scoring is designed to assign scores to the IP addresses of peers that are directly
connected. Direct peer scoring works as follows:

1. The local peer SHOULD keep a permanent list of all peers (identified by their IP addresses) it
   has connected to and their "scores" (starting at 0)
2. If a peer behaves in an unexpected way in contrary to the Malabar protocol, the peer's score
   SHOULD be incremented by a set number. The amount the score should be incremented is different
   for different infractions, and are listed below.
3. If at anytime the score of a peer exceeds or is equal to 100 the local peer SHOULD disconnect
   from the misbehaving peer and refuse all future connection attempts

The possible infractions and their corresponding scores are listed below:

| Infraction | Score Increase |
| ---------- | -------------- |
|            |                |

## 2.6. Proof-of-Entry

The peer scoring mechanism addresses the issue of non-compliant and misbehaving peers, but does not
address the situation where a misbehaving peer can simply use a new Ethereum address upon being
disconnected from. Proof-of-entry (or PoE) provides a solution to this problem.

Before sending a message for the first time, a peer MUST solve a PoE challenge specific to the
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

The concept of "gas" is to define a maximum fee that a peer is willing to pay for certain
interactions with the network. Gas is represented as the amount of the smallest denomination of the
currency that the peer is willing to pay.

# 3. Messages

This section describes the steps that the sender, recipient, and transport peers must undertake in
order to send a message through the Malabar protocol.

## 3.1. Preperations

### 3.1.1. Message ID

Each message uses a message ID that is used in each of the communications. The message ID is a hash
of the message, and as such, can also be used as a checksum. The message ID is the SHA-256 hash of
the concatenation of the `To` address, `From` address, the `Time`, and the `Body` of the message.

### 3.1.2. Digital Signature

The digital signature is a method of verifying the authenticity of a message. The digital signature
algorithm used by Malabar is ECDSA, and is used often in the Ethereum ecosystem. The curve used by
Malabar is the secp256k1, the same curve pioneered by Bitcoin. The signature is computing by
calculating the 72-byte ECDSA signature of the message ID.

## 3.2. Routing Communication

The first step in a message is the routing communication. The purpose of the routing communication
is to determine the cheapest route to the recipient without sending the entire payload of the
message. The process is as follows:

1. The sender assembles the routing communication using the format described in
   [section 4.2](#4.2.-routing-communication). Note that the list of stops will be empty upon
   sending, and as such will take up 0 bytes. After assembling the routing communication, the sender
   sends the routing communication to each of it's peers using the protocol identifier
   `/malabar/route/0.1.0`.
2. Any peer, upon recieving a routing communication:
   1. MUST ignore the communication if they have already seen a routing communication for the same
      message ID
   2. SHOULD ignore the communication if they have blacklisted the sender or recipient
   3. MAY add themselves to the stops list on the routing communication
   4. MUST increase the gas used
   5. MUST discard the communication if the gas used is greater than the gas limit
   6. MUST decrement the TTL by 1
   7. MUST forward the communication to each of their peers if the TTL is greater than 0, and MUST
      NOT otherwise
   8. If the local peer is the recipient of the message, the peer MUST respond with an
      [ACK communication](#3.3.-ack-communication)

## 3.3. ACK Communication

Upon the recipient receiving the routing communication, it MUST respond to the sender with an ACK
communication. The process is as follows:

1. The recipient assembles the ACK communication using the format described in
   [section 4.3](#4.3.-ack-communication). The list of stops MUST be copied directly from the
   routing communication.
2. The recipient MUST send the ACK communication to the peer it recieved the routing communication
   from and MUST NOT send it to other peers
3. Any peer, upon receiving an ACK communication:
   1. MUST ignore the communication if they have not previously received a routing communication
      with the same message ID (or are the sender for the message ID)
   2. If the local peer is the sender of the message, the peer MUST respond with the
      [payload communication](#3.4.-payload-communication)
   3. If not, the peer MUST send the ACK communication to the peer it recieved the routing
      communication from and MUST NOT send it to other peers

## 3.4. Payload Communication

The sender, after receiving the ACK communication from the recipient, sends the payload
communication to the recipient in the following way:

1. The sender assembles the payload communication using the format described in
   [section 4.4](#4.4.-payload-communication)
2. The sender MUST send the communication to the peer from which it received the ACK communication
   from
3. Any peer, upon receiving a payload communication:
   1. If the local peer is the recipient, then the peer MUST verify the signature signed by the
      sender
   2. If not, MUST forward the communication to the peer from which it received the ACK
      communication from and MUST NOT send it to other peer

## 3.5. Fees

After the sender receives the ACK communication it MUST follow the process described in
[section 3.3](#3.3.-ack-communication). In addition, the sender MUST pay the message fees to each
transport peer involved in the succesful transfer of the message. For each transport peer and gas
fee on the stops list on the ACK communication, the sender MUST make the appropriate payment using a
verifyiable method. If the total gas used is greater than the maximum gas then the sender MAY not
pay for the fees.

If the sender peer does not make the necessary payments to the transport peers (assuming that the
used gas is less than or equal to the maximum gas), the transport may choose to handle the
misbehaving sender peer in any way it sees fit. It is RECOMMENDED that if the sender does not make
the necessary fee payments to a given transport peer within a set amount of time, the peer
increments the foreign peer score of the sender by 100.

# 4. Binary Serialization

This section describes how objects in the Malabar protocol are serialized and deserialized into
their binary representations. Malabar uses network (big-endian) byte order in its binary
serialization.

## 4.1. Stop

The stop object holds the Ethereum address and the gas used of a transport peer engaged in the
transportation of a message.

| Name     | Size (bytes) | Description                                                                               |
| -------- | ------------ | ----------------------------------------------------------------------------------------- |
| Address  | 20           | Ethereum address                                                                          |
| Gas Used | 32           | How much gas was used by the peer to help in the successful transportation of the message |

## 4.2. Routing Communication

| Name       | Size (bytes)                | Description                                                                                                                                                               |
| ---------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Message ID | 32                          |                                                                                                                                                                           |
| To         | 20                          | Ethereum address of the recipient                                                                                                                                         |
| From       | 20                          | Ethereum address of the sender                                                                                                                                            |
| PoE        | 32                          |                                                                                                                                                                           |
| PoE Nonce  | 8                           | Nonce used to compute the valid PoE                                                                                                                                       |
| Gas Limit  | 32                          | How much gas the sender is willing to pay for the successful transportation of the message                                                                                |
| Gas Used   | 32                          | The total amount of gas used                                                                                                                                              |
| Time       | 8                           | Current time in seconds since the Unix epoch. Apart from providing a timestamp, this is useful so that two identical messages sent at different times have different IDs. |
| Size       | 4                           | Size of the body of the message, in bytes                                                                                                                                 |
| TTL        | 2                           | Time to live                                                                                                                                                              |
| Stops      | 52 bytes per transport peer | Concatenation of the binary representations of each [stop](#4.1.-stop)                                                                                                    |

## 4.3. ACK Communication

| Name       | Size (bytes)                | Description                                                            |
| ---------- | --------------------------- | ---------------------------------------------------------------------- |
| Message ID | 32                          |                                                                        |
| Stops      | 52 bytes per transport peer | Concatenation of the binary representations of each [stop](#4.1.-stop) |

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
that a single message would result in communications being made to every peer in the network. This
is not suitable to scale to a large number of concurrent messages. A potential fix to this issue is
"sharding". Sharding is where the network is subdivided such that there are many sub-networks, or
"shards", as part of the Malabar network.

# 6. Changelog

- Version 0.2.0
  - Initial specification
