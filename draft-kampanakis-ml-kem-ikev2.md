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

informative:


--- abstract

NIST recently standardized ML-KEM, a new key encapsulation mechanism, which can be used for quantum-resistant key establishment. This draft specifies how to use ML-KEM as an additionall key exchange mechanism in IKEv2 along with traditional (elliptic curve) Diffie-Hellman. This hybrid approach allows for negotiating IKE and Child SA keys which are safe against cryptanalytically-relevant quantum computers and theoretical weaknesses in new algorithm which have not been thoroughly analyzed.


--- middle

# Introduction

Quantum computing threat. Someone storing data today. Link to PPK draft as a temporary option.

{{!RFC9242}} defines how to do additional key exchanges for IKE SA rekey or Child SA establishment usine a new message INTERMEDIATE. This message can be fragmented. 

Key Encapsulation KEM and NIST.

{{!RFC9370}} defines how to do up to 7 additional key exchange and derive new SKEYSEED and KEYMAT in order to rekey an SA orcrete or rekey a Child SA. That way someone could do an additional post-quantum key exchange on top of the classical key exchange in the 
The derived keys for IPsec are quantum-sagfe a CRQC cannot decrypt. 

This draft describes how ML-KEM can be used as the quantum-safe key. ML-KEM is the NIST standardized KEM by NIST. [EDNOTE: Add reference to IETF Kyber fraft or NIST FIPS 203 spec.] 

This draft profiles  {{!RFC9370}} and specifies has ML-KEM algorithms can be used as the one additional INTERMEDIATE key exchange. This key exchange can be used to rekey IKE SAs or Child SAs or create a new Child SA. 

## KEMs

Explain how KEMs work. 

In the context of the NIST Post-Quantum Cryptography Standardization Project [NIST_PQ], key exchange algorithms are formulated as key encapsulation mechanisms (KEMs), which consist of three algorithms:

- 'KeyGen() -> (pk, sk)': A probabilistic key generation algorithm, which generates a public key 'pk' and a secret key 'sk'.
- 'Encaps(pk) -> (ct, ss)': A probabilistic encapsulation algorithm, which takes as input a public key 'pk' and outputs a ciphertext 'ct' and shared secret 'ss'.
- 'Decaps(sk, ct) -> ss': A decapsulation algorithm, which takes as input a secret key 'sk' and ciphertext 'ct' and outputs a shared secret 'ss', or in some cases a distinguished error value.

The main security property for KEMs is indistinguishability under adaptive chosen ciphertext attack (IND-CCA2), which means that shared secret values should be indistinguishable from random strings even given the ability to have arbitrary ciphertexts decapsulated. IND-CCA2 corresponds to security against an active attacker, and the public key / secret key pair can be treated as a long-term key or reused.  A weaker security notion is indistinguishability under chosen plaintext attack (IND-CPA), which means that the shared secret values should be indistinguishable from random strings given a copy of the public key.  IND-CPA roughly corresponds to security against a passive attacker, and sometimes corresponds to one-time key exchange.

## ML-KEM

Explain ML-KEM. 
Explain parameters. ML-KEM-512, 768, 1024.
Add reference to IETF Kyber fraft or NIST FIPS 203 spec.  

Security levels for NIST. 

ML-KEM-512 out of scope long confidentiality and studies for parformance and VPN tunnels stay up. Very efficient. Mo need for 512 for IKEv2 and thus not specified here. 

## Conventions and Definitions

{::boilerplate bcp14-tagged}



# ML-KEM in IKE_INTERMEDIATE 

As per {{!RFC9370}}, in its initial proposal in Security Associations the initiator includes an KE (ECDH usually) and ADDKE Transform Type with Transform ID of 35 or 36 for ML-KEM-768 or ML-KEM-1024. The responder will return a proposal which includes a KE Transform Type if it wants to negotiate the one from the initiator and an ADDKE1 with the ML-KEM it choses.

The IKE_SA_INIT

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

Section 2.2.2 {{!RFC9370}}
KEi(0), KEr(0) is regular ECDH key exchange in the first IKE_SA_INIT messages
As per {{!RFC9370}},  first a set of keying materials is derived, in particular SK_d, SK_a[i/r], and SK_e[i/r] are derived in the IKE_SA_INIT. 

Both peers then perform an IKE_INTERMEDIATE exchange, carrying new Key Exchange payload, which is protected with SK_e[i/r] and SK_a[i/r] keys from the IKE_SA_INIT  as per {{!RFC9242}} 3.3.1. Authenticated as per 3.3.2.  

Explain how the initiator message will include ML-KEM public key is encoded as raw bytes on-the-wire format (https://www.rfc-editor.org/rfc/rfc8031.html uses the X25519 encoding (search for encoding")). 
Section 2.2.2 {{!RFC9370}}
n = 1 one Intermediate Key Exchange
KEi(1), KEr(1) is ML-KEM.  
KEi(1) is the Keygen and public key encoded as 
KEi(1), KEr(1) takes place in an intermediate key exchange {{!RFC9242}} 
KEr(1) is the encaps with the ciphertext encoded as. Then initiator decapsulates 
Both sides reach SK(1) = ss. 

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
  |   Diffie-Hellman Group Num    |           RESERVED            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                                                               |
  ~                       Key Exchange Data                       ~
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
~~~
  
- Payload Length - For  ML-KEM-768, the public key is XYZ octets, so the Payload Length field will be XYZ.  For  ML-KEM-1024, the public key is XYZ octets, so the Payload Length field will be XYZ.
- The Key Exchange Method is 35 for ML-KEM-768 or 36 for ML-KEM-1024. 
- The Key Exchange Data is the 32 or 56 octets as described in Section 6 of [RFC7748].

IKE_INTERMEDIATE fragmented as per {{?RFC7383}} carrying ML-KEM in rfc9370 of how to combine it. 

## Key derivation

As per {{!RFC9370}},  first a set of keying materials is derived, in particular SK_d, SK_a[i/r], and SK_e[i/r] are derived in the IKE_SA_INIT. 
First key exchange IKE_SA_INIT SK_d(0) derived from IKE_SA_INIT 

Both peers then perform an IKE_INTERMEDIATE exchange, carrying new Key Exchange payload, which is protected with SK_e[i/r] and SK_a[i/r] keys from the IKE_SA_INIT. 

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

All security considerations of {{!RFC9242}}, {{!RFC9370}} apply. 


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

https://www.iana.org/assignments/ikev2-parameters/ikev2-parameters.xhtml#ikev2-parameters-8 


--- back

# Acknowledgments
{:numbered="false"}

[EDNOTE: Update here. ]
The authors would like to thank Valery Smyslov for his valuable contributions to the document. 
