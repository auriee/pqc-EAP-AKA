---
title: "Post-Quantum Cryptography enhancement in EAP-AKA'"
abbrev: "PQC in EAP-AKA'"
category: info

docname: draft-ar-emu-pqc-eapaka
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

informative:

--- abstract


The threat from CRQCs (Cryptographically relevant Quantum Computers) to public key cryptographic algorithms is quite real as is pervasive monitoring attacks that arise from this encryption breaking capability. These attacks develop over a period of time (Also termed as Harvest Now Decrypt Later attack) and are immune to perfect forward secrecy as well. An adversary can break a public key in the future even with perfect forward secrecy (PFS). if the adversary has also obtained knowledge of the long-term key, they could then compute the session keys as well as the shared secret, and any derived output keys.

This draft acknowledges the work done in {{?I-D.draft-ietf-emu-aka-pfs-11}} with regards to achieveing EAP-AKA' in perfect forward secrecy and aims to make the EAP-AKA' protocol defined in {{?RFC9048}} quantum-safe.

--- middle

# Introduction

Forward Secrecy for the Extensible Authentication Protocol Method for Authentication and Key Agreement (EAP-AKA' FS) defined in {{?I-D.draft-ietf-emu-aka-pfs-11}} updates the improved Extensible Authentication Protocol Method for 3GPP Mobile Network Authentication and Key Agreement (EAP-AKA') specified in RFC 9048, with an optional extension providing ephemeral key exchange. This prevents an attacker who has gained access to the long term key from obtaining session keys established in the past, assuming these have been properly deleted. EAP-AKA' FS mitigates passive attacks (e.g., large scale pervasive monitoring) against future sessions.
Nevertheless, EAP-AKA' FS uses traditional algorithms public-key algorithms (e.g., ECDH) which will be broken by a Cryptographically Relevant Quantum Computer (CRQC) using Shor's algorithm. The presence of a CRQC would render state-of-the-art, traditional public-key algorithms deployed today obsolete and insecure, since the assumptions about the intractability of the mathematical problems for these algorithms that offer confident levels of security today no longer apply in the presence of a CRQC. The migration to PQC is unique in the history of modern digital cryptography in that neither the traditional algorithms nor the post-quantum algorithms are fully trusted to protect data for the 
required data lifetimes. The post-quantum algorithms face uncertainty about the underlying mathematics, compliance issues, unknown vulnerabilities, hardware and software implementations that have not had sufficient maturing time to rule out classical cryptanalytic attacks and implementation bugs. During the transition from traditional to post-quantum algorithms, there is a desire or a requirement for protocols that use both algorithm types.

# Conventions and Defintions

{::boilerplate bcp14-tagged}

This document makes use of the terms defined in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}. For the purposes of this document, it is helpful to be able to divide cryptographic algorithms into two classes:

"Traditional Algorithm": An asymmetric cryptographic algorithm based on integer factorisation, finite field discrete logarithms or elliptic curve discrete logarithms. In the context of JOSE, examples of traditional key exchange algorithms include Elliptic Curve Diffie-Hellman Ephemeral Static {{?RFC6090}} {{?RFC8037}}. In the context of COSE, examples of traditional key exchange algorithms include Ephemeral-Static (ES) DH and Static-Static (SS) DH {{?RFC9052}}.

"Post-Quantum Algorithm": An asymmetric cryptographic algorithm that is believed to be secure against attacks using quantum computers as well as classical computers. Examples of PQC key exchange algorithms include Kyber.

"Hybrid" key exchange, in this context, means the use of two key exchange algorithms based on different cryptographic assumptions, e.g., one traditional algorithm and one Post-Quantum algorithm, with the purpose of the final shared secret key being secure as long as at least one of the component key exchange algorithms remains unbroken. It is referred to as PQ/T Hybrid Scheme in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}.

PQ/T Hybrid Key Encapsulation Mechanism: A Key Encapsulation mechanism (KEM) made up of two or more component KEM algorithms where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

# Background on EAP-AKA' with perfect forward secrecy

As described in the draft {{?I-D.draft-ietf-emu-aka-pfs-11}}, the UDM has the EAP identity of UE. The UDM runs AKA algorithm to generate RAND, AUTN, XRES, CK and IK. Further it also derives CK’ and IK’ keys which are tied to a serving network name. The UDM now generates the ephemeral key pair and sends the public key of that key pair and the first EAP method message to the UE. In this EAP message, AT_PUB_ECDHE (carries public key) and the AT_KDF_FS(carries other FS related parameters). Both of these might be ignored of UE doesn’t support the Forward Secrecy extension. The UE checks if it wants to have a Forward extension in EAP AKA'. If yes, then it will eventually respond with AT_PUB_ECDHE and MAC. If not, it will ignore AT_PUB_ECDHE. If UE wants to participate in FS extension, it will then generate its ECDH key pair, calculate a shared key based on its private key and UDM public key. The UDM will receive the RES from UE and AT_PUB_ECDHE. The shared key will be generated in UE and UDM with key pairs exchanged, and later master key is also generated.

MK_ECDHE = PRF'(IK'|CK'|SHARED_SECRET,"EAP-AKA' FS"| Identity).

# PQC Enhancements in Design

We suggest the following changes and enhancements:
- New attributes will be defined to carry the PQC KEM Public Key of HN_pk and UE_ct (cipher text) 
- New AT_KDF_FS attribute to indicate the key derivation function for Hybrid Master Key MK_HYBRID.
- Multiple AT_KDF_FS attributes will be included to handle the EAP peer not supporting PQC. 
- The Hybrid key derivation function will be included first to indicate higher priority than the traditional key derivation function.

# Protocol Construction
This section defines the construction for hybrid key exchange in EAP-AKA' FS. Hybrid key exchange refers to using multiple key exchange algorithms simultaneously and combining the result with the goal of providing security even if all but one of the component algorithms is broken. It is motivated by transition to post-quantum cryptography.

We outline the following steps in the protocol:

1. USIM performs the SUPI to SUCI concealment for initial registration and sends it to ME. 5G-GUTI is used by ME if it is already registered.
2. ME will send the registration request with SUCI or 5G-GUTI to AMF/SEAF via RAN. The RAN will forward the content to the network entity AUSF and UDM of HN.
3. UDM/ARPF/SIDF will perform the de-concealment and also selects the authentication method. UDM generates the home network PQC KEM Public key(HN_pk) and private 
key(HN_sk) pairs.
4. The UDM will generate the AKA challenge and sends the EAP AKA' Authentication Vector (AV).
5. The UDM will send the EAP AKA" AV along with SUPI, AKMA indication and Home network KEM private and public key to AUSF (HN_PQC_KEM_Public_key (HN_pk) and HN_PQC_KEM_Private_key (HN_sk)).
   kemKeyGen() -> (HN_pk, HN_sk)
6. The AUSF will store the expected response XRES and the Home network KEM private key HN_PQC_KEM_Private_key – HN_sk. The AUSF and AMF will forward the EAP AKA' 
AV to UE along with HN_PQC_KEM_Public_key (HN_pk).
7. The USIM will validate the AKA challenge received , also verifies the MAC-I. After the verification is successful and if the UE also supports the Forward secrecy , UE will invoke 
kemEncaps using HN_pk:
   kemEncaps (HN_pk) -> UE_ct, ss.
UE_ct is the ciphertext representing the encapsulated key and ss is shared secret. Hybrid shared key Ks is generated by the UE using the KEM combiner (e.g., 
{{?I-D.ounsworth-cfrg-kem-combiners}}) with parameters “ss” (Shared secret generated out of KemEncaps function) and traditional shared secret of ECDHE.
The KEM combiner can also be a "hash and concatenation" based approach to use a “hybrid” shared secret (Ks). 
8. The UE will send the Authentication response RES and UE_ct to AUSF via AMF. 
9. The AUSF will verify the RES with XRES. The AUSF will use the UE_ct and home network private key HN_sk to generate shared secret.
   Kemdecaps(UE_ct, HN_sk) -> ss.
The generated ss from KEMdecaps along with traditional shared secret of ECHDE used as input to KEM combiner to generate the hybrid shared secret Ks. The UE and AUSF first generate the MK_HYBRID and subsequently generate MSK, EMSK as shown below.
   MK_HYBRID = PRF'(IK'|CK'| HYBRID_SHARED_SECRET,"EAP-AKA' FS"| Identity) 
      K_encr = MK[0..127]
      K_aut = MK[128..383]
      K_re = MK_HYBRID [0..255] 
      MSK = MK_HYBRID [256..767] 
      EMSK = MK_HYBRID [768..1279]

# IANA Considerations

# Acknowledgement
This draft leverages text from {{?I-D.draft-ietf-emu-aka-pfs-11}}

