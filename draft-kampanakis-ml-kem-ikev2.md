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

  NIST-PQ:
    title: "Post-Quantum Cryptography"
    author:
      org: National Institute of Standards and Technology (NIST)
    date: 2016-2024
    seriesinfo:
      https://csrc.nist.gov/projects/post-quantum-cryptography

--- abstract

[EDNOTE: The intention of this draft is to get IANA KE codepoints for ML-KEM. It could be a standards track draft given that ML-KEM will see a lot of adoption, an AD sponsored draft, or even a individual stable draft which gets codepoints from Expert Review. The approach is to be decided by the IPSECME WG. ] 

NIST recently standardized ML-KEM, a new key encapsulation mechanism, which can be used for quantum-resistant key establishment. This draft specifies how to use ML-KEM as an additionall key exchange mechanism in IKEv2 along with traditional (Elliptic Curve) Diffie-Hellman. This hybrid approach allows for negotiating IKE and Child SA keys which are safe against cryptanalytically-relevant quantum computers and theoretical weaknesses in ML-KEM as it is relatively new.


--- middle

# Introduction

A Cryptanalytically-relevant Quantum Computer (CRQC), if it became a reality, could threaten public key encryption algorithms used today for key exhange. Someone storing encrypted communications which use (Elliptic Curve) Diffie-Hellman ((EC)DH) to negotiate keys could decrypt these communications in the future after a CRQC was available. This could include Internet Key Exchange Protocol Version 2 (IKEv2)/IPsec tunnels which negotiate IKE and Child SA keys by using ECDH key exchange in their IKE_SA_INIT messages. 

To address this concern, the Mixing Preshared Keys in IKEv2 specification {{?RFC8784}} introduced Post-quantum Preshared Keys as a temporary option for stirring a pre-shared key of adequate entropy in the derived Child SA encryption keys in order to provide quantum-resistance. This specification can be used in conjunction with PPK as defined {{?RFC8784}}. Since then, the Intermediate Exchange in IKEv2 document {{!RFC9242}} defined how to do additional large message exchanges by using a new IKE_INTERMEDIATE message. As post-quantum keys are usualy larger than common network Maximum Transport Units (MTU), IKE_INTERMEDIATE messages can be fragmented which could allow for the peers to do post-quantum key exchanges without IP fragmentation. The Multiple Key Exchanges in IKEv2 specification {{!RFC9370}} defined how to do up to seven additional key exchanges by using IKE_INTERMEDIATE messages and derive new SKEYSEED and KEYMAT key materials. This allows for new post-quantum key exchanges to be used in the derived IKE and Child SA keys and provide quantum resistance. 

NIST has been working on a public project {{NIST-PQ}} for standardizing quantum-safe algorithms which include key ensapsulation and signatures. At the end of Round 3, they picked Kyber as the first Key Encapsulation Mechanism (KEM) which was  standardized as ML-KEM {{FIPS203}} in 2024. 

This document describes how ML-KEM can be used as the quantum-safe KEM in IKEv2 by using one additional IKE_INTERMEDIATE key exchange after the classical (EC)DH exchange in IKE_SA_INIT. The result is a new Child SA key or an IKE or Child SA rekey with keying material which is safe against a CRQC. This specification is a profile of the Multiple Key Exchanges in IKEv2 specification {{!RFC9370}} and registers new algorithm identifiers for ML-KEM key exchanges in IKEv2. 

## KEMs

In the context of the NIST Post-Quantum Cryptography Standardization Project {{NIST-PQ}}, key exchange algorithms are formulated as KEMs, which consist of three steps:

- 'KeyGen() -> (pk, sk)': A probabilistic key generation algorithm, which generates a public key 'pk' and a secret key 'sk'.
- 'Encaps(pk) -> (ct, ss)': A probabilistic encapsulation algorithm, which takes as input a public key 'pk' and outputs a ciphertext 'ct' and shared secret 'ss'.
- 'Decaps(sk, ct) -> ss': A decapsulation algorithm, which takes as input a secret key 'sk' and ciphertext 'ct' and outputs a shared secret 'ss', or in some cases a distinguished error value.

The main security property for KEMs standardized by NIST is indistinguishability under adaptive chosen ciphertext attacks (IND-CCA2), which means that shared secret values should be indistinguishable from random strings even given the ability to have arbitrary ciphertexts decapsulated. IND-CCA2 corresponds to security against an active attacker, and the public key / secret key pair can be treated as a long-term key or reused.  A weaker security notion is indistinguishability under chosen plaintext attacks (IND-CPA), which means that the shared secret values should be indistinguishable from random strings given a copy of the public key. IND-CPA roughly corresponds to security against a passive attacker, and sometimes corresponds to one-time key exchange.

## ML-KEM

ML-KEM is a recently standardized lattice-based key encapsulation mechanism {{FIPS203}}. It use using Module Learning with Errors as its underlying primitive which is a structured lattices variant that offers good performance and relatively small and balanced key and ciphertext sizes. ML-KEM was standardized with three parameters, ML-KEM-512, ML-KEM-768, and ML-KEM-1024. These were mapped by NIST to the three security levels defined in the NIST PQC Project, Level 1, 3, and 5. These levels correspond to the hardness of breaking AES-128, AES-192 and AES-256 respectively. 

This specification introduces ML-KEM-768 and ML-KEM-1024 to IKEv2 key exchanges as conservative security level parameters which will not have material performance impact on IKEv2/IPsec tunnels which usually stay up for long periods of time. Since the ML-KEM-768 and ML-KEM-1024 public key and ciphertext sizes can exceed the typical network MTU, these key exchanges will usually require two or three network IP packets from both the initiator and the responder.  

## Conventions and Definitions

{::boilerplate bcp14-tagged}


# ML-KEM in IKEv2 

## ML-KEM in IKE_INTERMEDIATE messages

ML-KEM key exchanges can be negotiated in IKE_INTERMEDIATE messages. As per the Multiple Key Exchanges in IKEv2 specification {{!RFC9370}}, in its initial proposal in Security Associations the initiator includes a Key Exchange (KE) ((EC)DH usually) and Additional Key Exchange 1 (ADDKE1) Transform Type 4 with ID of 35 or 36 for ML-KEM-768 or ML-KEM-1024 respectively. The responder returns a proposal which includes a KE Transform Type 4 ID if it wants to negotiate the (EC)DH one from the initiator and an ADDKE1 with the ML-KEM parameter it choses.

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

As explained in Section 2.2.2 of the Multiple Key Exchanges in IKEv2 doscument {{!RFC9370}}, KEi(0), KEr(0) are regular (EC)DH key exchange messages in the first IKE_SA_INIT exchange. A first set of keying materials, in particular SK_d, SK_a[i/r], and SK_e[i/r], are derived from the IKE_SA_INIT as with every IKEv2 exchange. 

Both peers then perform an IKE_INTERMEDIATE exchange, carrying new Key Exchange payloads, which are protected with the SK_e[i/r] and SK_a[i/r] keys which were derived from the IKE_SA_INIT as per Section 3.3.1 of the Intermediate Exchange in IKEv2 document {{!RFC9242}}. The IKE_INTERMEDIATE messages are authenticated as per 3.3.2 of {{!RFC9242}}.  

KEi(1) carries the ML-KEM public key generated by using an (sk, pk) keypair with KeyGen(). The public key is encoded as raw bytes in little-endian encoding. [ EDNOTE: Confirm this makes sense. ] The responder uses the initiator's public key and encapsulates a 256-bit ML-KEM shared secret ss to a ciphertext ct by using Encaps(pk). ct is encoded as raw bytes in little-endian encoding in KEr(1) from the responder. [ EDNOTE: Confirm this makes sense. ] Then the initiator decapsulates the 256-bit ML-KEM shared secret ss from the ciphertext by using it private key sk in Decaps(sk, ct). Both peers have now reached a common ss at the end of this KE(1) key exchange. 

IKE_INTERMEDIATE messages, KEi(1), KEr(1), can be fragmented as per the IKEv2 Message Fragmentation specification {{?RFC7383}} since the ML-KEM-768 and ML-KEM-1024 public keys and ciphertexts often are close to network MTUs.

Afterwards both peers continue to the IKE_AUTH exchange phase. 

## Key Exchange Payload

HDR, the IKE header, of the IKE_INTERMEDIATE messages carrying the ML-KEM key exchange have a Next Payload value of 34 (Key Exchange), Exchange Type of 43 (IKE_INTERMEDIATE) and Message ID of 1 assuming this is the first additional key exchange (ADDKE1). 

The IKE_INTERMEDIATE payload which is protected with SK_e[i/r] and SK_a[i/r] keys from the IKE_SA_INIT ML-KEM key exchange is shown below as defined in Section 3.4 of the IKEv2 standard {{!RFC7296}},:
   
~~~
                     1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Next Payload  |C|  RESERVED   |         Payload Length        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Key Exchange Method Num    |           RESERVED             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
~                       Key Exchange Data                       ~
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
~~~
  
- Payload Length: The ML-KEM-768 public key is 1184 bytes, so the Payload Length field included in the payload of the IKE_INTERMEDIATE from the initiator is 1192. The ML-KEM-768 ciphertext is 1088 bytes, so the Payload Length of IKE_INTERMEDIATE from the responder is 1096. The ML-KEM-1024 public key is 1568 bytes, so the Payload Length field included in the payload of the IKE_INTERMEDIATE from the initiator is 1576. The ML-KEM-1024 ciphertext is 1568 bytes, so the Payload Length of IKE_INTERMEDIATE from the responder is 1576. 
- The Key Exchange Method Num identifier is 35 for ML-KEM-768 or 36 for ML-KEM-1024. ML-KEM-768 and ML-KEM-1024 Key Exchange Method identifiers SHOULD be used in IKE_INTERMEDIATE exchanges. They SHOULD NOT be used in IKE_SA_INIT because they often could be introducing IP fragmentation which is not possible in IKE_SA_INIT exchanges.
- The Key Exchange Data is the 1184 or 1568 octets of the ML-KEM-768 or ML-KEM-1024 public key respectively for the IKE_INTERMEDIATE message from the initiator. The response from the responder is 1568 octets as the size of the ML-KEM-768 or ML-KEM-1024 ciphertexts respectively. 
 

## Key derivation

As per the Multiple Key Exchanges in IKEv2 document {{!RFC9370}}, the first a set of keying materials, SK_d, SK_a[i/r], is derived using (EC)DH key exchange in the IKE_SA_INIT exchange.  

Then, both peers perform an IKE_INTERMEDIATE exchange, carrying new Key Exchange payloads, which are protected with SK_e[i/r] and SK_a[i/r]. Section 2.2.2 of {{!RFC9370}} specifies that the peers then derive a new SKEYSEED as: 

~~~
SKEYSEED(1) = prf(SK_d, SK(1) | Ni | Nr)
~~~

SK_d is the output of the first IKE_SA_INIT key exchange. SK(1) is the 256-bit shared secret ss established in the ML-KEM key exchange. 

Subsequently, new keying materials are generated from the SKEYSEED(1) as per {{!RFC9370}}: 

~~~
{SK_d(1) | SK_ai(1) | SK_ar(1) | SK_ei(1) | SK_er(1) | SK_pi(1) |
       SK_pr(1)} = prf+ (SKEYSEED(1), Ni | Nr | SPIi | SPIr)
~~~

These keys are used in the rest of the IKEv2 negotiation as typical keys

~~~
{SK_d | SK_ai  | SK_ar | SK_ei | SK_er | SK_pi | SK_pr }
~~~

The new updated key values are used in the subsequent IKE_AUTH as defined in Section 3.3.2 of the Intermediate Exchange in IKEv2 specification {{!RFC9242}} and presented here for convenience: Both peers compute IntAuth_i1 and IntAuth_r1 using the SK_pi(1) and SK_pr(1) keys, respectively. These values are required in the IKE_AUTH phase of the exchange. IntAuth_i1 and IntAuth_r1 are used to compute IntAuth, which, in turn, is used to calculate InitiatorSignedOctets and ResponderSignedOctets blobs.

Creating additional Child SAs, rekeying these Child SAs, and rekeying IKE SA itself uses a CREATE_CHILD_SA exchange. When creating or rekeying Child SAs, the peers optionally perform an ML-KEM key exchange to add a fresh entropy into the session keys. These situations negotiate hybrid with ECDH and ML-KEM. They first use a CREATE_CHILD_SA exchange in a similar fashion to the initial IKE exchange. Then an ML-KEM key exchange is performed using IKE_FOLLOWUP_KE {{!RFC9370}}. The derived KEYMAT combines SK_d and stirs all the newly negotiated SK(0) in the CREATE_CHILD_SA exchange and 256-bit SK(1) negotiated in IKE_FOLLOWUP_KE exchange as defined in Section 2.2.4 of {{!RFC9370}}: 

~~~
KEYMAT = prf+ (SK_d, SK(0) | Ni | Nr | SK(1))
~~~

IKE SAs can be rekeyed similarly by negotiating a new (EC)DH and ML-KEM key. The SKEYSEED combines SK_d and stirs all the negotiated keys ((EC)DH shared key SK(0), ML-KEM shared key SK(1)) as per Section 2.2.4 of {{!RFC9370}}: 

~~~
SKEYSEED = prf(SK_d, SK(0) | Ni | Nr | SK(1))
~~~

## Recipient Tests {#receipent-tests}

Receiving and handling of malformed ML-KEM public key or ciphertext MUST follow the input validation described in {{FIPS203}}. 

In particular, entities receiving the ML-KEM public key to encapsulate to MUST perform the type and modulus checks in Sections 6.1 of {{FIPS203}} and reject the ML-KEM public key, if malformed. Entities receiving an ML-KEM ciphertext for decapsulation MUST perform the ciphertext and decapsulation key type checks in Section 6.2 of {{FIPS203}} and reject the ciphertext or key, if malformed. These checks could be performed separately before performing the encapsulation or decapsulation steps or be part of the encapsulation or decapsulation process

Note that at decapsulation, ML-KEM uses implicit rejection which leads the decapsulating entity to implicitly reject the decapsulated shared secret by setting it to a hash of the ciphertext together with a random value stored in the ML-KEM secret when the re-encrypted shared secret does not match the original one. 


# Security Considerations

All security considerations from {{!RFC9242}} and {{!RFC9370}} apply to the ML-KEM exchanges described in this specification. 


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


--- back

# Acknowledgments
{:numbered="false"}

[EDNOTE: Update here. ]
The authors would like to thank Valery Smyslov for his valuable contributions to the document. 
