---
title: Observations on Deploying New DNSSEC Cryptographic Algorithms 
abbrev: I-D
docname: draft-york-dnsop-deploying-dnssec-crypto-algs-00
date: 2016-03-21
category: info
ipr: trust200902
wg: DNSOP Working Group
area: Ops

author:
 -
    ins: D. York
    name: Dan York
    org: Internet Society
    email: york@isoc.org
 -
    ins: O. Sury
    name: Ondrej Sury
    org: CZ.NIC
    email: ondrej.sury@nic.cz
 -
    ins: P. Wouters
    name: Paul Wouters
    org: Red Hat
    email: pwouters@redhat.com
 -
    ins: O. Gudmundsson
    name: Olafur Gudmundsson
    org: CloudFlare
    email: olafur+ietf@cloudflare.com

normative:
  RFC2119:

informative:

--- abstract

As new cryptographic algorithms are developed for use in DNSSEC 
signing and validation, this document captures the steps needed 
for new algorithms to be deployed and enter general usage. The 
intent is to ensure a common understanding of the typical deployment 
process and potentially identify opportunities for improvement of 
operations.

--- middle

# Introduction

The DNS Security Extensions (DNSSEC), broadly defined in {{?RFC4033}},
 {{?RFC4034}} and {{?RFC4035}}, make use of cryptographic algorithms 
in both the signing of DNS records and the validation of DNSSEC 
signatures by recursive resolvers.

The current list of cryptographic algorithms can be found in the IANA
"Domain Name System Security (DNSSEC) Algorithm Numbers" registry 
located at http://www.iana.org/assignments/dns-sec-alg-numbers/ 
Algorithms are added to this IANA registry through a process defined 
in {{?RFC6014}}.  Note that {{?RFC6944}} provides some guidance as 
to which of these algorithms should be implemented and supported.

Historically DNSSEC signatures have primarily used cryptographic 
algorithms based on RSA keys. As deployment of DNSSEC has 
increased there has been interest in using newer and more secure 
algorithms, particularly those using elliptic curve cryptography.  
The ECDSA algorithm {{?RFC6605}} has seen some adoption and two 
new algorithms are being proposed: Ed25519 
{{?I-D.ietf-curdle-dnskey-ed25519}} and Ed448 
{{?I-D.ietf-curdle-dnskey-ed448}}.

The challenge is that the deployment of a new cryptographic 
algorithm for DNSSEC is not a simple process. DNSSEC algorithms 
are used throughout the DNS infrastructure for tasks such as:

* Generation of keys (`DNSKEY` record) for signing

* Creation of DNSSEC signatures in zone files (`RRSIG`)

* Usage in a Delegation Signer (`DS`) record {{?RFC3658}} for the 
"chain of trust" connecting back to the root of DNS

* Validation of DNSSEC signatures by DNS resolvers

In order for a new cryptographic algorithm to be fully deployed, 
all aspects of the DNS infrastructure that interact with DNSSEC 
must be updated to use the new algorithm.

This document outlines the current understanding of the components 
of the DNS infrastructure that need to be updated to deploy a new 
cryptographic algorithm.

It should be noted that DNSSEC is not alone in complexity of 
deployment.  The IAB documented "Guidelines for Cryptographic 
Algorithm Agility" in {{?RFC7696}} to highlight the importance 
of this issue.

## Terminology

In this document, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" are to be interpreted as described in BCP 14, RFC 2119
{{RFC2119}}.

# Aspects of Deploying New Algorithms

For a new cryptographic algorithm to be deployed in DNSSEC, the 
following aspects of the DNS infrastructure must be updated:

* DNS resolvers performing validation

* Authoritative DNS servers

* Signing software

* Registries

* Registrars

* DNS Hosting Operators

* Applications

Each of these aspects is discussed in more detail below.

## DNS Resolvers Performing Validation

DNS recursive resolvers perform "validation" to check the 
DNSSEC signatures of records received in a DNS query.  To 
validate the signatures, the resolvers need to be able to 
understand the algorithm used to create the signatures.

In the case of a new algorithm, the resolver software needs
to be updated.  In some cases this could require waiting 
until an underlying library is updated to support the new
algorithm.

Once the software is updated, the updates need to be deployed 
to all resolvers using that software.  This can be challenging 
in cases of customer-premises equipment (CPE) that does not
have any mechanism for automatic updating.

### Resolvers and Unknown Algorithms

It should be noted that section 5.2 of {{?RFC4035}} states:

“If the resolver does not support any of the algorithms listed 
in an authenticated DS RRset, then the resolver will not be 
able to verify the authentication path to the child zone. 
In this case, the resolver SHOULD treat the child zone as 
if it were unsigned.”

This means that signing a zone with a new algorithm that is 
not widely supported by DNS resolvers would result in the 
signatures being ignored and the zone treated as unsigned
until resolvers were updated to recognize the new algorithm.

## Authoritative DNS Servers

Authoritative DNS servers serve out signed DNS records.  Serving new
DNSSEC signing algorithms should not be a problem as a well-written
authoritative DNS server implementation should be agnostic to the RR
DATA they serve.  Note that some authoritative server implementations 
could include DNSSEC signing as part of the server and thus also fall 
into the "Signing Software" category below.

[NOTE(OS): Do we also address new NSEC/NSEC3 hashing algorithms?
Because that would require update in the authoritative DNS server.]

## Signing Software

The software performing the signing of the records needs to
be updated with the new cryptographic algorithm.

User interfaces that allow users to interact with the 
DNSSEC signing software may also need to be updated to 
reflect the existence of the new algorithm.

Note that the key and signatures with the new algorithm will
need to co-exist with the existing key and signatures for 
some period of time, which would have some impact on the size
of the DNS records. 

[NOTE(OS): Shouldn't we just update the language that requires the
resolver to be so strict and finally be done with this requirement?
Or just give a recommendation in the paragraph on resolver here?]

[NOTE(DY): I just noticed that "Signing software" or "Signers"
does not exist in the "DNS Terminology draft at
https://tools.ietf.org/html/rfc7719 ]

## Registries

The registry for a top-level domain (TLD) needs to accept 
DS records using the new cryptographic algorithm.

Observations to date have shown that some registries only 
accept DS records with certain algorithms.  Registry 
representatives have indicated that they verify the accuracy
of DS records to reduce technical support incidents and ensure 
customers do not mistakenly create any outages.  

However, this means that registries who perform this level
of checking must be able to understand new algorithms in 
order to successfully verify the DS records.

Separately, feedback from registrars has indicated that they
do not currently have any mechanism to understand what
DNSSEC algorithms a registry can accept.

## Registrars

Registrars perform a critical role in the DNSSEC "chain of trust" 
of passing the DS record up to the Registry to ensure that 
the signed zone can be authenticated from the root of DNS all 
the way to the zone.

If the registrar is also providing the DNS hosting services
for a domain, the registrar can easily create the `DS` record
from the `DNSKEY` record and pass the DS record up to the 
registry.

However, if the authoritative servers for a domain are not 
with the registrar, then the registrar needs to provide
some mechanism to accept a DS record to pass that up to the
registry.  Typically this is done through a web interface.

An issue is that many registrar web interfaces only allow
the input of DS records using a listed set of DNSSEC 
algorithms.  Any new cryptographic algorithms need to be
added to the web interface in order to be accepted into
the registrar's system.

Additionally, in a manner similar to registries, many 
registrars perform some level of verification on the DS
record to ensure it was entered "correctly".  To do this
verification, the registrar's software needs to understand 
the algorithm used in the DS record.  This requires the 
software to be updated to support the new algorithm.

Note that work is currently underway in {{?I-D.ietf-dnsop-maintain-ds}}
to provide an automated mechanism to update the DS records
with a registry.  If this method becomes widely adopted, 
registrar web interfaces will no longer be needed.

## DNS Hosting Operators

DNS hosting operators are entities that are operating the 
authoritative DNS servers for domains and with DNSSEC
are also providing the signing of zones.  In many cases they may 
also be the registrar for domain names, but in other cases they
are a separate entity providing DNS services to customers.

DNS hosting operators need to update their authoritative 
DNS server software to understand new cryptographic algorithms,
but they also need to update their web interfaces and provisioning
software to allow configuration and support of new algorithms.

## Applications

Beyond the recursive resolvers, authoritative servers, web 
interfaces and provisioning software, it has been observed that
some applications (or "apps"), particularly in the mobile 
environment, are including their own DNS resolvers within the
app itself.   These recursive resolvers are used by the app 
instead of the recursive resolver included with the underlying
operating system.  These applications that perform DNSSEC
validation would need to also be updated to understand a 
new algorithm.

In many cases, it may be that an underlying developer library
needs to be updated first. It will then depend upon how long
it takes the application developer to pull in the updated 
library.

Outside of applications, these developer libraries are also 
typically used by recursive resolver software and signing software.

# Conclusion

This document provides a view into the steps necessary for the
deployment of new cryptographic algorithms in DNSSEC at the time
of this publication.  In order to more rapidly roll out new
DNSSEC algorithms, these steps must be understood and hopefully
improved over time.  


# IANA Considerations

This document does not make any requests of IANA.

# Security Considerations

No new security considerations are created by this document.

It should be noted that there are security considerations regarding
changing DNSSEC algorithms that are mentioned in both {{?RFC6781}}
and {{?RFC7583}}.

# Acknowledgements

The information in this document evolved out of several mailing 
list discussions and also through engagement with participants 
in the DNSSEC Workshop sessions at ICANN 53 (Buenos Aires) and 
ICANN 55 (Marrakech).


