---
title: "Attester Issuer Protocol"
abbrev: "Attester-Issuer"
category: info

docname: draft-hendrickson-pp-attesterissuer-latest
submissiontype: IETF
v: 3
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
 -
    fullname: Thibault Meunier
    organization: Cloudflare Inc.
    email: ot-ietf@thibault.uk

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
        |      +~~~~~~ defined in this draft ~~~~~~~+   |           |
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
connection. They MAY use Mutual TLS for mutual authentication. The
"IssuerTokenRequest" defined below is written in TLS Presentation Layer (Section
4 of {{!RFC8446}}), although the Attester and Issuer may instead choose to use
another equivelent data interchange format such as JSON ({{!RFC8259}}).

~~~tls
struct {
   uint16_t token_type;
   select (token_type) {
      case (0x0001): /* Type VOPRF(P-384, SHA-384), RFC 9578 */
         uint8_t truncated_token_key_id;
         uint8_t blinded_msg[Ne];
      case (0x0002): /* Type Blind RSA (2048-bit), RFC 9578 */
         uint8_t truncated_token_key_id;
         uint8_t blinded_msg[Nk];
   }
} IssuerTokenRequest;
~~~

The structure fields are defined as follows:

- "token_type" is a 2-octet integer. TokenRequest MUST be prefixed with a uint16
  "token_type" indicating the token type.

- The rest of the structure after "token_type" is based on that type, within the
inner opaque token_request attribute. The above definition corresponds to
TokenRequest from {{RFC9578}}. For TokenRequest not defined in {{RFC9578}}, they
MAY be used as long as they are prefixed with a 2-octet token_type.

The Attester will encode the IssuerTokenRequest in the chosen interchange format,
and send the IssuerTokenRequest to the issuer over the chosen connection.

## Issuer behavior

In the simplest case, the issuer will recieve the IssuerTokenRequest, sign the
message, and return it to the Attester. The signature may be defined by Sections 5
or 6 of {{!RFC9578}}, depending on the token_type used.

## Issuer-to-Attester Response {#response}

After signing the request, the issuer assembles and returns an
"IssuerTokenResponse" to the attester.  The response should be sent in the same
HTTPS connection as the request was delivered on.  Like the request, the
response below is written in TLS Presentation Language, but any data interchange
format is acceptable.

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
