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

BODS data is composed of **Statements** — immutable, time-stamped assertions about entities, persons, and the relationships between them. Each Statement describes one **record** (identified by `recordId`) at a point in time; later Statements about the same record share its `recordId`. The standard uses JSON (or JSON Lines) and is defined using JSON Schema 2020-12.

**Key design principles:**
- Statements are immutable. Updates are published as new statements.
- Every statement has a globally unique `statementId` (32–64 characters, e.g. a UUID).
- Relationships point at other records by **`recordId`**, not by `statementId`.
- Statements can be linked to declarations (filings) via `declaration` and `declarationSubject`.
- The standard supports both direct and indirect beneficial ownership.
- Unknown or anonymous parties are first-class citizens in the model.

---

## Dataset Structure

A BODS dataset is an array (or newline-delimited stream) of Statement objects. A relationship's `subject` and `interestedParty` hold the **`recordId`** of the entity or person record they refer to (a plain string — there is no `describedByEntityStatement` / `describedByPersonStatement` wrapper in v0.4):

```json
[
  { "statementId": "…", "recordId": "entity-001", "recordType": "entity", "recordDetails": { /* entity */ } },
  { "statementId": "…", "recordId": "person-001", "recordType": "person", "recordDetails": { /* person */ } },
  {
    "statementId": "…",
    "recordId": "rel-001",
    "recordType": "relationship",
    "recordDetails": {
      "isComponent": false,
      "subject": "entity-001",
      "interestedParty": "person-001",
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
| `statementId` | string | Yes | Globally unique, persistent ID for this statement, 32–64 characters. Use a UUID or similar. |
| `statementDate` | date | Yes | Date (or date-time) the statement was made (e.g. `2024-01-15`). |
| `declarationSubject` | string | Yes | `recordId` of the entity that is the subject of the declaration this statement belongs to. |
| `recordId` | string | Yes | Stable ID for the record, shared by every statement about it (`new` / `updated` / `closed`). Unique within the publisher's system. |
| `recordType` | enum | Yes | `entity` \| `person` \| `relationship`. Selects which `recordDetails` schema applies. |
| `recordDetails` | object | Yes | The type-specific payload for the entity, person or relationship. |
| `recordStatus` | enum | No | `new` \| `updated` \| `closed`. Tracks lifecycle of the record. |
| `declaration` | string | No | ID of the parent declaration (filing/submission) this statement belongs to. |
| `publicationDetails` | object | No | `{ "publicationDate", "bodsVersion": "0.4", "publisher": { "name" / "url" }, "license" }` — if present, `publicationDate`, `bodsVersion` and `publisher` are required. |
| `source` | object | No | Provenance information (see [Source](#source-and-provenance)). |
| `annotations` | array | No | Annotations on the statement (see the `annotationMotivation` codelist: `commenting`, `correcting`, `identifying`, `linking`, `transformation`). |

---

## Entity Statement

Describes a legal entity: a company, trust, foundation, or other arrangement.

### `recordDetails` fields for entity

| Field | Type | Required | Description |
|---|---|---|---|
| `isComponent` | boolean | Yes | `true` if this record is a component of an indirect relationship. |
| `entityType` | object | Yes | `{ "type": ..., "subtype": ..., "details": ... }` — `type` required; `subtype` from the entitySubtype codelist (see [Codelists](#codelists)); `details` for a local name for the entity form. In v0.4 this replaces v0.3's separate `entitySubtype` / `entitySubtypeCategory`. |
| `name` | string | No | Primary name of the entity. |
| `alternateNames` | array[string] | No | Other names by which the entity is known. |
| `jurisdiction` | object | No | Jurisdiction of incorporation/registration: `{ "name": "United Kingdom", "code": "GB" }` — `name` required; `code` from ISO 3166-1 alpha-2 or ISO 3166-2. (Called `incorporatedInJurisdiction` before v0.3.) |
| `identifiers` | array | No | Registered identifiers (company numbers, LEIs, etc.) — see [Identifiers](#identifiers-and-schemes). |
| `foundingDate` | date | No | Date entity was formed/incorporated. |
| `dissolutionDate` | date | No | Date entity was dissolved/wound up. |
| `addresses` | array | No | Registered or operational addresses. |
| `uri` | string | No | URI for the entity's official record. |
| `publicListing` | object | No | Listed-company details: `hasPublicListing` (required), `companyFilingsURLs`, `securitiesListings`. |
| `unspecifiedEntityDetails` | object | No | Unspecified Record (`reason`, `description`) for an `anonymousEntity` or `unknownEntity`. |
| `formedByStatute` | object | No | *(v0.3)* For state bodies formed by legislation: `{ "name": "Act name", "date": "YYYY-MM-DD" }`. |

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
| `isComponent` | boolean | Yes | `true` if this record is a component of an indirect relationship. |
| `personType` | enum | Yes | `knownPerson` \| `anonymousPerson` \| `unknownPerson` |
| `unspecifiedPersonDetails` | object | No | Unspecified Record (`reason`, `description`) for an `anonymousPerson` or `unknownPerson`. |
| `names` | array | No | Name objects — see below. |
| `identifiers` | array | No | Identity documents or official IDs — see [Identifiers](#identifiers-and-schemes). |
| `nationalities` | array | No | Country objects: `{ "name": "United Kingdom", "code": "GB" }` (`name` required). |
| `birthDate` | date | No | `YYYY-MM-DD`, or `YYYY-MM` / `YYYY` where the full date is not known. |
| `deathDate` | date | No | Date of death, same format. |
| `placeOfBirth` | object | No | An Address object, e.g. `{ "type": "placeOfBirth", "address": "Leeds", "country": { "name": "United Kingdom", "code": "GB" } }` |
| `taxResidencies` | array | No | Country objects. |
| `addresses` | array | No | Address objects (see [Addresses](#addresses)). |
| `politicalExposure` | object | No | PEP status: `{ "status": "isPep" \| "isNotPep" \| "unknown", "details": [...] }` (`status` required). |

### Name object

```json
{
  "type": "legal",
  "fullName": "Jane Elizabeth Smith",
  "familyName": "Smith",
  "givenName": "Jane",
  "patronymicName": "Elizabeth"
}
```

`fullName` is required on every name object.

**Name types (`nameType` codelist, closed):**

| Code | Meaning |
|---|---|
| `legal` | The name used for legal, administrative and other official purposes — usually the one on official government documents |
| `translation` | A translation of the legal name in a different language |
| `transliteration` | A transliteration of the legal name in a different script |
| `former` | A name the person has used in the past |
| `alternative` | Another name the person is known by — an alias, a nickname or an "also known as" |
| `birth` | The legal name of the person at birth |

> There is **no** `individual` code (renamed to `legal` in v0.4) and **no** `alias` code (use `alternative`). When picking a display name from `names`, prefer `type == "legal"` — do not rely on array order.

---

## Relationship Statement

Describes the interests (ownership/control) an interested party holds in a subject entity.

### `recordDetails` fields for relationship

| Field | Type | Required | Description |
|---|---|---|---|
| `isComponent` | boolean | Yes | `true` if this relationship is a component of an indirect relationship. |
| `subject` | string or object | Yes | `recordId` of the entity being owned/controlled (always an entity), or an Unspecified Record. |
| `interestedParty` | string or object | Yes | `recordId` of the owner/controller (person or entity), or an Unspecified Record with a reason. |
| `interests` | array | No | Array of interest objects detailing the nature of the relationship. |
| `componentRecords` | array[string] | No | `recordId`s of the component records that make up an indirect relationship. If present, this relationship's `isComponent` MUST be `false`. |

### Unspecified Record

When the interested party cannot be identified, put an Unspecified Record object **directly** in `interestedParty` (there is no `unspecified` wrapper). `reason` is required:

```json
"interestedParty": {
  "reason": "unknown",
  "description": "The beneficial owner could not be determined."
}
```

Use it only where nothing is known about interested parties beyond this point. If a person or entity is known to exist but their identity is unavailable, give them a record with `personType: "anonymousPerson"` / `"unknownPerson"` (or `entityType.type: "anonymousEntity"` / `"unknownEntity"`) and reference its `recordId` instead.

**Unspecified reasons (`unspecifiedReason` codelist, closed — seven codes):**

| Code | Meaning |
|---|---|
| `noBeneficialOwners` | There are no beneficial owners who need to disclose ownership under the rules the statement is made under |
| `subjectUnableToConfirmOrIdentifyBeneficialOwner` | The subject, as the disclosing party, has been unwilling or unable to confirm or identify a beneficial owner |
| `interestedPartyHasNotProvidedInformation` | The interested party has not provided enough information to identify or confirm the beneficial owner |
| `subjectExemptFromDisclosure` | The subject is not required to disclose its beneficial owner |
| `interestedPartyExemptFromDisclosure` | The interested party is exempt from having their identity disclosed |
| `unknown` | The reason the party cannot be provided is not known |
| `informationUnknownToPublisher` | The publisher does not have access to the information. Should not generally be used where one party has the responsibility to provide it |

> **Exemption ≠ no beneficial owner.** `subjectExemptFromDisclosure` ("we were never required to look") asserts something materially different from `noBeneficialOwners` ("we looked and there is none"). Don't collapse exemptions onto `noBeneficialOwners`.
>
> These codes have been camelCase since v0.3 (hyphenated in v0.2, e.g. `subject-exempt-from-disclosure`). Values such as `unknown-unknown`, `unknownUnknown` or `information-unknown-to-register` do not exist in any BODS version.

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

### Share object

```json
{ "exact": 51.0 }
{ "minimum": 25.0, "maximum": 50.0 }
{ "exclusiveMinimum": 25.0, "maximum": 50.0 }
```
All values are percentages (0–100). Provide `exact` when known; otherwise give a range — don't combine `exact` with range bounds. Use `minimum`/`maximum` for inclusive bounds. Use `exclusiveMinimum`/`exclusiveMaximum` when the threshold itself is excluded (e.g. "more than 25%" = `exclusiveMinimum: 25`).

---

## Codelists

> **Codelist values in this file are BODS v0.4.** They are summaries — the CSVs in [`schema/codelists/`](https://github.com/openownership/data-standard/tree/0.4.0/schema/codelists) (or `libcovebods/data/schema-0-4-0/`) are authoritative. Check there before hard-coding values in a mapper: codelist spelling **and** membership have changed between versions, so a camelCased old value is not automatically a valid v0.4 value. All BODS v0.4 codelists are closed (`openCodelist: false`), so an unlisted value fails schema validation.

### Interest types (`interestType`, 23 codes)

| Code | Introduced | Description |
|---|---|---|
| `shareholding` | ≤ v0.2 | Economic interest gained by holding shares |
| `votingRights` | ≤ v0.2 | Controlling interest from the right to vote on matters of corporate policy |
| `appointmentOfBoard` | ≤ v0.2 | Absolute right to appoint members of the board |
| `otherInfluenceOrControl` | ≤ v0.2 | Influence or control distinct from shareholding, voting rights or board appointment |
| `seniorManagingOfficial` | ≤ v0.2 | Control over the management of the entity gained by employment |
| `settlor` | ≤ v0.2 | Person who creates a trust (or settlor-equivalent in a similar arrangement) |
| `trustee` | ≤ v0.2 | Person who administers a trust and holds legal title to its property |
| `protector` | ≤ v0.2 | Person appointed to protect the settlor's interests or wishes |
| `beneficiaryOfLegalArrangement` | ≤ v0.2 | Person who benefits from a trust or other legal arrangement |
| `rightsToSurplusAssetsOnDissolution` | ≤ v0.2 | Right to a share of surplus assets on winding up |
| `rightsToProfitOrIncome` | ≤ v0.2 | Rights to receive profits or income, granted by contract |
| `rightsGrantedByContract` | ≤ v0.2 | An interest granted by contract |
| `conditionalRightsGrantedByContract` | ≤ v0.2 | An interest that exists only if a contractual condition is met |
| `controlViaCompanyRulesOrArticles` | v0.3 | Control through company articles or a shareholder agreement |
| `controlByLegalFramework` | v0.3 | Control arising from legislation (typically state-linked entities) |
| `boardMember` | v0.3 | Member of the governing board |
| `boardChair` | v0.3 | Chair of the board |
| `unknownInterest` | v0.3 | An interest is known to exist but its nature is unknown |
| `unpublishedInterest` | v0.3 | The nature of the interest is known but not published |
| `enjoymentAndUseOfAssets` | v0.3 | Use of assets belonging to an entity |
| `rightToProfitOrIncomeFromAssets` | v0.3 | Right to profits or income from an entity's assets |
| `nominee` | v0.4 | Person who acts on behalf of a nominator in a specified capacity |
| `nominator` | v0.4 | Person who instructs a nominee to act on their behalf |

> **Version history:** codes were hyphenated up to v0.2 (e.g. `voting-rights`, `settlor-of-trust`) and became camelCase in **v0.3**, when the trust codes also lost `OfTrust` (`beneficiary-of-trust` → `beneficiaryOfLegalArrangement`) and `other-influence-or-control-of-trust` was removed. v0.4 added `nominee` and `nominator`.

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

### Entity subtype (`entityType.subtype`)

| Code | Allowed with `entityType.type` |
|---|---|
| `governmentDepartment` | `stateBody` |
| `stateAgency` | `stateBody` |
| `trust` | `arrangement` or `legalEntity` |
| `nomination` | `arrangement` |
| `other` | any |

### Record type (`recordType`)

`entity` | `person` | `relationship`

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

**Entity identifier schemes** — `scheme` SHOULD be a code from [org-id.guide](https://org-id.guide), e.g.:
- `GB-COH` — UK Companies House
- `XI-LEI` — Legal Entity Identifier (GLEIF)
- `US-EIN` — US Employer Identification Number

If there is no org-id.guide code, leave `scheme` blank and use `schemeName`.

**Person identifier schemes** — `scheme` SHOULD follow `{JURISDICTION}-{TYPE}`, where JURISDICTION is an ISO 3166-1 **alpha-3** code and TYPE is `PASSPORT`, `TAXID` or `IDCARD` (e.g. `GBR-PASSPORT`). Check data protection rules before publishing any person identifier. Internal person identifiers use `schemeName` of the form `{publisher name}-{identifier type}`.

---

## Source and Provenance

The `source` object records where the information came from.

```json
{
  "type": ["officialRegister"],
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

`source.type` is an **array** of codes. **Source types:** `selfDeclaration`, `officialRegister`, `thirdParty`, `primaryResearch`, `verified` (add `verified` alongside another code when the information has been through a verification process). There is no `other` code.

---

## Addresses

```json
{
  "type": "registered",
  "address": "123 High Street, London, EC1A 1BB",
  "postCode": "EC1A 1BB",
  "country": { "name": "United Kingdom", "code": "GB" }
}
```

`country` is a Country object (`name` required, `code` ISO 3166-1 alpha-2) — not a bare country code.

**Address types:** `placeOfBirth` (persons, `placeOfBirth` field only), `residence` (persons only), `registered` (entities only), `service` (persons only), `alternative`, `business`

---

## Examples — Complete Dataset

All examples below validate against the BODS v0.4 schema (`libcovebods jsonschemavalidate`).

### Simple direct ownership (one person owns one company)

```json
[
  {
    "statementId": "1dc0e987-5c57-4a1c-b3ad-61353b66a9b7",
    "statementDate": "2024-01-15",
    "declarationSubject": "acme-ltd-record",
    "recordId": "acme-ltd-record",
    "recordType": "entity",
    "recordStatus": "new",
    "publicationDetails": {
      "publicationDate": "2024-01-15",
      "bodsVersion": "0.4",
      "publisher": { "name": "Example Register" }
    },
    "source": { "type": ["officialRegister"], "description": "UK Companies House" },
    "recordDetails": {
      "isComponent": false,
      "entityType": { "type": "registeredEntity" },
      "name": "Acme Holdings Ltd",
      "jurisdiction": { "name": "United Kingdom", "code": "GB" },
      "identifiers": [{ "id": "12345678", "scheme": "GB-COH", "schemeName": "Companies House" }],
      "foundingDate": "2010-03-01",
      "addresses": [
        {
          "type": "registered",
          "address": "123 High Street, London, EC1A 1BB",
          "country": { "name": "United Kingdom", "code": "GB" }
        }
      ]
    }
  },
  {
    "statementId": "019a93f1-e470-42e9-957b-f8ab0a6c2d3e",
    "statementDate": "2024-01-15",
    "declarationSubject": "acme-ltd-record",
    "recordId": "jane-smith-record",
    "recordType": "person",
    "recordStatus": "new",
    "recordDetails": {
      "isComponent": false,
      "personType": "knownPerson",
      "names": [
        { "type": "legal", "fullName": "Jane Elizabeth Smith", "givenName": "Jane", "familyName": "Smith" },
        { "type": "alternative", "fullName": "Janey Smith" }
      ],
      "nationalities": [{ "name": "United Kingdom", "code": "GB" }],
      "birthDate": "1975-08"
    }
  },
  {
    "statementId": "fbfd0547-d0c6-4a00-b559-5c5e91c34f5c",
    "statementDate": "2024-01-15",
    "declarationSubject": "acme-ltd-record",
    "recordId": "acme-smith-ownership-record",
    "recordType": "relationship",
    "recordStatus": "new",
    "recordDetails": {
      "isComponent": false,
      "subject": "acme-ltd-record",
      "interestedParty": "jane-smith-record",
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
          "share": { "minimum": 75.0, "maximum": 100.0 }
        }
      ]
    }
  }
]
```

### Updating a record

A new statement, same `recordId`, `recordStatus: "updated"`. The earlier statement stays in the dataset unchanged.

```json
{
  "statementId": "8f0e1d2b-7a4c-4b8e-9c61-2d3f4a5b6c7d",
  "statementDate": "2024-06-01",
  "declarationSubject": "acme-ltd-record",
  "recordId": "acme-smith-ownership-record",
  "recordType": "relationship",
  "recordStatus": "updated",
  "recordDetails": {
    "isComponent": false,
    "subject": "acme-ltd-record",
    "interestedParty": "jane-smith-record",
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
  "statementId": "c3a1b2d4-5e6f-4a7b-8c9d-0e1f2a3b4c5d",
  "statementDate": "2024-01-15",
  "declarationSubject": "acme-ltd-record",
  "recordId": "acme-unknown-bo-record",
  "recordType": "relationship",
  "recordDetails": {
    "isComponent": false,
    "subject": "acme-ltd-record",
    "interestedParty": {
      "reason": "informationUnknownToPublisher",
      "description": "The register does not hold beneficial owner information for this entity."
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

### Exempt from disclosure

Where the law does not require the subject to disclose (not the same as "there is no beneficial owner"):

```json
{
  "statementId": "5b7d9f1a-3c5e-4f7a-9b1d-3f5a7c9e1b3d",
  "statementDate": "2024-01-15",
  "declarationSubject": "acme-ltd-record",
  "recordId": "acme-exempt-record",
  "recordType": "relationship",
  "recordDetails": {
    "isComponent": false,
    "subject": "acme-ltd-record",
    "interestedParty": {
      "reason": "subjectExemptFromDisclosure",
      "description": "Not required to file beneficial ownership information under the applicable law."
    }
  }
}
```

---

## Validation and Tooling

**Official schema (JSON Schema 2020-12):**
- v0.4 release: https://github.com/openownership/data-standard/tree/0.4.0/schema
- Codelists (authoritative values): https://github.com/openownership/data-standard/tree/0.4.0/schema/codelists

**Validator — `libcovebods`** (the library behind the BODS Data Review Tool):
```bash
pip install libcovebods
libcovebods jsonschemavalidate my-bods-data.json   # schema check (alias: jsv)
libcovebods pythonvalidate my-bods-data.json       # additional normative rules (alias: pv)
```
The bundled v0.4 schema lives at `libcovebods/data/schema-0-4-0/` and is a quick way to read codelist enums:
```bash
python -c "import libcovebods,pathlib,json; p=pathlib.Path(libcovebods.__file__).parent/'data/schema-0-4-0/components.json'; print(json.load(open(p))['\$defs']['UnspecifiedRecord']['properties']['reason']['enum'])"
```
Validate output built from **real** source data, not only fixtures — fixtures often exercise just one branch of a codelist mapping.

**Web validator (Data Review Tool):** https://datareview.openownership.org/

**Flatten Tool (CSV ↔ BODS):**
https://flatten-tool.readthedocs.io/en/latest/usage-bods/

**Data generator / test data:**
https://www.openownership.org/en/publications/beneficial-ownership-data-standard-generator/

**Official documentation:**
- v0.4 docs: https://standard.openownership.org/en/0.4.0/
- Schema browser: https://standard.openownership.org/en/0.4.0/standard/schema-browser.html
- Changelog: https://standard.openownership.org/en/0.4.0/standard/changelog.html
- GitHub: https://github.com/openownership/data-standard
