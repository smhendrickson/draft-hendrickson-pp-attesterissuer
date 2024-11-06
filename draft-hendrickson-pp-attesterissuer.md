---
title: "Attester Issuer Protocol"
abbrev: "Attester-Issuer"
category: info

docname: draft-hendrickson-pp-attesterissuer-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "smhendrickson/draft-hendrickson-pp-attesterissuer"
  latest: "https://smhendrickson.github.io/draft-hendrickson-pp-attesterissuer/draft-hendrickson-pp-attesterissuer.html"

author:
 -
    fullname: Scott Hendrickson
    organization: Google
    email: scott@shendrickson.com

normative:

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

This document specifies the protocol between the Attester and Issuer as defined
in {{!RFC9576}}. This draft is intended for those deploying the split Attester
Issuer as defined in Section 4.4 of {{!RFC9576}}.

The base Privacy Pass issuance protocol {{!RFC9578}} defines stateless anonymous
tokens, which can either be publicly verifiable or not. This draft is agnostic
to all cryptographic protocols and public/private verifiability.

This draft defines the protocol that will occur behind the issuer as defined in
{{!RFC9577}}.

## Protocol Overview

{{fig-overview}} shows how this draft is only specifying a protocol between the
Attester and Issuer, and makes no changes to the protocols between Attesters and
Clients, or Attesters and Origins.



~~~
                                                        +-----------+
      Client                                            |   Origin  |
        |                    Challenge                  |           |
        <----------------------------------------------------+      |
        |                                               |           |
        |      +~~~~~~~defined in this draft ~~~~~~~+   |           |
        |      |                                    |   |           |
        |      |   +-------------+                  |   |           |
        |      |   |   Attester  |                  |   |           |
        |      |   |             |                  |   |           |
        |      |   | Attest      |    +----------+  |   |           |
        +----------------->      |    |  Issuer  |  |   |           |
        |      |   |             |    |          |  |   |           |
        |   TokenRequest         |    |          |  |   |           |
        +----------------->      |    |          |  |   |           |
        |      |   |             |    |          |  |   |           |
        |      |   |       TokenRequest          |  |   |           |
        |      |   |     +------------------->   |  |   |           |
        |      |   |             |    |          |  |   |           |
        |      |   |             |    |          |  |   |           |
        |      |   |       TokenResponse         |  |   |           |
        |      |   |     <-------------------+   |  |   |           |
        |      |   |             |    |          |  |   |           |
        |   TokenResponse        |    |          |  |   |           |
        <-----------------+      |    |          |  |   |           |
        |      |   |             |    |          |  |   |           |
        |      |   +-------------+    +----------+  |   |           |
        |      |                                    |   |           |
        |      +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+   |           |
        |                                               |           |
        |                    Redeem                     |           |
        +---------------------------------------------------->      |
                                                        |           |
                                                        +-----------+
~~~
{: #fig-overview title="Issuance Protocol Overview"}

# Attester Issuer Protocol {#issuance}

This section describes the Issuance protocol for an Attester to request and
receive a token from an Issuer. This protocol occurs entirely between Section
5.1 and 5.2 of {{!RFC9578}} or Section 6.1 and 6.2 of {{!RFC9578}}.

1. The Client sends a token request to the attester as defined in Section 5.1 or
6.1 of {{!RFC9577}}.

1. The Attester authenticates the client as defined in Section 3.5.1 of {{!RFC9576}}.

1. The Attester sends an IssuerTokenRequest to the Issuer

1. The Issuer signs the token, and returns an IssuerTokenResponse to the Attester.

1. The Attester parses the signature from the IssuerTokenResponse, and sends the
client a TokenResponse as defined in 5.2 or 6.2 of {{!RFC9577}}.

1. The client uses the TokenResponse to particpate in a redemption protocol.

The Attester Issuer issuance protocol is designed such that the Issuer learns
only enough to satisfy issuance requirements, which can be as simple as the
token to sign. Notably the Issuer should not learn any information that may be
uniquely identifying at the Origin, especially if the Origin and Issuer are the
same entity, as defined in section 4.3 of {{!RFC9576}}.

## Attester-to-Issuer Request {#request}

The Attester and Issuer MUST use a mutually authenticated and secure HTTPS
connection. They MAY use Mutual TLS for mutual aumutual authentication.

   struct {
     uint16_t token_type;
     uint8_t token_key_id;
     uint8_t blinded_msg[Ne];
   } IssuerTokenRequest;

   The structure fields are defined as follows:

   *  "token_type" is a 2-octet integer, which matches the type in the
      challenge.

   *  "token_key_id" which matches the type in the challenge.

   *  "blinded_msg" is the Ne-octet blinded message to be signed, which
      matches the challenge.

## Issuer behavior

In the simplest case, the issuer will recieve the IssuerTokenRequest, sign the
message, and return it to the Attester. The signature is defined in Sections 5
and 6 of {{!RFC9578}}, depending on the cryptography used.

## Issuer-to-Attester Response {#response}

   struct {
     uint8_t blinded_signature[Ne];
   } IssuerTokenResponse;

   The structure fields are defined as follows:

   *  "blinded_signature" is a Ne-octet blinded signature over "blinded_msg".


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
