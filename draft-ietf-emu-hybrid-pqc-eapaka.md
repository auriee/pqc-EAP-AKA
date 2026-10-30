---
title: "Enhancing Security in EAP-AKA' with Hybrid Post-Quantum Cryptography"
abbrev: "PQC Hybrid in EAP-AKA prime"
category: std

docname: draft-ietf-emu-hybrid-pqc-eapaka-02
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "EMU"
keyword:
 - PQC
 - EAP-AKA'


venue:
  group: "emu"
  type: "Working Group"
  mail: "emu@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/emu/"


stand_alone: yes
pi: [toc, sortrefs, symrefs, strict, comments, docmapping]

author:
 -
    fullname: Aritra Banerjee
    organization: Nokia
    city: London
    country: United Kingdom
    email: "aritra.banerjee@nokia.com"
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"

normative:
  RFC4187:
  RFC9048:
  RFC9678:
  RFC9794:
  I-D.ietf-hpke-pq:
  I-D.ietf-emu-pqc-eapaka:

informative:
  RFC4086:
  RFC9958:
  FIPS203:
     title: "FIPS-203: Module-Lattice-based Key-Encapsulation Mechanism Standard"
     target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.pdf
     date: false

--- abstract

Forward Secrecy for the Extensible Authentication Protocol Method for Authentication and Key Agreement (EAP-AKA' FS) is specified in {{RFC9678}}, providing updates to {{RFC9048}} with an optional extension that offers ephemeral key exchange using the traditional Ephemeral Elliptic Curve Diffie-Hellman (ECDHE) key agreement algorithm for achieving perfect forward secrecy (PFS). However, it is susceptible to future threats from Cryptographically Relevant Quantum Computers, which could potentially compromise a traditional ephemeral public key. If the adversary has also obtained knowledge of the long-term key and ephemeral public key, it could compromise session keys generated as part of the authentication run in EAP-AKA'.

This draft aims to enhance the security of EAP-AKA' FS protocol by using Post-Quantum/Traditional (PQ/T) hybrid Key Encapsulation Mechanisms (KEMs) {{RFC9794}} to make it quantum-safe.

--- middle

# Introduction

Forward Secrecy for the Extensible Authentication Protocol Method for Authentication and Key Agreement (EAP-AKA' FS) defined in {{RFC9678}} updates the improved Extensible Authentication Protocol Method for 3GPP Mobile Network Authentication and Key Agreement (EAP-AKA') specified in {{RFC9048}}, with an optional extension providing ephemeral key exchange. The ephemeral key exchange prevents an attacker who subsequently obtains the long-term key from recovering session keys established in previous EAP-AKA' FS authentication runs, provided that the corresponding ephemeral secrets and derived session keys have been deleted. EAP-AKA' FS mitigates passive attacks (e.g., large scale pervasive monitoring) against future sessions.

Nevertheless, EAP-AKA' FS uses traditional public-key algorithms (e.g., ECDH) which will be broken by a Cryptographically Relevant Quantum Computer (CRQC) using Shor's algorithm. A CRQC would render commonly deployed traditional public-key algorithms based on factoring or discrete-logarithm problems insecure. The computational assumptions underlying these algorithms would no longer provide the intended security against an adversary with a sufficiently capable quantum computer. In particular, a CRQC could recover the ECDHE SHARED_SECRET from the exchanged public keys described in Section 6.3 of {{RFC9678}}. If the adversary has also obtained knowledge of the long-term key, it could then compute CK', IK', and the SHARED_SECRET, and any derived output keys. This means that the CRQC would disable the forward security capability provided by {{RFC9678}}.

The migration to PQC is unique in the history of modern digital cryptography in that neither the traditional algorithms nor the post-quantum algorithms are fully trusted to protect data for the required data lifetimes. The post-quantum algorithms face uncertainty about the underlying mathematics, compliance issues, unknown vulnerabilities, hardware and software implementations that have not had sufficient maturing time to rule out classical cryptanalytic attacks and implementation bugs. During the transition from traditional to post-quantum algorithms, there is a desire or a requirement for protocols that use both algorithm types.

This document specifies the use of the PQ/T hybrid KEMs defined in Section 4 of {{I-D.ietf-hpke-pq}} with EAP-AKA' FS. Only the KEM functions GenerateKeyPair(), Encap(), and Decap() are used; Hybrid Public Key Encryption (HPKE) encryption is not used. {{I-D.ietf-emu-pqc-eapaka}} specifies the use of ML-KEM with EAP-AKA' FS. This document uses its fragmentation mechanism and the same negotiation approach.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

This document makes use of the terms defined in {{RFC9794}}. For the purposes of this document, it is helpful to be able to divide cryptographic algorithms into two classes:

"Traditional Algorithm": An asymmetric cryptographic algorithm based on integer factorisation, finite field discrete logarithms or elliptic curve discrete logarithms. An example of a traditional key exchange algorithm is Elliptic Curve Diffie-Hellman (ECDH).

"Post-Quantum Algorithm": An asymmetric cryptographic algorithm that is believed to be secure against attacks using quantum computers as well as classical computers. An example of a post-quantum KEM is ML-KEM {{FIPS203}}.

"Hybrid" key exchange, in this context, means the use of two key exchange algorithms based on different cryptographic assumptions, e.g., one traditional algorithm and one Post-Quantum algorithm, with the purpose of the final shared secret key being secure as long as at least one of the component key exchange algorithms remains unbroken. It is referred to as PQ/T Hybrid Scheme in {{RFC9794}}.

PQ/T Hybrid Key Encapsulation Mechanism: A Key Encapsulation mechanism (KEM) made up of two or more component KEM algorithms where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

# Background on EAP-AKA' with perfect forward secrecy

In EAP-AKA', the authentication vector (AV) contains a random part RAND, an authenticator part AUTN used for authenticating the network to the USIM, an expected result part XRES, a 128-bit session key for integrity check IK, and a 128-bit session key for encryption CK.

As described in {{RFC9678}}, the server has the EAP identity of the peer. The server asks the Authentication Database (AD) to run AKA algorithm to generate RAND, AUTN, XRES, CK and IK. Further it also derives CK' and IK' keys which are tied to a particular network name. The server now generates the ephemeral key pair and sends the public key of that key pair and the first EAP method message to the peer. In this EAP message, AT_PUB_ECDHE (carries public key) and the AT_KDF_FS (carries other FS related parameters). Both of these are ignored by a peer that does not support the Forward Secrecy extension. The peer determines whether it supports and selects the EAP-AKA' FS extension offered by the server. If yes, then it will respond with AT_PUB_ECDHE and AT_MAC. If not, it will ignore AT_PUB_ECDHE. If the peer wants to participate in FS extension, it will then generate its ECDH key pair, calculate a shared key based on its private key and server public key. The server will receive the RES from peer and AT_PUB_ECDHE. The shared key will be generated both in the peer and the server with key pairs exchanged, and later master key is also generated.

~~~
MK_ECDHE = PRF'(IK'| CK'|SHARED_SECRET,"EAP-AKA' FS"|Identity)
~~~

# Hybrid Enhancements by Design

This specification defines the following extensions to EAP-AKA' FS:

- A new attribute, AT_PUB_HYBRID, carries the hybrid KEM encapsulation key from the EAP server and the hybrid KEM ciphertext from the EAP peer.

- New AT_KDF_FS values identify the hybrid KEMs ({{iana-kdf}}) used to derive the Hybrid Master Key MK_HYBRID.

- The server lists the hybrid KEM values after an ECDHE value in AT_KDF_FS and sends AT_PUB_HYBRID only after the peer selects a hybrid KEM ({{negotiation}}).

# Message Fragmentation and Reassembly {#frag}

For all hybrid KEMs registered in this document, AT_PUB_HYBRID exceeds the 1020-octet limit for EAP-AKA extensions in Section 8 of {{RFC4187}}. AT_PUB_HYBRID is therefore fragmented using AT_FRAGMENT, as specified in {{I-D.ietf-emu-pqc-eapaka}}.

AT_FRAGMENT MUST NOT be sent in an AKA'-Challenge message until the peer has selected a hybrid KEM ({{negotiation}}).

AKA'-Identity messages are not protected by AT_MAC. If AT_FRAGMENT is used in any AKA'-Identity message, AT_CHECKCODE MUST be included in the EAP-Request/AKA'-Challenge carrying AT_PUB_HYBRID and in the EAP-Response/AKA'-Challenge carrying AT_RES. Section 10.13 of {{RFC4187}} requires this because AT_FRAGMENT is an attribute not defined in {{RFC4187}}.

# Capability Negotiation {#negotiation}

AT_PUB_HYBRID and AT_FRAGMENT use a 2-octet Length field. Implementations that do not support this document parse a 1-octet Length field, as specified in {{RFC4187}}, and reject a message containing these attributes, even though their Type values are in the skippable range.

These attributes are therefore sent only after the peer has selected a hybrid KEM, using the AT_KDF_FS change procedure in Section 6.2 of {{RFC9678}}:

1. In the first EAP-Request/AKA'-Challenge, the server lists an ECDHE value from {{RFC9678}} as the first AT_KDF_FS attribute, followed by the hybrid KEM values, and includes AT_PUB_ECDHE. The server MUST NOT include AT_PUB_HYBRID or AT_FRAGMENT in this message.

2. A peer that does not support {{RFC9678}} ignores AT_KDF_FS and AT_PUB_ECDHE and continues as specified in {{RFC9048}}. A peer that supports {{RFC9678}} but not this document continues as specified in {{RFC9678}}.

3. A peer that supports this document and one of the offered hybrid KEMs MUST request it. Its EAP-Response/AKA'-Challenge contains only AT_KDF_FS with the selected value. The peer does not run the AKA algorithm on this message.

4. The server re-sends the EAP-Request/AKA'-Challenge with the selected value as the first AT_KDF_FS attribute, followed by the original list, and with AT_PUB_HYBRID instead of AT_PUB_ECDHE. The peer verifies the AT_KDF_FS list as specified in Section 6.2 of {{RFC9678}}.

Either side MAY require a hybrid KEM by local policy. A server that requires a hybrid KEM and finds that the peer did not select one MUST behave as if authentication failed. A peer that requires a hybrid KEM and finds no hybrid KEM value in AT_KDF_FS MUST behave as if AUTN were incorrect. This follows Section 6.5.4 of {{RFC9678}}.

# Protocol Construction

This section defines the construction for hybrid key exchange in EAP-AKA' FS. Hybrid key exchange refers to using multiple key exchange algorithms simultaneously and combining the result with the goal of providing security even if all but one of the component algorithms is broken. It is motivated by transition to post-quantum cryptography.

## Protocol Call Flow

~~~ artwork
     USIM           Peer                        Server              AD
      |              |                            |                |
      |              |           EAP-Req/Identity |                |
      |              |<---------------------------+                |
      |              |                            |                |
      |              | EAP-Resp/Identity          |                |
      |              | (Privacy-Friendly)         |                |
      |              +--------------------------->|                |
      |      +-------+----------------------------+----------------+--+
      |      | Server now has an identity for the peer. The server    |
      |      | then asks the help of AD to run AKA algorithms,        |
      |      | generating RAND, AUTN, XRES, CK, IK. Typically, the AD |
      |      | performs the first part of key derivations so that the |
      |      | authentication server gets the CK' and IK' keys        |
      |      | already tied to a particular network name.             |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              |                            | ID, key deriv. |
      |              |                            | function,      |
      |              |                            | network name   |
      |              |                            +--------------->|
      |              |                            |                |
      |              |                            |    RAND, AUTN, |
      |              |                            | XRES, CK', IK' |
      |              |                            |<---------------+
      |      +-------+----------------------------+----------------+--+
      |      | Server now has the needed authentication vector. It    |
      |      | generates an ECDHE key pair. It lists an ECDHE value   |
      |      | first in AT_KDF_FS, followed by the hybrid KEM values, |
      |      | and includes AT_PUB_ECDHE but not AT_PUB_HYBRID.       |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              | EAP-Req/AKA'-Challenge     |                |
      |              | AT_RAND, AT_AUTN, AT_KDF,  |                |
      |              | AT_KDF_INPUT,              |                |
      |              | AT_KDF_FS (ECDHE),         |                |
      |              | AT_KDF_FS (hybrid KEM),    |                |
      |              | AT_PUB_ECDHE, AT_MAC       |                |
      |              |<---------------------------+                |
    +--+--------------+----------------------------+---------+     |
    | A peer that does not support RFC 9678 ignores          |     |
    | AT_KDF_FS and AT_PUB_ECDHE and continues per RFC 9048. |     |
    | A peer that supports RFC 9678 but not this document    |     |
    | continues per RFC 9678. A peer that supports this      |     |
    | document requests the hybrid KEM value (Section 6.2 of |     |
    | RFC 9678). It does not run the AKA algorithm on this   |     |
    | message.                                               |     |
    +--+--------------+----------------------------+---------+     |
      |              |                            |                |
      |              | EAP-Resp/AKA'-Challenge    |                |
      |              | AT_KDF_FS (hybrid KEM)     |                |
      |              +--------------------------->|                |
      |      +-------+----------------------------+----------------+--+
      |      | The server generates a hybrid KEM key pair (skR, pkR). |
      |      | It re-sends the challenge with the selected hybrid KEM |
      |      | value first in AT_KDF_FS, followed by the original     |
      |      | list, and with AT_PUB_HYBRID carrying pkR instead of   |
      |      | AT_PUB_ECDHE.                                          |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              | EAP-Req/AKA'-Challenge     |                |
      |              | AT_RAND, AT_AUTN, AT_KDF,  |                |
      |              | AT_KDF_INPUT,              |                |
      |              | AT_KDF_FS (hybrid KEM),    |                |
      |              | AT_KDF_FS (ECDHE),         |                |
      |              | AT_KDF_FS (hybrid KEM),    |                |
      |              | AT_PUB_HYBRID (pkR), AT_MAC|                |
      |              |<---------------------------+                |
    +--+--------------+----------------------------+---------+     |
    | The peer queries the auth parameters from the USIM     |     |
    | card.                                                  |     |
    +--+--------------+----------------------------+---------+     |
      |              |                            |                |
      |   RAND, AUTN |                            |                |
      |<-------------+                            |                |
      |              |                            |                |
      | CK, IK, RES  |                            |                |
      +------------->|                            |                |
    +--+--------------+----------------------------+---------+     |
    | The peer verifies AT_MAC and checks that the AT_KDF_FS |     |
    | list differs from the original list only by the        |     |
    | requested change (Section 6.2 of RFC 9678). It         |     |
    | computes (ss, enc) = Encap(pkR), derives all EAP-AKA'  |     |
    | key values, and constructs a full response.            |     |
    +--+--------------+----------------------------+---------+     |
      |              |                            |                |
      |              | EAP-Resp/AKA'-Challenge    |                |
      |              | AT_RES, AT_PUB_HYBRID (enc),|                |
      |              | AT_MAC                     |                |
      |              +--------------------------->|                |
      |      +-------+----------------------------+----------------+--+
      |      | The server computes ss = Decap(enc, skR) and checks    |
      |      | AT_RES and AT_MAC. Success requires both to be         |
      |      | correct. An attacker who later obtains the long-term   |
      |      | key cannot derive the session keys from a recorded     |
      |      | exchange without also breaking the hybrid KEM.         |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              |                EAP-Success |                |
      |              |<---------------------------+                |
      |              |                            |                |
~~~

## Key Steps in Protocol Construction {#steps}

- After the peer selects a hybrid KEM ({{negotiation}}), the server generates an ephemeral key pair for that hybrid KEM: (skR, pkR) = GenerateKeyPair(), as defined in Section 4 of {{I-D.ietf-hpke-pq}}.

- The server associates XRES and skR with the current EAP session and sends pkR in AT_PUB_HYBRID.

- The peer runs the AKA algorithm, verifies AT_MAC, and computes (ss, enc) = Encap(pkR). It sends RES in AT_RES and enc in AT_PUB_HYBRID.

- The server verifies RES and computes ss = Decap(enc, skR). If Decap() fails, the server MUST behave as if authentication failed.

The peer and the server derive the keys as follows:

~~~
   MK        = PRF'(IK'|CK', "EAP-AKA'"|Identity)
   MK_HYBRID = PRF'(IK'|CK'|ss, "EAP-AKA' FS Hybrid"|
                    Length(Identity)|Identity|Length(enc)|enc)
   K_encr    = MK[0..127]
   K_aut     = MK[128..383]
   K_re      = MK_HYBRID[0..255]
   MSK       = MK_HYBRID[256..767]
   EMSK      = MK_HYBRID[768..1279]
~~~

Length(X) is the length of X in octets, encoded as a 2-octet unsigned integer in network byte order. The label "EAP-AKA' FS Hybrid" separates MK_HYBRID from MK_ECDHE (Section 6.3 of {{RFC9678}}).

For each authentication exchange, the server MUST generate a new hybrid KEM key pair. Key pairs MUST NOT be reused across authentication exchanges.

# Extensions to EAP-AKA' FS

## AT_PUB_HYBRID {#hybrid}

The format of the AT_PUB_HYBRID attribute is shown below.

~~~ artwork
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | AT_PUB_HYBRID |   Reserved    |            Length             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   |                       Value (variable)                        |
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

AT_PUB_HYBRID: Set to TBA1 by IANA.

Reserved: 1 octet. MUST be set to 0 on transmission and ignored on receipt.

Length: 2 octets. The length of the attribute in multiples of 4 octets, including the Type, Reserved, Length, and Value fields and any padding. This differs from {{RFC4187}}, where the Length field is 1 octet, because hybrid KEM encapsulation keys and ciphertexts exceed the 1020-octet maximum of a 1-octet Length field.

Value: In the EAP-Request, the hybrid KEM encapsulation key pkR. In the EAP-Response, the hybrid KEM ciphertext enc. Both are output by the hybrid KEM as defined in Section 4 of {{I-D.ietf-hpke-pq}} and are opaque to EAP-AKA'. The sender pads the Value field with zero octets to a multiple of 4 octets. The receiver removes the padding using the fixed encapsulation key or ciphertext length of the selected hybrid KEM.

# Security Considerations

The IND-CCA security of the hybrid KEMs is discussed in Section 7 of {{I-D.ietf-hpke-pq}}. Security levels and sizes of ML-KEM are discussed in {{RFC9958}}. For the security properties of EAP-AKA' FS, see Section 7 of {{RFC9678}}.

Implementations MUST generate hybrid KEM key pairs using a cryptographically secure random number generator ({{RFC4086}}).

The server MUST delete skR, and both sides MUST delete ss, once MK_HYBRID has been derived. A retained skR allows anyone who obtains it, together with the long-term key, to recover the session keys of the corresponding recorded exchange.

AT_MAC protects the AT_KDF_FS offer and the peer's selection, because the final EAP-Request/AKA'-Challenge carries both the selected value and the original list (Section 6.2 of {{RFC9678}}). An attacker that holds the long-term key can still remove the hybrid KEM offer. Deployments that need protection against this MUST require a hybrid KEM by local policy.

# IANA Considerations

## Attribute Type

IANA is requested to assign one new value in the "Attribute Types (Skippable Attributes 128-255)" registry under the "EAP-AKA and EAP-SIM Parameters" group:

~~~
   +=======+===============+=========================+
   | Value | Name          | Reference               |
   +=======+===============+=========================+
   | TBA1  | AT_PUB_HYBRID | THIS RFC                |
   +-------+---------------+-------------------------+
~~~

## AT_KDF_FS Key Derivation Function Values {#iana-kdf}

IANA is requested to add the following entries to the "EAP-AKA' AT_KDF_FS Key Derivation Function Values" registry under the "EAP-AKA and EAP-SIM Parameters" group:

~~~
   +=======+===============================+===========+
   | Value | Description                   | Reference |
   +=======+===============================+===========+
   | TBA2  | EAP-AKA' with MLKEM768-X25519 | THIS RFC  |
   +-------+-------------------------------+-----------+
   | TBA3  | EAP-AKA' with MLKEM768-P256   | THIS RFC  |
   +-------+-------------------------------+-----------+
   | TBA4  | EAP-AKA' with MLKEM1024-P384  | THIS RFC  |
   +-------+-------------------------------+-----------+
~~~

The hybrid KEMs are defined in Section 4 of {{I-D.ietf-hpke-pq}}.

--- back

# Acknowledgements

This draft leverages text from {{RFC9678}}.
