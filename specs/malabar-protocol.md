# Malabar Protocol Specification

Author: Sawyer Herbst

Version: 0.1.0

# 0. Preface

This document specifies the Malabar protocol, a decentralized, low-latency communication protocol.

## 0.1. Key words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as
described in [BCP 14](https://tools.ietf.org/html/bcp14)
[[RFC2119]](https://tools.ietf.org/html/rfc2119) [[RFC8174]](https://tools.ietf.org/html/rfc8174)
when, and only when, they appear in all capitals, as shown here.

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

### 1.2.3. Self Sustaining

Nodes are incentivized to participate in the transportation of messages throughout the network. The
network can run indefinitely without the maintenance of Malabar or a third party.

### 1.2.4. Low-Latency

The time it takes for a message to arrive at the recipient node should be short enouph to allow for
real-time communications. This ideally would be under a second.

### 1.2.5. Scalable

The Malabar network must be able to scale to allow for the sending of a large amount of concurrent
messages.

## 1.3. Scope

This document is the specification of the Malabar protcol. It describes how nodes discover peers,
connect, and send and transport messages. A developer should be able to implement a Malabar
protocol-compliant client by reading this specification alone.

## 1.4. Out of Scope

This document does not explain the workings of the currency used to pay fees on the Malabar network.
This document also will not provide an implementation of the Malabar protocol.

## 1.5. Terminology

| Term           | Definition                                                   |
| -------------- | ------------------------------------------------------------ |
| Node           | Any client that supports the Malabar protocol                |
| Network        | A group of interconnected nodes supporting the Malabar protocol |
| Peer           | A node that is directly connected to a given node            |
| Request        | Any communication defined by the Malabar protocol that is made between two nodes |
| Transaction    | A combination of requests made between nodes in order to send a message from one Ethereum address to another |
| Sender         | The node that initiates the transaction to send a message    |
| Recipient      | The node that the sender wishes to send the message to       |
| Transport node | Any node other than the sender or reciever that engages in the transportation of a message |

# 2. Network

This section describes how nodes discover and connect to each other, forming a network of nodes.
This section also introduces the underlying wire protocol, Libp2p.

# 3. Transactions

This section describes the steps that the sender, recipient, and transport nodes must undertake in
order to send a message through the Malabar protocol.

# 4. Future Work

This section describes potential future work that would improve the Malabar protocol.

## 4.1. Sharding

The current version of this specification assumes that there is no network subdivision. This means that a single message transaction would result in requests being made to every node in the network. This is not suitable to scale to a large number of concurrent transactions. A potential fix to this issue is "sharding". Sharding is where the network is subdivided such that there are many sub-networks, or "shards", as part of the Malabar network.