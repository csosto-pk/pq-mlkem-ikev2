---
stand_alone: true
ipr: trust200902
docname: draft-kampanakis-ml-kem-ikev2
cat: std
consensus: 'true'
pi:
  toc: 'yes'
  sortrefs: 'yes'
  symrefs: 'yes'
title: "Post-quantum Key Exchange with ML-KEM in the Internet Key Exchange Protocol Version 2 (IKEv2)"
abbrev: "ML-KEM IKEv2"
submissionType: IETF
area: Security
workgroup: IPSECME
kw: Internet-Draft
date: {DATE}
keyword:
 - PQ IKEv2

author:
 -
    fullname: Panos Kampanakis
    organization: Amazon Web Services 
    email: kpanos@amazon.com

 -
    fullname: Gerardo Ravago
    organization: Amazon Web Services 
    email: gcr@amazon.com

normative:
    RFC9370: 

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# ML-KEM 

Explain how KEMs work and explain how the responder response includes the encapsulated ciphertext.

This document models key agreement as key encapsulation mechanisms (KEMs), which consist of three algorithms:

-  KeyGen() -> (pk, sk): A probabilistic key generation algorithm, which generates a public key pk and a secret key sk.
-  Encaps(pk) -> (ct, ss): A probabilistic encapsulation algorithm, which takes as input a public key pk and outputs a ciphertext ct and shared secret ss.
-  Decaps(sk, ct) -> ss: A decapsulation algorithm, which takes as input a secret key sk and ciphertext ct and outputs a shared secret ss, or in some cases a distinguished error value.

The main security property for KEMs is indistinguishability under
   adaptive chosen ciphertext attack (IND-CCA2), which means that shared
   secret values should be indistinguishable from random strings even
   given the ability to have other arbitrary ciphertexts decapsulated.
   IND-CCA2 corresponds to security against an active attacker, and the
   public key / secret key pair can be treated as a long-term key or
   reused.  A common design pattern for obtaining security under key
   reuse is to apply the Fujisaki-Okamoto (FO) transform [FO] or a
   variant thereof [HHK].

# ML-KEM in IKE_INTERMEDIATE 

{{!RFC9370}}

How key SKEYSEED and KEYMAT is generated. 

IKE_INTERMEDIATE fragmented carrying ML-KEM in rfc9370 of how to combine it. Protection of additional key exchange as per rfc9242 3.3.1. Authenticated as per 3.3.2. 

Explain how the initiator message will include ML-KEM public key is encoded as raw bytes on-the-wire format (https://www.rfc-editor.org/rfc/rfc8031.html uses the X25519 encoding (search for encoding")). 

IANA Key Exchange identifier 

# Security Considerations

TODO Security


# IANA Considerations

New Transform Type 4 - Key Exchange Method Transform ID 35 ML-KEM-768 and 36 ML-KEM-1024 https://www.iana.org/assignments/ikev2-parameters/ikev2-parameters.xhtml#ikev2-parameters-8 


--- back

# Acknowledgments
{:numbered="false"}

[EDNOTE: Update here. ]
