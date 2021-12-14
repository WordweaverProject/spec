*This is a high-level draft and should not be used to build a client implementation yet.*

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Standard Transports](#standard-transports)
- [Optional Transports](#optional-transports)
- [Standard Interchange](#standard-interchange)
- [Forbidden/Diagnostic Actions](#forbiddendiagnostic-actions)
- [Message Types and Protocols](#message-types-and-protocols)
  - [Trust Exchange](#trust-exchange)
  - [Information Disclosure](#information-disclosure)
  - [Conversation](#conversation)
  - [Language](#language)
  - [Resend Request](#resend-request)

## Standard Transports

The Standard Transports must be supported by every Wordweaver client. They are used for initial exchange of information. Details marked as 'public' must be disclosed during a trust or information exchange.

* `mail` - SMTP + POP3
  * Public:
    * `address` - The address of the SMTP account. For example, "user@example.com"
  * Non-Public:
    * `server` - The domain name of the mailserver. For example, "pop.example.com"
    * `port` - The POP3 port of the mailserver. For example, "995"
    * `user` - The username with which to log in to the mailserver. For example, "user"
    * `password` - The password with which to log in to the mailserver. For example, "hunter2"
    * `ssl` - Whether the server supports SSL. For example, "true"

## Optional Transports

Optional transports are not required, but may be useful to include.

* `fs` - The filesystem
  * This could be used to easily create an internal Wordweaver server by using a networked filesystem, or it could be used to send your messages by mail on a USB drive.
  * Public:
    * `name` - The name of the filesystem transport. Filesystem transports with different names will be treated as different transports.
  * Non-public:
    * `mountpoint` - The mount point of the filesystem transport. The mountpoint must directly contain a file named `.wordweaver-fs-allow` for any further reads or writes to be made to the filesystem.

## Standard Interchange

The Standard Interchange must be supported by every Wordweaver client. It is used for all networked exchanges of information.

All messages should be serialized in the [CBOR](https://cbor.io/), and should have the following top-level keys:

* `v` - The data version number.
* `t` - An integer corresponding to a message type present in the associated data version.
* `s` - The hash of the public key of the sender.
* `u` - A decimal UUID (128-bit integer) which is unique for every conversation.
* `no` - A counter which increments for every message sent by other clients in this conversation.
* `ns` - A counter which increments for every message sent by this client in this conversation.
* `d` - Binary CBOR data compatible with the specified version.

All messages should be encrypted using the recipient's private key and encoded such that the transport can gracefully handle them.

Additionally, the SHA256 hash of a string following the pattern below should be included in a subject line to allow clients to identify messages intended for them.

String pattern:
```
The UTC date, formatted as "2021-12-14".
A colon.
The SHA256 hash of the recipient's public key.
```

Example string:
```
2021-12-14:6eb228fb5b59ab49a45a48bdd9e5f0ed65b43afb52ec72cc825a567986630827
...becomes...
332d034b9b2a64661a1bc73974b0d24fba034ad238dd5bb531001778bb820e08
```

Clients should also check the hash using several previous days. The number of days to check is left up to the implementor, but it should be at least one to prevent messages sent near UTC midnight from being dropped. The best number depends on how often the target device is connected to the internet. If the transport provides timestamp information, it may be best to use that to inform which dates are checked first.

If a transport does not include a subject line, the identification string may also be prepended to the ciphertext, separated by ASCII LF.

Finally, the sender's signature of the ciphertext should be appended to the ciphertext, separated by ASCII LF.

## Forbidden/Diagnostic Actions

Certain actions are forbidden for security reasons and will raise an alarm. Some actions don't indicate malicious activity, but can expose issues.

* Using the same key from multiple clients.
  * This looks the same as an attacker impersonating you using a compromised key, so you should use different keys for each of your clients.
* Sending a message with a lower data version than the maximum mutually supported version of a conversation.
  * This could potentially allow for downgrade attacks in the future.
  * This may occur if an Information Exchange is lost in transit, so it's not guaranteed to be an attack.
  * This is allowed for revocation messages.
  * A warning should be displayed.
* Reporting a lower maximum supported data version than has previously been reported.
  * This could potentially allow for downgrade attacks in the future.
  * This may occur if an Information Exchange is lost in transit, so it's not guaranteed to be an attack.
  * A warning should be displayed.
* Sending a message with unexpected counters.
  * When you receive a message with incorrect counters, its meaning depends on how the counters deviate from your expectations.
  * The following are the meanings of various incorrect counters:
    * The self counter is too high: You've missed a message from the sender in this conversation.
    * The others counter is too high: You've missed a message from someone else in this conversation.
    * The self counter is too low: The sender is being impersonated in this conversation.
    * The others counter is too low: The sender missed a message from someone.
* Sending a message with an invalid signature.
  * This only occurs when a message is forged.
  * A serious alarm should be raised.
* Sending a message to a conversation of which you are not a member.
  * This should be silently ignored if the signature is valid, otherwise raise an alarm.

## Message Types and Protocols

### Trust Exchange

* A Trust Exchange must be performed over a trusted channel.
  * A trusted channel can be firsthand, by communicating physically, or digital, via a middleman.
  * The information which is disclosed by both parties must be as follows:
    * `pubkey` - A Wordweaver public key (format currently undecided).
    * `addr[(name,data)]` - At least one set of details for one of the standard transports.
      * This is to ensure compatibility with all Wordweaver clients.
      * Extended transports may also be used, but only if you are absolutely certain both parties have a compatible version of the transport.
    * `name` - A human-readable name.
    * `data_ver` - The maximum supported interchange version of the client.
  * After a Trust Exchange, an Information Disclosure should occur as soon as possible.
* A digital Trust Exchange is message type 0.
* A digital Trust Exchange is initiated by the middleman sending the information they know about each party to the other.

### Information Disclosure

* An Information Disclosure occurs when a client has information to share.
* An Information Disclosure must be sent on all supported transports shared with the recipient.
* An Information Disclosure is message type 1.
* Most properties of an Information Disclosure are optional. Undefined properties will be assumed to remain the same.

### Conversation

* A Conversation must be sent on all supported transports shared with the recipient(s).
* Creating a conversation requires all conversation members to trust one another.
  * Clients may provide shortcuts to automatically initiate trust exchanges between all members.
* A Conversation is message type 2.

### Language

* A Language message must be sent on at least one supported transport shared with the recipient(s).
* A Language message is a length of human-readable text.
* A Language message is message type 3.

### Resend Request

* A Resend Request must be sent on at least one supported transport shared with the recipient.
* A Resend Request asks the recipient to send a range of messages again.
* Messages are identified using the self counter.
* If a resend request is sent for a message which has been discarded, no response should be sent.
* A Resend Request is message type 4.