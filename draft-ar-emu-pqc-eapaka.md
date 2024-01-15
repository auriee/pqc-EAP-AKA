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

