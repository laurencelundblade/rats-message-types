---
v: 3

title: RATS Message Types
abbrev: RATS Message Types
docname: draft-fossati-rats-message-types
cat: std

date:
consensus: true
stream: IETF
ipr: trust200902
area:  "Security"
workgroup: RATS
keyword: cbor


author:
- ins: T. Fossati
  name: Thomas Fossati
  org: Linaro
  email: lgl@securitytheory.com

- ins: L. Lundblade
  name: Laurence Lundblade
  org: Security Theory LLC
  email: lgl@securitytheory.com

normative:
  RFC2119:

  I-D.ietf-rats-eat:
  
  RFC7252:

  RFC9277:

  RFC6838:

  I-D.ietf-rats-msg-wrap:

informative:
  STD94:

--- abstract

This establishes that RATs messages types should be identified by either CoAP Content ID or Media Type (MIME type) or both.
CBOR tags and other type mechanisms should not be used.

This updates EAT, in a backwards-compatible way, so that the type of EAT nested tokens can be identified by CoAP Content ID and Media Type.

--- middle

# Introduction

When EAT [I-D.ietf-rats-eat] and CMW [I-D.ietf-rats-msg-wrap] were designed, some of the nesting requirements were not fully understood.
What is understood now is that composite and layered attestation has to accommodate an open-ended range of formats for attestation evidence messages (a.k. a., attestation tokens).

When composing attestation for a complex device like a car or airplane, there are likely to be hundreds of components from hundreds of suppliers.
Individual suppliers are likely to have picked an attestation token format convenient for their environment.
For example, traditional security-oriented components like HSMs are likely to choose ASN.1 based formats while higher level components may use JSON-based formats.
In many cases, the components and the attestation implementation will be subject to expensive certification requirements.
It is not possible for the manufacturer of the complex device to dictate that all suppliers use the same format.

Encoding-specific message type indicators like CBOR tags (See [STD94]) are not recommended, because they work only in the particular protocol environments.

Proprietary type indicators are also not recommended as there are perfectly usable standard type indicators.
Standard type indicators have the benefit of working with established protocols like HTTP and CoAP [RFC7252].

It is expected that attestation, and thus attestation messages, will be integrated into other protocols like TLS.
Some uniform means of identifying RATS messages is desirable.
This is akin to identify image format types in a general way that works for email, web pages and so on.

A concern with multiple formats for RATS messages is incurring n-squared complexity where n is the number of formats for something like evidence.
Orienting around a standard content types will mostly reduce from n-squared to simple order of n.

It is advantageous to identify one point in RATS protocols where message-formats can change, say from CBOR to JSON or DER (ASN.1) to CBOR.
Since RATS messages can be many formats, the logical place to do that is at the message level.
This is how EAT does it.
The only place a message format boundary is crossed is for nested tokens.

(EAT also allows a message format boundary crossing for manifests and measurements, but technically they are not attestation messages.
Never the less, in EAT this boundary makes use of CoAP content IDs).


# Type Identifiers and Receptacles

A type system needs identifiers for each message that plugs-in and a receptacle structure for the place the message plugs-in.

## Type Identifiers

A CoAP Content ID and Media Type SHOULD be registered for each RATS protocol message definition.

A CBOR tag SHOULD NOT be registered for RATS protocol messages.
If a CBOR tag is needed, one can be derived by the TN() mechanism described in [RFC7252].

## Receptacle Definition

A receptacle is a place in a protocol where a RATS message can occur.
For example, a nested-token in EAT or a cbor-record in CMW.
In a sense, HTTP is largely a receptacle.

A receptacle is a content type and a body. cbor-record and json-record in CMW are receptacles.
The content type header and message body in HTTP are a receptacle.

All RATS protocols SHOULD use a receptacle that is based on CoAP content type or media type or both.

CBOR-based protocols SHOULD use the cbor-record defined in CMW as a receptacle.
JSON-based protocols should use the json-record defined in CMW as a receptacle.

This does not deprecate previously registered CBOR tags or CBOR tag-using receptacles.

## Receptacle Commentary

The actual definition and encoding for a receptacle should match the format and encoding of the surround protocol.
For example, in a CBOR-format CMW token, a cbor-record is used for the receptacle.
In an ASN.1-based protocol the receptacle will be defined in ASN.1.
It is perhaps an INTEGER and an OCTET-STREAM or an OID that maps to a CoAP Content ID and an OCTET-STREAM.

A receptacle is a place where encoding formats can change.
A JSON-format message can be put into a CBOR-format message.
It is recommended that in RATS protocols a receptacle be the only place where the encoding format can change. Stated conversely, any place in a RATS protocol were the encoding format can change, a receptacle should be used.

A typical receptacle implementation will decode the message type, and then switch or dispatch to a handler for that type.
This may be by a look up table or such. The important part is the implementation of the receptacle doesn’t have to be changed for each new type.
Only a configuration has to be adjusted.

Some protocols, like TLS, don’t have receptacles and need one to transport RATS messages.

Note that in EAT, both the manifests and measurements claims are defined are receptacles as described here.


# EAT Updates

The definition of a nested-token in EAT [I-D.ietf-rats-eat] doesn’t meet the above criterion, so it needs to be adjusted.
This doesn’t deprecate anything in EAT, just refines some definitions.

In CBOR-format EAT tokens, the type of a nested token is identified by a CBOR tag.
This document updates EAT to say the CBOR tag SHOULD use CoAP Content IDs [RFC7252] and tn() described in [RFC9277] to turn them into CBOR tags.

In JSON-format EAT tokens, the JSON-Selector structure is used to identify the type of a nested token.
The string identifier in JSON-Selector may be one of a small number of strings defined by the EAT standard and may be update only by a standards document.

This document updates the JSON-Selector as follows.
When the string in the JSON-selector is one of the top-level media types (e.g., “application”) the JSON-selector is a media type identifier.

--- back


