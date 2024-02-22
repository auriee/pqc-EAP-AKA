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
The problem is EAP-AKA' FS uses traditional algorithms public-key algorithms (e.g., ECDH) which will fall to quantum cryptalanysis. The presence of a Cryptographically Relevant Quantum Computer (CRQC) would render state-of-the-art, traditional public-key algorithms deployed today obsolete and insecure, since the assumptions about the intractability of the mathematical problems for these algorithms that offer confident levels of security today no longer apply in the presence of a CRQC. The migration to PQC is unique in the history of modern digital cryptography in that neither the traditional algorithms nor the post-quantum algorithms are fully trusted to protect data for the 
required data lifetimes. The post-quantum algorithms face uncertainty about the underlying mathematics, compliance issues, unknown vulnerabilities, hardware and software implementations that have not had sufficient maturing time to rule out classical cryptanalytic attacks and implementation bugs. During the transition from traditional to post-quantum algorithms, there is a desire or a requirement for protocols that use both algorithm types.

# Conventions and Defintions

{::boilerplate bcp14-tagged}

# Background on EAP-AKA' with perfect forward secrecy

As described in draft https://datatracker.ietf.org/doc/draft-ietf-emu-aka-pfs/, the UDM has the EAP identity of UE. The UDM runs AKA algorithm to generate RAND, AUTN, XRES, CK and IK. Further it also derives CK’ and IK’ keys which are tied to a serving network name. The UDM now generates the ephemeral key pair and sends the public key of that key pair and the first EAP method message to the UE. In this EAP message, AT_PUB_ECDHE (carries public key) and the AT_KDF_FS(carries other FS related parameters). Both of these might be ignored of UE doesn’t support the Forward Secrecy extension. The UE checks if it wants to have a Forward extension in EAP AKA prime. If yes, then it will eventually respond with AT_PUB_ECDHE and MAC. If not, it will ignore AT_PUB_ECDHE. If UE wants to participate in FS extension, it will then generate its ECDH key pair, calculate a shared key based on its private key and UDM public key. The UDM will receive the RES from UE and AT_PUB_ECDHE. The shared key will be generated in UE and UDM with key pairs exchanged, and later master key is also generated.

MK_ECDHE = PRF'(IK'|CK'|SHARED_SECRET,"EAP-AKA' FS"| Identity).

# Overview of the PQC Enhancements

We suggest the following changes and enhancements:
• New attributes will be defined to carry the PQC KEM Public Key of HN_pk and 
UE_ct (cipher text) 
• New AT_KDF_FS attribute to indicate the key derivation function for Hybrid 
Master Key MK_HYBRID.
• Multiple AT_KDF_FS attributes will be included to handle the EAP peer not 
supporting PQC. 
• The Hybrid key derivation function will be included first to indicate higher priority 
than the traditional key derivation function.

# Protocol Design

# Acknowledgement
This draft leverages text from {{?I-D.draft-ietf-emu-aka-pfs-11}}

