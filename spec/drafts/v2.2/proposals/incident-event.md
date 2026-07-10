# STIX 2.2 Proposal: Incident and Event

This proposal addresses:

- [oasis-tcs/cti-stix2#1](https://github.com/oasis-tcs/cti-stix2/issues/1) - define a way to capture incident and event information.
- [oasis-tcs/cti-stix2#339](https://github.com/oasis-tcs/cti-stix2/issues/339) - review log transport in STIX, including the Open Cybersecurity Alliance `x-oca-event` extension as prior art.

## Goals

1. Add a first-class STIX Domain Object (SDO) for security incidents.
2. Add a first-class STIX Domain Object (SDO) for security-relevant events and logs.
3. Preserve the STIX 2.1 separation between assertions about security activity and raw cyber-observable evidence.
4. Provide a migration path for existing custom event extensions, especially `x-oca-event`.
5. Ensure that MISP Event packages from the MISP core format can be represented without losing the event-level context that MISP uses to group indicators, objects, reports, tags, galaxies, sightings, and sharing controls.
6. Avoid turning STIX into a high-volume log transport format while still allowing selected logs and events to be exchanged when they are relevant to cyber threat intelligence or incident response.

## Non-goals

- Defining a complete SIEM, EDR, or telemetry schema.
- Replacing OpenC2, CACAO, CASE, or ticketing/workflow systems.
- Requiring producers to normalize every vendor-specific log field into STIX properties.
- Standardizing every possible event action vocabulary in STIX 2.2.
- Replacing the MISP core format or requiring a lossless one-to-one transform for every MISP implementation-specific field.

## Summary of the proposal

STIX 2.2 should introduce two new SDOs:

- **Incident**: a higher-level security incident, investigation, or response case that may aggregate many events, observed data objects, indicators, courses of action, identities, and reports.
- **Event**: a discrete security-relevant occurrence, such as a process creation, file creation, network connection, authentication attempt, alert, or externally observed change in adversary infrastructure.

The Event SDO should describe the occurrence and its role in analysis. Raw log records and detailed observed telemetry should remain in **Artifact** and **Observed Data**. An Event can reference Observed Data, SCOs, and Artifacts to identify the evidence for the occurrence.

## Incident SDO

### Description

An Incident object represents an identified, suspected, or investigated security incident or response case. It can be used for internal incidents, externally reported incidents, public incident repositories, and cases that track incident-response activity.

### Properties

Incident SHOULD use the common STIX object properties and the following object-specific properties.

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string` | Required | A name used to identify the incident. |
| `description` | `string` | Optional | A description of the incident, including relevant context and impact. |
| `incident_types` | `list` of type `open-vocab` | Optional | Categories for the incident, such as `intrusion`, `malware-infection`, `ransomware`, `data-exposure`, `denial-of-service`, `policy-violation`, `phishing`, or `other`. |
| `status` | `open-vocab` | Optional | Current status of the incident. Suggested values: `new`, `triaged`, `investigating`, `contained`, `eradicated`, `recovering`, `resolved`, `closed`, `reopened`, `false-positive`, `duplicate`, `other`. |
| `severity` | `open-vocab` | Optional | Producer-assessed severity. Suggested values: `informational`, `low`, `medium`, `high`, `critical`, `unknown`. |
| `confidence` | `integer` | Optional | Existing STIX confidence property, used to express confidence that this is a real incident or that the incident characterization is accurate. |
| `first_seen` | `timestamp` | Optional | The earliest time the producer believes incident activity occurred or was observed. |
| `last_seen` | `timestamp` | Optional | The latest time the producer believes incident activity occurred or was observed. |
| `detected` | `timestamp` | Optional | The time the incident was detected. |
| `reported` | `timestamp` | Optional | The time the incident was reported to, or by, the producer. |
| `resolved` | `timestamp` | Optional | The time the incident was resolved or closed. |
| `impact` | `string` | Optional | Human-readable description of known or suspected impact. |
| `affected_refs` | `list` of type `identifier` | Optional | References to affected Identities, Locations, Infrastructure, SCOs, or other STIX Objects. |
| `external_references` | `list` of type `external-reference` | Optional | Existing STIX external references, suitable for ticket IDs, public incident IDs, and reports. |

### Relationships from Incident

| Relationship Type | Target | Description |
| --- | --- | --- |
| `based-on` | `observed-data`, `event`, `report`, `note`, SCOs | Evidence or source material used to assert the incident. |
| `contains` | `event`, `observed-data`, `indicator`, `malware`, `attack-pattern`, `course-of-action`, `report`, `note`, `grouping` | Objects that are part of the incident scope. |
| `targets` | `identity`, `location`, `infrastructure`, SCOs | Victims, environments, assets, or resources targeted by the incident. |
| `attributed-to` | `threat-actor`, `intrusion-set`, `campaign`, `identity` | Actor or campaign believed responsible. |
| `uses` | `attack-pattern`, `malware`, `tool`, `infrastructure` | TTPs and resources used in the incident. |
| `mitigated-by` | `course-of-action` | Courses of action that mitigate, contain, eradicate, or recover from the incident. |
| `related-to` | Any SDO/SCO | Generic relationship for cases not covered above. |

## Event SDO

### Description

An Event object represents a discrete security-relevant occurrence. Event is intended for selected events that provide context useful to threat intelligence, investigation, or incident response. Producers SHOULD NOT use Event as a bulk log transport mechanism.

Event is deliberately small and evidence-oriented. It captures the action, time, source/provider, outcome, severity, and references to the relevant Observed Data, SCOs, and raw records.

### Properties

Event SHOULD use the common STIX object properties and the following object-specific properties.

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `action` | `string` | Required | The normalized action or event name, such as `process-created`, `file-created`, `network-connection-created`, `authentication-succeeded`, or `alert-triggered`. |
| `category` | `list` of type `open-vocab` | Optional | High-level categories, such as `process`, `file`, `network`, `authentication`, `registry`, `named-pipe`, `alert`, `cloud`, `email`, `identity`, `external-observation`, or `other`. |
| `description` | `string` | Optional | Additional analyst- or producer-provided context. |
| `start_time` | `timestamp` | Optional | Start time of the event. For instantaneous events, use `start_time` and omit `end_time`. |
| `end_time` | `timestamp` | Optional | End time of the event. MUST NOT be earlier than `start_time` when both are present. |
| `duration` | `integer` | Optional | Duration in milliseconds. If present with `start_time` and `end_time`, it SHOULD be consistent with those timestamps. |
| `code` | `string` | Optional | Event ID, alert ID, or code from the original source. |
| `provider` | `string` | Optional | Product, service, sensor, or authority that produced the event, such as `Windows Security Event Log`, `Sysmon`, or `EDR`. |
| `module` | `string` | Optional | Subsystem, module, dataset, or channel that produced the event. |
| `outcome` | `open-vocab` | Optional | Result of the action. Suggested values: `success`, `failure`, `unknown`, `blocked`, `allowed`, `attempted`, `deferred`, `other`. |
| `severity` | `open-vocab` | Optional | Event severity. Suggested values: `informational`, `low`, `medium`, `high`, `critical`, `unknown`. |
| `source_ref` | `identifier` | Optional | The principal, process, account, host, sensor, or infrastructure that initiated or reported the event. |
| `target_refs` | `list` of type `identifier` | Optional | Objects acted upon or otherwise central to the event. |
| `observed_data_refs` | `list` of type `identifier` | Optional | Observed Data objects that contain SCO evidence for the event. |
| `raw_data_refs` | `list` of type `identifier` | Optional | Artifacts or SCOs that contain original raw records. Targets SHOULD be `artifact` when storing original log records. |

### Relationships from Event

| Relationship Type | Target | Description |
| --- | --- | --- |
| `based-on` | `observed-data`, `artifact`, SCOs | Evidence used to assert the event. |
| `caused` | `event`, `incident` | A later event or incident caused by this event. |
| `contained-in` | `incident`, `report`, `grouping` | A higher-level object that contains or summarizes this event. |
| `targets` | `identity`, `infrastructure`, `location`, SCOs | Object acted upon or affected by the event. |
| `uses` | `tool`, `malware`, `infrastructure`, `attack-pattern`, SCOs | Tools, malware, infrastructure, techniques, or observable resources used during the event. |
| `related-to` | Any SDO/SCO | Generic relationship for cases not covered above. |

## Representing log transport

Issue #339 raises whether STIX should carry logs. This proposal treats STIX as an exchange format for curated security events rather than high-volume raw telemetry.

Recommended pattern:

1. Put raw log records in Artifact when their original form must be preserved.
2. Put normalized observable facts in SCOs and collect them in Observed Data.
3. Use Event to explain what happened between those observables.
4. Use Sighting when an Event, Indicator, Malware, or other SDO is sighted in a particular place or time.
5. Use Incident to aggregate events, observations, response actions, and reporting.

This keeps log provenance and evidence available without requiring STIX consumers to ingest unbounded vendor-specific event streams.

## MISP Event compatibility

The MISP core format uses an Event as the top-level JSON structure for a coherent set of indicators, context, and metadata. A MISP Event may describe an incident, a security analysis report, or a threat actor analysis, so it is not always the same concept as the narrower STIX Event SDO proposed above. To avoid ambiguity, a MISP Event SHOULD map to the STIX object that best represents its analytical intent:

- Map to **Incident** when the MISP Event primarily describes an incident, investigation, response case, victim impact, or remediation activity.
- Map to **Report** when the MISP Event primarily packages an analytic report, finished intelligence, campaign write-up, or threat actor analysis.
- Map to **Grouping** when the MISP Event is mainly a contextual container for related indicators, observables, malware, vulnerabilities, tools, or other STIX objects and no stronger semantic object applies.
- Map to **Event** only for a discrete occurrence or alert-like record inside the MISP Event, not for the MISP Event container itself unless the MISP Event truly represents one occurrence.

This proposal therefore treats MISP Event support as a packaging and context-mapping requirement. STIX 2.2 should allow a producer to preserve the meaning of the MISP Event while expressing the contained attributes and objects using existing STIX concepts wherever possible.

### Mapping from MISP Event

| MISP core format element | Proposed STIX 2.2 representation | Notes |
| --- | --- | --- |
| `Event.uuid` | STIX object `id` UUID component, or `external_references.external_id` | Prefer deterministic STIX IDs derived from the MISP UUID when a producer needs stable round-tripping. |
| `Event.info` | `incident.name`, `report.name`, or `grouping.name`; longer text MAY go in `description` | MISP `info` is the human-readable event summary. |
| `Event.date` | `incident.first_seen`/`last_seen`, `report.published`, or object `created` as appropriate | MISP date is day-granular; preserve exact MISP value in an external reference or extension if needed. |
| `Event.timestamp` | STIX `modified` or extension property preserving the MISP timestamp | Use STIX `modified` only when it represents the STIX object version time. |
| `Event.publish_timestamp` / `published` | `report.published`, extension metadata, or external reference metadata | STIX does not have a general publication-state property for every SDO. |
| `Event.threat_level_id` | `incident.severity` or a MISP extension property | Suggested mapping: `1` to `high`, `2` to `medium`, `3` to `low`, `4` to `unknown`. |
| `Event.analysis` | `incident.status` or extension property | Suggested mapping: `0` to `new`/`triaged`, `1` to `investigating`, `2` to `resolved` or a producer-specific `analysis-complete` extension value. |
| `Org` / `Orgc` | `identity` objects referenced by `created_by_ref`, `object_marking_refs`, or relationships | Preserve both source organization and creator organization when they differ. |
| `Attribute` | STIX Indicator, Observed Data, SCO, or Artifact | Use Indicator when the MISP attribute is actionable detection logic or an IOC assertion; use SCO/Observed Data for observed facts; use Artifact for raw payloads. |
| `Object` | SCOs, SDOs, or Grouping depending on object template semantics | MISP object templates SHOULD be mapped by template-specific profiles rather than a single generic rule. |
| `ObjectReference` | STIX Relationship or embedded reference | Preserve the MISP relationship name as `relationship_type` when it is valid or as an extension property when it is not. |
| `Tag` and taxonomy tags | `labels`, `external_references`, `object_marking_refs`, or extension properties | TLP and PAP tags SHOULD become Marking Definitions where possible; other taxonomies can remain labels or extension metadata. |
| `Galaxy` / `GalaxyCluster` | STIX Malware, Tool, Attack Pattern, Intrusion Set, Threat Actor, Campaign, Vulnerability, Location, Identity, or external references | Map by galaxy type; preserve original galaxy and cluster UUIDs for round-tripping. |
| `EventReport` | STIX Report or Note | Use Report for standalone analyst reports and Note for annotations attached to a mapped Incident, Report, Grouping, or Event. |
| `Sighting` | STIX Sighting | Preserve sighting timestamp, source, and target where available. |
| `distribution`, `sharing_group_id`, local ACL fields | Marking Definitions or extension properties | STIX markings can capture common sharing constraints, but MISP sharing groups may require an extension or external policy reference. |
| `proposal` / `ShadowAttribute` | STIX Note, Opinion, or extension object | These are collaborative change suggestions rather than direct observations. |

### MISP Event representation pattern

A STIX bundle translated from a MISP Event SHOULD use one primary container object and relate the translated content to that object:

1. Create an Incident, Report, or Grouping for the MISP Event container.
2. Preserve the MISP Event UUID and local identifiers in `external_references` or a MISP extension.
3. Convert MISP Attributes and Objects into the most specific STIX Indicators, Observed Data, SCOs, Artifacts, or SDOs.
4. Attach converted content to the primary container with `contains`, `object`, `based-on`, `related-to`, or a more specific relationship when one exists.
5. Convert Event Reports to Report or Note and link them to the primary container.
6. Convert MISP Sightings to STIX Sighting objects.
7. Convert distribution constraints and sharing groups into STIX markings when possible and extension metadata when MISP-specific semantics must be retained.

### Example: MISP Event as a STIX Grouping

```json
{
  "type": "bundle",
  "id": "bundle--6d77cc83-9313-4f4c-bd98-54f5977b1a57",
  "objects": [
    {
      "type": "grouping",
      "spec_version": "2.2",
      "id": "grouping--8f70f1d1-2c88-4b3f-91f9-678098ab3f8f",
      "created": "2026-04-27T00:00:00.000Z",
      "modified": "2026-04-27T12:00:31.000Z",
      "name": "IoT malware - Gafgyt.Gen28 (active)",
      "context": "misp-event",
      "external_references": [
        {
          "source_name": "misp",
          "external_id": "8f70f1d1-2c88-4b3f-91f9-678098ab3f8f"
        }
      ],
      "object_refs": [
        "indicator--52bfa2cb-3f6b-4ee8-9845-230e14729210",
        "malware--0c7b5b88-8ff7-4a4d-aa9d-feb398cd0061"
      ]
    },
    {
      "type": "indicator",
      "spec_version": "2.2",
      "id": "indicator--52bfa2cb-3f6b-4ee8-9845-230e14729210",
      "created": "2026-04-27T00:00:00.000Z",
      "modified": "2026-04-27T12:00:31.000Z",
      "name": "MISP attribute: domain",
      "pattern_type": "stix",
      "pattern": "[domain-name:value = 'example.invalid']",
      "valid_from": "2026-04-27T00:00:00Z",
      "labels": ["misp:type=domain", "misp:category=Network activity"]
    }
  ]
}
```

## Mapping from `x-oca-event`

The OCA `x-oca-event` object is useful prior art. STIX 2.2 should reuse its core concepts while narrowing the standard object to properties that are broadly applicable across producers.

| `x-oca-event` property | Proposed STIX 2.2 representation |
| --- | --- |
| `action` | `event.action` |
| `category` | `event.category` |
| `code` | `event.code` |
| `description` | `event.description` |
| `start` | `event.start_time` |
| `end` | `event.end_time` |
| `duration` | `event.duration` in milliseconds |
| `module` | `event.module` |
| `provider` | `event.provider` |
| `outcome` | `event.outcome` |
| `severity` | `event.severity` |
| `original_ref` | `event.raw_data_refs` or `based-on` relationship to `artifact` |
| `host_ref`, `user_ref`, `process_ref`, `file_ref`, `network_ref`, `url_ref`, `domain_ref`, `registry_ref`, `ip_refs` | `event.source_ref`, `event.target_refs`, or `based-on` relationships to the referenced SCOs, depending on event semantics |
| `parent_process_ref`, `cross_process_target_ref` | Existing SCO relationships on `process`, plus `event.source_ref` / `event.target_refs` when needed for event semantics |
| `agent`, `dataset`, `timezone`, `pipe_name`, vendor-specific fields | Custom extension properties on Event, Artifact, or SCOs |

## Example: file creation event in an incident

```json
{
  "type": "bundle",
  "id": "bundle--d5f29f4d-189b-4f6a-bf41-189f3cfed303",
  "objects": [
    {
      "type": "incident",
      "spec_version": "2.2",
      "id": "incident--8b178041-e2f9-47f1-9725-1f1192f5c9f8",
      "created": "2026-04-16T13:00:00.000Z",
      "modified": "2026-04-16T13:00:00.000Z",
      "name": "Suspicious download on workstation",
      "incident_types": ["malware-infection"],
      "status": "investigating",
      "severity": "medium",
      "detected": "2026-04-16T12:42:00.000Z"
    },
    {
      "type": "event",
      "spec_version": "2.2",
      "id": "event--f5d76c11-9953-4df0-a770-70ff73d6cc69",
      "created": "2026-04-16T12:43:10.813Z",
      "modified": "2026-04-16T12:43:10.813Z",
      "action": "file-created",
      "category": ["file"],
      "start_time": "2026-04-16T12:43:10.813Z",
      "provider": "EDR",
      "outcome": "success",
      "source_ref": "process--5f4bbeb7-a991-4057-89cf-ea8f012f5e94",
      "target_refs": ["file--1bd39816-c38f-5d1c-a768-0420c397b79d"],
      "observed_data_refs": ["observed-data--7805aca6-b29d-4e1a-86b2-ba4eb1110046"],
      "raw_data_refs": ["artifact--ca17bcf8-9846-5ab4-8662-75c1bf6e63ee"]
    },
    {
      "type": "relationship",
      "spec_version": "2.2",
      "id": "relationship--d47393a9-b632-40c2-8f7c-fc5082a6b025",
      "created": "2026-04-16T13:00:00.000Z",
      "modified": "2026-04-16T13:00:00.000Z",
      "relationship_type": "contains",
      "source_ref": "incident--8b178041-e2f9-47f1-9725-1f1192f5c9f8",
      "target_ref": "event--f5d76c11-9953-4df0-a770-70ff73d6cc69"
    }
  ]
}
```

## Open questions for TC discussion

1. Should Event be an SDO, an SRO with additional properties, or both? This proposal recommends an SDO because events can be sighted, aggregated, enriched, and referenced as analysis objects.
2. Should Incident include workflow state (`status`) in core STIX, or should detailed workflow remain an extension? This proposal includes only a coarse status vocabulary.
3. Should `severity` be an open vocabulary or a numeric scale? This proposal recommends open vocabulary in core STIX and allows numeric scoring through extensions.
4. Should `source_ref` and `target_refs` be constrained to SCOs only, or allow SDOs and SCOs? This proposal allows both because externally observed events may target Identities, Infrastructure, Campaigns, or other SDOs.
5. Should the Event object include strongly typed convenience references such as `process_ref` and `file_ref`? This proposal recommends generic source/target/evidence references in core STIX, with domain-specific typed references in extensions.
6. Should STIX 2.2 define a standard MISP extension to preserve fields such as `published`, `distribution`, `sharing_group_id`, `analysis`, and local IDs, or should those remain implementation-specific extension content?

## Proposed next steps

1. Discuss whether Incident and Event should be accepted as STIX 2.2 candidate SDOs.
2. If accepted, add normative object definitions, relationship tables, examples, and vocabularies to the STIX 2.2 draft specification.
3. Create JSON schemas and conformance tests for both objects.
4. Publish migration guidance for common custom event objects, beginning with `x-oca-event`.
5. Publish a MISP Event conversion profile that defines recommended mappings for common MISP Attribute types, Object templates, taxonomies, galaxies, sightings, and sharing controls.
