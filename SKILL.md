---
name: bods
description: >
  Expert reference skill for the Beneficial Ownership Data Standard (BODS) v0.4, developed by Open Ownership. Use this skill whenever the user asks about BODS — including how to structure or validate entity statements, person statements, or relationship statements; how to model beneficial ownership chains; what interest types or codelists to use; how v0.4 differs from v0.3; how to implement or publish a BODS dataset; or any question about UBOs, beneficial ownership data, open ownership transparency, or the openownership.org standard. Trigger on: BODS, beneficial ownership data standard, UBO data, open ownership standard, entity statement, person statement, relationship statement, ownership-or-control statement, openownership.org, recordDetails, interestType codelist, BODS schema, BODS JSON.
metadata:
  doc_version: "0.4"
---

# Beneficial Ownership Data Standard (BODS) v0.4

BODS is an open JSON-based standard for publishing structured data about who owns and controls companies and other legal entities. Developed by Open Ownership (openownership.org) in partnership with Open Data Services.

## Core Concepts

A **BODS dataset** is a collection of **Statements**. Every statement is an immutable, time-stamped claim made by a publisher about a subject. Publishers never edit statements — they publish new statements to reflect changes.

There are three statement types, nested inside a `recordDetails` object (v0.4):

| Statement type | `recordType` value | Describes |
|---|---|---|
| Entity Statement | `entity` | A company, trust, or other legal arrangement |
| Person Statement | `person` | A natural person (beneficial owner or controller) |
| Relationship Statement | `relationship` | An ownership/control interest between a person/entity and an entity |

> **v0.3 note:** In v0.3, relationship statements were called "ownership-or-control statements" (`ownershipOrControlStatement`). In v0.4 the terminology changed to "relationship statement" (`relationshipStatement`).

---

## Statement Structure (v0.4)

### Top-level fields (all statement types)

| Field | Type | Notes |
|---|---|---|
| `statementId` | string | Globally unique identifier for this statement |
| `declarationSubject` | string | Links statement to the declaring entity/filing |
| `declaration` | string | Groups statements by parent declaration |
| `recordId` | string | Stable ID across statement versions (for the same record) |
| `recordStatus` | enum | `new`, `updated`, `closed` |
| `recordDetails` | object | Contains the type-specific payload (see below) |
| `statementDate` | date | When the statement was made |
| `source` | object | Provenance of the statement |

---

### Entity Statement (`recordDetails` for entity)

```json
{
  "statementId": "...",
  "recordId": "...",
  "recordStatus": "new",
  "statementDate": "2024-01-15",
  "recordDetails": {
    "entityType": {
      "type": "registeredEntity"
    },
    "name": "Acme Holdings Ltd",
    "incorporatedInJurisdiction": {
      "name": "United Kingdom",
      "code": "GB"
    },
    "identifiers": [
      {
        "id": "12345678",
        "scheme": "GB-COH",
        "schemeName": "Companies House"
      }
    ],
    "foundingDate": "2010-03-01",
    "addresses": [
      {
        "type": "registered",
        "address": "123 High Street, London, EC1A 1BB"
      }
    ]
  }
}
```

**Entity types (`entityType.type`):** `registeredEntity`, `legalEntity`, `arrangement`, `anonymousEntity`, `unknownEntity`, `state`, `stateBody`

**v0.4 addition:** `formedByStatute` object (with `name` and `date`) for state bodies; `entitySubtypeCategory` codelist for state body subtypes.

---

### Person Statement (`recordDetails` for person)

```json
{
  "statementId": "...",
  "recordId": "...",
  "recordStatus": "new",
  "statementDate": "2024-01-15",
  "recordDetails": {
    "personType": "knownPerson",
    "names": [
      {
        "type": "individual",
        "fullName": "Jane Smith"
      }
    ],
    "identifiers": [
      {
        "id": "AB123456",
        "scheme": "GB-PASS",
        "schemeName": "UK Passport"
      }
    ],
    "nationalities": [
      { "code": "GB", "name": "British" }
    ],
    "birthDate": "1975-08-22",
    "addresses": [
      {
        "type": "residence",
        "address": "45 Oak Avenue, London, SW1A 2AA",
        "country": "GB"
      }
    ]
  }
}
```

**Person types (`personType`):** `knownPerson`, `anonymousPerson`, `unknownPerson`

---

### Relationship Statement (`recordDetails` for relationship)

```json
{
  "statementId": "...",
  "recordId": "...",
  "recordStatus": "new",
  "statementDate": "2024-01-15",
  "recordDetails": {
    "subject": {
      "describedByEntityStatement": "<entity-statement-id>"
    },
    "interestedParty": {
      "describedByPersonStatement": "<person-statement-id>"
    },
    "interests": [
      {
        "type": "shareholding",
        "directOrIndirect": "direct",
        "beneficialOwnershipOrControl": true,
        "share": {
          "exact": 51,
          "minimum": 50,
          "maximum": 75
        },
        "startDate": "2019-06-01"
      },
      {
        "type": "votingRights",
        "directOrIndirect": "direct",
        "beneficialOwnershipOrControl": true,
        "share": {
          "exact": 51
        }
      }
    ]
  }
}
```

---

## Interest Types (v0.4 codelist — camelCase)

> **v0.3 → v0.4:** Interest type codes changed from hyphenated to camelCase (e.g. `voting-rights` → `votingRights`).

| Code | Meaning |
|---|---|
| `shareholding` | Ownership of shares |
| `votingRights` | Rights to vote at shareholder/member meetings |
| `appointmentOfBoard` | Power to appoint or remove directors |
| `otherInfluenceOrControl` | Other means of influence or control |
| `controlViaCompanyRulesOrArticles` | Control through articles of association or company rules *(new in v0.4)* |
| `controlByLegalFramework` | Control arising from legal or regulatory framework *(new in v0.4)* |
| `boardMember` | Membership of board *(new in v0.4)* |
| `boardChair` | Chair of board *(new in v0.4)* |
| `unknownInterest` | Type of interest is unknown *(new in v0.4)* |
| `unpublishedInterest` | Interest exists but is not published *(new in v0.4)* |
| `enjoymentAndUseOfAssets` | Right to enjoy/use assets *(new in v0.4)* |
| `rightToProfitOrIncomeFromAssets` | Right to profit or income from assets *(new in v0.4)* |

---

## Key Migration Notes: v0.3 → v0.4

1. **Flattened structure**: Statement fields are now at the top level. Type-specific data moves into `recordDetails`.
2. **Relationship statements**: Previously "ownership-or-control statements" — the concept is the same.
3. **Record management**: New `recordId`, `recordStatus`, `declaration`, and `declarationSubject` fields.
4. **JSON Lines**: v0.4 supports newline-delimited JSON (one statement per line) for large datasets.
5. **Interest type codelists**: All codes now camelCase; several new codes added.
6. **Entity enhancements**: `formedByStatute` and `entitySubtypeCategory` added.

---

## Modelling Guidance

### Representing a beneficial owner chain
Model each entity in the chain with an Entity Statement, the ultimate beneficial owner with a Person Statement, and each ownership link with a Relationship Statement. The `directOrIndirect` field on interests captures whether ownership is direct or flows through intermediate entities.

### Thresholds and unknowns
Use `share.minimum` / `share.maximum` when the exact percentage is not known. Use `share.exclusiveMinimum` / `share.exclusiveMaximum` when the bound itself is excluded. Use interest type `unknownInterest` when the type is not known, or `unpublishedInterest` when it exists but is deliberately withheld.

### Updates and history
Publish a new Statement with the same `recordId` and set `recordStatus: "updated"`. The previous statement remains in the dataset unchanged — BODS is append-only. To close a record, publish a statement with `recordStatus: "closed"`.

---

## Reference Files

Detailed documentation is in `references/bods.md`. Read it when you need full field-level schema details, all codelist values, or extended examples.

**Official resources:**
- Standard: https://standard.openownership.org/en/0.4.0/
- Schema reference: https://standard.openownership.org/en/latest/standard/reference.html
- Changelog: https://standard.openownership.org/en/0.4.0/standard/changelog.html
- GitHub: https://github.com/openownership/data-standard
