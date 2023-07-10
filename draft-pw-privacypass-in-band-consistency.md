---
title: "Privacy Pass In-Band Key Consistency Checks"
abbrev: "In-Band Key Consistency Checks"
category: std

docname: draft-pw-privacypass-in-band-consistency-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Privacy Pass"
keyword:
 - token
 - extensions
venue:
  group: "Privacy Pass"
  type: "Working Group"
  mail: "privacy-pass@ietf.org"

author:
 -
    ins: T. Pauly
    name: Tommy Pauly
    org: Apple Inc.
    street: One Apple Park Way
    city: Cupertino, California 95014
    country: United States of America
    email: tpauly@apple.com
 -
    ins: C. A. Wood
    name: Christopher A. Wood
    org: Cloudflare
    email: caw@heapingbits.net

normative:
  PRIVACYPASS: I-D.ietf-privacypass-architecture
  PRIVACYPASS-ISSUANCE: I-D.ietf-privacypass-protocol
  CONSISTENCY: I-D.ietf-privacypass-key-consistency
  OHTTP: I-D.ietf-ohai-ohttp

informative:
  K-CHECK: I-D.group-privacypass-k-check

--- abstract

This document describes an in-band key consistency enforcement mechanism
for Privacy Pass deployments wherein the Attester is split from the Issuer and Origin.

--- middle

# Introduction

Key and configuration consistency is an important preqrequisite for guaranteeing
security properties of Privacy Pass. In particular, privacy informally depends
on Clients using an Issuer key that many, if not all, other Clients use.
If a Client were to receive an Issuer key that was specific to them, or restricted
to a small set of Clients, then use of that Issuer key could be used to learn
targeted information about the Client. Clients that share the same Issuer key
are said to have a consisten key.

{{CONSISTENCY}} describes general patterns for implementing consistency in
protocols such as Privacy Pass, and {{K-CHECK}} describes a protocol-agnostic
mechanism for implementing consistency checks that can apply to Privacy Pass.
K-Check is an orthogonal protocol for checking consistency of a given key that
runs out-of-band of Privacy Pass.

This document specifies an in-band consistency check for Privacy Pass. It is only
applicable to deployments where the Attester is split from the Origin and Issuer.
This is because the Attester is responsible for enforcing consistency on behalf
of many Clients.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Attester Consistency Check

In Privacy Pass deployment models where the Attester is split from the Origin
and Issuer, Clients interact with the Attester to run the issuance protocol for
obtaining tokens. In particular, Clients obtain tokens from the Issuer by sending
token requests to the Attester, which forwards token requests to the Issuer.
As an active, on-path participant in the issuance protocol, the Attester is capable
of applying enforcement checks for these token requests. This section describes
a simple key consistency enforcement check to ensure that all Clients behind the
Attester share a consistent view of the Issuer key.

Upon receipt of a token request, which contains a key ID as described in
{{PRIVACYPASS-ISSUANCE}}, an Attester does the following:

1. The Attester checks that it has a cached copy of the private token directory
   from the Issuer corresponding to the token request. If the Attester does not,
   it first obtains a copy of the directory. If this fails, the Attester aborts
   the token request with a failure.
1. The Attester compares the key ID corresponding in the token request to the
   the valid keys in the directory. If no match is found, the Attester aborts token
   request with a failure.
1. If the above steps succeed, the Attester allows the token request to proceed.

Attesters can update their cached copy of Issuer token directories independently
of token requests. Attesters SHOULD refresh their cached copy of the Issuer directory
when it becomes invalid (according to cache control headers).

Attesters SHOULD enforce that Issuer directories contain few keys suitable for use
at any given point in time, and penalize or reject Issuers that advertise too many keys.
This is because Clients and Origins may choose different
keys from this directory based on their local state, e.g., their clock, and too many
keys could be misused for the purposes of partitioning Client anonymity sets.

# Security Considerations

Unlike K-Check, in which Clients can check key consistency against the first valid
key in an Issuer private token directory, the Attester consistency check in this document
needs to allow for greater flexibility, in particular because Client diversity and differences
may lead to different key selections. Attesters need to balance this flexibility against
the overall privacy goals of this consistency check. Admitting an unbounded number of keys
in the Issuer's directory effectively renders the consistency check meaningless, as each Client
could be given a unique key that belongs to the directory set.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

This document was the product of the consistency design team in the Privacy Pass working group.
