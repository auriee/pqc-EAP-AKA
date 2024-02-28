---
title: "Use of Post-quantum KEM in JOSE and COSE"
abbrev: "Use of Post-quantum KEM in JOSE and COSE"
category: std

docname: draft-ra-cose-hybrid-encrypt
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "COSE"
keyword:
 - PQC
 - COSE
 - JOSE
 - Hybrid

 

venue:
  group: "cose" 
  type: "Working Group"
  mail: "cose@ietf.org" 
  arch: "https://mailarchive.ietf.org/arch/browse/cose/" 
  

stand_alone: yes
pi: [toc, sortrefs, symrefs, strict, comments, docmapping]

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"
 
normative:
  RFC2119:
  RFC8174:
  RFC7516:
  JOSE-IANA:
     author:
        org: IANA
     title: JSON Web Signature and Encryption Algorithms
     target: https://www.iana.org/assignments/jose/jose.xhtml


informative:
 
  PQCAPI:
     title: "PQC - API notes"
     target: https://csrc.nist.gov/CSRC/media/Projects/Post-Quantum-Cryptography/documents/example-files/api-notes.pdf
     date: false
  FO:
     title: "Secure Integration of Asymmetric and Symmetric Encryption Schemes"
     target: https://link.springer.com/article/10.1007/s00145-011-9114-1
     date: false
  HHK:
     title: "A Modular Analysis of the Fujisaki-Okamoto Transformation"
     target: https://link.springer.com/chapter/10.1007/978-3-319-70500-2_12
     date: false
  FIPS203-ipd:
     title: "Module-Lattice-based Key-Encapsulation Mechanism Standard"
     target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.ipd.pdf
     date: false
     
--- abstract

This document describes the conventions for using a Post-quantum Key Encapsulation Mechanism algorithm (KEM) within JOSE and COSE. 

--- middle

# Introduction

Quantum computing is no longer perceived as a conjecture of computational sciences and theoretical physics.  Considerable research efforts and enormous corporate and government funding for the development of practical quantum computing systems are being invested currently. As such, as quantum technology advances, there is the potential for future quantum computers to have a significant impact on current cryptographic systems. 

Extensive research has developed Post-quantum key encapsulation mechanisms (PQ-KEM) in order to provide secure key establishment against an adversary with access to a quantum computer.

As the National Institute of Standards and Technology (NIST) is still in the process of selecting the new post-quantum cryptographic algorithms that are secure against both quantum and classical computers, the purpose of this document is to propose a Post-quantum KEM solution to protect the confidentiality of content encrypted using JOSE and COSE against the quantum threat.

Although this mechanism could thus be used with any post-quantum KEM, this docuemnt specifies the case where the
PQ-KEM algorithm is ML-KEM. The Module-Lattice-based Key-Encapsulation Mechanism (ML-KEM) Algorithm is a one-pass (store-and-forward) cryptographic mechanism for an originator to securely send keying material to a recipient
using the recipient's ML-KEM public key. Three parameters sets for the ML-KEM Algorithm are specified by NIST in {{FIPS203-ipd}}. In order of increasing security strength (and decreasing performance), these parameter sets
are ML-KEM-512, ML-KEM-768, and ML-KEM-1024.

# Conventions and Definitions

{::boilerplate bcp14-tagged}
This document makes use of the terms defined in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}. For the purposes of this document, it is helpful to be able to divide cryptographic algorithms into two classes:

"Traditional Algorithm":  An asymmetric cryptographic algorithm based on integer factorisation, finite field discrete logarithms or elliptic curve discrete logarithms. In the context of JOSE, examples of traditional key exchange algorithms include Elliptic Curve Diffie-Hellman Ephemeral Static {{?RFC6090}} {{?RFC8037}}. In the context of COSE, examples of traditional key exchange algorithms include Ephemeral-Static (ES) DH and Static-Static (SS) DH {{?RFC9052}}. 

"Post-Quantum Algorithm":  An asymmetric cryptographic algorithm that is believed to be secure against attacks using quantum computers as well as classical computers. Post-quantum algorithms can also be called quantum-resistant or quantum-safe algorithms. Examples of Post-Quantum Algorithm include ML-KEM.

## Post-Quantum Key Encapsulation Mechanisms

For the purposes of this document, we consider a Key Encapsulation Mechanism (KEM) to be any asymmetric cryptographic scheme comprised of algorithms satisfying the following interfaces [PQCAPI].  

* def kemKeyGen() -> (pk, sk)
* def kemEncaps(pk) -> (ct, ss)
* def kemDecaps(ct, sk) -> ss

where pk is public key, sk is secret key, ct is the ciphertext representing an encapsulated key, and ss is shared secret.

KEMs are typically used in cases where two parties, hereby refereed to as the "encapsulater" and the "decapsulater", wish to establish a shared secret via public key cryptography, where the decapsulater has an asymmetric key pair and has previously shared the public key with the encapsulater.
  
# Design Rationales

The JSON Web Algorithms (JWA) {{?RFC7518}} in Section 4.6 defines two ways of using the key agreement result. When Direct Key Agreement is employed, the shared secret established through the Traditional Algorithm will be the content encryption key (CEK). When Key Agreement with Key Wrapping is employed, the shared secret established through the Traditional Algorithm will wrap the CEK. If multiple recipients are needed, then the version with key wrap is used. Similarly, COSE in Sections 8.5.4 and 8.5.5 {{?RFC9052}} defines the Direct Key Agreement and Key Agreement with Key Wrap classes. This document proposes the use of Post-Quantum Algorithms in these two modes.

It is essential to note that in the PQ-KEM, one needs to apply Fujisaki-Okamoto {{FO}} transform or its variant {{HHK}} on the PQC KEM part to ensure that the overall scheme is IND-CCA2 secure as mentioned in {{?I-D.ietf-tls-hybrid-design}}. The FO transform is performed using the KDF such that the PQC KEM shared secret achieved is IND-CCA2 secure. In this case, one can re-use the PQC KEM public keys but depending on some upper bound that must adhered to.

Note that during the transition from traditional to post-quantum algorithms, there may be a desire or a requirement for protocols that incorporate both types of algorithms until the post-quantum algorithms are fully trusted. The terminology for Post-Quantum and Traditional Hybrid Schemes is defined in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}. HPKE with COSE and JOSE is presented in {{?I-D.ietf-rha-jose-hpke-encrypt}} and {{?I-D.ietf-cose-hpke}}. These specifications can be extended to support hybrid post-quantum Key Encapsulation Mechanisms (KEMs) as defined in {{?I-D.ietf-westerbaan-cfrg-hpke-xyber768d00}}.

# KEM PQC Algorithms

The National Institute of Standards and Technology (NIST) started a process to solicit, evaluate, and standardize one or more quantum-resistant public-key cryptographic algorithms, as seen [here](https://csrc.nist.gov/projects/post-quantum-cryptography). Said process has reached its [first announcement](https://csrc.nist.gov/publications/detail/nistir/8413/final) in July 5, 2022, which stated which candidates to be standardized for KEM:

* Key Encapsulation Mechanisms (KEMs): [CRYSTALS-Kyber](https://pq-crystals.org/kyber/): ML-KEM, previously known 
 as Kyber is a module learning with errors (MLWE)-based key encapsulation mechanism. These were mapped by NIST to the three security levels defined in the NIST PQC Project, Level 1, 3, and 5. These levels correspond to the hardness of breaking AES-128, AES-192 and AES-256 respectively.

NIST announced as well that they will be [opening a fourth round](https://csrc.nist.gov/csrc/media/Projects/post-quantum-cryptography/documents/round-4/guidelines-for-submitting-tweaks-fourth-round.pdf) to standardize an alternative KEM, and a [call](https://csrc.nist.gov/csrc/media/Projects/pqc-dig-sig/documents/call-for-proposals-dig-sig-sept-2022.pdf) for new candidates for a post-quantum signature algorithm.

## ML-KEM

ML-KEM offers several parameter sets with varying levels of security and performance trade-offs. This document specifies the use of the ML-KEM algorithm at three security levels: ML-KEM-512, ML-KEM-768, and ML-KEM-1024. ML-KEM key generation, encapsulation and decaspulation functions are defined in {{?I-D.cfrg-schwabe-kyber}}.    The main security property for KEMs standardized in the NIST Post-Quantum Cryptography Standardization Project 
is indistinguishability under adaptive chosen ciphertext attacks (IND-CCA2) (Section 10.2 of {{?I-D.ietf-pquip-pqc-engineers}}). The public/private key sizes, ciphertext key size, and PQ security levels of ML-KEM are detailed in Section 12 of {{?I-D.ietf-pquip-pqc-engineers}}.

## PQ-KEM Encryption

The message encryption process is as follows. 

1.  Generate a inital shared secret SS' and the associated ciphertext CT
    using the KEM encaspulation function and the recipient's public
    key recipPubKey:

          (SS', CT) = kemEncaps(recipPubKey)

2.  Derive a final shared secret SS of length SSLen bytes from
    the initial shared secret SS' using the underlying key derivation
    function:

          SS = KDF(SS', SSLen)

In Direct Key Agreement mode, the output of the KDF MUST be a key of the same length as that used by encryption algorithm. In Key Agreement with Key Wrapping mode, the output of the KDF MUST be a key of the length needed for the specified key wrap algorithm. 

When Direct Key Agreement is employed, SS is the CEK. When Key Agreement with Key Wrapping is employed, SS is used to wrap the CEK.

## PQ-KEM Decryption {#decrypt}

The message decryption process is as follows.

1.  Decapsulate the ciphertext CT using the KEM decaspulation
    function and the recipient's private key to retrieve the initial shared
    secret SS':

          SS' = kemDecaps(recipPrivKey, CT)

    If the decapsulation operation outputs an error, output "decryption error", and stop.

2.  Derive the final shared secret SS of length SSLen bytes from
    the inital secret SS' using the underlying key derivation
    function:

          SS = KDF(SS', SSLen)

# Post-quantum KEM in JOSE

JSON Web Algorithms (JWA) Section 4.6 of {{RFC7518}} defines two ways to use public key cryptography with JWE:

* Direct Key Agreement
* Key Agreement with Key Wrapping

This specification describes these two modes of use for PQ-KEM in JWE. Unless otherwise stated, no changes to the processes described in {{RFC7516}} have been made.

If the 'alg' header parameter is set to the 'PQ-Direct' value (as defined in {{IANA}}), PQ KEM is used in Direct Key Agreement mode; otherwise, it is in Key Agreement with Key Wrapping.

## Direct key agreement 

*  The "alg" Header Parameter MUST be "PQ-Direct", "enc" MUST be an PQ-KEM algorithm from JSON Web Signature and Encryption Algorithms in {{JOSE-IANA}} and they MUST occur only within the JWE Protected Header.

*  The JWE Ciphertext must include the concatenation of the output ('ct') from the PQ KEM Encaps algorithm, encoded using base64url, along with the base64url-encoded ciphertext output obtained by encrypting the plaintext using the Content Encryption Key (CEK). This encryption process corresponds to step 15 of the {{RFC7518}}. 

* The recipient will seperate the 'ct' (output from the PQ KEM Encaps algorithm) from JWE Ciphertext to decode it and then derive the CEK using the process defined in {{decrypt}}. The ciphertext sizes of ML-KEM are discussed in Section 12 of {{?I-D.ietf-pquip-pqc-engineers}}.

*  The JWE Encrypted Key MUST be absent.

## Key Agreement with Key Wrapping

The CEK will be generated using the process explained in {{decrypt}}. Subsequently, the plaintext will be encrypted using the CEK, as detailed in Step 15 of Section 5.1 of {{RFC7516}}. The 'enc' (Encryption Algorithm) Header Parameter MUST specify a content encryption algorithm from the JSON Web Signature and Encryption Algorithms defined in {{JOSE-IANA}}.

# JOSE Ciphersuite Registration {#JOSE-PQ-KEM}

This specification registers a number of PQ-KEM ciphersuites for use with JOSE. A ciphersuite is a group of algorithms, often sharing component algorithms such as hash functions, targeting a security level.

An HPKE ciphersuite, is composed of the following choices:

- PQ-KEM Algorithm
- KDF Algorithm
- AEAD Algorithm

All security levels of ML-KEM internally utilize SHA3-256, SHA3-512, SHAKE256, and SHAKE512. This internal usage influences the selection of the Key Derivation Function (KDF) within this document.

For readability the algorithm ciphersuites labels are built according to the following scheme: 

~~~
PQ-<PQ-KEM>-<KDF>-<AEAD>
~~~

* In Direct key agreement, the parameter "enc" MUST be specified, and its value MUST be one of the values specified in the table below:
  
              +===============================+===================================+
              | alg                           | Description                       |
              +===============================+===================================+
              | PQ-MLKEM512-SHA3-256-AES128   | ML-KEM-512 + SHA3-256 + AES128    |
              +===============================+===================================+              
              | PQ-MLKEM768-SHA3-384-AES256   | ML-KEM-768 + SHA3-384 + AES256    |
              +===============================+===================================+
              | PQ-MLKEM1024-SHA3-512-AES256  | ML-KEM-1024 + SHA3-512 + AES256   |
              +===============================+===================================+

                                 Table 1

* In Key Agreement with Key Wrapping, the parameter "alg" MUST be specified, and its value MUST be one of the values specified in the table above.

The specification allows a small number of "known good" PQ-KEM ciphersuites instead of allowing arbitrary combinations of PQC algorithms, HKDF and AEAD Algorithms. It follows the recent trend in protocols to only allow a small number of "known good" configurations that make sense, instead of allowing arbitrary combinations of individual configuration choices that may interact in dangerous ways. 

# COSE Ciphersuite Registration {#COSE-PQ-KEM}

The following table maps terms between JOSE and COSE for PQ-KEM ciphersuites.

        +==============+===================+====================+================================+
        | Name                          | Value  | Description                     | Recommended |
        +===================+===========+========+======================---========+=============+
        | PQ-MLKEM512-SHA3-256-AES128   | TBD1   | ML-KEM-512 + SHA3-256 + AES256  | No          |
        +-------------------------------+--------+---------------------------------+-------------+        
        | PQ-MLKEM768-SHA3-384-AES256   | TBD2   | ML-KEM-768 + SHA3-384 + AES256  | No          |
        +-------------------------------+--------+---------------------------------+-------------+
        | PQ-MLKEM768-SHA3-512-AES256   | TBD3   | ML-KEM-1024 + SHA3-512 + AES256 | No          |
        +-------------------------------+--------+---------------------------------+-------------+   

                                       Table 2

# Security Considerations

PQC KEMs used in the manner described in this document MUST explicitly be designed to be secure in the event that the public key is reused, such as achieving IND-CCA2 security. ML-KEM has such security properties.

# IANA Considerations {#IANA}

## JOSE

The following entries are added to the "JSON Web Signature and Encryption Algorithms" registry:

- Algorithm Name: PQ-Direct
- Algorithm Description: Post Quantum Direct Key Agreement.
- Algorithm Usage Location(s): "alg"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): [[TBD: This RFC]]
- Algorithm Analysis Documents(s): TODO

- Algorithm Name: PQ-MLKEM512-SHA3-256-AES128
- Algorithm Description: Cipher suite for PQ-KEM that uses ML-KEM-512 PQ-KEM, the SHA3-256 KDF and the AES-128-GCM AEAD.
- Algorithm Usage Location(s): "alg, enc"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): [[TBD: This RFC]]
- Algorithm Analysis Documents(s): TODO

- Algorithm Name: PQ-MLKEM768-SHA3-384-AES256
- Algorithm Description: Cipher suite for PQ-KEM that uses ML-KEM-768 PQ-KEM, the SHA3-384 KDF and the AES-256-GCM AEAD.
- Algorithm Usage Location(s): "alg, enc"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): [[TBD: This RFC]]
- Algorithm Analysis Documents(s): TODO

- Algorithm Name: PQ-MLKEM1024-SHA3-512-AES256
- Algorithm Description: Cipher suite for PQ-KEM that uses ML-KEM-1024 PQ-KEM, the SHA3-512 KDF and the AES-256-GCM AEAD.
- Algorithm Usage Location(s): "alg, enc"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): [[TBD: This RFC]]
- Algorithm Analysis Documents(s): TODO

## COSE

The following has to be added to the "COSE Algorithms" registry:

- Name: PQ-MLKEM512-SHA3-256-AES128
- Value: TBD1
- Description: Cipher suite for PQ-KEM that uses ML-KEM-512 PQ-KEM, the SHA3-256 KDF and the AES-128-GCM AEAD.
- Reference: This document (TBD)
- Recommended: No

- Name: PQ-MLKEM768-SHA3-384-AES256
- Value: TBD2
- Description: Cipher suite for PQ-KEM that uses ML-KEM-768 PQ-KEM, the SHA3-384 KDF and the AES-256-GCM AEAD.
- Reference: This document (TBD)
- Recommended: No

- Name: PQ-MLKEM1024-SHA3-512-AES256
- Value: TBD3
- Description: Cipher suite for PQ-KEM that uses ML-KEM-1024 PQ-KEM, the SHA3-512 KDF and the AES-256-GCM AEAD.
- Reference: This document (TBD)
- Recommended: No

# Acknowledgments
{: numbered="false"}

TODO.