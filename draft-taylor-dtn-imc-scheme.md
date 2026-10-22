---
title: "The 'imc' URI Scheme for Bundle Protocol Multicast"
abbrev: "imc URI Scheme"
category: std

docname: draft-taylor-dtn-imc-scheme-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Delay/Disruption Tolerant Networking"
keyword:
 - DTN
 - Bundle Protocol
 - BPv7
 - multicast
 - URI scheme
venue:
  group: "Delay/Disruption Tolerant Networking"
  type: "Working Group"
  mail: "dtn@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dtn/"
  github: "ricktaylor/imc-scheme"
  latest: "https://ricktaylor.github.io/imc-scheme/draft-taylor-dtn-imc-scheme.html"

author:
 -
    fullname: Rick Taylor
    organization: Aalyria Technologies
    email: rtaylor@aalyria.com

normative:
  RFC3986:
  RFC5234:
  RFC7595:
  RFC8126:
  RFC8610:
  RFC8949:
  RFC9171:
  RFC9758:
  I-D.ietf-dtn-eid-pattern:

informative:
  RFC9172:
  RFC9174:
  I-D.burleigh-dtn-multicast:
  I-D.burleigh-dtnrg-imc:
  I-D.cavallini-dtn-iac:
  I-D.ietf-dtn-bibe:

--- abstract

This document defines the 'imc' URI scheme, a concise integer-based scheme
for identifying Bundle Protocol Version 7 (BPv7) endpoints that are not
singleton endpoints, termed multicast endpoints. The structure of 'imc' URIs
mirrors that of the 'ipn' URI scheme: each 'imc' URI comprises an Allocator
Identifier, a Group Number, and a Service Number. This document specifies the
textual syntax of 'imc' URIs, their interpretation when used as BPv7 Endpoint
Identifiers, their CBOR encoding, and the corresponding EID Pattern, and
requests the associated IANA registrations.


--- middle

# Introduction {#intro}

The Bundle Protocol Version 7 (BPv7) {{RFC9171}} identifies the destination
of every bundle by an Endpoint Identifier (EID), which is a URI. A bundle
endpoint is a set of zero or more bundle nodes, the members of which are
determined by their registrations in it. Although BPv7 permits endpoints with any
number of members, the 'ipn' URI scheme {{RFC9758}}, the scheme most commonly
used in deployed BPv7 networks, only identifies singleton endpoints. There is
no equivalently concise scheme for identifying endpoints that may have many
members.

This document defines the 'imc' URI scheme for this purpose. An 'imc' URI
identifies an endpoint that is never a singleton endpoint; such an endpoint
is termed a "multicast endpoint". The name 'imc' is derived from
"InterPlanetary MultiCast", in the same way that 'ipn' was derived from
"InterPlanetary Network", but like 'ipn' the term is now used as a
non-acronymous name.

The structure of an imc URI mirrors that of an 'ipn' URI, with a Group Number
in place of the 'ipn' Node Number. However, whereas a Node Number identifies a
node, a Group Number identifies neither a node nor a set of nodes. As
throughout BPv7, it is endpoints that are addressed, and nodes are members of
endpoints: each imc URI identifies a single multicast
endpoint, with its own membership, and the Group Number names a group of
related multicast endpoints.

The 'imc' scheme was first proposed in {{I-D.burleigh-dtnrg-imc}} for use
with Compressed Bundle Header Encoding and the experimental Bundle Protocol
version 6, and was subsequently specified for BPv7 as part of the Bundle
Multicast protocol {{I-D.burleigh-dtn-multicast}}. This document separates the
definition of the 'imc' scheme from that protocol, so that the scheme can be
standardized independently of any particular multicast forwarding mechanism,
and aligns its structure, terminology, and encodings with the 'ipn' URI scheme
as updated by {{RFC9758}}.

Like the 'ipn' URI scheme, the 'imc' URI scheme defines identifiers only. An
'imc' URI does not carry any topological information describing how to route
bundles to the members of the identified multicast endpoint. The procedures by
which bundles destined for multicast endpoints are forwarded, and by which
information about membership is distributed, are out of scope for this
document; one such set of procedures is defined in
{{I-D.burleigh-dtn-multicast}}.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

For the remainder of this document, the term "imc URI" is used to refer to a
URI that uses the 'imc' URI scheme, and the term "imc EID" is used to refer to
an imc URI that is used as a BPv7 EID.

The term "multicast endpoint" refers to the BPv7 endpoint identified by an imc
EID. The members of a multicast endpoint are the bundle nodes that have a
registration ({{Section 3.1 of RFC9171}}) in that imc EID ({{membership}});
a multicast endpoint is therefore never a
singleton endpoint ({{Section 3.1 of RFC9171}}).

The term "group" refers to the set of all multicast endpoints whose imc EIDs
share the same Allocator Identifier and Group Number, differing only in their
Service Numbers ({{group-numbers}}). A group has no members of its own.

The terms "Allocator", "Allocator Identifier", "Default Allocator", "Node
Number", and "Service Number" are used as defined in {{RFC9758}}.


# Core Concepts {#core}

Every imc URI, no matter whether it is expressed with a textual
representation or a binary encoding, MUST be considered as a tuple of the
following three components:

* The Allocator Identifier

* The Group Number

* The Service Number

The Allocator Identifier indicates the entity responsible for assigning Group
Numbers, maintaining uniqueness whilst avoiding the need for a single registry
for all assigned Group Numbers. See {{allocators}}.

The Group Number names a group of multicast endpoints in the context of an
Allocator. See {{group-numbers}}.

The Service Number distinguishes between the multicast endpoints of a group.
See {{service-numbers}}.

The combination of all three components identifies a single multicast
endpoint. The combination of the Allocator Identifier and the Group Number
uniquely identifies a group. See {{group-numbers}}.

Membership is a property of each multicast endpoint, not of a group: the
multicast endpoints of a group can have entirely different members. See
{{membership}}.


## Allocator Identifiers {#allocators}

The Allocator Identifier of an imc URI has the same structure, purpose, and
range as the Allocator Identifier of an 'ipn' URI defined in
{{Section 3.2 of RFC9758}}, and is drawn from the same "'ipn' Scheme URI
Allocator Identifiers" registry. An Allocator Identifier is an unsigned
integer in the range 0 to 2^32-1.

An Allocator that has been assigned an Allocator Identifier is authorized to
assign Group Numbers for use with the 'imc' URI scheme, in addition to Node
Numbers for use with the 'ipn' URI scheme. Each Allocator assigns Group
Numbers according to its own policies. Other than ensuring that any Group
Numbers it assigns are unique amongst all Group Numbers it assigns, an
Allocator does not need to coordinate its assignments with other Allocators.

Group Numbers and Node Numbers are distinct namespaces. An Allocator may
assign the same numeric value as both a Group Number and a Node Number, and
there is no relationship between a group and a node that happen to share the
same Allocator Identifier and number.

In any case where an encoded imc URI does not explicitly include an Allocator
Identifier, an implementation MUST assume that the Group Number has been
assigned by the Default Allocator ({{Section 3.2.2 of RFC9758}}).

A new IANA registry, "'imc' Scheme URI Default Allocator Group Numbers", is
defined to control the assignment of Group Number values by the Default
Allocator; see {{iana-groups}}. This registry reserves some values as defined
in {{special-groups}} below.

If a system does not require interoperable deployment of 'imc' scheme URIs,
then the Private Use Group Numbers ({{private-groups}}), reserved by the
Default Allocator for this purpose, are to be used.


## Group Numbers {#group-numbers}

A Group Number names a group in the context of an Allocator. A Group Number is
an unsigned integer. A single Group Number assigned by a single Allocator MUST
refer to a single group.

A group is the set of all multicast endpoints whose imc EIDs share an
Allocator Identifier and Group Number, distinguished by their Service Numbers,
in the same way that an 'ipn' FQNN names a node whose services are
distinguished by their Service Numbers. An Allocator might assign a Group
Number to a community of interest, a mission, or a function, with the
multicast endpoints of the group identifying the services of that community.
A Group Number does not identify any node or set of nodes, and assigning it
does not make any node a member of any multicast endpoint.

All Group Number assignments, by all Allocators, MUST be in the range 0 to
2^32-1.

It is RECOMMENDED that Group Number zero (0) not be assigned by an Allocator,
to avoid confusion with the Null imc URI ({{null-imc}}).

Unlike the Fully Qualified Node Number of an 'ipn' URI
({{Section 3.3.1 of RFC9758}}), this document defines no representation of an
Allocator Identifier and Group Number packed together as a single integer, in
either the text or the CBOR representation of imc URIs.


## Special Group Numbers {#special-groups}

Some special-case Group Numbers are defined by the Default Allocator; see
{{iana-groups}}.


### The Zero Group Number {#zero-group}

The Default Allocator assigns the use of Group Number zero (0) solely for
identifying the Null imc URI ({{null-imc}}).

This means that any imc URI with a zero (0) Allocator Identifier and a zero
(0) Group Number, but a non-zero Service Number component, is invalid. Such
imc URIs MUST NOT be composed, and processors of such imc URIs MUST consider
them as the Null imc URI.

[^grp0]

[^grp0]: Appendix A of {{I-D.burleigh-dtn-multicast}} suggests that imc:0.0
    could identify an implicit multicast group of which every IMC-capable node
    is a member, to be used for distributing group membership information,
    and ION uses imc:0.0 in this way today. This document instead defines
    imc:0.0 as the null multicast endpoint, which never has any members (see
    {{null-group}} and {{sec-implicit}}), so that usage is not interoperable
    with this specification.


### The Maximum Group Number {#max-group}

The Default Allocator reserves Group Number 2^32-1 (0xFFFFFFFF) as invalid.
This Group Number MUST NOT be assigned, and any imc URI with a zero (0)
Allocator Identifier and a Group Number of 2^32-1 is invalid and MUST NOT be
composed.

This reservation exists solely to guard against confusion with LocalNode ipn
URIs ({{Section 3.4.2 of RFC9758}}), which use the same Node Number. For
example, if this Group Number were assignable, the mistyped URI
imc:4294967295.7, intended as the LocalNode ipn URI ipn:4294967295.7, could
silently identify a real multicast endpoint. Reserving it ensures that such an
error causes the bundle to be discarded rather than delivered to unintended
recipients.

The reservation does not give imc URIs any LocalNode semantics, and the
'imc' URI scheme has no equivalent of LocalNode ipn URIs. Services do not
receive bundles; they receive the application data units (ADUs) of bundles
delivered to them by their local bundle protocol agent
({{Section 5.7 of RFC9171}}). An imc URI identifying "the local services"
would therefore not identify anything to which a bundle could meaningfully be
sent.

A bundle node that receives or originates a bundle whose destination is an
imc EID with a zero (0) Allocator Identifier and a Group Number of 2^32-1
MUST NOT forward or deliver it, and MUST delete it
({{Section 5.10 of RFC9171}}). If a bundle deletion status report is
requested, the reason code "Destination endpoint ID unavailable"
({{Section 6.1.1 of RFC9171}}) SHOULD be used.

This reservation applies only to the Default Allocator. Other Allocators MAY
assign Group Number 2^32-1 according to their own policies.


### Private Use Group Numbers {#private-groups}

The Default Allocator reserves several ranges of Group Numbers for Private
Use, as defined in {{RFC8126}}. These ranges are interleaved with ranges of
registered Group Numbers, so that Private Use Group Numbers are available at
each length of CBOR encoding; see {{iana-groups}}.

Any imc URI with a zero (0) Allocator Identifier and a Group Number reserved
for Private Use is not guaranteed to be unique beyond a single administrative
domain. An administrative domain, as used here, is defined as the set of
nodes that share a unique allocation of Private Use Group Numbers.

Because of this lack of uniqueness, any implementation of a protocol using
imc URIs that resides on the border between administrative domains MUST have
suitable mechanisms in place to prevent protocol units using such Private Use
Group Numbers from crossing between different administrative domains.


## Service Numbers {#service-numbers}

A Service Number is an unsigned integer, as defined in
{{Section 3.5 of RFC9758}}. In an imc URI, the Service Number distinguishes
the multicast endpoints of a group, and so identifies the service to which a
bundle destined for that multicast endpoint is delivered at each of its
members.


## The Null imc URI {#null-imc}

There is value in defining a unique imc URI that identifies a multicast
endpoint that can never have any members. This imc URI is termed the "Null
imc URI" and has all three components (the Allocator Identifier, Group
Number, and Service Number) set to the value zero (0). The multicast endpoint
identified by the Null imc URI is termed the "null multicast endpoint". No
node is ever a member of the null multicast endpoint, and any destination
identified by the Null imc URI is therefore by definition unreachable.

In particular, the Null imc URI MUST NOT be interpreted as identifying a
multicast endpoint of which all nodes, or all nodes supporting the 'imc'
scheme, are implicitly members. See {{sec-implicit}}.


# Textual Representation of imc URIs {#text}

All 'imc' scheme URIs comply with {{RFC3986}} and are therefore represented
by a scheme identifier and a scheme-specific part. The scheme identifier is
imc, and the scheme-specific parts are represented as a sequence of numeric
components separated with the '.' character. A formal definition is provided
below (see {{text-syntax}}) and can be informally considered as:

~~~
imc:[<allocator-identifier>.]<group-number>.<service-number>
~~~

To keep the text representation concise, the following rules apply:

* All leading '0' characters MUST be omitted. A single '0' is valid.

* If the Allocator Identifier is zero (0), then the
  &lt;allocator-identifier&gt; and '.' MAY be omitted.

Examples of the textual representation of imc URIs can be found in
{{text-examples}}.


## 'imc' URI Scheme Text Syntax {#text-syntax}

The text syntax of an imc URI MUST comply with the following ABNF syntax from
{{RFC5234}} and repeats the core ABNF syntax rule for DIGIT defined by that
specification:

~~~ abnf
imc-uri = "imc:" imc-hier-part

imc-hier-part = group-part "." service-number

group-part = [allocator-identifier "."] group-number

allocator-identifier = number

group-number = number

service-number = number

number = "0" / non-zero-number

non-zero-number = (%x31-39 *DIGIT)

DIGIT = %x30-39
~~~

None of the reserved characters defined in the generic URI syntax are used as
delimiters within URIs of the 'imc' scheme, other than '.'.

[^text-single]

[^text-single]: The ABNF in {{I-D.burleigh-dtn-multicast}} names the
    single-number form of the group label "fqgn", implying that it carries
    a packed 64-bit Allocator Identifier and Group Number. This document
    instead follows Section 4 of {{RFC9758}}: the single-number form is a
    Group Number assigned by the Default Allocator, and no packed form is
    defined ({{group-numbers}}).

Unlike the 'ipn' URI scheme, the 'imc' URI scheme has no '!' form, because
it has no equivalent of LocalNode ipn URIs; see {{max-group}}.


# Usage of imc URIs with BPv7 {#bpv7}

This section identifies the behavior and interpretation of 'imc' scheme URIs
that MUST be followed when using this URI scheme to represent EIDs in BPv7.

This document does not define the use of imc URIs with the experimental
Bundle Protocol version 6.


## Multicast Endpoint Membership {#membership}

The members of a multicast endpoint are those bundle nodes that have a
registration ({{Section 3.1 of RFC9171}}) in its imc EID. A single bundle
node may be a member of any number of multicast endpoints.

This document does not define any mechanism by which a node joins or leaves
a multicast endpoint. As for any BPv7 endpoint, the means by which a
registration is created or removed is an implementation matter
({{Section 3.3 of RFC9171}}), and may for example be a local application
interface, configuration, or a separately specified protocol.

Membership is determined separately for each imc EID. Being a member of one
multicast endpoint of a group does not make a node a member of any other
multicast endpoint of that group: for example, the members of imc:977000.17.1
and of imc:977000.17.2 are independent sets of nodes, either of which may be
empty. A bundle destined for an imc EID is delivered only at members of that
multicast endpoint.

Forwarding implementations may nevertheless aggregate state, such as
distribution trees or membership information, across the multicast endpoints
of a group, or across any other set of multicast endpoints, for example one
described by an EID Pattern ({{patterns}}). Such aggregation is an
implementation choice that does not alter which nodes are members of each
multicast endpoint, and does not imply that Group Numbers carry topological
information ({{late-binding}}).

As noted in {{Section 3.1 of RFC9171}}, registrations are local, and BPv7 does
not include a mechanism for distributing information about registrations.
Mechanisms by which information about the membership of multicast endpoints
is distributed are out of scope for this document.


## The Null Multicast Endpoint {#null-group}

When the Null imc URI ({{null-imc}}) is used as a BPv7 EID, it is termed the
"Null imc EID", and the endpoint it identifies is the null multicast endpoint.

The null multicast endpoint never has any members: a bundle node MUST NOT
hold a registration in the Null imc EID, by whatever means registrations are
made.

Because the null multicast endpoint has no members, a bundle destined for the
Null imc EID can never be delivered. Such a bundle SHOULD NOT be created, and
a bundle node that receives or originates such a bundle MUST NOT forward it to
any other node. If such a bundle is deleted and a bundle deletion status
report is requested, the reason code "Destination endpoint ID unavailable"
({{Section 6.1.1 of RFC9171}}) SHOULD be used.

The null multicast endpoint is distinct from the BPv7 null endpoint ("dtn:none",
{{Section 4.2.5.1.1 of RFC9171}}). Although both identify endpoints that never
have any members, the Null imc EID MUST NOT be used in place of the null
endpoint ID, for example to indicate an anonymous bundle source or that no
status reports are to be sent. Equally, the null endpoint ID and the Null ipn
EID ({{Section 5.2 of RFC9758}}) are not imc EIDs.


## Node IDs and Bundle Sources {#node-ids}

{{Section 4.2.5.2 of RFC9171}} requires that a Node ID be the EID of a
singleton endpoint. Because a multicast endpoint is never a singleton
endpoint, an imc EID MUST NOT be used as a Node ID.

As a consequence, an imc EID MUST NOT be used as the source node ID of a
bundle ({{Section 5.2 of RFC9171}}), as the node ID carried in a Previous Node
Block ({{Section 4.4.1 of RFC9171}}), or as the security source of a Bundle
Protocol Security (BPSec) {{RFC9172}} security block.


## Private Use imc EIDs {#private-eids}

Bundles destined for imc EIDs with a zero (0) Allocator Identifier and a
Group Number within the Private Use ranges of the Default Allocator
({{private-groups}}) are
not universally unique; therefore, they are only valid within the scope of the
current administrative domain. This means that any bundle using a Private Use
imc EID as a bundle destination MUST NOT be allowed to cross administrative
domains. All implementations that could be deployed as a gateway between
administrative domains MUST be sufficiently configurable to ensure that this
is enforced, and operators MUST ensure correct configuration.

Private Use imc EIDs MUST NOT be present in any other part of a bundle that is
destined for another administrative domain when the lack of uniqueness
prevents correct operation.


## Administrative Multicast Endpoints {#admin}

The administrative endpoint of a node is a singleton endpoint, and its EID can
serve as the node's Node ID ({{Section 3.2 of RFC9171}}). No imc EID
identifies the administrative endpoint of any node.

However, consistent with the meaning of Service Number zero (0) in 'ipn' EIDs
({{Section 5.7 of RFC9758}}), the multicast endpoint of a group with Service
Number zero (0), for example imc:977000.17.0, is the "administrative
multicast endpoint" of that group. Bundles destined for an administrative
multicast endpoint carry administrative records ({{Section 6.1 of RFC9171}}),
and are delivered at each member to the administrative element of its
application agent ({{Section 3.1 of RFC9171}}), rather than to any
application-specific element.

As for every multicast endpoint, the members of an administrative multicast
endpoint are the nodes registered in its imc EID ({{membership}}), and its
membership is independent of that of the other multicast endpoints of the
group. A node is not
implicitly a member of the administrative multicast endpoint of any group,
including a group of whose other multicast endpoints it is a member.

The "ADU is an administrative record" bundle processing control flag
({{Section 4.2.3 of RFC9171}}) MUST be set in every bundle destined for an
administrative multicast endpoint, and a bundle destined for an
administrative multicast endpoint without that flag set MUST NOT be delivered.
Conversely, a bundle with that flag set MUST NOT be destined for an imc EID
whose Service Number is not zero (0), consistent with the rule for 'ipn' EIDs
in {{Section 5.7 of RFC9758}}.

The receiver of an administrative record delivered from an administrative
multicast endpoint MUST be prepared for the same record to be delivered to
many nodes. A specification of an administrative record type that is intended
to be sent to administrative multicast endpoints SHOULD state so, and
administrative record types not so specified SHOULD NOT be sent to them.
This document specifies that bundle status reports
({{Section 6.1.1 of RFC9171}}) MAY be sent to an administrative multicast
endpoint: the report-to EID of a bundle MAY be an administrative multicast
endpoint, so that a set of monitoring nodes all receive its status reports.

Because an administrative multicast endpoint is not the administrative
endpoint of any node, it cannot be used as a Node ID ({{node-ids}}).

[^admin-mc]

[^admin-mc]: Appendix A of {{I-D.burleigh-dtn-multicast}} uses imc:0.0 to
    convey administrative information about membership. Under this
    document, that function would use the administrative multicast endpoint
    of an explicitly assigned group, whose membership is determined by the
    deployment, rather than the null multicast endpoint ({{null-group}}, {{sec-implicit}}).


## Well-Known Service Numbers {#well-known}

The Service Number of an imc EID is interpreted in the same way as the
Service Number of an 'ipn' EID. In particular, the well-known Service Numbers
registered in the "'ipn' Scheme URI Well-Known Service Numbers for BPv7"
registry ({{Section 5.6 of RFC9758}}) identify the same services when they
appear in imc EIDs. Service Number zero (0), registered there as the
Administrative Endpoint, identifies the administrative multicast endpoint of
a group ({{admin}}).

[^svc-reg]

[^svc-reg]: {{I-D.cavallini-dtn-iac}} requests that the registries shared by
    'ipn' and 'iac' be renamed. The WG needs to coordinate how these shared
    registries are named and referenced across 'ipn', 'imc', and 'iac'.


# CBOR Representation of imc URIs with BPv7 {#cbor}

{{Section 4.2.5.1 of RFC9171}} requires that any URI scheme used to represent
BPv7 EIDs MUST define how the scheme-specific part of the URI scheme is
encoded with Concise Binary Object Representation (CBOR) {{RFC8949}}. To meet
this requirement, this section describes the CBOR encoding of imc EIDs. The
formal definition of the CBOR representation is specified in
{{cbor-syntax}}.

The examples in this section use the value 3 as the BPv7 URI scheme type of
the 'imc' scheme. This value is a placeholder pending assignment by IANA; see
{{iana-bp}}.


## imc EID CBOR Encoding {#cbor-encoding}

When encoding imc EIDs for use with BPv7, the scheme-specific part of an imc
URI MUST be represented as a CBOR array of exactly three (3) elements,
regardless of the Allocator Identifier. Each element of the array MUST be
encoded as a single CBOR unsigned integer, and:

* the first element of the array is the Allocator Identifier,

* the second element of the array is the Group Number, and

* the third element of the array is the Service Number.

This encoding corresponds to the three-element scheme-specific encoding of
'ipn' EIDs ({{Section 6.1.2 of RFC9758}}). There is no equivalent of the
two-element scheme-specific encoding of 'ipn' EIDs
({{Section 6.1.1 of RFC9758}}): the Allocator Identifier is always encoded,
including when it is zero (0), and no element packs the Allocator Identifier
and Group Number into a single integer.

For example, the imc EID of imc:977000.17.1 would result in the three-element
array of \[977000,17,1\], which would be encoded in CBOR as the following
10-octet sequence:

~~~ cbor-pretty
82                # 2-Element Endpoint Encoding
   03             # uri-code: 3 ('imc' URI scheme)
   83             # 3-Element imc EID encoding
      1A 000EE868 # Allocator Identifier
      11          # Group Number
      01          # Service Number
~~~

Further examples are provided in {{cbor-examples}}.

An imc EID whose scheme-specific part is not a CBOR array of exactly three
(3) unsigned integers, or whose Allocator Identifier or Group Number is not
less than 2^32, is malformed. In particular, a two-element array is
malformed. A bundle containing a malformed imc EID in its primary block MUST
be treated as malformed.

[^no-2el]

[^no-2el]: {{I-D.burleigh-dtn-multicast}} defines a two-element encoding
    whose first element packs the Allocator Identifier and Group Number
    into a single 64-bit integer, mirroring the 'ipn' scheme, and ION
    implements it. The 'ipn' two-element encoding was retained in RFC 9758
    only for backwards compatibility with ION, and CCSDS is already
    deprecating it, so this document does not carry it into the new scheme,
    in either its packed form or an unpacked form for the Default Allocator.
    The cost is one additional octet for imc EIDs using the Default
    Allocator, in exchange for each imc EID having a single encoding.
    Implementations that emit the two-element form will not interoperate
    with this specification.


## 'imc' URI Scheme CBOR Syntax {#cbor-syntax}

When encoded in CBOR {{RFC8949}}, a BPv7 endpoint identified by an imc URI
MUST comply with the following Concise Data Definition Language (CDDL)
{{RFC8610}} specification:

~~~ cddl
eid = $eid .within eid-structure

eid-structure = [
  uri-code: uint,
  SSP: any
]

; ... Syntax for other uri-code values defined in RFC 9171 ...

$eid /= [
  uri-code: 3,  ; placeholder pending IANA assignment (TBD1)
  SSP: imc-ssp
]
imc-ssp = [
  allocator-identifier: uint .lt 4294967296,
  group-number: uint .lt 4294967296,
  service-number: uint
]
~~~


## imc EID Matching {#matching}

imc EID matching MUST be performed on the decoded EID information itself:
two imc EIDs are equal if and only if their Allocator Identifiers, Group
Numbers, and Service Numbers are equal.

Because there is a single scheme-specific encoding, and {{Section 4.1 of
RFC9171}} requires the core deterministic encoding of {{RFC8949}} for bundle
blocks, each imc EID has exactly one CBOR representation. Unlike 'ipn' EIDs
({{Section 6.4 of RFC9758}}), no imc EID can be represented by different
encodings that need to be treated as equivalent.


# Special Considerations {#special}

## Relationship to Other URI Schemes {#other-schemes}

Although imc URIs share their structure, Allocator Identifiers, and Service
Numbers with 'ipn' URIs, the two schemes identify different kinds of
endpoint. An imc EID and an 'ipn' EID with identical numeric components, such
as imc:977000.17.1 and ipn:977000.17.1, identify unrelated endpoints.

[^iac]

[^iac]: {{I-D.cavallini-dtn-iac}} proposes that Group Numbers have the same
    meaning in the 'iac' anycast scheme and in multi-destination schemes such
    as 'imc', so that one Group Number can be used with either scheme.
    The WG needs to decide whether to adopt that alignment.


## Late Binding {#late-binding}

{{RFC9171}} mandates the concept of the "late binding" of an EID, whereby the
address of the destination of a bundle is resolved from its identifier
hop-by-hop as it transits a BPv7 network. The concept of late binding is
preserved in the 'imc' URI scheme. Elements of an imc URI MUST NOT be
regarded as carrying information relating to the location or reachability of
the members of a multicast endpoint, or other addressing or routing concerns.


# imc EID Patterns {#patterns}

{{I-D.ietf-dtn-eid-pattern}} defines EID Patterns, a means of matching sets
of BPv7 EIDs, and requires that each URI scheme supporting patterns define a
scheme-specific pattern item, comprising a logical information model, EID
matching logic, pattern set logic, and both text and CBOR forms
({{Section 1.1 of I-D.ietf-dtn-eid-pattern}}). This section defines the
scheme-specific pattern item for the 'imc' URI scheme, termed the "imc
pattern".

Because the information model of an imc URI is identical to that of an 'ipn'
URI, being three unsigned integer elements with the same domains, the imc
pattern is defined by reference to the IPN Scheme Pattern Item defined in
{{Section 2.4 of I-D.ietf-dtn-eid-pattern}}, termed the "IPN pattern". Except
as specified in this section, every requirement of that section and its
subsections applies to the imc pattern, with "imc" substituted for "IPN" or
"ipn".


## Information Model {#pattern-model}

An imc pattern SHALL logically contain exactly three elements corresponding
to the elements of an imc EID:

* Allocator Identifier, with domain 0 to 2^32-1 inclusive

* Group Number, with domain 0 to 2^32-1 inclusive

* Service Number, with domain 0 to 2^64-1 inclusive

Each element is matched as a specific value, a range, or a wildcard, and is
subject to the same constraints on ranges, normalization, decoding failure,
and minimum supported number of intervals as the corresponding element of an
IPN pattern ({{Section 2.4 of I-D.ietf-dtn-eid-pattern}}), with the Group
Number taking the place of the Node Number.


## EID Matching and Pattern Set Logic {#pattern-matching}

An imc pattern matches an imc EID according to the rules of
{{Section 2.4.1 of I-D.ietf-dtn-eid-pattern}}, applied to the decoded
Allocator Identifier, Group Number, and Service Number of the EID. An imc
pattern never matches an EID of any other scheme; in particular, an imc
pattern and an IPN pattern with identical elements match disjoint sets of
EIDs ({{other-schemes}}).

Set logic between imc patterns is as defined for IPN patterns in
{{Section 2.4.2 of I-D.ietf-dtn-eid-pattern}}.


## Text Form {#pattern-text}

The text form of the imc pattern is identical to the three-element text form
of the IPN pattern defined in {{Section 2.4.3 of I-D.ietf-dtn-eid-pattern}},
except that the scheme name is "imc" rather than "ipn", and that there is no
two-element form containing a packed value or the '!' character. It conforms
to the following ABNF, which reuses the rules ipn-part-pat, ipn-decimal, and
nbr-delim (and the rules on which they depend) from Figure 1 of
{{I-D.ietf-dtn-eid-pattern}}:

~~~ abnf
eid-pattern-item =/ imc-pat-item
imc-pat-item = "imc:" (imc-pat-ssp3 / imc-pat-ssp2)

; Full pattern for three elements
imc-pat-ssp3 = ipn-part-pat
               nbr-delim ipn-part-pat
               nbr-delim ipn-part-pat

; Accept a single EID in two-element text form
imc-pat-ssp2 = ipn-decimal nbr-delim ipn-decimal
~~~

When decoding a two-element imc pattern, the first element SHALL be treated
as a Group Number assigned by the Default Allocator, and the pattern SHALL be
treated as the three-element pattern whose Allocator Identifier element is
the specific value zero (0), consistent with the two-element text form of an
imc URI ({{text}}). Unlike a two-element IPN pattern, the first element is not
a packed value, and has the domain of a Group Number.

As for the IPN pattern, the canonical text form of an imc pattern SHALL use
three elements, and every imc URI is itself a valid imc pattern that matches
only that imc URI, satisfying {{Section 2.1.1 of I-D.ietf-dtn-eid-pattern}}.

For example, the following imc pattern matches every Service Number of the
group with Group Number 17 assigned by Allocator 977000:

~~~
imc:977000.17.*
~~~

The following imc pattern matches Service Number 1 of every group
assigned by the Default Allocator from the first Private Use range
({{iana-groups}}):

~~~
imc:0.[1-127].1
~~~


## CBOR Form {#pattern-cbor}

The CBOR form of the imc pattern is identical to the CBOR form of the IPN
pattern defined in {{Section 2.4.4 of I-D.ietf-dtn-eid-pattern}}, including
the encoding of specific values, ranges, and wildcards, except that the scheme
number is the 'imc' URI scheme type ({{iana-bp}}) rather than 2. It conforms
to the following CDDL, which reuses the rule ipn-part-pat (and the rules on
which it depends) from Figure 2 of {{I-D.ietf-dtn-eid-pattern}}:

~~~ cddl
$eid-pat-item /= [
  scheme-num: 3,  ; placeholder pending IANA assignment (TBD1)
  SSP: imc-pat-ssp
]
imc-pat-ssp = [
  ; This CDDL representation does not restrict to the
  ; true domain of each element.
  3*3 ipn-part-pat,
]
~~~

As for the IPN pattern, the CBOR form always has three elements; there is no
two-element CBOR form of an imc pattern.

For example, the imc pattern imc:977000.17.\* would be encoded in CBOR as the
following 11-octet sequence, using the placeholder value 3 for the 'imc' URI
scheme type:

~~~ cbor-pretty
81                   # EID Pattern with one item
   82                # imc pattern item
      03             # scheme-num: 3 ('imc' URI scheme)
      83             # 3-Element imc pattern SSP
         1A 000EE868 # Allocator Identifier
         11          # Group Number
         F5          # Service Number: wildcard (true)
~~~

[^pattern-dep]

[^pattern-dep]: This section depends normatively on
    {{I-D.ietf-dtn-eid-pattern}}. If the WG prefers not to create that
    dependency, this section could instead be moved to that document, or
    to a separate document.


# Security Considerations {#security}

This section is derived from the security considerations for the 'imc' scheme
in {{I-D.burleigh-dtn-multicast}}, and follows the structure of
{{Section 8 of RFC9758}}.


## Reliability and Consistency

None of the BPv7 endpoints identified by imc EIDs are guaranteed to be
reachable at any time, and the identity of the processing entities operating
on those endpoints is never guaranteed by the Bundle Protocol itself.
Verification of the signature provided by the Block Integrity Block (BIB)
targeting the bundle's primary block, as defined by BPSec {{RFC9172}}, is
required for this purpose.


## Malicious Construction

Malicious construction of a conformant imc URI is limited to the malicious
selection of Allocator Identifiers, Group Numbers, and Service Numbers. That
is, a maliciously constructed imc EID could be used to direct a bundle to a
multicast endpoint whose members might be damaged by the arrival of that bundle.
A bundle that asserts an imc EID as its source node ID is not valid
({{node-ids}}). In all cases (and indeed in all bundle processing), the node
that receives a bundle should verify its authenticity and validity before
operating on it in any way.


## Unauthorized Membership

Membership of a multicast endpoint is determined by local registrations
({{membership}}), and the 'imc' URI scheme itself provides no means of
restricting which nodes may become members of a multicast endpoint. Bundles destined for a multicast endpoint may therefore
be delivered to nodes that are not intended recipients. Where this is a
concern, the confidentiality of bundle contents should be protected using BPSec
{{RFC9172}}, and multicast protocols should authenticate membership
information before acting on it.


## Implicit Membership {#sec-implicit}

This document does not define any multicast endpoint of which all nodes, or all
nodes supporting the 'imc' scheme, are implicitly members. In particular, the
Null imc URI ({{null-imc}}), which earlier specifications suggested for such
a purpose, instead identifies the null multicast endpoint, which never has any
members ({{null-group}}).

Such an endpoint would allow any node able to inject a single bundle into a
BPv7 network to cause that bundle to be replicated to, and delivered at, every
node in the network. This is an amplification mechanism that could be used
to exhaust transmission opportunities, storage, and power across the whole
network, which is a particularly severe threat in networks with scarce or
scheduled connectivity. Because membership would be implicit rather than
determined by local registrations, no node could decline to receive such
bundles, and the scope of the endpoint could not be confined to a single
administrative domain. Mechanisms that require communication with many nodes,
such as the distribution of membership information, should instead use
multicast endpoints of explicitly assigned groups, whose membership and
origination can be controlled by the deployment.


## Status Report Implosion {#sec-reports}

{{Section 6.1.1 of RFC9171}} notes that requesting status reports for a
single unicast bundle can result in (1 + (2 * (N-1))) status report bundles,
where N is the number of nodes on the path to its destination, and for that
reason requires that the generation of status reports be disabled by default
and enabled only when the risk of excessive network traffic is deemed
acceptable. {{Section 8 of RFC9171}} further notes that malicious initiation
of status reporting could be used to effect a denial-of-service attack.

These risks are multiplied for bundles destined for multicast endpoints. A
single bundle destined for an imc EID may be replicated along many paths and
delivered at every member of the multicast endpoint, and each node that
receives, forwards, delivers, or deletes a copy may generate a status report.
The number of status reports generated by a single bundle can therefore grow
with the number of members of the multicast endpoint as well as with the
length of each path, and all of those reports converge on the single
report-to endpoint, which may be overwhelmed. A bundle destined for a
multicast endpoint that requests status reports is also an amplification
mechanism, since a single bundle sent by an attacker can cause a much larger
number of bundles to be sent to a victim named as its report-to endpoint.

Implementations of the 'imc' scheme MUST retain the default of
{{Section 6.1.1 of RFC9171}} that status report generation is disabled, and
SHOULD NOT generate status reports in response to bundles destined for imc
EIDs unless explicitly configured to do so for the multicast endpoints
concerned. Where status reporting for bundles destined for multicast
endpoints is enabled, operators should take into account the number of
members and the capacity of the report-to endpoint. Naming an administrative
multicast endpoint ({{admin}}) as the report-to endpoint distributes the
reports to a set of monitoring nodes, but does not reduce their number, and
further multiplies the resulting traffic.

Integrity protection of the primary block using BPSec {{RFC9172}} allows a
node to verify that status reports were genuinely requested by the source of
a bundle.


## Private Use imc EIDs {#sec-private}

Private Use ({{private-groups}}) imc URIs present a risk to the stability of
deployed BPv7 networks. If such imc URIs are allowed to propagate beyond the
domain in which they are valid, then the required uniqueness of imc URIs no
longer holds, and bundles may be delivered to unintended multicast endpoints in
other administrative domains.

See {{private-eids}} for required behaviors to mitigate against this form of
abuse.


## Back-End Transcoding

The limited expressiveness of URIs of the 'imc' scheme effectively eliminates
the possibility of threats due to errors in back-end transcoding.


## Rare IP Address Formats

Not relevant, as IP addresses do not appear anywhere in conformant imc URIs.


## Sensitive Information

Because imc URIs are used only to represent the numeric identities of
multicast endpoints, the risk of disclosure of sensitive information due to
interception of these URIs is minimal. Examination of imc URIs could be used
to support traffic analysis; where traffic analysis is a plausible danger,
bundles should be conveyed by bundle-in-bundle encapsulation
{{I-D.ietf-dtn-bibe}} or by secure convergence-layer protocols that do not
expose endpoint IDs, such as TCPCLv4 {{RFC9174}}.


## Semantic Attacks

The simplicity of the 'imc' URI scheme syntax minimizes the possibility of
misinterpretation of a URI by a human user.


## EID Patterns

The security considerations of {{Section 5 of I-D.ietf-dtn-eid-pattern}}
apply to imc patterns ({{patterns}}). Because the text forms of imc and IPN
patterns differ only in their scheme name, care is needed that a pattern
intended to match 'ipn' EIDs is not written with the 'imc' scheme name, or
vice versa, as the two match disjoint sets of EIDs.


# IANA Considerations {#iana}

## 'imc' URI Scheme {#iana-uri}

IANA is requested to register the following URI scheme in the "Uniform
Resource Identifier (URI) Schemes" registry, using the template in
{{Section 7.4 of RFC7595}}:

Scheme name:
: imc

Status:
: Permanent

Applications/protocols that use this scheme name:
: The Delay-Tolerant Networking (DTN) Bundle Protocol Version 7 (BPv7)
  {{RFC9171}}.

Contact:
: IESG &lt;iesg@ietf.org&gt;

Change controller:
: IETF

References:
: This document.


## Bundle Protocol URI Scheme Types {#iana-bp}

IANA is requested to assign the following value in the "Bundle Protocol URI
Scheme Types" registry within the "Bundle Protocol" registry group, including
the "EID Pattern Reference" column added by
{{Section 6.1 of I-D.ietf-dtn-eid-pattern}}:

| Value | Description | BP Utilization Reference | URI Definition Reference | EID Pattern Reference |
|-------|-------------|--------------------------|--------------------------|-----------------------|
| TBD1  | imc         | This document            | This document            | This document, {{patterns}} |
{: title="New Value in the Bundle Protocol URI Scheme Types Registry"}

[^tbd1]

[^tbd1]: Implementations of {{I-D.burleigh-dtn-multicast}}, such as ION, use
    the value 3 for 'imc', and {{I-D.cavallini-dtn-iac}} also suggests the
    value 3 for 'iac'. The WG needs to resolve this conflict before IANA
    assignment.


## 'ipn' Scheme URI Allocator Identifiers {#iana-allocators}

IANA is requested to add this document as a reference to the "'ipn' Scheme
URI Allocator Identifiers" registry, and to add the following note to that
registry:

> Allocator Identifiers in this registry are also used to identify the
> Allocators of Group Numbers for the 'imc' URI scheme.


## 'imc' Scheme URI Default Allocator Group Numbers Registry {#iana-groups}

IANA is requested to create a new registry titled "'imc' Scheme URI Default
Allocator Group Numbers" in the "Uniform Resource Identifier (URI) Schemes"
registry group. Using terms defined in {{RFC8126}}, the registration
procedures for this registry are:

| Range                | Registration Procedures |
|----------------------|-------------------------|
| 1..127               | Private Use             |
| 128..255             | Standards Action        |
| 0x0100..0x7FFF       | Private Use             |
| 0x8000..0xFFFF       | Specification Required  |
| 0x10000..0xFFFFFFFE  | Private Use             |
| >=0x100000000        | Invalid                 |
{: title="Registration Procedures for the 'imc' Scheme URI Default Allocator Group Numbers Registry"}

These ranges follow the CBOR encoding of unsigned integers
({{Section 3 of RFC8949}}), in which values less than 24 are encoded in one
octet, values less than 256 in two octets, values less than 65536 in three
octets, and values less than 2^32 in five octets. The two-octet and
three-octet encoded lengths are each divided between Private Use and
registered Group Numbers, so that deployments using Private Use Group Numbers
are not forced to use less efficiently encoded values than registered
groups. All one-octet values other than zero (0), and all
five-octet values other than 2^32-1, are reserved for Private Use. This is
the same structure as the "'ipn' Scheme URI Well-Known Service Numbers for
BPv7" registry ({{Section 9.3 of RFC9758}}).

The initial values in the registry are:

| Value      | Description                                       | Reference                    |
|------------|---------------------------------------------------|------------------------------|
| 0          | Reserved for the Null imc URI                     | This document, {{null-imc}}  |
| 4294967295 | Invalid; avoids confusion with LocalNode ipn URIs | This document, {{max-group}} |
{: title="Initial Values in the 'imc' Scheme URI Default Allocator Group Numbers Registry"}

Each entry in this registry associates a single Group Number with a
group of interest to the wider BPv7 community. Within the registry,
the group is identified using the "Description" field, and the
"Reference" field identifies a publicly available specification of the
purpose of the group.

[^groups-ranges]

[^groups-ranges]: The ranges of the "'ipn' Scheme URI Default Allocator Node
    Numbers" registry (Section 9.2 of {{RFC9758}}) are not used as a model,
    because that registry retains its pre-existing structure and assignments
    rather than following CBOR encoding lengths. How this registry relates to
    the "Bundle Protocol Well-Known Group Numbers" registry proposed by
    {{I-D.cavallini-dtn-iac}}, which uses the same CBOR-aligned ranges, is to
    be decided.


### Guidance for Designated Experts {#iana-groups-experts}

Due to the nature of the CBOR encoding of unsigned integers used for Group
Numbers with BPv7, Group Numbers with a low value are encoded more
efficiently than larger numbers. The two-octet registered range (128..255)
is therefore reserved for Standards Action. Designated experts reviewing
requests in the three-octet registered range (0x8000..0xFFFF) are strongly
recommended to familiarize themselves with the CBOR encoding of unsigned
integers in {{RFC8949}}.

Group Numbers assigned by the Default Allocator are intended for multicast
groups that are of use to the entire BPv7 community. Each registration
outside the Private Use ranges MUST have a corresponding publicly available
specification describing the purpose of the group. Multicast
groups that are specific to a particular deployment or organization are
expected to use Group Numbers assigned by that organization's own Allocator,
or Private Use Group Numbers, and do not require an entry in this registry.

Designated experts MUST NOT approve the registration of any Group Number
whose specification requires bundle nodes to be implicitly members of the
multicast endpoints of the identified group (see {{sec-implicit}}).


--- back

# 'imc' URI Scheme Text Representation Examples {#text-examples}

This section provides some example imc URIs in their textual representation.


## Using the Default Allocator

Consider the imc URI identifying the multicast endpoint with Service Number 1
of the group with Group Number 5 assigned by the Default Allocator (0).

The recommended seven-character representation of this URI would be as
follows:

~~~
imc:5.1
~~~

The nine-character representation of this URI, with the explicit Allocator
Identifier, would be as follows:

~~~
imc:0.5.1
~~~


## Using a Non-Default Allocator

Consider the imc URI identifying the multicast endpoint with Service Number 1
of the group with Group Number 17 assigned by Allocator 977000.

The 15-character representation of this URI would be as follows:

~~~
imc:977000.17.1
~~~


## The Null imc URI

The Null imc URI ({{null-imc}}) is represented as:

~~~
imc:0.0
~~~


# 'imc' URI Scheme CBOR Encoding Examples {#cbor-examples}

This section provides some example CBOR encodings of imc EIDs. As in
{{cbor}}, these examples use the placeholder value 3 for the 'imc' URI scheme
type.


## Using the Default Allocator

Consider the imc EID imc:5.1. This textual representation of an imc EID
identifies the multicast endpoint with Service Number 1 of the group with Group
Number 5 assigned by the Default Allocator (0).

The six-octet encoding of this EID would be as follows:

~~~ cbor-pretty
82       # 2-Element Endpoint Encoding
   03    # uri-code: 3 ('imc' URI scheme)
   83    # 3-Element imc EID encoding
      00 # Default Allocator
      05 # Group Number
      01 # Service Number
~~~


## Using a Non-Default Allocator

Consider the imc EID imc:977000.17.1. This textual representation of an imc
EID identifies the multicast endpoint with Service Number 1 of the group with
Group Number 17 assigned by Allocator 977000.

The 10-octet encoding of this EID would be as follows:

~~~ cbor-pretty
82                # 2-Element Endpoint Encoding
   03             # uri-code: 3 ('imc' URI scheme)
   83             # 3-Element imc EID encoding
      1A 000EE868 # Allocator Identifier
      11          # Group Number
      01          # Service Number
~~~


## The Null imc EID

The six-octet encoding of the Null imc EID ({{null-group}}) of imc:0.0 would
be as follows:

~~~ cbor-pretty
82       # 2-Element Endpoint Encoding
   03    # uri-code: 3 ('imc' URI scheme)
   83    # 3-Element imc EID encoding
      00 # Default Allocator
      00 # Group Number
      00 # Service Number
~~~


# Acknowledgments
{:numbered="false"}

The 'imc' URI scheme was designed by Scott Burleigh, and much of the text of
this document is derived from {{I-D.burleigh-dtn-multicast}} and
{{I-D.burleigh-dtnrg-imc}}. The structure of this document follows
{{RFC9758}}.
