---
v: 3

title: Quantum-Resistant Cipher Suites for LAKE
abbrev: LAKE PQC
docname: draft-ietf-lake-pqsuites-latest
category: std
submissiontype: IETF

v3xml2rfc:
  silence:
  - Found SVG with width or height specified

ipr: trust200902
area: Security
workgroup: LAKE Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

coding: utf-8


venue:
  group: "Lightweight Authenticated Key Exchange"
  type: "Working Group"
  mail: "lake@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/lake/"
  github: "lake-wg/pqsuites"

author:
-
    ins: G. Selander
    name: Göran Selander
    org: Ericsson
    email: goran.selander@ericsson.com
-
    ins: J. Preuß Mattsson
    name: John Preuß Mattsson
    org: Ericsson
    email: john.mattsson@ericsson.com

normative:
  RFC9528:
  I-D.ietf-cose-dilithium:
  I-D.ietf-jose-pqc-kem:

informative:
  I-D.ietf-iotops-7228bis:
  I-D.ietf-lake-edhoc-psk:
  I-D.connolly-cfrg-xwing-kem:
  I-D.sfluhrer-cfrg-ml-kem-security-considerations:
  I-D.connolly-cfrg-ml-dsa-security-considerations:
  FIPS202:
    target: https://doi.org/10.6028/NIST.FIPS.202
    title: SHA-3 Standard - Permutation-Based Hash and Extendable-Output Functions
    seriesinfo:
      "NIST": "FIPS 202"
    author:
    date: August 2015
  FIPS203:
    target: https://doi.org/10.6028/NIST.FIPS.203
    title: Module-Lattice-Based Key-Encapsulation Mechanism Standard
    seriesinfo:
      "NIST": "FIPS 203"
    author:
    date: August 2024
  FIPS204:
    target: https://doi.org/10.6028/NIST.FIPS.204
    title: Module-Lattice-Based Digital Signature Standard
    seriesinfo:
      "NIST": "FIPS 204"
    author:
    date: August 2024
  CNSA20:
    title: Commercial National Security Algorithm Suite 2.0
    author:
      - org: National Security Agency
    date: September 2022
    target: "https://www.nsa.gov/Press-Room/Press-Releases-Statements/Press-Release-View/Article/3148990/nsa-releases-future-quantum-resistant-qr-algorithm-requirements-for-national-se/"
  IANA_edhoc_cipher_suites:
    title: EDHOC Cipher Suites
    author:
      - org: IANA
    date:
    target: https://www.iana.org/assignments/edhoc#edhoc-cipher-suites
  IANA_edhoc_method_types:
    title: EDHOC Method Types
    author:
      - org: IANA
    date:
    target: https://www.iana.org/assignments/edhoc#edhoc-method-types


--- abstract

The Lightweight Authenticated Key Exchange (LAKE) protocol, also known as Ephemeral Diffie-Hellman over COSE (EDHOC), achieves post-quantum security by adding new cipher suites with quantum-resistant algorithms, such as ML-DSA for digital signatures and ML-KEM for key exchange. This document specifies how the LAKE protocol operates in a post-quantum setting using both signature-based and PSK-based authentication methods, and defines corresponding cipher suites.

--- middle


# Introduction

The Lightweight Authenticated Key Exchange (LAKE) protocol defined in {{RFC9528}}, also known as Ephemeral Diffie-Hellman over COSE (EDHOC), supports the use of multiple authentication methods and the negotiation of cipher suites based on COSE algorithms. Currently, four asymmetric authentication methods (0, 1, 2, and 3) are defined. In addition, a symmetric key-based authentication method is being developed, see {{I-D.ietf-lake-edhoc-psk}}.

Currently defined cipher suites rely on Elliptic Curve Cryptography (ECC) for key exchange and authentication, making them vulnerable in the event that a Cryptographically Relevant Quantum Computer (CRQC) is constructed.

This document specifies how the LAKE protocol can operate in a post-quantum setting using both signature-based and PSK-based authentication, and defines corresponding cipher suites. With this modification the protocol is no longer dependent on Diffie-Hellman which makes EDHOC a misnomer and we henceforth use the name LAKE for the protocol.

## Terminology # {#terminology}

{::boilerplate bcp14}

Readers are expected to be familiar with {{RFC9528}}. To avoid misunderstanding of the capabilities of the protocol, the name EDHOC is replaced by LAKE. To avoid misunderstanding with terminology from {{RFC9528}}, the prefix EDHOC is retained when needed, for example in the IANA registries.


# LAKE with Quantum-Resistant Algorithms

Method 0 in {{RFC9528}}, which uses digital signatures for authentication by both the Initiator and Responder, and also the PSK method in {{I-D.ietf-lake-edhoc-psk}}, is straightforward to use with standardized post-quantum algorithms.

A quantum-resistant signature algorithm, such as ML-DSA {{I-D.ietf-cose-dilithium}}, is a drop-in replacement for classical signature algorithms such as ECDSA. For post-quantum secure key exchange, in order to replace the Ephemeral Diffie-Hellman key exchange, a quantum-resistant Key Encapsulation Mechanism (KEM), such as ML-KEM {{I-D.ietf-jose-pqc-kem}}, can be applied directly to the LAKE protocol, as is detailed in {{KEM}}.

To enable post-quantum security support for LAKE it suffices to register new cipher suites using COSE registered algorithms. Cipher suites using ML-KEM-512 and ML-KEM-1024 {{I-D.ietf-jose-pqc-kem}} for key exchange, and ML-DSA-44 and ML-DSA-87 {{I-D.ietf-cose-dilithium}} for digital signatures are specified in {{suites-registry}}. As both ML-KEM {{FIPS203}} and ML-DSA {{FIPS204}} internally use SHAKE256 {{FIPS202}}, it was natural to also use SHAKE256 for key derivation. Additional post-quantum cipher suites may be specified.

However, note that for application-layer use, e.g., with EDHOC_exporter, the use of SHA-256 or SHA-384 provides sufficient security against quantum pre-image attacks, offering 128-bit (or 192-bit) security levels, see {{suites-registry}}.

Methods 1–3 in {{RFC9528}} use a Diffie-Hellman/Non-Interactive Key Exchange (NIKE) based API for authentication. As of this writing, no standardized post-quantum algorithms for these methods exist. To highlight which methods that require DH/NIKE a column is added to the EDHOC Method Type registry, see {{method-update}}. To highlight matching cipher suites a corresponding column indicating support for DH/NIKE is added, see {{suites-registry}}.

An alternative path to post-quantum support for the LAKE protocol, not pursued in this document, is to define new authentication methods based on Key Encapsulation Mechanisms (KEMs).

Compared to elliptic curve algorithms such as ECDHE, ECDSA, and EdDSA, ML-KEM-512 and ML-DSA-44 (and ML-KEM-1024 and ML-DSA-87) introduce significantly higher overhead {{FIPS203}}{{FIPS204}}, but currently are the most lightweight standardized post-quantum algorithms to use with LAKE. More efficient post-quantum signature schemes are being standardized, such as FN-DSA, which could offer smaller signatures. This remains a possible direction for future research, analysis and standardization, after which they may be included in new cipher suites.

However, it is important to note that these cipher suites may not be usable for certain classes of constrained devices (see {{I-D.ietf-iotops-7228bis}}) due to  for example, increased size of signatures or of KEM keys in quantum-resistant algorithms.

# Using KEMs in the Key Exchange {#KEM}

Given a quantum-resistant KEM, such as ML-KEM-512, with encapsulation key ek, decapsulation key dk, ciphertext c, and shared secret key K (using the notation of {{FIPS203}}), the Diffie-Hellman procedure in the key exchange is replaced by a KEM procedure as follows:


* The Initiator generates a new encapsulation / decapsulation key pair matching the selected cipher suite.

* The encapsulation key ek is transported in the G_X field in message_1.

* The Responder calculates (K,c) = Encaps(ek).

* The ciphertext c is transported in the G_Y field in message_2.

* The Initiator calculates the shared secret K = Decaps(dk, c).

* G_XY is the shared secret key K.

The security requirements and security considerations of {{RFC9528}} and the KEM algorithm used apply. For example, the Initiator MUST generate a new encapsulation / decapsulation key pair for each LAKE session.

Note that G_Y does not contain a public key when a KEM is used in this way. The definition of LAKE message_2 in {{Section 5.3.1 of RFC9528}} remains the same:

~~~~~~~~~~~ CDDL
message_2 = (
  G_Y_CIPHERTEXT_2 : bstr,
)
~~~~~~~~~~~

and G_Y_CIPHERTEXT_2 remains the concatenation of G_Y and CIPHERTEXT_2, the latter is defined in {{Section 5.3.2 of RFC9528}}. But now G_Y is a KEM ciphertext.

Just as with the ephemeral key G_Y, the length of KEM ciphertext c is known from the corresponding algorithm in the selected cipher suite, see {{fig-ct-length}}. Hence the Initator can separate out the concatenated ciphertexts and decapsulate and decrypt, respectively.

~~~~~~~~~~~
+-------------+------------------------------+
|     KEM     | Length of ciphertext (bytes) |
+=============+==============================+
| ML‑KEM‑512  |                          768 |
| ML‑KEM‑768  |                         1088 |
| ML‑KEM‑1024 |                         1568 |
+-------------+------------------------------+
~~~~~~~~~~~
{: #fig-ct-length title="Length of ML-KEM Ciphertext."}

Note also that this use of KEM applies both to standalone KEM and hybrid KEMs such as, e.g., X-wing {{I-D.connolly-cfrg-xwing-kem}}.

Conventions for using post-quantum KEMs within COSE are described in {{I-D.ietf-jose-pqc-kem}}. The shared secret key K corresponds to the initial shared secret SS' in that document.

# Security Considerations

The cipher suites defined in {{RFC9528}} rely on Elliptic Curve Cryptography (ECC) for key exchange and authentication, which would be broken by a Cryptographically Relevant Quantum Computer (CRQC). In this section we discuss the security considerations brought by the new cipher suites.

## Usual LAKE security properties

When used with Method 0 from {{RFC9528}}, where both the Initiator and Responder authenticate using digital signatures, or with the PSK method defined in {{I-D.ietf-lake-edhoc-psk}}, these cipher suites preserve the security properties discussed in {{Section 9 of RFC9528}} (for Method 0) and in {{Section 9 of I-D.ietf-lake-edhoc-psk}} (for PSK method). Let us cite, for example, mutual authentication and confidentiality, keys security, identity protection, External Authorization Data (EAD) security, etc.

This is because the security properties of LAKE (methods 0 and PSK) are affected by cipher suites only through the security of the algorithms involved. Since the algorithms introduced in these cipher suites -- ML-KEM, ML-DSA and SHAKE256 -- are post-quantum secure, i.e., secure against a quantum adversary and, by extension, secure against a classical adversary, the security properties are guaranteed.


## Post-quantum security

Cipher suites specified in this document use ML-KEM for ephemeral key exchange, and ML-DSA for authentication. These algorithms are believed secure against a quantum adversary. Security considerations of ML-KEM are discussed in {{I-D.sfluhrer-cfrg-ml-kem-security-considerations}}, and those of ML-DSA are addressed in {{I-D.connolly-cfrg-ml-dsa-security-considerations}}.

The security of LAKE for methods 0 to 3, specified in {{RFC9528}}, and for the PSK mode, estalished in {{I-D.ietf-lake-edhoc-psk}} has been estalished in the Random Oracle Model (ROM), i.e., against "classical" adversaries. Proving the same security properties against quantum adversaries, i.e., in the Quantum Random Oracle Model (QROM), for LAKE Method 0 and PSK, is left for futur work. To date, only the standalone primitives ML-KEM and ML-DSA have been analyzed in the QROM, and their integration into cipher suites is not sufficient to claim the overall post-quantum security of LAKE Method 0 and PSK in post-quantum settings.

The first two proposals have Category 1 security level (according to NIST), while proposal 3 is in Category 5.

### Store Now Decrypt Later

The use of PQ-KEM, e.g., ML-KEM, for ephemeral key exchange in LAKE Method 0 and PSK protects against Store Now Decrypt Later (SNDL) attacks from an adversary equipped with a CRQC.

### Hybridation


In the event that a feasible attack against ML-KEM or ML-DSA is discovered (that does not require a CRQC), the use of hybrid algorithms in a cipher suite, i.e., a cipher suite combining classical and post-quantum algorithms for ephemeral key exchange and signature-based authentication, ensures the continuity of the (classical) security of the LAKE Method 0 and PSK protocols in post-quantum settings (as long as the classical algorithms remain secure).


### Side-channel considerations

Implementation of lattice-based algorithms have been shown to be susceptible to side-channel attacks, e.g., regarding timing or power analysis attacks. Side-channel resistance of ML-KEM and ML-DSA depends both on their implementation and on how they are used within the protocol itself.

Implementations MUST follow the side-channel guidance given in the specifications of ML-KEM {{I-D.sfluhrer-cfrg-ml-kem-security-considerations}} and ML-DSA {{I-D.connolly-cfrg-ml-dsa-security-considerations}}. Moreover, ML-KEM key used for ephemeral key exchange MUST be freshly generated for each LAKE protocol session. In addition, analyzing the resistance of LAKE Method 0 and PSK to side-channel attacks, in post-quantum settings is out of scope of this document and left for future work.


# Privacy Considerations

This document does not add any new privacy considerations to those discussed in {{RFC9528}}.

# IANA Considerations

This section specifies IANA updates for EDHOC Method Types and Cipher Suites registration.

## EDHOC Method Type Registry {#method-update}

IANA is requested to update the EDHOC Method Type registry {{IANA_edhoc_method_types}} with a column with heading "Requires DH/NIKE" indicating that the method requires Diffie-Hellman or Non-Interactive Key Exchange. Valid table entries in this column are "Yes" and "No".

For the existing Method Types, the following entries are inserted in the new "Requires DH/NIKE" column:

~~~~~~~~~~~~~~~~~~~~~~~
Value: 0, Requires DH/NIKE: No
Value: 1, Requires DH/NIKE: Yes
Value: 2, Requires DH/NIKE: Yes
Value: 3, Requires DH/NIKE: Yes
~~~~~~~~~~~~~~~~~~~~~~~

This note is to be removed before publishing as an RFC.
Once the LAKE PSK authentication method {{I-D.ietf-lake-edhoc-psk}} is standardized and registered with IANA, add the line:

~~~~~~~~~~~~~~~~~~~~~~~
Value: 4, Requires DH/NIKE: No
~~~~~~~~~~~~~~~~~~~~~~~

## EDHOC Cipher Suites Registry {#suites-registry}

IANA is requested to update the EDHOC Cipher Suites registry {{IANA_edhoc_cipher_suites}} with a column with heading "Supports DH/NIKE" indicating that the cipher suite supports Diffie-Hellman or Non-Interactive Key Exchange. Valid table entries in this column are "Yes" and "No".

For the existing cipher suites 0-6, 24, 25, the entry "Yes" is inserted in the new "Supports DH/NIKE" column.

Furthermore, IANA is requested to register the following entries in the EDHOC Cipher Suites registry:

~~~~~~~~~~~~~~~~~~~~~~~
Value: TBD1
Array: 30, -45, 16, TBD10, -48, 10, -16
Description: AES-CCM-16-128-128, SHAKE256, 16, MLKEM512, ML-DSA-44,
             AES-CCM-16-64-128, SHA-256
Supports DH/NIKE: No
Reference: [[This document]]
~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~
Value: TBD2
Array: 3, -45, 16, TBD10, -48, 3, -16
Description: A256GCM, SHAKE256, 16, MLKEM512, ML-DSA-44,
             A256GCM, SHA-256
Supports DH/NIKE: No
Reference: [[This document]]
~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~
Value: TBD3
Array: 3, -43, 16, TBD12, -48, 3, -43
Description: A256GCM, SHA-384, 16, MLKEM1024, ML-DSA-85,
             A256GCM, SHA-384
Supports DH/NIKE: No
Reference: [[This document]]
~~~~~~~~~~~~~~~~~~~~~~~

Cipher suite TBD3 is intended for for high security applications such as government use and financial applications. This cipher suites consists of algorithms from the Commercial National Security Algorithm (CNSA) 2.0 suite {{CNSA20}}.

--- back


# Acknowledgments # {#acknowledgment}
{: numbered="no"}

This work was supported partially by Vinnova - the Swedish Agency for Innovation Systems - through the EUREKA CELTIC-NEXT project CYPRESS.
