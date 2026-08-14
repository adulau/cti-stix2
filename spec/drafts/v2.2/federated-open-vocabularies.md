# STIX 2.2 Proposal: Federated Open Vocabularies and Vocabulary Registries

This proposal addresses the current STIX concept of **open vocabularies** and proposes a mechanism to make them genuinely open, extensible, machine-readable, versioned, and independently maintainable.

The proposal is inspired by:

* the existing STIX `open-vocab` datatype and STIX vocabularies;
* the [MISP taxonomy format](https://www.misp-standard.org/rfc/misp-standard-taxonomy-format.html);
* the [MISP galaxy format](https://www.misp-standard.org/rfc/misp-standard-galaxy-format.html);
* the existing public MISP taxonomy and galaxy repositories;
* operational requirements to extend vocabularies without requiring a new version of the STIX specification.

## Goals

1. Preserve the existing STIX `open-vocab` datatype as a string.
2. Preserve all existing STIX 2.0 and STIX 2.1 open-vocabulary values without modification.
3. Move the definition and maintenance of open vocabularies from static lists embedded in the STIX specification toward machine-readable vocabulary registries.
4. Assign stable UUIDs to vocabularies and individual vocabulary entries.
5. Allow vocabularies to evolve independently of the STIX specification lifecycle.
6. Allow organizations and communities to define their own vocabulary extensions.
7. Provide namespaces to avoid collisions between independently maintained extensions.
8. Allow existing MISP taxonomy repositories to be used as STIX vocabulary sources.
9. Allow MISP galaxy repositories to provide richer UUID-based knowledge vocabularies where appropriate.
10. Allow private, community, sector-specific, national, and vendor-specific vocabulary repositories.
11. Ensure that a STIX consumer does not need network access to process a STIX object.
12. Ensure that unknown vocabulary entries remain valid open-vocabulary strings.
13. Keep STIX Enumerations separate from Open Vocabularies; enumerations remain closed and normative.

## Non-goals

* Replacing STIX Enumerations with externally maintained lists.
* Requiring producers or consumers to contact an online vocabulary service.
* Requiring every STIX implementation to support MISP.
* Converting all MISP galaxies into STIX open-vocabulary values.
* Making vocabulary repositories authoritative over STIX objects already exchanged.
* Replacing SDOs such as Malware, Threat Actor, Attack Pattern, Tool, or Vulnerability with vocabulary values.
* Requiring UUIDs to replace the human-readable strings currently used in STIX properties.

## Current STIX open-vocabulary model

STIX 2.1 defines `open-vocab` as a string.

Properties using this datatype identify a suggested vocabulary. Producers SHOULD select a value from that vocabulary but MAY use another string.

For example:

```json
{
  "type": "threat-actor",
  "roles": [
    "director",
    "infrastructure-operator"
  ]
}
```

The current Threat Actor Role Vocabulary defines values such as:

```text
agent
director
independent
infrastructure-architect
infrastructure-operator
malware-author
sponsor
```

A producer may already introduce another value:

```json
{
  "roles": [
    "initial-access-broker"
  ]
}
```

This is syntactically valid STIX because `roles` uses an open vocabulary.

However, there is currently no machine-readable mechanism to determine:

* who defined `initial-access-broker`;
* which vocabulary contains it;
* its semantic definition;
* whether two independently defined strings have the same meaning;
* whether the value has been renamed or deprecated;
* whether aliases exist;
* which version of the vocabulary introduced it;
* where additional metadata can be retrieved;
* whether another vocabulary extends the STIX vocabulary.

The proposed registry mechanism addresses these limitations without changing the serialized property.

## Summary of the proposal

STIX 2.2 SHOULD retain:

```text
open-vocab = string
```

STIX 2.2 SHOULD additionally define a machine-readable **Open Vocabulary Registry model**.

The model separates:

```text
STIX object
       |
       | contains a string
       v
"ransomware"
       |
       | optionally resolved
       v
Vocabulary entry
       |
       +-- UUID
       +-- description
       +-- namespace
       +-- vocabulary
       +-- version
       +-- aliases
       +-- deprecation state
       +-- references
       +-- relationships
```

Vocabulary resolution is OPTIONAL.

The STIX object remains understandable and valid without resolving the vocabulary entry.

## Core design principle

The value serialized in existing STIX objects MUST NOT change solely because the registry mechanism is introduced.

For example:

```json
{
  "malware_types": [
    "ransomware"
  ]
}
```

MUST remain valid.

It MUST NOT become:

```json
{
  "malware_types": [
    {
      "value": "ransomware",
      "uuid": "..."
    }
  ]
}
```

and SHOULD NOT become:

```json
{
  "malware_types": [
    "urn:uuid:..."
  ]
}
```

The UUID belongs to the **vocabulary definition**, not to the base STIX wire representation.

This allows existing implementations to continue treating an open-vocabulary property as a string.

## Vocabulary identity

Every registered vocabulary SHOULD have a stable UUID.

Every registered vocabulary value SHOULD also have a stable UUID.

For example, conceptually:

```text
STIX Malware Type Vocabulary
UUID: <vocabulary-uuid>

    ransomware
        UUID: <ransomware-value-uuid>

    rootkit
        UUID: <rootkit-value-uuid>

    spyware
        UUID: <spyware-value-uuid>
```

The UUID provides semantic identity independently from:

* the human-readable value;
* the repository hosting the vocabulary;
* the repository URL;
* the vocabulary version;
* translations;
* aliases;
* display names.

UUID version 4 or UUID version 5 SHOULD be used for newly created vocabulary identifiers.

Existing UUIDs from compatible vocabulary repositories MUST be preserved.

## UUID stability

A vocabulary UUID MUST remain unchanged across updates to the same vocabulary.

A vocabulary entry UUID MUST remain unchanged when:

* its description is improved;
* references are added;
* aliases are added;
* translations are added;
* non-semantic metadata is modified.

A new UUID SHOULD be assigned when the semantic meaning of a vocabulary entry changes substantially.

A UUID MUST NOT be reused for a different concept.

## Core STIX vocabulary registry

The existing STIX open vocabularies SHOULD form the initial **STIX Core Vocabulary Registry**.

For example:

```text
stix
 |
 +-- account-type-ov
 +-- attack-motivation-ov
 +-- attack-resource-level-ov
 +-- grouping-context-ov
 +-- hashing-algorithm-ov
 +-- identity-class-ov
 +-- indicator-type-ov
 +-- industry-sector-ov
 +-- infrastructure-type-ov
 +-- malware-type-ov
 +-- report-type-ov
 +-- threat-actor-type-ov
 +-- threat-actor-role-ov
 +-- threat-actor-sophistication-ov
 +-- tool-type-ov
 +-- ...
```

Each existing vocabulary becomes a persistent vocabulary identified by UUID.

Each existing value receives a persistent UUID.

The canonical string representations defined by earlier STIX versions MUST remain unchanged.

For example:

```text
threat-actor-role-ov
    agent
    director
    independent
    infrastructure-architect
    infrastructure-operator
    malware-author
    sponsor
```

remain serialized exactly as they are today.

## Compatibility with the MISP taxonomy model

The STIX Vocabulary Registry model SHOULD intentionally align with the MISP taxonomy model.

A MISP taxonomy consists conceptually of:

```text
namespace
    |
    +-- predicate
            |
            +-- value
            +-- value
            +-- value
```

A STIX vocabulary can be represented in the same structure:

```text
namespace = stix

predicate = threat-actor-role-ov

values =
    agent
    director
    independent
    infrastructure-architect
    infrastructure-operator
    malware-author
    sponsor
```

This means that a machine-readable representation of the current STIX vocabulary can closely follow the MISP taxonomy format.

For example, the following is illustrative; the UUIDs are placeholders and would need to be allocated by the TC:

```json
{
  "namespace": "stix",
  "description": "STIX open vocabularies",
  "version": 1,
  "uuid": "11111111-1111-4111-8111-111111111111",
  "predicates": [
    {
      "value": "threat-actor-role-ov",
      "expanded": "Threat Actor Role Vocabulary",
      "uuid": "22222222-2222-4222-8222-222222222222"
    }
  ],
  "values": [
    {
      "predicate": "threat-actor-role-ov",
      "entry": [
        {
          "value": "agent",
          "expanded": "Agent",
          "uuid": "33333333-3333-4333-8333-333333333331"
        },
        {
          "value": "director",
          "expanded": "Director",
          "uuid": "33333333-3333-4333-8333-333333333332"
        },
        {
          "value": "infrastructure-operator",
          "expanded": "Infrastructure Operator",
          "uuid": "33333333-3333-4333-8333-333333333333"
        },
        {
          "value": "malware-author",
          "expanded": "Malware Author",
          "uuid": "33333333-3333-4333-8333-333333333334"
        }
      ]
    }
  ]
}
```

The exact schema does not need to be identical to the MISP taxonomy schema, but SHOULD preserve its important design characteristics:

* namespace;
* vocabulary/predicate;
* value;
* UUID;
* version;
* description;
* expanded/display value;
* optional numerical value;
* optional metadata.

## Vocabulary bindings

A vocabulary registry SHOULD describe where a vocabulary is normally used.

For example:

```json
{
  "vocabulary": "threat-actor-role-ov",
  "vocabulary_uuid": "22222222-2222-4222-8222-222222222222",
  "bindings": [
    {
      "object_type": "threat-actor",
      "property": "roles"
    }
  ]
}
```

Another vocabulary may apply to several properties or object types.

Bindings are informative.

They MUST NOT prevent an open-vocabulary value from being used elsewhere when permitted by the STIX specification.

## External vocabulary extensions

Organizations MUST be able to publish additional vocabulary entries without modifying the STIX specification.

For example, an organization may require additional malware types:

```text
loader-as-a-service
browser-injector
credential-relay-malware
```

A custom vocabulary can declare that it extends the STIX Malware Type Vocabulary.

Conceptually:

```text
STIX malware-type-ov
       ^
       |
       | extends
       |
example-org malware taxonomy
```

The external vocabulary has:

* its own namespace;
* its own UUID;
* its own version;
* UUIDs for all entries;
* an explicit reference to the vocabulary it extends.

## Namespaces

Core STIX vocabulary values SHOULD continue to use their existing unqualified representation.

For example:

```json
{
  "malware_types": [
    "ransomware"
  ]
}
```

External vocabulary values SHOULD use a qualified representation when there is a risk of ambiguity or collision.

A MISP-compatible machine-tag representation MAY be used.

For example:

```json
{
  "malware_types": [
    "ransomware",
    "example-org:malware-type=\"loader-as-a-service\""
  ]
}
```

A STIX 2.0 or 2.1 consumer sees both entries as strings.

It understands:

```text
ransomware
```

and may treat:

```text
example-org:malware-type="loader-as-a-service"
```

as an unknown custom open-vocabulary value.

A STIX 2.2 consumer aware of vocabulary registries can additionally resolve the namespace and retrieve its semantic definition.

This provides graceful degradation.

## Why qualified custom values are useful

Consider two organizations that independently define:

```text
broker
```

One may mean:

> a threat actor selling initial network access

while another may mean:

> an intermediary selling stolen data.

Without namespaces, both are represented as:

```json
"broker"
```

With qualified values:

```text
example-a:threat-actor-role="broker"
example-b:threat-actor-role="broker"
```

the concepts remain distinguishable.

The vocabulary entry UUID provides an additional persistent identity.

## Backwards compatibility

The proposed model is designed so that existing STIX objects remain unchanged.

### Existing STIX value

STIX 2.1:

```json
{
  "roles": [
    "infrastructure-operator"
  ]
}
```

STIX 2.2:

```json
{
  "roles": [
    "infrastructure-operator"
  ]
}
```

No change.

### New STIX core value

Suppose the STIX vocabulary registry later adds:

```text
initial-access-broker
```

A STIX 2.2 producer can use:

```json
{
  "roles": [
    "initial-access-broker"
  ]
}
```

A STIX 2.1 consumer already treats this as a syntactically valid unknown open-vocabulary value.

A STIX 2.2 consumer can additionally resolve its UUID and definition.

### External value

```json
{
  "roles": [
    "example-org:threat-actor-role=\"access-broker\""
  ]
}
```

An older implementation still sees a valid string.

A registry-aware implementation understands the namespace and semantic identifier.

## Compatibility matrix

| Producer         | Value                      | STIX 2.0/2.1 Consumer    | Registry-aware STIX 2.2 Consumer  |
| ---------------- | -------------------------- | ------------------------ | --------------------------------- |
| STIX 2.0/2.1     | Existing core value        | Native                   | Native + registry metadata        |
| STIX 2.2         | Existing core value        | Native                   | Native + registry metadata        |
| STIX 2.2         | New core registry value    | Valid unknown open-vocab | Native                            |
| STIX 2.2         | External unqualified value | Valid unknown open-vocab | Resolvable if uniquely registered |
| STIX 2.2         | Qualified external value   | Valid unknown open-vocab | Namespace and UUID resolvable     |
| MISP/STIX bridge | MISP machine tag           | Valid unknown open-vocab | Taxonomy directly resolvable      |

## Vocabulary versioning

Vocabulary definitions SHOULD contain a monotonically increasing version.

For example:

```text
malware-type-ov
UUID: A
Version: 1

  ransomware
  rootkit
  spyware
```

may evolve into:

```text
malware-type-ov
UUID: A
Version: 2

  ransomware
  rootkit
  spyware
  bootkit
```

The vocabulary UUID remains:

```text
A
```

Existing value UUIDs remain unchanged.

Only the registry version changes.

This allows vocabulary updates without requiring:

```text
STIX 2.2
STIX 2.3
STIX 2.4
...
```

for every vocabulary addition.

## Deprecation

Removing vocabulary values creates interoperability problems and SHOULD generally be avoided.

Instead, entries SHOULD support deprecation metadata.

For example:

```json
{
  "value": "old-term",
  "uuid": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "deprecated": true,
  "replaced_by_uuid": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"
}
```

The old entry remains resolvable.

Consumers can recommend the newer value without making previously exchanged STIX objects invalid.

## Aliases and synonyms

Vocabulary entries MAY define aliases.

For example:

```json
{
  "value": "initial-access-broker",
  "uuid": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "aliases": [
    "access-broker",
    "iab"
  ]
}
```

Aliases are useful for:

* search;
* translation;
* import;
* normalization;
* correlation.

The canonical serialized value remains:

```text
initial-access-broker
```

Aliases SHOULD NOT silently modify STIX content during ingestion.

## Registry directory

Vocabulary repositories SHOULD provide a machine-readable manifest.

This follows the deployment model successfully used by the MISP taxonomy repository.

Conceptually:

```json
{
  "version": 1,
  "description": "STIX Open Vocabulary Registry",
  "uuid": "aaaaaaaa-1111-4111-8111-aaaaaaaaaaaa",
  "vocabularies": [
    {
      "name": "STIX Core Open Vocabularies",
      "namespace": "stix",
      "version": 1,
      "format": "stix-open-vocab",
      "path": "stix-vocabularies.json"
    },
    {
      "name": "Example Organization Vocabulary",
      "namespace": "example-org",
      "version": 4,
      "format": "misp-taxonomy",
      "path": "example-org/machinetag.json"
    }
  ]
}
```

A registry MAY reference another registry.

This permits federation.

## Federated vocabulary model

There SHOULD NOT be a requirement for a single global repository containing every possible vocabulary.

Instead:

```text
                  STIX Core Registry
                         |
          +--------------+--------------+
          |                             |
     FIRST registry                MISP registry
          |                             |
     sector vocab                +------+------+
                                 |             |
                             taxonomy       galaxy
                                 |
                      organization registry
```

Organizations can select which registries they trust or enable.

This supports:

* OASIS-managed vocabularies;
* FIRST vocabularies;
* MISP community vocabularies;
* national CSIRT vocabularies;
* ISAC/ISAO vocabularies;
* sector-specific vocabularies;
* vendor vocabularies;
* private organizational vocabularies.

## Registry discovery is optional

A critical requirement is that consuming STIX MUST NOT depend on a live external service.

For example:

```json
{
  "malware_types": [
    "example-org:malware-type=\"special-loader\""
  ]
}
```

remains valid even when:

```text
example-org vocabulary registry
```

cannot be reached.

Registry resolution provides additional semantics but MUST NOT be required to parse the STIX object.

Implementations MAY:

* ship vocabulary snapshots;
* periodically synchronize repositories;
* cache vocabulary definitions;
* operate entirely offline;
* disable external registry resolution;
* trust only explicitly configured registries.

## Registry security

Implementations MUST NOT automatically trust arbitrary vocabulary locations received from untrusted STIX content.

Vocabulary repositories SHOULD be configured or discovered through trusted manifests.

Registries SHOULD support integrity mechanisms such as:

* hashes;
* signed manifests;
* pinned repository revisions;
* trusted distribution channels.

A remote vocabulary update MUST NOT retrospectively alter the interpretation of an existing UUID.

The UUID identifies the semantic concept, while a URL merely identifies one possible location from which metadata can be retrieved.

## MISP taxonomy compatibility

MISP taxonomies are especially suitable for representing STIX open vocabularies.

A MISP taxonomy provides:

```text
namespace
predicate
value
UUID
version
description
```

A STIX implementation SHOULD therefore be able to register a MISP taxonomy as a vocabulary source without requiring the taxonomy to be converted into a new conceptual model.

For example:

```text
admiralty-scale:information-credibility="2"
```

can remain exactly the same machine tag in MISP and, when appropriate, be used as an external open-vocabulary value in STIX.

The UUID associated with the MISP taxonomy entry remains its semantic identifier.

## MISP galaxy compatibility

MISP galaxies provide a related but richer model.

A Galaxy defines a collection, and Galaxy Clusters provide:

* persistent UUIDs;
* values;
* descriptions;
* metadata;
* synonyms;
* references;
* relationships to other UUID-backed clusters.

For example:

```text
Galaxy
   |
   +-- Cluster A
   |      UUID A
   |      aliases
   |      references
   |
   +-- Cluster B
          UUID B
          |
          +---- related-to ---> UUID A
```

This model is useful when a vocabulary entry represents more than a simple classification value.

## Classification vocabularies versus knowledge vocabularies

The registry model SHOULD distinguish between two broad categories.

### Classification vocabulary

A classification vocabulary provides values used directly in an open-vocabulary property.

Examples include:

```text
threat-actor-role
malware-type
industry-sector
report-type
infrastructure-type
```

These map naturally to the MISP taxonomy model.

### Knowledge vocabulary

A knowledge vocabulary represents named concepts with richer metadata and relationships.

Examples include:

```text
known threat actors
malware families
tools
attack techniques
ransomware groups
country-specific threat classifications
```

These map naturally to the MISP galaxy model.

Where STIX already has an appropriate SDO, the SDO SHOULD normally be used to represent the actual concept.

For example:

```text
MISP Galaxy Cluster
        |
        | maps to
        v
Threat Actor SDO
```

rather than placing:

```text
APT28
```

in a generic open-vocabulary property.

The Galaxy registry remains useful for:

* UUID identity;
* synonyms;
* mappings;
* import/export;
* enrichment;
* external references;
* relationships;
* cross-ecosystem interoperability.

## Registry source formats

A vocabulary registry SHOULD be capable of describing multiple source formats.

For example:

```text
stix-open-vocab
misp-taxonomy
misp-galaxy
```

A manifest entry might declare:

```json
{
  "name": "Example Threat Vocabulary",
  "namespace": "example",
  "format": "misp-taxonomy",
  "version": 8,
  "path": "machinetag.json"
}
```

or:

```json
{
  "name": "Example Threat Actor Knowledge Base",
  "namespace": "example-actors",
  "format": "misp-galaxy",
  "version": 12,
  "path": "threat-actors.json"
}
```

This allows the existing MISP repositories to participate directly instead of requiring duplicated STIX-specific copies.

## Private vocabularies

Nothing in the registry model SHOULD require a vocabulary to be public.

For example:

```text
banking-isac:fraud-type="account-takeover"
```

could resolve only inside a banking information-sharing community.

Similarly:

```text
internal-soc:incident-origin="honeypot-cluster-3"
```

could be meaningful only within one organization.

The same namespace and UUID mechanisms apply.

## Extending another vocabulary

A vocabulary MAY explicitly extend another vocabulary.

For example:

```json
{
  "namespace": "example-org",
  "name": "Example Extended Malware Types",
  "uuid": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "version": 3,
  "extends": [
    {
      "vocabulary": "malware-type-ov",
      "namespace": "stix",
      "uuid": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"
    }
  ]
}
```

The extension MAY:

* add values;
* add aliases;
* provide mappings;
* add descriptions;
* add translations.

An extension MUST NOT redefine the meaning of an existing value UUID.

## Multiple extensions

Several communities may independently extend the same STIX vocabulary.

For example:

```text
                    STIX malware-type-ov
                           |
          +----------------+----------------+
          |                |                |
       Vendor A          CSIRT A          ISAC B
          |                |                |
      extension        extension        extension
```

No central approval is necessary for private extensions.

Public registries MAY establish their own governance requirements before accepting an extension.

## Vocabulary relationships and mappings

Vocabulary entries MAY contain relationships to other entries.

For example:

```json
{
  "value": "initial-access-broker",
  "uuid": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "related": [
    {
      "type": "broader-than",
      "dest_uuid": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"
    }
  ]
}
```

Useful relationship types MAY include:

```text
equivalent-to
similar-to
broader-than
narrower-than
derived-from
replaced-by
related-to
```

The relationship mechanism SHOULD follow the same UUID-based principle used by MISP Galaxy clusters.

## Cross-vocabulary equivalence

One important benefit of UUID-backed registries is explicit mappings between independently maintained vocabularies.

For example:

```text
STIX:
    malware-type-ov:ransomware
             |
             | equivalent-to
             v
Community taxonomy:
    incident-classification:ransomware
```

This mapping SHOULD be represented as metadata rather than silently replacing one vocabulary value with another.

## Numerical values

Vocabulary entries MAY expose a numerical value when this is semantically useful.

This is compatible with MISP taxonomies.

For example:

```json
{
  "value": "very-high",
  "uuid": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "numerical_value": 90
}
```

The textual value remains the canonical STIX representation.

The numeric value is supplemental metadata.

## Human-readable labels and translations

Vocabulary entries SHOULD distinguish the machine-readable value from display text.

For example:

```json
{
  "value": "infrastructure-operator",
  "expanded": "Infrastructure Operator",
  "uuid": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa"
}
```

A registry MAY additionally provide localized display strings:

```json
{
  "translations": {
    "fr": "Opérateur d'infrastructure",
    "de": "Infrastrukturbetreiber"
  }
}
```

The STIX wire value remains:

```text
infrastructure-operator
```

This avoids localization affecting interoperability.

## Updating STIX Section 2.14

The definition of `open-vocab` SHOULD be updated conceptually from:

> a string which SHOULD come from a suggested vocabulary defined by the STIX specification

to:

> a string whose value MAY be defined by a STIX core vocabulary or another vocabulary source. STIX core vocabularies provide recommended values. Additional values MAY be defined by registered or private vocabularies. Producers SHOULD use registered values where an appropriate value exists. Consumers MUST NOT reject an object solely because an open-vocabulary value is unknown.

The existing lowercase and hyphen recommendation SHOULD remain for unqualified custom values.

Qualified vocabulary values MAY additionally use a registered namespace syntax.

## Updating STIX Section 10

The existing Section 10 vocabulary definitions SHOULD remain in STIX 2.2 as the **baseline STIX vocabulary snapshot**.

This preserves:

* human-readable documentation;
* backwards compatibility;
* reproducible specification versions.

Each open vocabulary SHOULD additionally identify:

```text
Vocabulary Name
Vocabulary UUID
Registry Namespace
Registry Location
Registry Version
```

For example:

```text
Vocabulary Name: threat-actor-role-ov
Registry Namespace: stix
Vocabulary UUID: <TC-assigned UUID>
```

The values documented in STIX 2.2 become the initial registry values.

Subsequent compatible values MAY be added to the registry without requiring modification of the STIX core object model.

## Enumerations are unchanged

This proposal applies only to:

```text
open-vocab
```

It does not apply to:

```text
enum
```

For example, where STIX defines an Enumeration:

```text
value MUST be one of:
    x
    y
    z
```

the permitted values remain controlled by the STIX specification.

A registry MUST NOT be used to extend an Enumeration.

This distinction remains explicit:

```text
Enumeration
    CLOSED
    specification-controlled

Open Vocabulary
    OPEN
    registry-assisted
```

## Example: current STIX vocabulary

A STIX object:

```json
{
  "type": "threat-actor",
  "spec_version": "2.2",
  "id": "threat-actor--8e2e2d2b-17d4-4cbf-938f-98ee46b3cd3f",
  "created": "2026-08-14T08:00:00.000Z",
  "modified": "2026-08-14T08:00:00.000Z",
  "name": "Example Actor",
  "roles": [
    "infrastructure-operator"
  ]
}
```

A registry-aware consumer resolves:

```text
property:
    threat-actor.roles

vocabulary:
    threat-actor-role-ov

namespace:
    stix

value:
    infrastructure-operator

value UUID:
    <stable UUID>

description:
    <description maintained by the vocabulary>

version:
    <registry version>
```

Nothing additional is required in the STIX object.

## Example: external extension

An organization publishes:

```text
namespace:
    example-org

vocabulary:
    threat-actor-role

extends:
    STIX threat-actor-role-ov

entry:
    initial-access-broker

UUID:
    <stable UUID>
```

It can serialize:

```json
{
  "roles": [
    "infrastructure-operator",
    "example-org:threat-actor-role=\"initial-access-broker\""
  ]
}
```

A legacy implementation can still process the object.

A registry-aware implementation can resolve both terms.

## Example: MISP taxonomy reused directly

Suppose an existing taxonomy contains:

```text
admiralty-scale:information-credibility="2"
```

and its taxonomy entry already has a stable UUID.

A STIX implementation that uses this vocabulary SHOULD preserve:

* the namespace;
* predicate;
* value;
* UUID.

It SHOULD NOT allocate a second STIX-specific UUID for the same imported vocabulary entry.

This enables:

```text
MISP
   |
   | same namespace/value/UUID
   v
STIX
   |
   | same namespace/value/UUID
   v
MISP
```

without creating duplicate vocabulary identities.

## Example: Galaxy-backed knowledge entry

Suppose a MISP Galaxy contains:

```text
Threat Actor: Example Panda
UUID: A
Aliases:
    Example Group
    Panda Team
References:
    ...
```

A STIX converter can create:

```text
threat-actor--...
```

while retaining:

```text
Galaxy Cluster UUID A
```

as an external semantic identifier or mapping.

The vocabulary registry can retain the mapping:

```text
Galaxy Cluster UUID A
           |
           | maps-to
           v
STIX Threat Actor
```

This is preferable to replacing the Threat Actor SDO with a string vocabulary value.

## Why not put UUIDs directly in open-vocab properties?

Changing:

```json
"roles": [
  "director"
]
```

to:

```json
"roles": [
  {
    "value": "director",
    "uuid": "..."
  }
]
```

would change the STIX datatype and break existing implementations.

Changing it to:

```json
"roles": [
  "urn:uuid:..."
]
```

would preserve the JSON datatype but lose human readability and break existing value-based processing.

The registry approach provides both:

```text
human-readable stable wire value
+
persistent semantic UUID
```

without changing existing STIX objects.

## Why not keep vocabularies only in the specification?

Static specification vocabularies create unnecessary coupling between:

```text
vocabulary evolution
```

and:

```text
STIX specification evolution
```

A new malware category, infrastructure role, industry sector, threat actor role, or report type should not necessarily require a new STIX specification revision.

The object model should remain stable while the terminology can evolve.

## Benefits

### Backwards compatibility

No existing STIX vocabulary value changes.

### Forward compatibility

Older consumers already have a defined behavior for unknown open-vocabulary strings.

### Independent evolution

Vocabulary updates do not require changes to the STIX object model.

### Stable semantic identity

UUIDs make vocabulary entries persistent across renames, repository moves, and metadata updates.

### Decentralization

Organizations can maintain their own vocabularies without requesting additions to the core STIX specification.

### Federation

Multiple repositories can coexist and cross-reference one another.

### Offline use

Registry access is optional and vocabularies can be cached or distributed as files.

### Existing ecosystem reuse

MISP taxonomy and galaxy repositories can be directly reused instead of recreating equivalent vocabulary infrastructure.

### Improved mapping

UUID-based equivalence relationships make mappings between MISP, STIX, sector vocabularies, vendor vocabularies, and other CTI standards explicit.

## Open questions for TC discussion

1. Should STIX 2.2 formally separate **open vocabulary definitions** from the STIX specification lifecycle?
2. Should every existing STIX open vocabulary receive a stable UUID?
3. Should every existing STIX open-vocabulary value receive a stable UUID?
4. Should UUIDv5 be used to deterministically assign UUIDs to the existing STIX core vocabulary entries?
5. Should the STIX Core Vocabulary Registry use a format directly compatible with the MISP taxonomy format?
6. Should STIX define its own vocabulary format while requiring lossless conversion to and from the MISP taxonomy format?
7. Should MISP machine-tag syntax be RECOMMENDED for qualified external vocabulary values?
8. Should qualified external values use:

```text
namespace:predicate="value"
```

or a simpler STIX-specific representation such as:

```text
namespace:value
```

9. Should unqualified values be implicitly interpreted as belonging to the STIX core vocabulary associated with the property?
10. Should external vocabularies explicitly declare which STIX vocabulary UUID they extend?
11. Should registry manifests be standardized as part of STIX 2.2?
12. Should registry manifests support multiple formats such as:

```text
stix-open-vocab
misp-taxonomy
misp-galaxy
```

13. Should registry entries support aliases and translations?
14. Should registry entries support numerical values?
15. Should vocabulary entries support UUID-based relationships such as `equivalent-to`, `broader-than`, and `replaced-by`?
16. Should deprecated vocabulary entries remain permanently resolvable?
17. Should the STIX TC maintain an official vocabulary registry repository independently from the STIX specification repository?
18. Should third-party registry discovery be explicitly outside STIX validation?
19. Should vocabulary registry integrity or signing mechanisms be standardized or left to repository implementations?
20. Should a common crosswalk format be defined for mappings between STIX vocabularies, MISP taxonomies, MISP galaxies, and other CTI vocabularies?

## Proposed next steps

1. Confirm that the `open-vocab` wire datatype remains a string.
2. Identify all existing STIX open vocabularies and distinguish them from Enumerations.
3. Allocate a persistent UUID to each existing STIX open vocabulary.
4. Allocate persistent UUIDs to all existing vocabulary entries.
5. Define the STIX Core Vocabulary Registry.
6. Create a machine-readable representation of all existing Section 10 open vocabularies.
7. Define registry versioning and UUID stability rules.
8. Define namespace rules for third-party vocabulary extensions.
9. Define the behavior of qualified and unqualified values.
10. Define a vocabulary registry manifest.
11. Define compatibility rules for MISP taxonomy repositories.
12. Define compatibility and mapping rules for MISP galaxy repositories.
13. Define extension, deprecation, alias, and equivalence mechanisms.
14. Update Section 2.14 to describe registry-backed open vocabularies.
15. Update Section 10 so that its existing vocabulary values remain the STIX 2.2 baseline snapshot.
16. Add conformance tests demonstrating that:

    * all STIX 2.0/2.1 values remain valid;
    * unknown strings remain valid open-vocabulary values;
    * new core registry values are accepted;
    * qualified external values are accepted;
    * registry resolution is optional;
    * enumerations cannot be extended through the registry.
17. Publish a reference STIX vocabulary repository.
18. Demonstrate interoperability by consuming existing `misp-taxonomies` and `misp-galaxy` repositories without changing their UUID identities.

