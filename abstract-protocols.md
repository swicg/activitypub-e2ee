# Abstract protocols

Evan Prodromou

## Overview

This document discusses several abstract protocols for end-to-end encrypted messaging (E2EE) in ActivityPub. It then maps each abstract protocol into Activity Streams 2.0 (AS2) data structures as well as the ActivityPub (AP) API and federation protocol. Finally, it discusses pros and cons of each protocol for implementing E2EE in ActivityPub.

## Protocols

### PGP/MIME

The [OpenPGP Message Format](https://datatracker.ietf.org/doc/html/rfc4880) is a standard for encrypting and signing messages using various encryption algorithms.

[RFC 3156](https://datatracker.ietf.org/doc/html/rfc3156) ("MIME Security with OpenPGP") defines a way to embed OpenPGP messages into MIME messages. It provides a content type for OpenPGP messages, `application/pgp-encrypted`.

Together, these RFCs help people send end-to-end encrypted email. They can be adapted for use in ActivityPub.

Text-oriented ActivityPub object types, like `Note` and `Article` can have a `mediaType` property that determines the Internet media type of the `content` property. An implementation can use the `application/pgp-encrypted` media type for encrypted messages.

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Note",
  "id": "https://example.com/notes/1",
  "attributedTo": "https://example.com/users/evan",
  "to": ["https://example.com/users/alice"],
  "mediaType": "application/pgp-encrypted",
  "content": "<OpenPGP encrypted message>",
  "published": "2024-06-14T00:00:00Z"
}
```

Binary data, such as images, can be embedded directly into the `content` property of an ActivityPub object.

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Image",
  "id": "https://example.com/image/1",
  "attributedTo": "https://example.com/users/evan",
  "to": ["https://example.com/users/alice"],
  "mediaType": "application/pgp-encrypted",
  "content": "<OpenPGP encrypted image data>",
  "published": "2024-06-14T00:00:00Z"
}
```

Alternately, the binary data can be uploaded separately and referenced by URL.

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Image",
  "id": "https://example.com/image/1",
  "attributedTo": "https://example.com/users/evan",
  "to": ["https://example.com/users/alice"],
  "url": {
    "type": "Link",
    "mediaType": "application/pgp-encrypted",
    "href": "https://example.com/image/1.jpg.pgp"
  },
  "published": "2024-06-14T00:00:00Z"
}
```

Well-behaved ActivityPub servers should be able to handle this form of encrypted message and route them to the correct recipient.

Clients that don't support OpenPGP can't read the encrypted content. They will see the encrypted content as plain text. This is not ideal, but also does not leak the content.

Most of the architecture variations can work with this method.

#### Pros

- This format can satisfy most of the use cases defined for E2EE for ActivityPub.
- It's much simpler than the alternatives.

#### Cons

- PGP/MIME is usually used for email, not real-time messaging.
- Since keys are addressed on a per-message basis in OpenPGP, adding thousands of recipients to a message could be slow.
- It lacks some of the advanced features of other abstract protocols, like forward secrecy and quantum resistance.
- ActivityPub clients that don't check the `mediaType` property might show the encrypted content as plain text.

### Signal Protocol

Signal is a protocol developed by the Signal Foundation for end-to-end encrypted messaging. It is used in the Signal messaging app, WhatsApp, Facebook Messenger, Google RCS messages, and other messaging services.

The protocol is not standardized; most implementations use a single implementation, `libsignal`, developed by the Signal Foundation. The library is licensed under the Affero General Public License (AGPL) version 3.0.

The Signal messaging system uses gRPC for its transport protocol.

Specifying the Signal protocol layered over ActivityPub will require two main components:

- Read-only methods mirroring those in the Signal gRPC API.
- Write methods that can be implemented as ActivityPub Activity types.

#### Read-only methods

-

#### Write methods

The payloads for Activity types need to mirror these `libsignal` message types:

- SignalMessage
- PreKeySignalMessage
- SenderKeyMessage
- PlaintextContent

### Message Layer Security (MLS)

Message Layer Security (MLS) is a protocol for end-to-end encrypted messaging. It is implemented by Cisco Webex for their messaging service.

MLS explicitly defines two low-level abstract services: a key store and a message delivery service. These can be implemented on top of various transport protocols, like ActivityPub.

The read-only methods of these services can be implemented as ActivityPub API endpoints, or can use existing endpoints as a fallback. The write methods can be implemented as ActivityPub Activity types, posted to the `outbox` endpoint for the actor.

#### Key store

The key store has several responsibilities:

-
-
-

#### Delivery service

The delivery service is required to deliver messages to all members of a group. It can be implemented as a set of custom Activity types on top of ActivityPub.

- KeyPackage
- Proposal:
- Commit:
- Welcome:
- GroupInfo:

#### Pros

- MLS is an official standard from the IETF. Its intellectual property profile is "royalty-free", in line with W3C policies.
- MLS is designed from the ground up for group messaging.
- MLS was designed to work on top of various low-level transport protocols, like ActivityPub.

#### Cons

- Tree format
- Few implementations
- Cannot be bridged end-to-end with Signal protocol implementations like WhatsApp, Signal, and Facebook Messenger.

## Changelog

- 2024-06-14: Initial draft.
- 2024-07-25: OpenPGP, Signal, and MLS protocols added.