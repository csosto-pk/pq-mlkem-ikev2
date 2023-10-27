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

informative:


--- abstract

A CRQC could threated the encrypted data run over IKEv2/IPsec tunnels. Some could store data and decrypt. New standardized KEMs are coming. This draft specifies how to use ML-KEM on top of classical key exchange in IKEv2. This will allows to negotiate quantum safe keys in IKE SA and Child SAs to address the concern.


--- middle

# Introduction

Quantum computing threat. Someone storing data today. Link to PPK draft as a temporary option.

{{!RFC9242}} defines how to do additional key exchanges for IKE SA rekey or Child SA establishment usine a new message INTERMEDIATE. This message can be fragmented. 

{{!RFC9370}} defines how to do up to 7 additional key exchange and derive new SKEYSEED and KEYMAT in order to rekey an SA orcrete or rekey a Child SA. That way someone could do an additional post-quantum key exchange on top of the classical key exchange in the 
The derived keys for IPsec are quantum-sagfe a CRQC cannot decrypt. 

This draft describes how ML-KEM can be used as the quantum-safe key. ML-KEM is the NIST standardized KEM by NIST. [EDNOTE: Add reference to IETF Kyber fraft or NIST FIPS 203 spec.] 

This draft profiles  {{!RFC9370}} and specifies has ML-KEM algorithms can be used as the one additional INTERMEDIATE key exchange. This key exchange can be used to rekey IKE SAs or Child SAs or create a new Child SA. 

## KEMs

Explain how KEMs work and explain how the responder response includes the encapsulated ciphertext.

This document models key agreement as key encapsulation mechanisms (KEMs), which consist of three algorithms:

-  KeyGen() -> (pk, sk): A probabilistic key generation algorithm, which generates a public key pk and a secret key sk.
-  Encaps(pk) -> (ct, ss): A probabilistic encapsulation algorithm, which takes as input a public key pk and outputs a ciphertext ct and shared secret ss.
-  Decaps(sk, ct) -> ss: A decapsulation algorithm, which takes as input a secret key sk and ciphertext ct and outputs a shared secret ss, or in some cases a distinguished error value.

The main security property for KEMs is indistinguishability under adaptive chosen ciphertext attack (IND-CCA2), which means that shared secret values should be indistinguishable from random strings even given the ability to have other arbitrary ciphertexts decapsulated. IND-CCA2 corresponds to security against an active attacker, and the public key / secret key pair can be treated as a long-term key or  reused. A common design pattern for obtaining security under key reuse is to apply the Fujisaki-Okamoto (FO) transform [FO] or a variant thereof [HHK].

## ML-KEM

Explain parameters 

[EDNOTE: Add reference to IETF Kyber fraft or NIST FIPS 203 spec.] 
ML-KEM-512, 768, 1024. 

Security levels for NIST. 
512 out of scope long confidentiality and studies for parformance and VPN tunnels stay up. Very efficient. Mo need for 512 for IKEv2 and thus not specified here. 


## Conventions and Definitions

{::boilerplate bcp14-tagged}

# ML-KEM in IKE_INTERMEDIATE 

Section 2.2.2 {{!RFC9370}}
n = 1 one Intermediate Key Exchange
KEi(0), KEr(0) is regular ECDH key exchange in the first IKE_SA_INIT messages

First key exchange SK_d(0) derived from IKE_SA_INIT 

KEi(1), KEr(1) is ML-KEM. 
KEi(1), KEr(1) takes place in an intermediate key exchange {{!RFC9242}} 

Section 2.2.2 {{!RFC9370}}
n = 1 one Intermediate Key Exchange
KEi(0), KEr(0) is regular ECDH key exchange in the first IKE_SA_INIT messages

First key exchange SK_d(0) derived from IKE_SA_INIT 

KEi(1), KEr(1) is ML-KEM. 
KEi(1), KEr(1) takes place in an intermediate key exchange {{!RFC9242}} 

## Key Exchange Payload

[ EDNOTE: From https://www.rfc-editor.org/rfc/rfc8031.html#section-3.1 . Update accordingly

   The diagram for the Key Exchange payload from Section 3.4 of RFC 7296
   is copied below for convenience:
   
~~~
                       1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  | Next Payload  |C|  RESERVED   |         Payload Length        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |   Diffie-Hellman Group Num    |           RESERVED            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                                                               |
  ~                       Key Exchange Data                       ~
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
~~~
  
- Payload Length - For Curve25519, the public key is 32 octets, so
  the Payload Length field will be 40.  For Curve448, the public key
  is 56 octets, so the Payload Length field will be 64.
  
- The Diffie-Hellman Group Num is 31 for Curve25519 or 32 for
  Curve448.
  
- The Key Exchange Data is the 32 or 56 octets as described in
      Section 6 of [RFC7748].

Explain how the initiator message will include ML-KEM public key is encoded as raw bytes on-the-wire format (https://www.rfc-editor.org/rfc/rfc8031.html uses the X25519 encoding (search for encoding")). 
KEi(1) is the Keygen and public key encoded as 
KEr(1) is the encaps with the ciphertext encoded as. Then initiator decapsulates 
Both sides reach SK(1) = ss. 


## Key derivation

As per {{!RFC9370}} combines 

~~~
SKEYSEED(1) = prf(SK_d(0), SK(1) | Ni | Nr)
~~~

The other keying materials, SK_d, SK_ai, SK_ar, SK_ei, SK_er, SK_pi, and SK_pr, are generated from the SKEYSEED(n) as follows: 

~~~
{SK_d(1) | SK_ai(1) | SK_ar(1) | SK_ei(1) | SK_er(1) | SK_pi(1) |
       SK_pr(1)} = prf+ (SKEYSEED(1), Ni | Nr | SPIi | SPIr)
~~~

 SK_d(1) is used as SK_d in generating the Child SA keying material.

Afterwards both peers continue to the IKE_AUTH exchange phase. use these updated key values in the next exchange IKE_AUTH

As per {{!RFC9242}}, both peers compute IntAuth_i1 and IntAuth_r1 using the SK_pi(1) and SK_pr(1) keys, respectively. These values are required in the IKE_AUTH phase of the exchange. IntAuth_i1 and IntAuth_r1 are used to compute IntAuth, which, in turn, is used to calculate InitiatorSignedOctets and ResponderSignedOctets blobs (see Section 3.3.2 of {{!RFC9242}}).

IKE_INTERMEDIATE fragmented as per {{?RFC7383}} carrying ML-KEM in rfc9370 of how to combine it. Protection of additional key exchange as per {{!RFC9242}}  3.3.1. Authenticated as per 3.3.2. 

The CREATE_CHILD_SA exchange is used in IKEv2 for the purposes of creating additional Child SAs, rekeying these Child SAs, and rekeying IKE SA itself as per 2.2.4 {{!RFC9370}}. 

When creating or rekeying Child SAs, the peers may optionally perform a key exchange to add a fresh entropy into the session keys. These situations will negotiate hybrid with ECDH and ML-KEM. For Child SA creation or rekey CREATE_CHILD_SA exchange is negotiated in a similar fashion to the initial IKE exchange. And then a ML-KEM is performed using IKE_FOLLOWUP_KE as per 2.2.4 {{!RFC9370}}. 
the KEYMAT combine SK_d and stir all the negotiated keys as per per 2.2.4 {{!RFC9370}}

For IKE SA rekey, peers will negotiate a new key and renegotiate with ECDH with ML-KEM as per 2.2.4 {{!RFC9370}}. 
The SKEYSEED combine SK_d and stir all the negotiated keys as per 2.2.4 {{!RFC9370}}. 

## Recipient Tests {#receipent-tests}

[EDNOTE: This is from https://www.rfc-editor.org/rfc/rfc8031.html#section-5 . Update here] 

Receiving and handling of incompatible point formats MUST follow the considerations described in Section 5 of [RFC7748]. In particular, receiving entities MUST mask the most-significant bit in the final byte for X25519 (but not X448), and implementations MUST accept non-canonical values.

# Security Considerations

All security considerations of {{!RFC9242}}, {{!RFC9370}} apply. 


# IANA Considerations

IANA Key Exchange identifier 

New Transform Type 4 - Key Exchange Method Transform ID 35 ML-KEM-768 and 36 ML-KEM-1024 https://www.iana.org/assignments/ikev2-parameters/ikev2-parameters.xhtml#ikev2-parameters-8 

|--------|------------|--------|-----------------|-----------|
| Number |    Name    | Status | Recipient Tests | Reference |
|--------|------------|--------|-----------------|-----------|
|   35   | mlkem-768  |        | [TBD, this draft, {{receipent-tests}}],  | [TBD, this draft]  |
|   36   | mlkem-1024 |        | [TBD, this draft, {{receipent-tests}}],  |  [TBD, this draft]  |
|--------|------------|--------|--------------|-----------|
        
--- back

# Acknowledgments
{:numbered="false"}

[EDNOTE: Update here. ]
