# Architectural variations

## Overview

This document describes some of the architectural variations that can be used to implement the protocols described in the [abstract protocols](abstract-protocols.md) document.

It presupposes the recommended integration model presented in the [integration models](integration-models.md) document, namely, that the ActivityPub API and ActivityPub federation protocol are used as the distribution channel for end-to-end encrypted (E2EE) messages.

## Encoding data structures in AS2

For protocols that work over ActivityPub, we'll need to encode the data structures as `Activity` objects in AS2. For some protocols, this might mean using extended properties or custom object types.

## Public key discovery

For public key encryption, the message sender needs a way to discover the recipient's public key. We consider multiple methods for this, including:

- **Negotiating a key using `Offer` and `Accept` activities in AS2.** The two parties send `Offer` and `Accept` activities to each other with the public key as the `object` property. This is a simple way to exchange keys that doesn't require support from the ActivityPub server, but it requires both parties to be online at the same time.

- **Public keys stored in the Actor profile.** Actors store one or more public keys in their Actor profile or another well-known location. Correspondents can discover the key by looking up the Actor's profile. This requires support from the ActivityPub server to store and retrieve the public key. It will also mean avoiding interference with HTTP Signatures support, which also uses the Actor profile for key discovery.

- **Public keys stored in a dedicated key server.** Actors store their public keys in a separate key server. To send an encrypted message, another actor looks up the key according the actor's ID. Some form of verification, like key signatures, are needed to prevent key substitution attacks.

## Conversation archive storage

Actors need to be able to read the backlog of messages in a conversation, especially if the conversation takes place over days, weeks, or years. This requires some form of conversation archive storage.

- **ActivityPub inbox**. A client can search through the actor's `inbox` for encrypted messages from the correspondent, and use the `inReplyTo`, `replies`, or `context` properties to connect the messages into a conversation. The `inbox` is a serial structure, and can contain hundreds of thousands of messages, so this is likely to be a slow operation.

- **Dedicated conversation archive**. The server can filter and store messages in a dedicated conversation archive, which is already and structured for fast retrieval. For example, it could be an `OrderedCollection` object that in turn contains `OrderedCollection` objects for each conversation in reverse chronological order. This requires the server to filter incoming messages and store them in the archive.

- **External archive**. A third-party server can be used to store the conversation archive. This could be a dedicated server, or a cloud storage service like Apple iCloud or Dropbox. This requires the server to send messages to the third-party server, and to retrieve them when needed.

- **Client-side archive**. Clients store the conversation archive on the client device. Multiple clients for the same actor either have a mechanism to synch the archive, or the conversation is fragmented across clients.

## Validating end-to-end encryption

One model for integration of end-to-end encryption into ActivityPub is to use the ActivityPub API to post an encrypted message to the sender's server; the sender's server then forwards the message to the recipient's server, and the recipient client pulls the message from the recipient server.

As an attack, either the sending server or the recipient server could present their own key to the sending client, decrypt the message, and then re-encrypt it with the correct key. This would allow the server to read the message, defeating the purpose of E2EE.

- **Trusted servers**. One model is to simply trust servers not to do this. This could be because the server software is Open Source and has been audited, or based on the reputation of the server operator.
- **Third-party key validation**. Another model is to use a third-party service to validate the public keys of the receiving actor out-of-band. This could be a trusted key server, or a signature from one or more trusted actors (for example, mutual friends).
- **Out-of-band validation**. The sending actor and the receiving actor could validate their public keys out-of-band, for example by comparing a fingerprint or a QR code.

## Changelog

- 2024-06-14: Initial draft.