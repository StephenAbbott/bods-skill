# BODS v0.4 — Detailed Reference

Source: https://standard.openownership.org/en/0.4.0/

---

## Table of Contents

1. [Overview and purpose](#overview)
2. [Dataset structure and serialization](#dataset-structure)
3. [Top-level statement fields](#top-level-statement-fields)
4. [Entity Statement — full field reference](#entity-statement)
5. [Person Statement — full field reference](#person-statement)
6. [Relationship Statement — full field reference](#relationship-statement)
7. [Interests object](#interests-object)
8. [Codelists](#codelists)
9. [Identifiers and schemes](#identifiers-and-schemes)
10. [Source and provenance](#source-and-provenance)
11. [Addresses](#addresses)
12. [Examples — complete dataset](#examples)
13. [Validation and tooling](#validation-and-tooling)

---

## Overview

The Beneficial Ownership Data Standard (BODS) provides a specification for modelling and publishing information on the beneficial ownership and control of companies and other legal entities. It is designed to support transparency and anti-money laundering efforts globally.

BODS data is composed of **Statements** — immutable, time-stamped assertions about entities, persons, and the relationships between them. The standard uses JSON (or JSON Lines) and is defined using JSON Schema 2020-12.

**Key design principles:**
- Statements are immutable. Updates are published as new statements.
- Every statement has a globally unique `statementId`.
- Statements can be linked to declarations (filings) via `declaration` and `declarationSubject`.
- The standard supports both direct and indirect beneficial ownership.
- Unknown or anonymous parties are first-class citizens in the model.

---

## Dataset Structure

A BODS dataset is an array (or newline-delimited stream) of Statement objects. Statements reference each other by `statementId`:

```json
[
  { "statementId": "entity-001", "recordDetails": { /* entity */ } },
  { "statementId": "person-001", "recordDetails": { /* person */ } },
  {
    "statementId": "rel-001",
    "recordDetails": {
      "subject": { "describedByEntityStatement": "entity-001" },
      "interestedParty": { "describedByPersonStatement": "person-001" },
      "interests": [ ... ]
    }
  }
]
```

**Serialization options:**
- Standard JSON array: `[{...}, {...}, {...}]`
- JSON Lines (JSONL): one statement per line, no array wrapper — preferred for large datasets

---

## Top-Level Statement Fields

These fields appear on every statement regardless of type.

| Field | Type | Required | Description |
|---|---|---|---|
| `statementId` | string | Yes | Globally unique, persistent ID for this statement. Use a UUID or similar. |
| `declarationSubject` | string | No | ID of the entity/subject this statement is filed against. |
| `declaration` | string | No | ID of the parent declaration (filing/submission) this statement belongs to. |
| `recordId` | string | No | Stable ID shared across versions of the same record. Same value across `new`/`updated`/`closed` statements for one record. |
| `recordStatus` | enum | No | `new` \| `updated` \| `closed`. Tracks lifecycle of the record. |
| `statementDate` | date | No | ISO 8601 date when the statement was made (e.g. `2024-01-15`). |
| `source` | object | No | Provenance information (see [Source](#source-and-provenance)). |
| `annotations` | array | No | Free-form annotations or qualifications on the statement. |
| `recordDetails` | object | Yes | The type-specific payload — contains entity, person, or relationship data. Also specifies the record type via its structure. |

---

## Entity Statement

Describes a legal entity: a company, trust, foundation, or other arrangement.

### `recordDetails` fields for entity

| Field | Type | Required | Description |
|---|---|---|---|
| `entityType` | object | Yes | Type classification of the entity (see below). |
| `name` | string | No | Primary name of the entity. |
| `alternateNames` | array[string] | No | Other names by which the entity is known. |
| `incorporatedInJurisdiction` | object | No | Jurisdiction of incorporation (`name`, `code` using ISO 3166-1 alpha-2). |
| `identifiers` | array | No | Registered identifiers (company numbers, LEIs, etc.) — see [Identifiers](#identifiers). |
| `foundingDate` | date | No | Date entity was formed/incorporated. |
| `dissolutionDate` | date | No | Date entity was dissolved/wound up. |
| `addresses` | array | No | Registered or operational addresses. |
| `uri` | string | No | URI for the entity's official record. |
| `formedByStatute` | object | No | *(v0.4 new)* For state bodies formed by legislation: `{ "name": "Act name", "date": "YYYY-MM-DD" }`. |
| `entitySubtypeCategory` | enum | No | *(v0.4 new)* Subtype for state bodies — e.g. `stateAuthority`, `stateCorporation`. |

### Entity type codes (`entityType.type`)

| Code | Meaning |
|---|---|
| `registeredEntity` | A company or other entity registered with an official registry |
| `legalEntity` | A legal entity not registered with a standard registry |
| `arrangement` | A legal arrangement such as a trust or partnership |
| `anonymousEntity` | Entity exists but cannot be identified |
| `unknownEntity` | It is unknown whether an entity exists |
| `state` | A national or subnational state |
| `stateBody` | A body established by or on behalf of a state |

---

## Person Statement

Describes a natural person — a beneficial owner, controller, or other interested party.

### `recordDetails` fields for person

| Field | Type | Required | Description |
|---|---|---|---|
| `personType` | enum | Yes | `knownPerson` \| `anonymousPerson` \| `unknownPerson` |
| `names` | array | No | Name objects — see below. |
| `identifiers` | array | No | Identity documents or official IDs — see [Identifiers](#identifiers). |
| `nationalities` | array | No | Array of `{ "code": "GB", "name": "British" }` objects. |
| `birthDate` | date | No | ISO 8601 date of birth (`YYYY-MM-DD` or partial `YYYY-MM`). |
| `deathDate` | date | No | ISO 8601 date of death. |
| `placeOfBirth` | object | No | `{ "address": "...", "country": "GB" }` |
| `addresses` | array | No | Residential or other addresses. |
| `politicalExposure` | object | No | PEP status: `{ "status": "isPep" \| "isNotPep" \| "unknown", "details": [...] }` |
| `source` | object | No | Statement-level source override. |

### Name object

```json
{
  "type": "individual",
  "fullName": "Jane Elizabeth Smith",
  "familyName": "Smith",
  "givenName": "Jane",
  "patronymicName": "Elizabeth"
}
```

**Name types:** `individual`, `translation`, `former`, `alias`, `birth`

---

## Relationship Statement

Describes the interests (ownership/control) an interested party holds in a subject entity.

### `recordDetails` fields for relationship

| Field | Type | Required | Description |
|---|---|---|---|
| `subject` | object | Yes | The entity being owned/controlled. One of: `describedByEntityStatement` (statementId), or `describedByRecord` for a recordId reference. |
| `interestedParty` | object | Yes | The owner/controller. One of: `describedByPersonStatement`, `describedByEntityStatement`, or `unspecified` (with reason). |
| `interests` | array | No | Array of interest objects detailing the nature of the relationship. |
| `isComponent` | boolean | No | Whether this statement is a component of a broader indirect ownership chain. |

### `interestedParty.unspecified`

When the interested party cannot be identified:
```json
{
  "unspecified": {
    "reason": "unknown-unknown",
    "description": "The beneficial owner could not be determined."
  }
}
```
**Unspecified reasons:** `no-beneficial-owners`, `subject-unable-to-confirm`, `information-unknown-to-register`, `interested-party-has-not-responded`, `unknown-unknown`

---

## Interests Object

Each item in the `interests` array describes one type of ownership or control interest.

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | enum | Yes | Interest type — see codelist below. |
| `directOrIndirect` | enum | No | `direct` \| `indirect` \| `unknown` |
| `beneficialOwnershipOrControl` | boolean | No | Whether this interest constitutes beneficial ownership or control under the relevant jurisdiction's definition. |
| `details` | string | No | Free-text description of the interest. |
| `share` | object | No | Proportion of interest (see below). |
| `startDate` | date | No | When this interest began. |
| `endDate` | date | No | When this interest ended. |
| `isComponent` | boolean | No | Whether this is a component interest in a chain. |

### Share object

```json
{
  "exact": 51.0,
  "minimum": 50.0,
  "maximum": 75.0,
  "exclusiveMinimum": 50.0,
  "exclusiveMaximum": 75.0
}
```
Provide `exact` when known. Use `minimum`/`maximum` for ranges (inclusive bounds). Use `exclusiveMinimum`/`exclusiveMaximum` when the threshold itself is excluded (e.g. "more than 25%" = `exclusiveMinimum: 25`).

---

## Codelists

### Interest types (v0.4 — camelCase)

| Code | Added | Description |
|---|---|---|
| `shareholding` | v0.1 | Ownership of shares/equity |
| `votingRights` | v0.1 | Rights to vote |
| `appointmentOfBoard` | v0.1 | Power to appoint/remove board members |
| `otherInfluenceOrControl` | v0.1 | Other influence or control not captured elsewhere |
| `controlViaCompanyRulesOrArticles` | v0.4 | Control through articles or company constitution |
| `controlByLegalFramework` | v0.4 | Control arising from law or regulation |
| `boardMember` | v0.4 | Membership of governing board |
| `boardChair` | v0.4 | Chair of governing board |
| `unknownInterest` | v0.4 | Type of interest is unknown |
| `unpublishedInterest` | v0.4 | Interest exists but is not published |
| `enjoymentAndUseOfAssets` | v0.4 | Right to enjoy or use assets |
| `rightToProfitOrIncomeFromAssets` | v0.4 | Right to profit or income from assets |

> **v0.3 migration:** Codes were hyphenated in v0.3 (e.g. `voting-rights`, `appointment-of-board`). In v0.4 all codes use camelCase.

### Record status

| Code | Meaning |
|---|---|
| `new` | First publication of this record |
| `updated` | Replaces a previously published record (same `recordId`) |
| `closed` | Record is no longer active |

### Person type

`knownPerson` | `anonymousPerson` | `unknownPerson`

### Direct or indirect

`direct` | `indirect` | `unknown`

---

## Identifiers and Schemes

Identifiers link statements to real-world registrations and databases.

```json
{
  "id": "12345678",
  "scheme": "GB-COH",
  "schemeName": "Companies House",
  "uri": "https://find-and-update.company-information.service.gov.uk/company/12345678"
}
```

**Common entity identifier schemes:**
- `GB-COH` — UK Companies House
- `US-EIN` — US Employer Identification Number
- `LEI` — Legal Entity Identifier (Global)
- `EU-VAT` — EU VAT registration number
- Country-specific registry codes follow the format `{ISO2}-{REGISTRY}`

**Common person identifier schemes:**
- `GB-PASS` — UK passport
- `MISC-NationalId` — National identity document (use with `schemeName` to specify country/type)

---

## Source and Provenance

The `source` object records where the information came from.

```json
{
  "type": "officialRegister",
  "description": "UK Companies House PSC register",
  "url": "https://find-and-update.company-information.service.gov.uk/",
  "retrievedAt": "2024-01-15T10:30:00Z",
  "assertedBy": [
    {
      "name": "Companies House",
      "uri": "https://www.gov.uk/government/organisations/companies-house"
    }
  ]
}
```

**Source types:** `officialRegister`, `selfDeclaration`, `thirdParty`, `primaryResearch`, `verified`, `other`

---

## Addresses

```json
{
  "type": "registered",
  "address": "123 High Street, London, EC1A 1BB",
  "postCode": "EC1A 1BB",
  "country": "GB"
}
```

**Address types:** `registered`, `service`, `residence`, `business`, `alternative`

---

## Examples — Complete Dataset

### Simple direct ownership (one person owns one company)

```json
[
  {
    "statementId": "entity-acme-001",
    "statementDate": "2024-01-15",
    "recordId": "acme-ltd-record",
    "recordStatus": "new",
    "recordDetails": {
      "entityType": { "type": "registeredEntity" },
      "name": "Acme Holdings Ltd",
      "incorporatedInJurisdiction": { "code": "GB", "name": "United Kingdom" },
      "identifiers": [{ "id": "12345678", "scheme": "GB-COH", "schemeName": "Companies House" }],
      "foundingDate": "2010-03-01"
    }
  },
  {
    "statementId": "person-smith-001",
    "statementDate": "2024-01-15",
    "recordId": "jane-smith-record",
    "recordStatus": "new",
    "recordDetails": {
      "personType": "knownPerson",
      "names": [{ "type": "individual", "fullName": "Jane Smith" }],
      "nationalities": [{ "code": "GB", "name": "British" }],
      "birthDate": "1975-08-22"
    }
  },
  {
    "statementId": "rel-acme-smith-001",
    "statementDate": "2024-01-15",
    "recordId": "acme-smith-ownership-record",
    "recordStatus": "new",
    "recordDetails": {
      "subject": { "describedByEntityStatement": "entity-acme-001" },
      "interestedParty": { "describedByPersonStatement": "person-smith-001" },
      "interests": [
        {
          "type": "shareholding",
          "directOrIndirect": "direct",
          "beneficialOwnershipOrControl": true,
          "share": { "exact": 75.0 },
          "startDate": "2019-06-01"
        },
        {
          "type": "votingRights",
          "directOrIndirect": "direct",
          "beneficialOwnershipOrControl": true,
          "share": { "exact": 75.0 }
        }
      ]
    }
  }
]
```

### Updating a record

```json
{
  "statementId": "rel-acme-smith-002",
  "statementDate": "2024-06-01",
  "recordId": "acme-smith-ownership-record",
  "recordStatus": "updated",
  "recordDetails": {
    "subject": { "describedByEntityStatement": "entity-acme-001" },
    "interestedParty": { "describedByPersonStatement": "person-smith-001" },
    "interests": [
      {
        "type": "shareholding",
        "directOrIndirect": "direct",
        "beneficialOwnershipOrControl": true,
        "share": { "exact": 100.0 },
        "startDate": "2024-06-01"
      }
    ]
  }
}
```

### Unknown beneficial owner

```json
{
  "statementId": "rel-acme-unknown-001",
  "statementDate": "2024-01-15",
  "recordDetails": {
    "subject": { "describedByEntityStatement": "entity-acme-001" },
    "interestedParty": {
      "unspecified": {
        "reason": "information-unknown-to-register",
        "description": "The register does not hold beneficial owner information for this entity."
      }
    },
    "interests": [
      {
        "type": "unknownInterest",
        "beneficialOwnershipOrControl": true
      }
    ]
  }
}
```

---

## Validation and Tooling

**Official schema (JSON Schema 2020-12):**
https://github.com/openownership/data-standard/tree/main/schema

**BODS validator:**
Use the `bods-validate` Python package:
```bash
pip install bods-validate
bods-validate my-bods-data.json
```

**Flatten Tool (CSV ↔ BODS):**
https://flatten-tool.readthedocs.io/en/latest/usage-bods/

**Data generator / test data:**
https://www.openownership.org/en/publications/beneficial-ownership-data-standard-generator/

**Official documentation:**
- v0.4 docs: https://standard.openownership.org/en/0.4.0/
- Schema browser: https://standard.openownership.org/en/0.4.0/standard/schema-browser.html
- Changelog: https://standard.openownership.org/en/0.4.0/standard/changelog.html
- GitHub: https://github.com/openownership/data-standard
