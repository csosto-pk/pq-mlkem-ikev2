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
title: "Post-quantum Hybrid Key Exchange with ML-KEM in the Internet Key Exchange Protocol Version 2 (IKEv2)"
abbrev: "ML-KEM IKEv2"
submissionType: IETF
area: Security
workgroup: IPSECME
kw: Internet-Draft
date: {DATE}
keyword:
 - PQ key exchange in IKEv2
 - ML-KEM in IKEv2
 - Hybrid Key exchange in IKEv2

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
  FIPS203: DOI.10.6028/NIST.FIPS.203
 
informative:
  FIPS203-ipd:
    title: "Module-Lattice-based Key-Encapsulation Mechanism Standard"
    author:
      org: National Institute of Standards and Technology (NIST)
    date: 2023-08-24
    seriesinfo:
      NIST: Federal Information Processing Standards
    format:
      PDF: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.ipd.pdf

  NIST-PQ:
    title: "Post-Quantum Cryptography"
    author:
      org: National Institute of Standards and Technology (NIST)
    date: 2016-2024
    seriesinfo:
      https://csrc.nist.gov/projects/post-quantum-cryptography

--- abstract

NIST recently standardized ML-KEM, a new key encapsulation mechanism, which can be used for quantum-resistant key establishment. This draft specifies how to use ML-KEM as an additionall key exchange mechanism in IKEv2 along with traditional (elliptic curve) Diffie-Hellman. This hybrid approach allows for negotiating IKE and Child SA keys which are safe against cryptanalytically-relevant quantum computers and theoretical weaknesses in new algorithm which have not been thoroughly analyzed.


--- middle

# Introduction

A Cryptographically Relevant Quantum Computer (CRQC), if it became a reality, could threaten public key encryption algorithms used today for key exhange. Someone storing encrypted communications which use (Elliptic Curve) Diffie-Hellman ((EC)DH) to negotiate keys could decrypt these communications in the future after a CRQC was available. This could include Internet Key Exchange Protocol Version 2 (IKEv2)/IPsec tunnels which negotiate IKE and Child SA keys by using classical public key algorithms in their IKE_SA_INIT messages. 

To address this concern, {{?RFC8784}} introduced Post-quantum Preshared Keys as a temporary option for stirring a pre-shared key of adequate entropy in the derived Child SA encryption keys in order to provide quantum-resistance. Since then, {{!RFC9242}} defined how to do additional large message exchanges by using a new IKE_INTERMEDIATE message. As post-quantum keys are usualy larger than common network Maximum Transport Units (MTU), IKE_INTERMEDIATE messages can be fragmented which could allow for the peers to do post-quantum key exchanges without IP fragmentation. {{!RFC9370}} defined how to do up to seven additional key exchanges by using IKE_INTERMEDIATE messages and derive new SKEYSEED and KEYMAT key materials. This allows for new post-quantum key exchanges to be used in the derived IKE and Child SA keys and provide quantum resistance. 

NIST has been working on a public project {{NIST-PQ}} for standardizing quantum-safe algorithms which include key enscapsulation and signatures. At the end of Round 3, they picked Kyber as the first Key Encapsulation Mechanism (KEM) for standardization {{?I-D.draft-cfrg-schwabe-kyber-03}}. Kyber was then standardized as Module-Lattice-based Key-Encapsulation Mechanism (ML-KEM) in {{FIPS203-ipd}}. ML-KEM was standardized in 2024 {{FIPS203}}. [ EDNOTE: Reference normatively the ratified version {{?I-D.draft-cfrg-schwabe-kyber-03}} if it is ever ratified. Otherwise keep a normative reference of {{FIPS203}}. And remove the reference to {{FIPS203-ipd}}. ]

This document describes how ML-KEM can be used as the quantum-safe KEM in IKEv2 by using one additional IKE_INTERMEDIATE key exchange negotiation after the classical (EC)DH exchange in IKE_SA_INIT. The result is a new Child SA key or an IKE or Child SA rekey with keying material which is safe against a CRQC. This specification is a profile of {{!RFC9370}} and registers new algorithm identifiers for ML-KEM key exchanges in IKEv2. 

## KEMs

In the context of the NIST Post-Quantum Cryptography Standardization Project {{NIST-PQ}}, key exchange algorithms are formulated as KEMs, which consist of three steps:

- 'KeyGen() -> (pk, sk)': A probabilistic key generation algorithm, which generates a public key 'pk' and a secret key 'sk'.
- 'Encaps(pk) -> (ct, ss)': A probabilistic encapsulation algorithm, which takes as input a public key 'pk' and outputs a ciphertext 'ct' and shared secret 'ss'.
- 'Decaps(sk, ct) -> ss': A decapsulation algorithm, which takes as input a secret key 'sk' and ciphertext 'ct' and outputs a shared secret 'ss', or in some cases a distinguished error value.

The main security property for KEM standardized by NIST is indistinguishability under adaptive chosen ciphertext attack (IND-CCA2), which means that shared secret values should be indistinguishable from random strings even given the ability to have arbitrary ciphertexts decapsulated. IND-CCA2 corresponds to security against an active attacker, and the public key / secret key pair can be treated as a long-term key or reused.  A weaker security notion is indistinguishability under chosen plaintext attack (IND-CPA), which means that the shared secret values should be indistinguishable from random strings given a copy of the public key. IND-CPA roughly corresponds to security against a passive attacker, and sometimes corresponds to one-time key exchange.

## ML-KEM

ML-KEM is a recently standardized lattice-based key encapsulation mechanism {{FIPS203}}. [ EDNOTE: Reference normatively the ratified version {{?I-D.draft-cfrg-schwabe-kyber-03}} if it is ever ratified. Otherwise keep a normative reference of {{FIPS203}}. ]

ML-KEM is using Module Learning with Errors as it underlying primitive which is a structure lattices variant that offer good performance and relatively small and balanced key and ciphertext sizes. ML-KEM was standardized with three variants, ML-KEM-512, ML-KEM-768, and ML-KEM-1024. These were mapped by NIST to the three security levels defined in the NIST PQC Project, Level 1, 3, and 5. These levels correspond to the hardness of solving breaking AES-128, AES-192 and AES-256 respectively. 

This specification introduces ML-KEM-768 and ML-KEM-1024 to IKEv2 key exchanges as conservative security level parameters which will not have material impact on IKEv2/IPsec tunnels which usually stay up for long periods of time. Since the ML-KEM-768 and ML-KEM-1024 public key and ciphertext sizes can exceed the typical network MTU, these key exchanges will usually require two or three network packets from both the initiator and the responder.  

## Conventions and Definitions

{::boilerplate bcp14-tagged}


# ML-KEM in IKEv2 

## ML-KEM in IKE_INTERMEDIATE messages

ML-KEM key exchanges can be negotiated in IKE_INTERMEDIATE messages. As per {{!RFC9370}}, in its initial proposal in Security Associations the initiator includes a Key Exchange (KE) ((EC)DH usually) and Additional Key Exchange 1 (ADDKE1) Transform Type 4 with ID of 35 or 36 for ML-KEM-768 or ML-KEM-1024 respectively. The responder returns a proposal which includes a KE Transform Type 4 ID if it wants to negotiate the (EC)DH one from the initiator and an ADDKE1 with the ML-KEM parameter it choses.

The IKE_SA_INIT and IKE_INTERMEDIATE messages between the initiator and responder, as per {{!RFC9370}}, will be:

~~~
Initiator                                Responder
-------------------------------------  ------------------------------
HDR(IKE_SA_INIT), SAi1, KEi, Ni,
[N(INTERMEDIATE_EXCHANGE_SUPPORTED)]-->
                                   <-- HDR(IKE_SA_INIT), SAr1, KEr,
                                                     Nr, [CERTREQ],
                                   [N(INTERMEDIATE_EXCHANGE_SUPPORTED)

HDR(IKE_INTERMEDIATE), SK {KEi(1)}  -->
                                   <-- HDR(IKE_INTERMEDIATE),SK{KEr(1)}

                  HDR(IKE_AUTH) ... -->
                                   <--  HDR(IKE_AUTH) ...
~~~

As explained in Section 2.2.2 of {{!RFC9370}}, KEi(0), KEr(0) are regular (EC)DH key exchange messages in the first IKE_SA_INIT exchange. A first set of keying materials, in particular SK_d, SK_a[i/r], and SK_e[i/r], are derived from the IKE_SA_INIT as with every IKEv2 exchange. 

Both peers then perform an IKE_INTERMEDIATE exchange, carrying new Key Exchange payloads, which are protected with the SK_e[i/r] and SK_a[i/r] keys which were derived from the IKE_SA_INIT as per Section 3.3.1 of {{!RFC9242}}. The IKE_INTERMEDIATE messages are authenticated as per 3.3.2 of {{!RFC9242}}.  

KEi(1) carries the ML-KEM public key generated by using an (sk, pk) keypair with KeyGen(). The public key is encoded as raw bytes in little-endian encoding. [ EDNOTE: Confirm this makes sense. ] The responder uses the initiator's public key and encapsulates an ML-KEM shared secret ss to a ciphertext ct by using Encaps(pk). ct is encoded as raw bytes in little-endian encoding in KEr(1) from the responder. [ EDNOTE: Confirm this makes sense. ] Then the initiator decapsulates the ML-KEM shared secret ss from the ciphertext by using it private key sk in Decaps(sk, ct). Both peers have now reached a common ss at the end of this KE(1) key exchange. 

## Key Exchange Payload

[ EDNOTE: From https://www.rfc-editor.org/rfc/rfc8031.html#section-3.1 . Update accordingly ]

The HDR of the IKE_INTERMEDIATE messages carrying the ML-KEM exchange will have a Next Payload value of 34 (Key Exchange), Exchange Type of 43 (IKE_INTERMEDIATE) and Message ID of 1 (first and only additional key exchange). 

The protected with SK_e[i/r] and SK_a[i/r] keys from the IKE_SA_INIT ML-KEM key exchange payload which use the diagram for the Key Exchange payload from Section 3.4 of RFC 7296 is copied below for convenience:
   
~~~
                       1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  | Next Payload  |C|  RESERVED   |         Payload Length        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |   Key Exchange Method Num    |           RESERVED            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                                                               |
  ~                       Key Exchange Data                       ~
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
~~~
  
- Payload Length - For  ML-KEM-768, the public key is XYZ octets, so the Payload Length field will be XYZ.  For  ML-KEM-1024, the public key is XYZ octets, so the Payload Length field will be XYZ.
- The Key Exchange Method is 35 for ML-KEM-768 or 36 for ML-KEM-1024. 
- The Key Exchange Data is the 32 or 56 octets of ciphertext ct as described in Section 6 of [RFC7748].

IKE_INTERMEDIATE fragmented as per {{?RFC7383}} carrying ML-KEM in rfc9370 of how to combine it. 

ML-KEM-768 and ML-KEM-1024 Key Exchange Method identifiers SHOULD be used in IKE_INTERMEDIATE exchanges. They SHOULD NOT be used in IKE_SA_INIT exchanges because they could be introducing fragmentation. 

## Key derivation

As per {{!RFC9370}},  first a set of keying materials is derived, in particular SK_d, SK_a[i/r], and SK_e[i/r] are derived in the IKE_SA_INIT. 
First key exchange IKE_SA_INIT SK_d(0) derived from IKE_SA_INIT 

Both peers then perform an IKE_INTERMEDIATE exchange, carrying new Key Exchange payload, which is protected with SK_e[i/r] and SK_a[i/r] keys from the IKE_SA_INIT. 

Section 2.2.2 {{!RFC9370}}

~~~
SKEYSEED(1) = prf(SK_d(0), SK(1) | Ni | Nr)
~~~

The other new keying materials, SK_d, SK_ai, SK_ar, SK_ei, SK_er, SK_pi, and SK_pr, are generated from the SKEYSEED(1) as follows: 

~~~
{SK_d(1) | SK_ai(1) | SK_ar(1) | SK_ei(1) | SK_er(1) | SK_pi(1) |
       SK_pr(1)} = prf+ (SKEYSEED(1), Ni | Nr | SPIi | SPIr)
~~~

SK_d(1) is used as SK_d in generating the Child SA keying material.

Afterwards both peers continue to the IKE_AUTH exchange phase. use these updated key values in the next exchange IKE_AUTH

As per {{!RFC9242}}, both peers compute IntAuth_i1 and IntAuth_r1 using the SK_pi(1) and SK_pr(1) keys, respectively. These values are required in the IKE_AUTH phase of the exchange. IntAuth_i1 and IntAuth_r1 are used to compute IntAuth, which, in turn, is used to calculate InitiatorSignedOctets and ResponderSignedOctets blobs (see Section 3.3.2 of {{!RFC9242}}).

The CREATE_CHILD_SA exchange is used in IKEv2 for the purposes of creating additional Child SAs, rekeying these Child SAs, and rekeying IKE SA itself as per 2.2.4 {{!RFC9370}}. 

When creating or rekeying Child SAs, the peers may optionally perform a key exchange to add a fresh entropy into the session keys. These situations will negotiate hybrid with ECDH and ML-KEM. For Child SA creation or rekey CREATE_CHILD_SA exchange is negotiated in a similar fashion to the initial IKE exchange. And then a ML-KEM is performed using IKE_FOLLOWUP_KE as per 2.2.4 {{!RFC9370}}. 
the KEYMAT combine SK_d and stir all the negotiated keys as per per 2.2.4 {{!RFC9370}}

For IKE SA rekey, peers will negotiate a new key and renegotiate with ECDH with ML-KEM as per 2.2.4 {{!RFC9370}}. 
The SKEYSEED combine SK_d and stir all the negotiated keys as per 2.2.4 {{!RFC9370}}. 

## Recipient Tests {#receipent-tests}

[EDNOTE: Update here about implicit rejection of the public key at the responder or the ciphertext at the initiator. ] 

[ EDNOTE: From This is from https://www.rfc-editor.org/rfc/rfc8031.html#section-5 about curve25519 ] 
Receiving and handling of incompatible ML-KEM public key or ciphertext formats MUST follow the considerations described in Section 5 of [RFC7748]. In particular, receiving entities MUST mask the most-significant bit in the final byte for X25519 (but not X448), and implementations MUST accept non-canonical values.

[ EDNOTE: From https://www.rfc-editor.org/rfc/rfc6989.html#section-2.3 about P256 ]
IKEv2 can be used with elliptic curve groups defined over a field GF(p) [RFC5903] [RFC5114].  According to [Menezes], Section 2.3, there is some informational leakage possible.  A receiving peer MUST check that its peer's public value is valid; that is, the x and y parameters from the peer's public value satisfy the curve equation, y^2 = x^3 + ax + b mod p (where for groups 19, 20, and 21, a=-3 (mod p), and all other values of a, b, and p for the group are listed in the RFC defining the group). We note that an additional check to ensure that the public value is not the point at infinity is not needed, because IKE (see Section 7 of [RFC5903]) does not allow for encoding this value. 


# Security Considerations

All security considerations from {{!RFC9242}} and {{!RFC9370}} apply to the ML-KEM exchanges decribed in this specification. 


# IANA Considerations

IANA is requested to assign two values for the names "mlkem-768" and "mlkem-1024" in the IKEv2 "Transform Type 4 - Key Exchange Method Transform IDs" and has listed this document as the reference.  The Recipient Tests field should also point to this document:

|--------|------------|--------|-----------------|-----------|
| Number |    Name    | Status | Recipient Tests | Reference |
|--------|------------|--------|-----------------|-----------|
|   35   | mlkem-768  |        | [TBD, this draft, {{receipent-tests}}],  | [TBD, this draft]  |
|   36   | mlkem-1024 |        | [TBD, this draft, {{receipent-tests}}],  |  [TBD, this draft]  |
| 37-1023 |	Unassigned|        |                 |           |       
|--------|------------|--------|-----------------|-----------|
{: #tab1 title="Updates to the IANA \"Transform Type 4 - Key Exchange Method Transform IDs\" table" }

[ EDNOTE: Remove link 
https://www.iana.org/assignments/ikev2-parameters/ikev2-parameters.xhtml#ikev2-parameters-8 . ]


--- back

# Acknowledgments
{:numbered="false"}

[EDNOTE: Update here. ]
The authors would like to thank Valery Smyslov for his valuable contributions to the document. 
