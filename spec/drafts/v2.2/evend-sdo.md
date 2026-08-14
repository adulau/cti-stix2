# STIX 2.2 Proposal: Event

This proposal addresses:

* [oasis-tcs/cti-stix2#1](https://github.com/oasis-tcs/cti-stix2/issues/1) - define a way to capture incident and event information.
* [oasis-tcs/cti-stix2#340](https://github.com/oasis-tcs/cti-stix2/pull/340) - STIX 2.2 proposal for Incident and Event.
* The need for a first-class STIX Domain Object able to represent a coherent unit of cyber threat intelligence, similar to the Event concept defined by the [MISP core format](https://www.misp-standard.org/rfc/misp-standard-core.html#name-event).

## Goals

1. Add a first-class **Event SDO** representing a coherent and persistent unit of cyber threat intelligence.
2. Allow an Event to contain or reference arbitrary STIX content, including SDOs, SCOs, SROs, and SMOs.
3. Represent event-level lifecycle information such as analysis state, publication state, threat level, and a reference time.
4. Allow events to evolve over time using the normal STIX object versioning mechanism.
5. Support events which represent different analytical purposes, including incidents, investigations, threat actor analyses, malware analyses, vulnerability analyses, and collections of related indicators.
6. Provide a straightforward mapping from the MISP Event data model without requiring the MISP Event to be heuristically converted into a Report, Grouping, or Incident.
7. Keep operational or implementation-specific MISP properties outside the core Event SDO where they are not generally applicable to CTI producers.
8. Keep raw logs and high-volume telemetry outside the Event SDO.

## Non-goals

* Defining a log-entry or generic telemetry format.
* Replacing Observed Data or Cyber-observable Objects.
* Replacing the Incident SDO.
* Replacing the Report SDO.
* Replacing Grouping for simple collections where no Event lifecycle or identity is required.
* Making STIX a SIEM or EDR event transport format.
* Reproducing every implementation-specific MISP database property as a STIX property.
* Replacing the MISP core format.

## Summary of the proposal

STIX 2.2 SHOULD introduce an **Event SDO**.

An Event represents a coherent unit of cyber threat intelligence with its own identity and lifecycle.

An Event is primarily a contextual container. Its meaning is determined by the information associated with the Event and the STIX Objects referenced by it.

For example, an Event MAY represent:

* an ongoing threat investigation;
* an incident analysis;
* a malware analysis;
* a threat actor analysis;
* a vulnerability investigation;
* a phishing campaign under investigation;
* an intelligence collection assembled from multiple indicators and observables;
* a collaborative analysis which is not yet mature enough to be published as a Report.

An Event MAY reference an Incident, Report, Grouping, Indicator, Malware, Threat Actor, Vulnerability, Observed Data, Sighting, Note, Opinion, Relationship, or any other STIX Object.

An Event SHOULD NOT be interpreted as a single operating-system, network, application, EDR, or SIEM log entry.

Those records SHOULD continue to be represented using SCOs, Observed Data, Artifacts, or a dedicated log-entry SCO if such an object is standardized.

## Event SDO

### Description

An Event object represents a coherent set of cyber threat intelligence and contextual information maintained as a single logical unit.

An Event has a stable identity and MAY evolve as additional information becomes available.

The Event does not prescribe the semantic nature of its contents. The referenced objects determine whether the Event describes an incident, investigation, threat actor, malware family, vulnerability, campaign, collection of indicators, or another CTI subject.

This follows the model used by MISP, where an Event is a meta-structure containing a coherent set of indicators and metadata, and where the meaning of the Event depends on the information contained in it.

Event therefore differs from a low-level occurrence or log record.

For example:

```text
Event: Investigation of infrastructure associated with Foo ransomware

  ├── Malware
  ├── Threat Actor
  ├── Indicator
  ├── Indicator
  ├── Domain Name
  ├── IPv4 Address
  ├── Relationship
  ├── Sighting
  ├── Note
  └── Report
```

The Event provides the persistent analytical context binding these objects together.

### Properties

Event SHOULD use the common STIX Domain Object properties and the following object-specific properties.

| Property              | Type                        | Required | Description                                                                                                                                                                                                                                                   |
| --------------------- | --------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                | `string`                    | Required | A human-readable name or summary identifying the Event.                                                                                                                                                                                                       |
| `description`         | `string`                    | Optional | Additional human-readable information describing the Event and its analytical context.                                                                                                                                                                        |
| `event_types`         | `list` of type `open-vocab` | Optional | Describes the general purpose or subject of the Event. Suggested values include `incident`, `investigation`, `threat-report`, `threat-actor-analysis`, `malware-analysis`, `vulnerability-analysis`, `campaign-analysis`, `suspicious-activity`, and `other`. |
| `event_time`          | `timestamp`                 | Optional | A reference time associated with the subject of the Event. This is distinct from the STIX `created` and `modified` timestamps, which describe the STIX object lifecycle.                                                                                      |
| `event_time_fidelity` | `open-vocab`                | Optional | Indicates the precision of `event_time`. Suggested values are `second`, `minute`, `hour`, `day`, `month`, and `year`.                                                                                                                                         |
| `analysis_status`     | `open-vocab`                | Optional | The maturity or analysis state of the Event. Suggested values are `initial`, `ongoing`, and `complete`.                                                                                                                                                       |
| `threat_level`        | `open-vocab`                | Optional | A producer-assessed threat level for the Event. Suggested values are `high`, `medium`, `low`, and `unknown`.                                                                                                                                                  |
| `publication_status`  | `open-vocab`                | Optional | Indicates whether the Event is currently intended for publication or dissemination. Suggested values are `unpublished` and `published`.                                                                                                                       |
| `first_published`     | `timestamp`                 | Optional | Time at which the Event was first published by its creator.                                                                                                                                                                                                   |
| `last_published`      | `timestamp`                 | Optional | Time at which the Event was most recently published or republished.                                                                                                                                                                                           |
| `producer_ref`        | `identifier`                | Optional | Reference to an Identity representing the organization or entity currently producing or distributing this representation of the Event.                                                                                                                        |
| `object_refs`         | `list` of type `identifier` | Optional | References to STIX Objects which form part of the Event. If present, the list MUST contain at least one reference.                                                                                                                                            |
| `extends_ref`         | `identifier`                | Optional | Reference to another Event that this Event explicitly extends. The referenced object MUST be of type `event`.                                                                                                                                                 |

The existing common STIX property `created_by_ref` SHOULD identify the original creator of the Event.

`producer_ref` is intentionally separate from `created_by_ref`, as an Event MAY be redistributed, synchronized, enriched, or republished by an organization other than its original creator.

### Event identity and versioning

An Event is a persistent STIX Domain Object.

New information concerning the same logical Event SHOULD normally result in a new version of the same Event object using the normal STIX `modified` versioning rules.

For example:

```text
event--8f70f1d1-2c88-4b3f-91f9-678098ab3f8f
    modified: 2026-08-10T08:00:00Z

event--8f70f1d1-2c88-4b3f-91f9-678098ab3f8f
    modified: 2026-08-11T14:15:00Z

event--8f70f1d1-2c88-4b3f-91f9-678098ab3f8f
    modified: 2026-08-13T09:42:00Z
```

A separate Event SHOULD be created when the producer intends to create a distinct analytical unit rather than another version of the same Event.

`extends_ref` MAY be used when a new Event intentionally extends another Event while preserving both Event identities.

## Relationships from Event

Most membership SHOULD be expressed using `object_refs`.

Semantic relationships between individual objects SHOULD continue to be expressed using normal STIX Relationship Objects.

The following Event relationships are useful in addition to the common STIX relationships:

| Relationship Type | Target                                      | Description                                                                 |
| ----------------- | ------------------------------------------- | --------------------------------------------------------------------------- |
| `extends`         | `event`                                     | The source Event extends the target Event while remaining a distinct Event. |
| `related-to`      | `event` or any STIX Object                  | Generic relationship when Events or their subjects are related.             |
| `derived-from`    | `event`, `report`, `grouping`, or other SDO | The Event was derived from another CTI object or Event.                     |
| `duplicate-of`    | `event`                                     | Indicates that two Event identities represent the same logical Event.       |

The TC may choose either an embedded `extends_ref` property or an `extends` SRO. Both are included here for discussion; only one mechanism is necessary in the final specification.

## Event compared with existing STIX objects

The Event SDO is not intended to replace existing STIX containers or analytical objects.

### Bundle

A Bundle is a transport container.

Objects occurring in the same Bundle do not imply that the objects have any semantic relationship.

Event explicitly asserts that its `object_refs` form part of the same analytical context.

### Grouping

Grouping expresses that a set of STIX Objects share some context.

Event adds a persistent analytical lifecycle to that concept, including:

* an Event reference time;
* analysis status;
* threat level;
* publication lifecycle;
* original creator and current producer;
* Event-to-Event extension;
* stable identity intended for collaborative enrichment.

Grouping remains appropriate for lightweight contextual collections where this lifecycle is unnecessary.

### Report

Report represents a finished or publishable intelligence product.

An Event can exist while analysis is initial or ongoing and can later contain a Report when a finished intelligence product is produced.

For example:

```text
Event
  analysis_status = ongoing
       |
       +-- Indicators
       +-- Malware
       +-- Threat Actor
       +-- Sightings
       +-- Notes
       |
       +-- Report
             "Final analysis of Foo campaign"
```

The Event remains the persistent collaborative context while the Report represents the resulting intelligence product.

### Incident

Incident represents an actual or suspected security incident.

Not every Event is an Incident.

For example, an Event could contain an analysis of threat actor infrastructure without describing an incident involving a victim.

When an Event does describe an incident, the Event MAY reference an Incident SDO:

```text
Event
  |
  +-- Incident
  +-- Indicators
  +-- Observed Data
  +-- Sightings
  +-- Course of Action
  +-- Notes
```

### Observed Data

Observed Data describes observable facts and evidence.

Event provides the analytical context around those observations.

A producer SHOULD NOT use Event as a replacement for Observed Data.

## Relationship to log entries

The Event SDO proposed here does not represent low-level log entries.

For example, these are not Event SDOs in the sense defined by this proposal:

```text
Windows Event ID 4688
Sysmon process creation event
HTTP access log entry
NetFlow record
EDR telemetry record
DNS query log
```

Such records are observations rather than persistent CTI analytical units.

A typical representation could instead be:

```text
Artifact / SCO
      |
Observed Data
      |
   Sighting
      |
    Event
      |
   Incident
```

This separation prevents STIX from becoming a high-volume logging format while retaining the ability to exchange selected evidence and its analytical context.

## MISP Event compatibility

The MISP core format defines Event as the top-level meta-structure used to form a coherent set of indicators and contextual information.

A MISP Event can represent an incident, a security analysis report, a threat actor analysis, or another analytical context.

This maps naturally to the proposed STIX Event SDO.

A MISP Event SHOULD therefore normally map to **one STIX Event SDO**, rather than requiring a producer to select Incident, Report, or Grouping as the replacement for the MISP Event container.

The individual contents of the MISP Event SHOULD then be mapped to their corresponding STIX representations and referenced from `event.object_refs`.

### Mapping MISP Event metadata

| MISP core format element                  | Proposed STIX Event representation                        | Notes                                                                                                                                                                           |
| ----------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Event.uuid`                              | `event.id`                                                | When permitted by STIX identifier requirements, the MISP UUID SHOULD be reused as the UUID component of `event--<uuid>` to provide stable round-tripping.                       |
| `Event.info`                              | `event.name`                                              | Direct mapping of the human-readable Event summary.                                                                                                                             |
| Event description or additional narrative | `event.description`                                       | Where available.                                                                                                                                                                |
| `Event.date`                              | `event.event_time` + `event_time_fidelity = "day"`        | MISP Event dates have day precision.                                                                                                                                            |
| `Event.timestamp`                         | `modified`                                                | Represents creation or latest modification of the MISP Event. On initial conversion it MAY also be used to initialize `created` when no better creation timestamp is available. |
| `Event.analysis = 0`                      | `analysis_status = "initial"`                             | Direct semantic mapping.                                                                                                                                                        |
| `Event.analysis = 1`                      | `analysis_status = "ongoing"`                             | Direct semantic mapping.                                                                                                                                                        |
| `Event.analysis = 2`                      | `analysis_status = "complete"`                            | Direct semantic mapping.                                                                                                                                                        |
| `Event.threat_level_id = 1`               | `threat_level = "high"`                                   | Direct semantic mapping.                                                                                                                                                        |
| `Event.threat_level_id = 2`               | `threat_level = "medium"`                                 | Direct semantic mapping.                                                                                                                                                        |
| `Event.threat_level_id = 3`               | `threat_level = "low"`                                    | Direct semantic mapping.                                                                                                                                                        |
| `Event.threat_level_id = 4`               | `threat_level = "unknown"`                                | MISP uses undefined.                                                                                                                                                            |
| `Event.published = true`                  | `publication_status = "published"`                        | Event is currently published.                                                                                                                                                   |
| `Event.published = false`                 | `publication_status = "unpublished"`                      | Event is currently not published.                                                                                                                                               |
| `Event.first_publication`                 | `first_published`                                         | First publication by the original creator.                                                                                                                                      |
| `Event.publish_timestamp`                 | `last_published`                                          | Most recent publication time.                                                                                                                                                   |
| `Event.Orgc`                              | `created_by_ref`                                          | Identity representing the original creator organization.                                                                                                                        |
| `Event.Org`                               | `producer_ref`                                            | Identity representing the organization producing or synchronizing the current representation.                                                                                   |
| `Event.extends_uuid`                      | `extends_ref` or `extends` relationship                   | References the Event extended by this Event.                                                                                                                                    |
| `Event.Tag`                               | `labels`, Marking Definitions, or referenced STIX objects | Depends on taxonomy semantics.                                                                                                                                                  |
| `Event.distribution`                      | `object_marking_refs`                                     | Distribution constraints SHOULD use STIX Data Markings where the semantics can be represented.                                                                                  |
| `Event.sharing_group_id`                  | Marking Definition or extension                           | MISP Sharing Groups may require additional metadata beyond standard STIX markings.                                                                                              |
| `Event.RelatedEvent`                      | Event SDO + `related-to` Relationship                     | Each related MISP Event becomes another Event SDO.                                                                                                                              |

### Mapping MISP Event contents

| MISP structure                   | STIX representation                                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `Attribute` with `to_ids = true` | Indicator when the value represents detection or indicator semantics                                                                |
| Observable `Attribute`           | SCO and/or Observed Data                                                                                                            |
| Attachment or malware sample     | Artifact and File SCO                                                                                                               |
| `Object`                         | Appropriate SDO/SCO structures based on the MISP Object template                                                                    |
| `ObjectReference`                | STIX Relationship or embedded STIX reference                                                                                        |
| `Galaxy` / `GalaxyCluster`       | Threat Actor, Intrusion Set, Malware, Tool, Attack Pattern, Campaign, Vulnerability, Location, Identity, or another appropriate SDO |
| `EventReport`                    | Report or Note                                                                                                                      |
| `Sighting`                       | Sighting SRO                                                                                                                        |
| Analyst `Note`                   | Note                                                                                                                                |
| Analyst `Opinion`                | Opinion                                                                                                                             |
| Analyst `Relationship`           | Relationship SRO                                                                                                                    |
| `RelatedEvent`                   | Event SDO related to the current Event                                                                                              |
| `ShadowAttribute` / proposals    | Note, Opinion, or a dedicated collaborative-change extension                                                                        |

All translated objects SHOULD be included in the Event's `object_refs` when they form part of the Event context.

## MISP-specific implementation metadata

Some properties in a MISP Event describe MISP implementation or synchronization behavior rather than generally applicable CTI semantics.

These SHOULD NOT become core STIX Event properties solely for the purpose of achieving byte-for-byte MISP round-tripping.

Examples include:

* local numeric `id`;
* `attribute_count`;
* `locked`;
* `proposal_email_lock`;
* `disable_correlation`;
* local `sharing_group_id`;
* local organization numeric IDs;
* implementation-specific creator email fields.

When lossless MISP round-tripping is required, these fields MAY be preserved using:

1. `external_references`;
2. a MISP Extension Definition; or
3. implementation-specific extension properties.

This keeps the core Event SDO generic while allowing MISP implementations to retain all original information.

## Example: MISP-style STIX Event

The following example represents an ongoing malware investigation containing an Indicator, Malware object, Sighting, and analyst Note.

```json
{
  "type": "bundle",
  "id": "bundle--6d77cc83-9313-4f4c-bd98-54f5977b1a57",
  "objects": [
    {
      "type": "identity",
      "spec_version": "2.2",
      "id": "identity--55f6ea5e-2c60-40e5-964f-47a8950d210f",
      "created": "2026-08-10T08:00:00.000Z",
      "modified": "2026-08-10T08:00:00.000Z",
      "name": "Example CSIRT",
      "identity_class": "organization"
    },
    {
      "type": "event",
      "spec_version": "2.2",
      "id": "event--8f70f1d1-2c88-4b3f-91f9-678098ab3f8f",
      "created_by_ref": "identity--55f6ea5e-2c60-40e5-964f-47a8950d210f",
      "created": "2026-08-10T08:00:00.000Z",
      "modified": "2026-08-13T12:00:31.000Z",
      "name": "Investigation of Foo ransomware infrastructure",
      "description": "Ongoing collaborative analysis of infrastructure and indicators associated with Foo ransomware.",
      "event_types": [
        "malware-analysis",
        "investigation"
      ],
      "event_time": "2026-08-10T00:00:00.000Z",
      "event_time_fidelity": "day",
      "analysis_status": "ongoing",
      "threat_level": "high",
      "publication_status": "published",
      "first_published": "2026-08-11T09:00:00.000Z",
      "last_published": "2026-08-13T12:00:31.000Z",
      "object_refs": [
        "indicator--52bfa2cb-3f6b-4ee8-9845-230e14729210",
        "malware--0c7b5b88-8ff7-4a4d-aa9d-feb398cd0061",
        "sighting--98fa81c2-a23d-4828-a8ed-d11e3c9ccf95",
        "note--49b73241-7dc4-49c5-96db-a61ad33d70d8"
      ]
    },
    {
      "type": "indicator",
      "spec_version": "2.2",
      "id": "indicator--52bfa2cb-3f6b-4ee8-9845-230e14729210",
      "created": "2026-08-10T08:10:00.000Z",
      "modified": "2026-08-10T08:10:00.000Z",
      "name": "Infrastructure associated with Foo ransomware",
      "pattern_type": "stix",
      "pattern": "[domain-name:value = 'example.invalid']",
      "valid_from": "2026-08-10T00:00:00.000Z"
    },
    {
      "type": "malware",
      "spec_version": "2.2",
      "id": "malware--0c7b5b88-8ff7-4a4d-aa9d-feb398cd0061",
      "created": "2026-08-10T08:15:00.000Z",
      "modified": "2026-08-10T08:15:00.000Z",
      "name": "Foo ransomware",
      "is_family": true,
      "malware_types": [
        "ransomware"
      ]
    },
    {
      "type": "note",
      "spec_version": "2.2",
      "id": "note--49b73241-7dc4-49c5-96db-a61ad33d70d8",
      "created": "2026-08-12T10:00:00.000Z",
      "modified": "2026-08-12T10:00:00.000Z",
      "content": "Infrastructure is still active and additional indicators are being collected.",
      "object_refs": [
        "indicator--52bfa2cb-3f6b-4ee8-9845-230e14729210",
        "malware--0c7b5b88-8ff7-4a4d-aa9d-feb398cd0061"
      ]
    }
  ]
}
```

## Example: Event containing an Incident and a Report

Event does not replace Incident or Report.

Instead, it can provide persistent context around both:

```text
event--A
  name: "Compromise of Example Corp"
  analysis_status: "complete"

  object_refs:
    incident--B
    indicator--C
    observed-data--D
    sighting--E
    report--F
```

This allows the Event to exist during the complete collaborative lifecycle:

```text
Initial Event
     |
     v
Ongoing investigation
     |
     +---- Incident identified
     |
     +---- Additional observations
     |
     +---- Sightings
     |
     +---- Analyst notes
     |
     v
Analysis complete
     |
     +---- Final Report
```

The identity of `event--A` remains stable throughout the process.

## MISP round-trip pattern

A MISP-to-STIX converter could follow the following procedure:

1. Create one Event SDO using the MISP Event UUID.
2. Map `info`, `date`, `analysis`, threat level, publication metadata, creator, and producer to Event properties.
3. Convert MISP Attributes to Indicators, SCOs, Observed Data, or Artifacts.
4. Convert MISP Objects using template-specific mappings.
5. Convert ObjectReferences to STIX Relationships.
6. Convert Galaxies to their corresponding STIX SDOs.
7. Convert EventReports to Reports or Notes.
8. Convert Sightings to STIX Sighting SROs.
9. Convert analyst Notes, Opinions, and Relationships to the corresponding STIX objects.
10. Add the resulting STIX Objects to `event.object_refs`.
11. Map MISP distribution policy to STIX Data Markings where possible.
12. Preserve remaining MISP-specific synchronization metadata in an extension when lossless round-tripping is required.

The reverse STIX-to-MISP conversion can use the Event SDO as the natural boundary for reconstructing the MISP Event.

## Advantages of a first-class Event SDO

### Stable exchange boundary

An Event creates an explicit semantic boundary for exchanging a coherent CTI dataset.

A Bundle alone cannot provide this because Bundle membership does not imply context.

### Incremental collaborative analysis

An Event can start with only a small amount of information and evolve as analysts add Indicators, Observed Data, Sightings, Notes, Opinions, Incidents, and Reports.

### No forced semantic conversion

A producer does not have to determine whether a generic CTI Event is primarily:

* a Report;
* an Incident; or
* a Grouping.

The Event can reference any of these objects when their semantics become applicable.

### Improved interoperability with MISP

MISP Event is one of the primary CTI exchange structures in operational use.

Providing a corresponding STIX semantic object removes the current need to map the MISP Event container differently depending on its contents.

### Separation from telemetry

The Event SDO represents analytical context rather than raw machine-generated events, keeping telemetry and observables separate from CTI knowledge objects.

## Alignment with the Incident Extension Event

The Incident Extension Suite already defines an Event SDO concept containing properties such as:

* `event_types`;
* `name`;
* `description`;
* `status`;
* event timestamps;
* sightings;
* Event-to-Event references.

This proposal SHOULD reuse compatible concepts from that work.

However, this proposal additionally makes `object_refs` and analytical lifecycle information central to Event so that Event can represent a complete CTI context rather than only an activity occurring on a timeline.

The two approaches are not necessarily mutually exclusive.

A real-world activity represented by the Incident Extension Event model can be represented as an Event with appropriate `event_types`, timestamps, and referenced Observed Data.

A higher-level MISP-style Event can use the same SDO with broader `object_refs` and analytical lifecycle properties.

The TC SHOULD determine whether one Event SDO can cleanly support both use cases or whether the low-level activity concept requires a separate object.

## Open questions for TC discussion

1. Should Event explicitly be defined as a **persistent CTI contextual container**, rather than as a low-level log or telemetry event?
2. Should `object_refs` be the primary mechanism used to associate STIX content with Event?
3. Should `object_refs` be optional so that an Event can exist before intelligence objects are added?
4. Should `analysis_status` be part of the Event core properties?
5. Should `threat_level` be standardized in Event or represented using labels or extensions?
6. Should publication lifecycle (`publication_status`, `first_published`, `last_published`) be part of Event?
7. Should Event distinguish the original creator (`created_by_ref`) from the current producer or distributor (`producer_ref`)?
8. Should MISP `extends_uuid` map to an embedded `extends_ref` or an `extends` Relationship SRO?
9. Should Event reuse `start_time` / `end_time` from the Incident Extension Event proposal instead of defining a generic `event_time`?
10. Should time-fidelity properties from the Incident Extension work be included in core STIX 2.2?
11. Should the Event SDO support both the Incident Extension real-world activity model and the MISP contextual-container model?
12. If these concepts are considered too different, should low-level machine-generated events instead be represented by a `log-entry` SCO?
13. Should the TC define a standard MISP extension for properties required for fully lossless MISP/STIX/MISP round-tripping?

## Proposed next steps

1. Agree on the semantic definition of Event: a CTI contextual object rather than a raw telemetry record.
2. Review the Event properties against the existing Incident Extension Suite.
3. Determine which Event properties are sufficiently generic to become STIX 2.2 core properties.
4. Add the Event SDO and `event--` identifier namespace to the STIX 2.2 draft.
5. Define Event-specific open vocabularies.
6. Add JSON schemas and conformance tests.
7. Add examples covering:

   * ongoing investigation;
   * Event containing an Incident;
   * Event containing a Report;
   * threat actor analysis;
   * MISP Event conversion;
   * Event extension and versioning.
8. Define a MISP-to-STIX Event conversion profile.
9. Optionally define a MISP Extension Definition for implementation-specific properties required for lossless round-tripping.

