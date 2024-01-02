---
title: "Cryptovolense"
abbrev: "cryptovolense"
category: info

docname: draft-steele-spice-cryptovolense-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Secure Patterns for Internet CrEdentials"
keyword:
 - fountain codes
 - animated qr codes
 - digital presentations
 - hpke
venue:
  group: "Secure Patterns for Internet CrEdentials"
  type: "Working Group"
  mail: "spice@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/spice/"
  github: "OR13/draft-steele-spice-cryptovolense"
  latest: "https://OR13.github.io/draft-steele-spice-cryptovolense/draft-steele-spice-cryptovolense.html"

author:
 -
    fullname: "Orie Steele"
    organization: Transmute
    email: "orie@transmute.industries"

normative:
  RFC2397: DataURL
  RFC6330: RaptorQ
  RFC9285: Base45
  RFC1952: GZIP
  RFC8878: ZSTD
  IANA.media-types: IANA-Media-Types

informative:
  RFC7516: JOSE-JWE
  RFC9053: COSE-Encrypt
  BCP205: RFC7942
  I-D.draft-ietf-cose-hpke: COSE-HPKE
  I-D.draft-rha-jose-hpke-encrypt: JOSE-HPKE


--- abstract

Digital presentations enable a holder of digital credentials to present proofs to a verifier.
Using QR Codes for digital presentations introduces challenges regarding maximum transmission size, error correction and confidentiality.
This document describes a generic optical transmission protocol suitable for digital credential presentations.

--- middle

# Introduction

The data density limitations of a single QR Code can be overcome through the use of animated QR Codes, where each frame of the animation is a valid QR Code.

Because QR Codes were originally developed to support transmission of text, not binary, it is desirable to apply specific base encoding, compression and forward error correction, to provide a generic binary content transmission capability through animated qr codes.

{{-RaptorQ}} describes a fully specified error correction scheme, {{-Base45}} describes an optimal base encoding for QR Codes, and {{-DataURL}} describes a content identifier scheme suitable for encoding binary data of a known content type.

This document describes how to use these ingredients to transmit arbitrary content of a known type from a sender to a receiver through the presentation of an animated QR Code by the sender to the receiver.

~~~aasvg
                 .----------.
holder   -->    |  Content   |
                 '----+-----'
                      |
                      v
                 .----+-------.
                / Encryption /
               '------+-----'
                      |
                      v
                 .----+-----.
                |  Data URL  |
                 '----+-----'
                      |
                      v
                 .----+-----------.
                / Fountain Codes /
               '------+---------'
                      |
                      v
                 .----+--------.
                / Compression /
               '------+------'
                      |
                      v
                 .----+-----.
                / Base45   /
               '------+---'
                      |
                      v
                 .----+-------------.
                |  Animated QR Code  |  -->  verifier
                 '------------------'
~~~

# Terminology

{::boilerplate bcp14-tagged}

`message`:
: The data (bytes / octets) to be transmitted.

`message data url`:
: The `message` encoded as a data URL, including its content type.

`transmission configuration`:
: The raptor q details necessary to recover the `message data url` from a series of raptor q packets.

`transmission packets`:
: The raptor q packets produced from the `message data url` interpretted as bytes.

`compressed transmission packets`:
: The `transmission packets` compressed by a protocol parameter "compression algorithm".

`base encoded transmission configuration`:
: The base45 encoded `transmission configuration` (without compression).

`base encoded transmission packets`:
: The base45 encoded `compressed transmission packets`.

`qr encoded transmission configuration`:
: The `base encoded transmission configuration` expressed as an image, for example `image/svg+xml` or `image/jpeg`.

`qr encoded transmission packets`:
: The `base encoded transmission packets` expressed as images, for example `image/svg+xml` or `image/jpeg`.

`transmission images`:
: The unordered set of images composed of `qr encoded transmission configuration` and `qr encoded transmission packets`.

`animated transmission image`:
: An animated image, constructed from a frame and duration for each of the `transmission images`, represented for example as `image/gif`, or `image/webp`.

# Usage

Although this document describes a generic data transmission capability, the primary motivating use case for developing this approach is transmission of large encrypted credential presentations, post quantum capable public keys, and hybrid public keys.

Large binary data can quickly exceed the limitations of single QR Code presentations, motivating a need for animated qr codes and forward error correction capabilities.

# Transmission

The sender MUST prepare their `message` for transmission by first encoding their data as a `message data url` according to the process described in {{-DataURL}}.

Next, the `message data url` MUST be converted to bytes, and the Raptor Q encoding algorithm described in {{-RaptorQ}} must be applied.

The result is `transmission configuration` as bytes, and `transmission packets` as bytes.

Edtiors note: We might need to define a content type for `transmission configuration`, ending in `+json` or `+cbor`.

In cases where this protocol is used with static `transmission configuration`, those details may be hard coded or discovered through some reliable out of band mechansism.

Editors note: We may want to define fountain code agility, such that coding schemes other than RaptorQ can be used.

Next, the packet bytes produced by {{-RaptorQ}} are compressed using gzip as described in {{-GZIP}} or zstd as described in {{-ZSTD}}.

Edtior note: Need to decide if compression agility is valuable, how to signal it, or which compression scheme to require.

Next, the `transmission configuration` and `compressed transmission packets` are encoded using {{-Base45}}.

Finally, the `base encoded transmission configuration` and `base encoded transmission packets` are converted to QR Codes, and each of the `transmission images` is used as a frame in the resulting animated QR Code.

# Recovery

The receiver MUST read the frames of the `animated transmission image`, storing each unique base45 encoded text string.

Once the `transmission configuration` has been recovered, the recovery of the original `message data url` can be attempted using {{-RaptorQ}}.

The recovery process ends when either:

1. a timeout set by the verifier is reached, a failure result indicated by a null / nil MUST be returned.
1. a data url is recovered successfully, a valid Data URL MUST be returned.

# Profile Discovery

The `transmission configuration` MAY include additional data elements, for example the compression algorithm, or public key discovery related meta data in the case that authenticated encryption capable content types are transmitted, for example auth mode hpke as described in {{-JOSE-HPKE}} or {{-COSE-HPKE}}.

In the case the `transmission configuration` contains parameters beyond what are required by {{-RaptorQ}}, it should be encoded as a data url, with a content type expressing the serialization of the parameters, for example `appliation/json` or `application/cbor`.

~~~
data:application/cose;base64,SGVsb...xkIQ==
~~~
{: #encrypted-data-uri-example align="left" title="Example Encrypted Data URL"}

The transmission configuration, and other protocol parameters, such as supported compression or encryption algorithms MAY be discovered out of band.

# Implementation Status

Note to RFC Editor: Please remove this section as well as references to {{BCP205}} before AUTH48.

This section records the status of known implementations of the protocol defined by this specification at the time of posting of this Internet-Draft, and is based on a proposal described in {{BCP205}}.
The description of implementations in this section is intended to assist the IETF in its decision processes in progressing drafts to RFCs.
Please note that the listing of any individual implementation here does not imply endorsement by the IETF.
Furthermore, no effort has been spent to verify the information presented here that was supplied by IETF contributors.
This is not intended as, and must not be construed to be, a catalog of available implementations or their features.
Readers are advised to note that other implementations may exist.

According to {{BCP205}}, "this will allow reviewers and working groups to assign due consideration to documents that have the benefit of running code, which may serve as evidence of valuable experimentation and feedback that have made the implemented protocols more mature. It is up to the individual working groups to use this information as they see fit".

## transmute.codes

An open-source implementation was initiated and is maintained by the Transmute Industries Inc. - Transmute, and is available at:

- [transmute-industries/transmute.codes](https://github.com/transmute-industries/transmute.codes)

An application demonstrating these concepts is available at [https://transmute.codes](https://transmute.codes).

The code's level of maturity is considered to be "prototype".

The current version ('main') implements the transmission and recovery algorithms of this draft.

The project and all corresponding code and data maintained on GitHub are provided under the Apache License, version 2.

The implementation uses a wasm module, built from this rust implementation of RaptorQ {{-RaptorQ}}, maintained at:
- [cberner/raptorq](https://github.com/cberner/raptorq)

Several other dependencies are used, but the RaptorQ implementation is the most relevant.

# Security Considerations

TODO Security

## Public Key Discovery

When encrypting data to a public key, it is important that the sender believe the corrosponding private key is under exclusive control of the receiver.
There are many different mechanisms for delivering a public key to a sender, such that the sender can be assured of this property.
A detailed analysis of identifier to public key binding, and recommendations suitable for use cases is outside the scope of this document.

## Encryption

Note that {{-RaptorQ}} and {{-Base45}} and Base64 as used in {{-DataURL}} are encoding schemes, NOT encryption schemes, and they provide no confidentiality.

Because the data that is encoded within QR Codes is visible to any system that can see the images, any sensitive data should be encrypted before transmission.

In the context of this specification, this means the `message data url` MUST include a content type for an encryption envelope, suitable for the confientiality requirements of the use case.

In the case that an encrypted message is transmitted as a Data URL, the content type of the message MUST be registered {{-IANA-Media-Types}}, for example `application/jose` might be used to transmit a JSON Web Encryption as described in {{-JOSE-JWE}}, and `application/cose` might be used to transmit a cose encrypt envelope as described in {{-COSE-Encrypt}}.

## Context Binding

It is recommended that encryption envelopes supporting multiple key agreement and content encryption schemes ensure that ciphertexts commit to the specific keys and algorithms used.

Additional context binding such as external aad in HPKE might be useful to achieve this.

## Browser APIs

[WICG/identity-credential](https://github.com/WICG/identity-credential) discusses a similar proposal related to invoking a presentation from a mobile device running a mobile operating system, to a browser requesting a presentation on behalf of a web origin.

It is possible that some content types could be shared between this browser API use case, and the QR Code transport use case.

## Proximal and Remote Presentations

It is possible to transmit animated QR Codes from a holder's handheld device to a verifier's camera / scanner, and to transmit animated QR Codes over an established video channel.
Additional security guidance is required regarding replay attack and proxy attacks on in person and remote presentations, that is beyond the scope of this document.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
