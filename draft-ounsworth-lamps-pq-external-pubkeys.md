---
title: External Keys For Use In Internet X.509 Certificates
abbrev: External X.509 Keys
docname: draft-ounsworth-lamps-pq-external-pubkeys-latest


# <!-- stand_alone: true -->
# area: Security
wg: LAMPS
kw: Internet-Draft
cat: std
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
consensus: true
v: 3
keyword:
 - X.509
 - External Public Key
venue:
#  group: "Limited Additional Mechanisms for PKIX and SMIME (lamps)"
#  type: "Working Group"
#  mail: "spasm@ietf.org"
#  arch: "https://mailarchive.ietf.org/arch/browse/spasm/"
  github: "EntrustCorporation/draft-pq-external-pubkeys"
  latest: "https://EntrustCorporation.github.io/draft-pq-external-pubkeys/draft-ounsworth-pq-external-pubkeys.html"

coding: us-ascii
pi:    # can use array (if all yes) or hash here
  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:
    -
      ins: M. Ounsworth
      name: Mike Ounsworth
      org: Entrust Limited
      abbrev: Entrust
      street: 1000 Innovation Drive
      city: Ottawa, Ontario
      country: Canada
      code: K2K 1E3
      email: mike.ounsworth@entrust.com

    -
      name: Markku-Juhani O. Saarinen
      org: PQShield
      email: mjos@pqshield.com

    -
      fullname: J. Gray
      organization: Entrust
      email: john.gray@entrust.com
    -
      fullname: D. Hook
      organization: KeyFactor
      email: david.hook@keyfactor.com

normative:
  RFC4648:
  RFC5280:
  RFC8411:

# <!-- EDNOTE: full syntax for this defined here: https://github.com/cabo/kramdown-rfc2629 -->

informative:

updates:
  RFC5280:

--- abstract
Many of the post quantum cryptographic algorithms have large public keys. In the interest of reducing bandwidth of transitting X.509 certificates, this document defines new public key and algorithms for referencing external public key data by hash, and location, for example URL. This mechanism is designed to mimic the behaviour of an Authority Information Access extension.

<!-- End of Abstract -->


--- middle

# Introduction {#sec-intro}


# External Value {#sec-pub}

The id-external-value algorithm identifier is used for identifying a public key or signature which is provided as a reference to external data.

~~~
id-external-value OBJECT IDENTIFIER  ::=  { iso(1)
            identified-organization(3) dod(6) internet(1)
            security(5) mechanisms(5) pkix(7) algorithms(6)
            TBDOID }
~~~

EDNOTE: for prototyping purposes, `id-external-value ::= 1.3.6.1.4.1.22554.4.2`

The corresponding subjectPublicKey is the DER encoding of the following structure:

~~~
ExternalValue ::= SEQUENCE {
  location     GeneralNames,
  hashAlg      AlgorithmIdentifier,
  hashVal      OCTET STRING
}
~~~

Upon retrieval of the referenced data, the hash of the OCTET STRING of the retrieved data (removing base64 encoding as per [RFC4648] if necessary) MUST be verified using hashAlg to match the `ExternalPublicKey.hash` value.

`GeneralNames` is defined in [!RFC5280] as 

~~~
GeneralNames ::= SEQUENCE SIZE (1..MAX) OF GeneralName
~~~

which we use instead of `GeneralName` so that certificate issuers can
specify multiple backup key servers for high availability or specify key 
identifiers in multiple formats if the corresponding public keys will
be distributed in multiple keystore formats. When multiple key locations 
are specified, they MUST represent alternative locations for retrieval of the
same key and MUST NOT be used as a mechanism to place multiple subject
keys into a single certificate. Thus, when multiple key locations 
are specified, the client MAY try them in any order and stop when it 
successfully retrieves a public key whose hash matches `hashVal`.



## External Public Key

When used with a public key, algorithm parameters for id-external-value are absent.

When ExternalValue is placed into a SubjectPublicKeyInfo.subjectPublicKey, the ExternalValue.location MUST refer to a DER-encoded SubjectPublicKeyInfo, which MAY be base64 encoded as per [RFC4648] for easier transport over text protocols.




<!-- End of Introduction section -->


# IANA Considerations {#sec-iana}
##  Object Identifier Allocations

###  Module Registration - SMI Security for PKIX Module Identifier

-  Decimal: IANA Assigned - **Replace TBDMOD**
-  Description: EXTERNAL-PUBKEY-2023 - id-mod-external-pubkey
-  References: This Document

###  Object Identifier Registrations - SMI Security for PKIX Algorithms

- Attest Statement
  - Decimal: IANA Assigned - Replace **TBDOID**
  - Description: id-external-value
  - References: This Document

<!-- End of IANA Considerations section -->


# Security Considerations

There are no security implications to externalizing a public key from a certificate as described in this draft. It is of course possible for a malicious actor to replace or tamper with the public key data at the referenced location, but since the hash of the public key data is included in the signed certificate, any such tampering will be detected and the certificate verification will fail. For this reason, external public key data MAY be served over an insecure channel such as HTTP.

## CSRs and CT logs

In practice, situations will arise where the ExternalPublicKey.location refers to a location which is not publicly available either because it is in a local keystore, on a private network, or no longer being hosted.

Not having the public key in a certificate signing request (CSR) could make it substantially harder for CAs to perform vetting of the key, for example for cryptographic strength or checking for prior revocation due to key compromise. A certificate requester MUST make the full public key available to the CA at the time of certificate request either by ensuring that the link in the ExternalPublicKey.location is visible to the CA, or by supplying the full public key to the CA out of band.


Not having the public key in Certificate Transparency (CT) logs could make it substantially harder for researchers to perform auditing tasks on CT logs. This may require additional CT mechanisms.

<!-- End of Security Considerations section -->

# Appendices

## ASN.1 Module

~~~
{::include EXTERNAL-PUBKEY-2023.asn}
~~~

## Samples

Here is a sample of a Kyber1024 end entity certificate with an external public key. A trust anchor certificate using the algorithm ecdsaWithSHA256 is provided so that the Kyber1024 End Entity certificate can be verified.

This is a modest example demonstrating a 550 byte Kyber1024 certificate and a 2.2 kb external Kyber1024 public key. This "compression" effect will be even more pronounced with algorithms such as Classic McEliece which have public keys in the hundreds of kilobytes; with the external public key mechanism, the size of the certificate remains constant regardless of how large the externalized subject public key is.

End entity Kyber1024 Certificate with `ExternalValue` public key:

~~~
{::include samples/ee_external_cert.pem}
~~~

For illustrative purposes, the `SubjectPublicKeyInfo` within the end entity certificate decodes as:

~~~
subjectPublicKeyInfo SubjectPublicKeyInfo SEQUENCE (2 elem)
      algorithm AlgorithmIdentifier SEQUENCE (1 elem)
        algorithm OBJECT IDENTIFIER 1.3.6.1.4.1.22554.4.2 ExternalValue
      subjectPublicKey BIT STRING (688 bit)
        SEQUENCE (3 elem)
          [6] (35 byte) file://local_keyserver/surveyors.db
          SEQUENCE (1 elem)
            OBJECT IDENTIFIER 2.16.840.1.101.3.4.2.1 sha-256
          OCTET STRING (32 byte) E73D4BC89752FD359...
~~~

The external public key object referenced by the end entity certificate is:

~~~
{::include samples/ee_key.pem}
~~~

For illustrative purposes, the key data, which is itself a `SubjectPublicKeyInfo`, decodes as:

~~~
SEQUENCE (2 elem)
  SEQUENCE (1 elem)
    OBJECT IDENTIFIER 1.3.6.1.4.1.22554.5.6.3 Kyber1024
  BIT STRING (12544 bit) 01101111…
~~~

The following trust anchor certificate can be used to validate the above end entity certificate.

~~~
{::include samples/ca_cert.pem}
~~~

## Intellectual Property Considerations

None.



# Contributors and Acknowledgements

This document incorporates contributions and comments from a large group of experts. The Editors would especially like to acknowledge the expertise and tireless dedication of the following people, who attended many long meetings and generated millions of bytes of electronic mail and VOIP traffic over the past year in pursuit of this document:

Serge Mister (Entrust).

We are grateful to all, including any contributors who may have
been inadvertently omitted from this list.

This document borrows text from similar documents, including those referenced below. Thanks go to the authors of those
   documents.  "Copying always makes things easier and less error prone" - [RFC8411].

## Making contributions

Additional contributions to this draft are welcome. Please see the working copy of this draft at, as well as open issues at:

https://github.com/EntrustCorporation/draft-ounsworth-pq-external-keys


<!-- End of Contributors section -->
