---
title: "Enhancing Security in EAP-AKA' with Hybrid Post-Quantum Cryptography"
abbrev: "PQC Hybrid in EAP-AKA prime"
category: std

docname: draft-ar-emu-pqc-eapaka-03
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
    city: Munich
    country: Germany
    email: "aritra.banerjee@nokia.com"
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"

normative:
  RFC9180:
  RFC9048:
  RFC9678:

informative:
  
  FIPS203:
     title: "FIPS-203: Module-Lattice-based Key-Encapsulation Mechanism Standard"
     target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.pdf
     date: false

--- abstract

Forward Secrecy for the Extensible Authentication Protocol Method for Authentication and Key Agreement (EAP-AKA' FS) is specified in {{RFC9678}}, providing updates to {{RFC9048}} with an optional extension that offers ephemeral key exchange using the traditional Ephemeral Elliptic Curve Diffie-Hellman (ECDHE) key agreement algorithm for achieving perfect forward secrecy (PFS). However, it is susceptible to future threats from Cryptographically Relevant Quantum Computers, which could potentially compromise a traditional ephemeral public key. If the adversary has also obtained knowledge of the long-term key and ephemeral public key, it could compromise session keys generated as part of the authentication run in EAP-AKA'.

This draft aims to enhance the security of EAP-AKA' FS protocol by leveraging PQ/T Hybrid {{?I-D.ietf-pquip-pqt-hybrid-terminology}} algorithms to make it quantum-safe.

--- middle

# Introduction

Forward Secrecy for the Extensible Authentication Protocol Method for Authentication and Key Agreement (EAP-AKA' FS) defined in {{RFC9678}} updates the improved Extensible Authentication Protocol Method for 3GPP Mobile Network Authentication and Key Agreement (EAP-AKA') specified in {{RFC9048}}, with an optional extension providing ephemeral key exchange. This prevents an attacker who has gained access to the long term key from obtaining session keys established in the past, assuming these have been properly deleted. EAP-AKA' FS mitigates passive attacks (e.g., large scale pervasive monitoring) against future sessions.

Nevertheless, EAP-AKA' FS uses traditional algorithms public-key algorithms (e.g., ECDH) which will be broken by a Cryptographically Relevant Quantum Computer (CRQC) using Shor's algorithm. The presence of a CRQC would render state-of-the-art, traditional public-key algorithms deployed today obsolete and insecure, since the assumptions about the intractability of the mathematical problems for these algorithms that offer confident levels of security today no longer apply in the presence of a CRQC. A CRQC could recover the SHARED_SECRET from the ECDHE public keys (Section 6.3 of {{RFC9678}}). If the adversary has also obtained knowledge of the long-term key, it could then compute CK', IK', and the SHARED_SECRET, and any derived output keys. This means that the CRQC would disable the forward security capability provided by {{RFC9678}}.

The migration to PQC is unique in the history of modern digital cryptography in that neither the traditional algorithms nor the post-quantum algorithms are fully trusted to protect data for the required data lifetimes. The post-quantum algorithms face uncertainty about the underlying mathematics, compliance issues, unknown vulnerabilities, hardware and software implementations that have not had sufficient maturing time to rule out classical cryptanalytic attacks and implementation bugs. During the transition from traditional to post-quantum algorithms, there is a desire or a requirement for protocols that use both algorithm types.

This specification defines PQ/T Hybrid KEMs for use with EAP-AKA' FS {{?I-D.irtf-cfrg-hybrid-kems}}. Although the proposed mechanism is applicable to any post-quantum Key Encapsulation Mechanism (PQ-KEM), this document specifies its use with Module-Lattice-based Key Encapsulation Mechanisms (ML-KEMs). ML-KEM provides a one-pass (store-and-forward) cryptographic method for an originator to securely transmit keying material to a recipient using the recipient’s ML-KEM public key. Three parameter sets for ML-KEM are defined in {{FIPS203}}, namely ML-KEM-512, ML-KEM-768, and ML-KEM-1024, listed in order of increasing security strength and decreasing performance.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

This document makes use of the terms defined in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}. For the purposes of this document, it is helpful to be able to divide cryptographic algorithms into two classes:

"Traditional Algorithm": An asymmetric cryptographic algorithm based on integer factorisation, finite field discrete logarithms or elliptic curve discrete logarithms. In the context of JOSE, examples of traditional key exchange algorithms include Elliptic Curve Diffie-Hellman Ephemeral Static {{?RFC6090}} {{?RFC8037}}. In the context of COSE, examples of traditional key exchange algorithms include Ephemeral-Static (ES) DH and Static-Static (SS) DH {{?RFC9052}}.

"Post-Quantum Algorithm": An asymmetric cryptographic algorithm that is believed to be secure against attacks using quantum computers as well as classical computers. Examples of PQC key exchange algorithms include Kyber.

"Hybrid" key exchange, in this context, means the use of two key exchange algorithms based on different cryptographic assumptions, e.g., one traditional algorithm and one Post-Quantum algorithm, with the purpose of the final shared secret key being secure as long as at least one of the component key exchange algorithms remains unbroken. It is referred to as PQ/T Hybrid Scheme in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}.

PQ/T Hybrid Key Encapsulation Mechanism: A Key Encapsulation mechanism (KEM) made up of two or more component KEM algorithms where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

# Background on EAP-AKA' with perfect forward secrecy

 In EAP-AKA', The authentication vector (AV) contains a random part RAND, an authenticator part AUTN used for authenticating the network to the USIM, an expected result part XRES, a 128-bit session key for integrity check IK, and a 128-bit session key for encryption CK.

As described in the draft {{RFC9678}}, the server has the EAP identity of the peer. The server asks the AD to run AKA algorithm to generate RAND, AUTN, XRES, CK and IK. Further it also derives CK’ and IK’ keys which are tied to a particular network name. The server now generates the ephemeral key pair and sends the public key of that key pair and the first EAP method message to the peer. In this EAP message, AT_PUB_ECDHE (carries public key) and the AT_KDF_FS(carries other FS related parameters). Both of these might be ignored of USIM doesn’t support the Forward Secrecy extension. The peer checks if it wants to have a Forward extension in EAP AKA'. If yes, then it will eventually respond with AT_PUB_ECDHE and MAC. If not, it will ignore AT_PUB_ECDHE. If the peer wants to participate in FS extension, it will then generate its ECDH key pair, calculate a shared key based on its private key and server public key. The server will receive the RES from peer and AT_PUB_ECDHE. The shared key will be generated both in the peer and the server with key pairs exchanged, and later master key is also generated.

~~~
MK_ECDHE = PRF'(IK'| CK'|SHARED_SECRET,"EAP-AKA' FS"|Identity)
~~~

# Hybrid Enhancements by Design

We suggest the following changes and enhancements:

- A new attribute, AT_PUB_HYBRID, is defined to carry the public key, which is the concatenation of traditional and PQC KEM public keys from the EAP server. The AT_PUB_HYBRID attribute will carry the encapsulated key, which is formed by concatenating the encapsulated key (enc) from the traditional KEM algorithm and the ciphertext (ct) from the PQC KEM Encapsulation function from the EAP peer.

- The AT_KDF_FS attribute is updated to indicate the PQ/T KEM and HKDF for generating the Hybrid Master Key MK_HYBRID.

- Multiple AT_KDF_FS attributes is included in the EAP-Request to handle the EAP peer not supporting PQ/T Hybrid KEM.

- The Hybrid key derivation function will be included first in the EAP-Request to indicate a higher priority than the traditional key derivation function.

# Protocol Construction

This section defines the construction for hybrid key exchange in EAP-AKA' FS. Hybrid key exchange refers to using multiple key exchange algorithms simultaneously and combining the result with the goal of providing security even if all but one of the component algorithms is broken. It is motivated by transition to post-quantum cryptography.
 
## Protocol Call Flow


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
      |      | generating RAND, AUTN, XRES, CK, IK. Typically, the    |
      |      | AD performs the first part of key derivations so that  |
      |      | the authentication server gets the CK' and IK' keys    |
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
      |      | generates a PQC KEM key pair, an ephemeral ECDHE key   |
      |      | pair, sends the hybrid (PQC + ECDHE) public key of that|
      |      | key pair and the first EAP method message to the peer  |
      |      | In the message the AT_PUB_HYBRID attribute             |
      |      | carries the concatenation of PQC KEM and ECDHE public  |
      |      | keys and the AT_KDF_FS attribute carries other         |
      |      | FS-related parameters. Both of these are               |
      |      | skippable attributes that can be ignored if the peer   |
      |      | does not support this extension.                       |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              |     EAP-Req/AKA'-Challenge |                |
      |              |  AT_RAND, AT_AUTN, AT_KDF, |                |
      |              |   AT_KDF_FS, AT_KDF_INPUT, |                |
      |              |       AT_PUB_HYBRID, AT_MAC|                |
      |              |<---------------------------+                |
    +--+--------------+----------------------------+---------+     |
    | The peer checks if it wants to do the FS extension. If |     |
    | yes, it will eventually respond with AT_PUB_HYBRID and |     |
    | AT_MAC. If not, it will ignore AT_PUB_HYBRID and       |     |
    | AT_KDF_FS and base all calculations on basic EAP-AKA'  |     |
    | attributes, continuing just as in EAP-AKA' per RFC     |     |
    | 9048 rules. In any case, the peer needs to query the   |     |
    | auth parameters from the USIM card.                    |     |
    +--+--------------+----------------------------+---------+     |
      |              |                            |                |
      |   RAND, AUTN |                            |                |
      |<-------------+                            |                |
      |              |                            |                |
      | CK, IK, RES  |                            |                |
      +------------->|                            |                |
    +--+--------------+----------------------------+---------+     |
    | The peer now has everything to respond. If it wants to |     |
    | participate in the FS extension, it will then generate |     |
    | its ECDHE key pair, calculate a hybrid shared secret   |     |
    | key based on the server's PQC KEM public key, its ECDHE|     |
    | key pair and the server's ECDHE public key. Finally,   |     |
    | it proceeds to derive all EAP-AKA' key values and      |     |
    | constructs a full response.                            |     |
    +--+--------------+----------------------------+---------+     |
      |              |                            |                |
      |              | EAP-Resp/AKA'-Challenge    |                |
      |              | AT_RES, AT_PUB_HYBRID,     |                |
      |              | AT_MAC                     |                |
      |              +--------------------------->|                |
      |      +-------+----------------------------+----------------+--+
      |      | The server now has all the necessary values. It        |
      |      | generates the Hybrid shared secret and checks the RES  |
      |      | and MAC values received in AT_RES and AT_MAC,          |
      |      | respectively. Success requires both to be found        |
      |      | correct. Note that when this document is used,         |
      |      | the keys generated from EAP-AKA' are based on CK, IK,  |
      |      | and the ECDHE and PQC KEM values. Even if there was an |
      |      | attacker who held the long-term key, only an active    |
      |      | attacker could have determined the generated session   |
      |      | keys; additionally an attacker with a cryptographically|
      |      | relevant quantum computer cannot get access to the     |
      |      | server KEM private key and decrypt the data.           |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              |                EAP-Success |                |
      |              |<---------------------------+                |
      |              |                            |                |
      

## Key Steps in protocol construction

We outline the following key steps in the protocol:

- Server generates the PQC KEM Public key(pq_PK), private key (pq_SK) pair and the ECDH public key (trad_PK), private key (trad_SK) pair. The server will generate and send the EAP AKA' Authentication Vector (AV). As defined in section 5.2 of {{?I-D.irtf-cfrg-hybrid-kems}} the server PQC KEM and ECDH key pairs are derived as:

~~~

   def expandDecapsulationKey(sk):
      expanded = SHAKE256(sk, 112)
      (pq_PK, pq_SK) = ML-KEM-768.KeyGen_internal(expanded[0:32], expanded[32:64])
      trad_SK = P-256.ScalarFromBytes(expanded[64:112])
      trad_PK = P-256.SerializeElement(P-256.ScalarMultBase(trad_SK))
      return (pq_SK, trad_SK, pq_PK, trad_PK)
   
   def KeyGen():
      sk = random(32)
      (pq_SK, trad_SK, pq_PK, trad_PK) = expandDecapsulationKey(sk)
      return sk, concat(pq_PK, trad_PK)

~~~

- The server will store the expected response XRES, the ECDH private key trad_SK and the PQC KEM private key pq_SK. The server will forward the EAP AKA' AV to peer along with pq_PK and trad_PK.

- The USIM will validate the AUTH received, also verifies the MAC. After the verification is successful and if the peer also supports the Forward secrecy, peer will invoke Encapsulate using concat(pq_PK, trad_PK) as defined in section 5.4 of {{?I-D.irtf-cfrg-hybrid-kems}}:

~~~
   def Encaps(pk):
      pq_PK = pk[0:1184]
      trad_PK = P-256.DeserializeElement(pk[1184:1217])
      (pq_SS, pq_CT) = ML-KEM-768.Encaps(pq_PK)
      ek = P-256.RandomScalar()
      trad_CT = P-256.SerializeElement(P-256.ScalarBaseMult(ek))
      trad_SS = P-256.SerializeElementAsSharedSecret(P-256.ScalarMult(trad_PK, ek))
      ss = SHA3-256(pq_SS, trad_SS, trad_CT, pk[1184:1217], label)
      ct = concat(pq_CT, trad_CT)
      return (ss, ct)
~~~

"ct" is the concatenation of the ciphertext from PQC KEM and encapsulated key from ECDH whereas "ss" is hybrid shared secret key. Hybrid shared key ss is generated by the peer using the Encapsulate() ({{?I-D.irtf-cfrg-hybrid-kems}}).

- The peer will send the Authentication response RES and ct to the server.

- The server will verify the RES with XRES. The server will use the ct, PQC KEM private key pq_SK and ECDH private key trad_SK to generate shared secret:

~~~
   def Decaps(sk, ct):
      (pq_SK, trad_SK, pq_PK, trad_PK) = expandDecapsulationKey(sk)
      pq_CT = ct[0:1088]
      trad_CT = P-256.DeserializeElement(ct[1088:1121])
      pq_SS = ML-KEM-768.Decapsulate(pq_SK, pq_CT)
      trad_SS = P-256.SerializeElementAsSharedSecret(P-256.ScalarMult(trad_SK, trad_CT))
      return SHA3-256(pq_SS, trad_SS, ct[1088:1121], trad_PK, label)
~~~

The generated ss from Decapsulate is the hybrid shared secret key derived from PQC KEM and traditional ECDH. The peer and the server first generate the MK_HYBRID and subsequently generate MSK, EMSK as shown below:

~~~
   MK = PRF'(IK'|CK',"EAP-AKA'"|Identity)
   HYBRID_SHARED_SECRET, ct = Encapsulate(pKR)
   MK_HYBRID = PRF'(IK'|CK'| HYBRID_SHARED_SECRET,"EAP-AKA' FS"| Identity) 
   K_encr = MK[0..127]
   K_aut = MK[128..383]
   K_re = MK_HYBRID [0..255] 
   MSK = MK_HYBRID [256..767] 
   EMSK = MK_HYBRID [768..1279]
~~~

where, pkR is concatenation of PQC KEM and traditional public keys of the receiver, ct is concatenation of the ciphertext from the PQC KEM and encapsulated key from ECDH; the Encapsulate() function is perfomed by the peer only.

# Extensions to EAP-AKA' FS

## AT_PUB_HYBRID {#hybrid}

   The format of the AT_PUB_HYBRID attribute is shown below.

~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | AT_PUB_HYBRID    |   Reserved    |      Length (in bytes)     |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     |                       Value (variable)                        |
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
      
   The fields are as follows:

   AT_PUB_HYBID:
   
   This is set to TBA1 BY IANA.

   Reserved:
   
   A 1-byte field reserved for future use. Including this field ensures that the fixed header (Type, Reserved, Length) is 4 bytes long, maintaining 4-byte alignment for the following Value field. The Reserved field MUST be set to 0 on transmission and ignored on receipt.
   
   Length:
   
   A 2-byte unsigned integer indicating the total length of the attribute in bytes, including the Type, Reserved, Length, and Value fields, as well as any padding. The length is expressed in multiples of 4 bytes.

   This differs from the attribute format used in EAP-AKA {{!RFC4187}}, where the Length field is 1 byte.The modification is necessary because PQC KEM public keys, such as those defined in ML-KEM-1024, will be 1568 bytes, which would exceed the 1024-byte limit imposed by the original EAP-AKA format.

   Value: 

      *  EAP-Request: It contains the public key, which is the concatenation of 
         traditional and PQC KEM public keys from the EAP server.
      *  EAP-Response: It contains the encapsulated key, which is formed by 
         concatenating the ciphertext (ct) from the PQC KEM Encapsulation function
         and the encapsulated key (enc) from the traditional KEM algorithm and from the 
         EAP peer.

   Because the length of the attribute must be a multiple of 4 bytes, the sender pads the Value field with zero bytes when necessary. To retain the security of the keys, the sender SHALL generate a fresh value for each run of the protocol.

# Security Considerations

ML-KEM is IND-CCA2 secure based on multiple analyses. The ML-KEM variant and its underlying components should be selected consistently with the desired security level. For further clarity on the sizes and security levels of ML-KEM variants, please refer to the tables in Sections 12 and 13 of {{?I-D.ietf-pquip-pqc-engineers}}.

The security of the ML-KEM algorithm depends on a high-quality pseudo-random number generator. For further discussion on random number generation, see {{?RFC4086}}.

In general, good cryptographic practice dictates that a given ML-KEM key pair should be used in only one EAP session. This practice mitigates the risk that compromising one EAP session will not compromise the security of another EAP session and is essential for maintaining forward security.

For security properties of traditional ECDHE for EAP-AKA FS, see section 7 of {{RFC9678}}. The overall Hybrid scheme needs to be IND-CCA2 robust; i.e., atleast one of the schemes should be IND-CCA2 secure.

# IANA Considerations

   One new value (TBA1) in the skippable range need to be assigned by IANA 
   for AT_PUB_HYBRID ({{hybrid}}) in the "Attribute Types" registry 
   under the "EAP-AKA and EAP-SIM Parameters" group.

   IANA is requested to update the registry "EAP-AKA' AT_KDF_FS
   Key Derivation Function Values" with the Hybrid key derivation 
   function entry:

~~~
   +=========+===============================+=========================+
   | Value   | Description                   | Reference               |
   +=========+===============================+=========================+
   | TBA2    | X25519Kyber768Draft00         | [TBD BY IANA: THIS RFC] |
   +=========+===============================+=========================+
~~~

--- back

# Acknowledgements

This draft leverages text from {{RFC9678}}.

