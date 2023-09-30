---
title: External Keys And Signatures For Use In Internet PKI
abbrev: External keys and sigs
# <!-- EDNOTE: Edits the draft name -->
docname: draft-ounsworth-pq-external-pubkeys-00
# <!-- date: 2012-01-13 -->
# <!-- date: 2012-01 -->
# <!-- date: 2012 -->

# <!-- stand_alone: true -->
area: Security
wg: LAMPS
kw: Internet-Draft
cat: std

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
Many of the post quantum cryptographic algorithms have either large public keys or signatures. In the interest of reducing bandwidth of transitting X.509 certificates, this document defines new public key and signature algorithms for referencing external public key and signature data by hash, URL, etc. This mechanism is designed to mimic the behaviour of an Authority Information Access extension.  

<!-- End of Abstract -->


--- middle

# Introduction {#sec-intro}


# External Value {#sec-pub}

The id-external-value algorithm identifier is used for identifying a public key or signature which is provided as a reference to external data.

~~~
id-external-value ::= < OID >
~~~

EDNOTE: for prototyping purposes, `id-external-value ::= 1.3.6.1.4.1.22554.4.2`

The corresponding subjectPublicKey is the DER encoding of the following structure:

~~~
ExternalValue ::= SEQUENCE {
  location     GeneralName,
  hashAlg      AlgorithmIdentifier,
  hashVal      OCTET STRING
}
~~~


Upon retrieval of the referenced data, the hash of the OCTET STRING of the retrieved data (removing base64 encoding as per [RFC4648] if necessary) MUST be verified using hashAlg to match the ExternalPublicKey.hash value.

## External Public Key

When used with a public key, algorithm parameters for id-external-value are absent.

When ExternalValue is placed into a SubjectPublicKeyInfo.subjectPublicKey, the ExternalValue.location MUST refer to a DER-encoded SubjectPublicKeyInfo, which MAY be base64 encoded as per [RFC4648] for easier transport over text protocols.

## External Signature

When used with a signatureAlgorithm, algorithm parameters are to contain the AlgorithmIdentifier of the signature that is being externalized.

When ExternalValue is placed into a signatureValue, the location MUST refer to the BIT STRING of a signatureValue, which MAY be base64 encoded as per [RFC4648] for easier transport over text protocols.




<!-- End of Introduction section -->


# IANA Considerations {#sec-iana}
The ASN.1 module OID is TBD.  The id-alg-composite OID is to be assigned by IANA. 


<!-- End of IANA Considerations section -->


# Security Considerations

## CSRs and CT logs

In practice, situations will arise where the ExternalPublicKey.location refers to a location which is not publicly available either because it is in a local keystore, on a private network, or no longer being hosted.

Not having the public key in a certificate signing request (CSR) could make it substantially harder for CAs to perform vetting of the key, for example for cryptographic strength or checking for prior revocation due to key compromise. A certificate requester MUST make the full public key available to the CA at the time of certificate request either by ensuring that the link in the ExternalPublicKey.location is visible to the CA, or by supplying the full public key to the CA out of band.


Not having the public key in Certificate Transparency (CT) logs could make it substantially harder for researchers to perform auditing tasks on CT logs. This may require additional CT mechanisms.

<!-- End of Security Considerations section -->

# Appendices

## ASN.1 Module

## Intellectual Property Considerations

The following IPR Disclosure relates to this draft:

https://datatracker.ietf.org/ipr/3588/



# Contributors and Acknowledgements
This document incorporates contributions and comments from a large group of experts. The Editors would especially like to acknowledge the expertise and tireless dedication of the following people, who attended many long meetings and generated millions of bytes of electronic mail and VOIP traffic over the past year in pursuit of this document:

John Gray (Entrust),
Serge Mister (Entrust).

We are grateful to all, including any contributors who may have
been inadvertently omitted from this list.

This document borrows text from similar documents, including those referenced below. Thanks go to the authors of those
   documents.  "Copying always makes things easier and less error prone" - [RFC8411].

## Making contributions

Additional contributions to this draft are weclome. Please see the working copy of this draft at, as well as open issues at:

https://github.com/EntrustCorporation/draft-ounsworth-pq-external-keys


<!-- End of Contributors section -->
