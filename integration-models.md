# Integration models of end-to-end encrypted messaging into ActivityPub

Evan Prodromou

## Overview

This document considers multiple modes of integration of end-to-end encrypted
messaging (E2EE) into ActivityPub. The different models progress in both the
level of effort for software developers implementing the integration, and ease
of use for both end users and system administrators.

A brief description of each model is provided. Pros and cons of the model are
given. Finally, a conclusion is drawn as to the best way to move forward.

This document does **not** define which abstract or concrete E2EE protocol to implement; that is left up to a separate, orthogonal effort.

## Models

### None

In this level, there's no integration of end-to-end encryption in ActivityPub.
This is the status quo as of this writing.

In order to get end-to-end encrypted messages, users have to share their account information with other users in the clear, either as part of their bio, or in a private or public message.

#### Pros

- no effort for developers
- no effort for system
administrators.

#### Cons

- end users need to use separate software for encrypted messaging
- there's no way to determine a user's messaging address automatically
- direct messaging in ActivityPub remains insecure from server administrators
- negotiating which messaging platform to use is a manual process

### Links

In this mode, users have a way to set a structured link to their messaging address on another platform. This could be via a link in their ActivityPub profile, or in a special field of their actor object, probably as an extension property.

#### Pros

- low effort for developers
- no effort for system administrators
- verification of the link between the ActivityPub account and the messaging account, possibly using rel=me or other bidirectional link verification

#### Cons

- Users have to find their correspondent's account, then the link to the messaging address, and then initiate the conversation
- No continuity in identity between ActivityPub and messaging platform after the conversation is established
- User experience entirely handed over to a separate messaging platform
- Big chance of mismatch between users on which messaging platform to use

### Initiation and handoff

There is some automated integration of one or more messaging platforms. The remote user has a way to specify their preferred messaging platform and address on that platform. When the end user wants to start an E2EE messaging conversation, their ActivityPub client and/or server identifies the remote user's messaging platform and identity, initiates the conversation, and then hands off the interaction to a client and/or server for that protocol. The clients might even negotiate the best platform to use, based on the end user's and remote user's configured accounts and preferences.

#### Pros

- Medium effort for developers
- Low effort for system administrators

#### Cons

- May require some configuration and/or network interaction on the server side
- No continuity in identity between ActivityPub and messaging platform after the conversation is established
- User experience entirely handed over to a separate messaging platform
- Either a low number of messaging platforms supported, or a high level of effort to support multiple platforms

### External protocol support

The ActivityPub client and/or server also support a distributed E2EE messaging protocol fully, such as Matrix.  This is either implemented directly in the software, or implemented by a close integration between the ActivityPub software and the messaging software, such as an API or plugin. Each user on the ActivityPub server has a corresponding account in the messaging namespace, such as by domain name.

#### Pros

- Some distributed protocols are available that are well-suited to E2EE messaging, like XMPP or Matrix
- One-to-one correspondence between ActivityPub identity and messaging identity
- Leverage existing protocols and/or software
for security and privacy
- High level of user experience integration

#### Cons

- High effort for developers
- High effort for system administrators
- Abstraction mismatch between ActivityPub and messaging protocol may cause subtle bugs or even security vulnerabilities

### Mastodon API and ActivityPub protocol

An abstract protocol for E2EE is implemented with the Mastodon API as the distribution channel. Mastodon API clients do the encryption, decryption, key management, and other E2EE functions. Mastodon or an API clone provide inter-server distribution of E2EE messages via ActivityPub.

#### Pros

- A number of high-quality abstract models for E2EE are available
- Seamless integration for end users
- Low or no effort for system administrators
- Mastodon API is widely implemented by other ActivityPub servers
- Mastodon API is widely implemented by official and third-party clients

#### Cons

- High effort for client developers
- Mastodon API does not support arbitrary datatypes as well as the ActivityPub API does
- Shoehorning E2EE messages into the Mastodon API may result in ugly line noise for non-E2EE clients, bugs, or security vulnerabilities
- Mastodon API is not a formal standard

### ActivityPub API and ActivityPub protocol

An abstract protocol for E2EE is implemented with the ActivityPub API as the distribution channel. ActivityPub clients do the encryption, decryption, key management, and other E2EE functions. ActivityPub servers provide inter-server distribution of E2EE messages via ActivityPub.

#### Pros

- A number of high-quality abstract models for E2EE are available
- Seamless integration for end users
- Low or no effort for system administrators
- ActivityPub API is a formal standard with clear IP rights
- ActivityPub API is extremely extensible and flexible, and can handle arbitrary datatypes and properties, giving a close fit for any abstract E2EE protocol and resulting in fewer bugs and security vulnerabilities
- Servers that implement the ActivityPub API have a high likelihood of handling extension properties and datatypes well
- E2EE may provide an incentive for non-compliant servers to implement the ActivityPub API

#### Cons

- High effort for client developers
- ActivityPub API is not implemented by Mastodon, currently the most popular ActivityPub server software
- Many other ActivityPub servers do not implement the ActivityPub API

## Conclusion

Of these options, the flexibility of the ActivityPub API and protocol gives a promising path forward. There are many abstract E2EE protocols that can fit well into the ActivityPub API model. It will make for slower growth of E2EE messaging in ActivityPub, but will provide a more robust and secure system in the long run.

## Changelog

- 10 Jun 2024: Initial draft.