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

---

## Tools & Resources

### Official BODS Standard

| Resource | URL |
|---|---|
| Standard documentation (v0.4) | https://standard.openownership.org/en/0.4.0/ |
| Schema reference | https://standard.openownership.org/en/latest/standard/reference.html |
| Changelog | https://standard.openownership.org/en/0.4.0/standard/changelog.html |
| GitHub (data-standard) | https://github.com/openownership/data-standard |
| OpenOwnership website | https://www.openownership.org/en/ |
| OpenOwnership publications | https://www.openownership.org/en/publications/ |

---

### Data Review Tool (CoVE-BODS): Validation

The **BODS Data Review Tool** (CoVE-BODS) validates BODS JSON data against the schema and runs additional compliance checks.

| Resource | URL |
|---|---|
| OpenOwnership publication | https://www.openownership.org/en/publications/beneficial-ownership-data-standard-data-review-tool/ |
| Web validator | https://datareview.openownership.org/ |
| GitHub (cove-bods) | https://github.com/openownership/cove-bods |
| GitHub (lib-cove-bods) | https://github.com/openownership/lib-cove-bods |
| PyPI (libcovebods) | https://pypi.org/project/libcovebods/ |

The web tool accepts BODS JSON directly and returns validation results. For CLI use:
```bash
pip install libcovebods
libcovebods your-data.json
```
Checks: required fields, valid enums, internal reference integrity, version compliance (BODS 0.1–0.4), plus 26 additional regulatory compliance assessments.

---

### Visualisation Library: Rendering Ownership Diagrams

The **BODS Visualisation Library** implements the Beneficial Ownership Visualisation System (BOVS) for rendering BODS-structured data as interactive ownership diagrams. In production use in the registers of Armenia, Bermuda, and Botswana.

| Resource | URL |
|---|---|
| OpenOwnership publication | https://www.openownership.org/en/publications/beneficial-ownership-data-standard-visualisation-library/ |
| GitHub (visualisation-tool / bods-dagre) | https://github.com/openownership/visualisation-tool |
| npm package | https://www.npmjs.com/package/@openownership/bods-dagre |
| Visualisation spec | https://github.com/openownership/visualisation-tool/blob/main/docs/spec.md |

```bash
npm install @openownership/bods-dagre
```
Input: JSON array of BODS statements. Output: SVG/canvas network diagram with person nodes, entity nodes, and directed ownership/control edges. Handles temporal data — filters to a snapshot at a given date.

---

### Analysis Notebooks & Dashboards (bodsanalysis)

**bodsanalysis** provides Jupyter notebooks and Python functions for reading, summarising, and analysing BODS data. Runs on local Jupyter, Deepnote, or Google Colab.

| Resource | URL |
|---|---|
| OpenOwnership publication | https://www.openownership.org/en/publications/analysis-notebooks-and-dashboards-for-beneficial-ownership-data-standard-bods-data/ |
| GitHub (bodsanalysis) | https://github.com/openownership/bodsanalysis |

Key notebooks: `latvia_demo.ipynb` (Latvia register), `Insights_UK_PSC_BODS-02.ipynb` (UK PSC data). Note: currently targets BODS 0.2 — adapt field names for 0.4.

---

### Data Processing Tools (bodsdata)

**bodsdata** converts BODS JSON into flat formats for database and tabular analysis.

| Resource | URL |
|---|---|
| OpenOwnership publication | https://www.openownership.org/en/publications/beneficial-ownership-data-analysis-tools/ |
| BODS data explorer | https://bods-data.openownership.org/ |
| GitHub (bodsdata) | https://github.com/openownership/bodsdata |

Output formats: CSV/TSV, SQLite, Parquet, PostgreSQL dump. Also runs consistency checks (required fields, duplicate statement IDs, broken references). The **BODS data explorer** hosts processed datasets from multiple jurisdictions for exploring real-world BODS data.

---

### RDF Vocabulary (bodsld): Linked Data & SPARQL

The **BODS RDF Vocabulary** enables querying BODS data as Linked Data using SPARQL, supporting cross-dataset linking and semantic reasoning over ownership graphs.

| Resource | URL |
|---|---|
| OpenOwnership publication | https://www.openownership.org/en/publications/rdf-vocabulary-for-the-beneficial-ownership-data-standard/ |
| RDF vocabulary documentation | https://vocab.openownership.org/ |
| RDF terms | https://vocab.openownership.org/terms/ |
| BODS RDF Turtle 0.4 | https://vocab.openownership.org/terms/bods-vocabulary-0.4.0.ttl |
| GitHub (bodsld) | https://github.com/openownership/bodsld |

---

### BODS Conversion Repositories (Stephen Abbott Pugh)

A suite of open-source converters between BODS v0.4 and other data formats/platforms:

| Repository | Conversion | URL |
|---|---|---|
| bods-ftm | BODS 0.4 ↔ FollowTheMoney (bidirectional) | https://github.com/StephenAbbott/bods-ftm |
| bods-aml-ai | BODS 0.4 → Google AML AI | https://github.com/StephenAbbott/bods-aml-ai |
| bods-neo4j | BODS 0.4 ↔ Neo4j graph database (bidirectional) | https://github.com/StephenAbbott/bods-neo4j |
| bods-gql | BODS 0.4 → Google BigQuery / GQL (ISO/IEC 39075) | https://github.com/StephenAbbott/bods-gql |
| bods-brightquery | BrightQuery → BODS 0.4 | https://github.com/StephenAbbott/bods-brightquery |
| bods-kyckr | Kyckr → BODS 0.4 | https://github.com/StephenAbbott/bods-kyckr |
| bods-icij-offshoreleaks | ICIJ Offshore Leaks → BODS 0.4 | https://github.com/StephenAbbott/bods-icij-offshoreleaks |
| bods-opencorporates | OpenCorporates → BODS 0.4 | https://github.com/StephenAbbott/bods-opencorporates |
| bods-xml | BODS 0.4 → XML (canonical + MRAS preBODS) | https://github.com/StephenAbbott/bods-xml |
| bods-stream | Live UK PSC stream → BODS v0.4 (web app, SSE) | https://github.com/StephenAbbott/bods-stream |
| bods-mapper | Shared CH PSC → BODS v0.4 library (used by opencheck + bods-stream) | https://github.com/StephenAbbott/bods-mapper |
| opencheck | CDD tool: LEI → 29 sources → BODS v0.4 (live demo) | https://github.com/StephenAbbott/opencheck |

---

### Testing & Validation Libraries

| Resource | URL |
|---|---|
| bods-fixtures (canonical test fixture pack) | https://github.com/StephenAbbott/bods-fixtures |
| bods-v04-fixtures (PyPI) | https://pypi.org/project/bods-v04-fixtures/ |
| pytest-bods-fixtures (pytest plugin) | https://github.com/StephenAbbott/pytest-bods-fixtures |
| pytest-bods-v04-fixtures (PyPI) | https://pypi.org/project/pytest-bods-v04-fixtures/ |
| bods-validator (validation + visualisation tool) | https://github.com/StephenAbbott/bods-validator |

**opencheck** (https://opencheck.onrender.com/) is a full CDD application — paste a LEI, and it fans out across 29 national and international corporate data sources, maps everything to BODS v0.4, and applies a 12-signal risk layer mirroring the EU AMLA draft CDD RTS conditions (trust/arrangement, non-EU jurisdiction, nominee, ≥3 ownership layers, FATF list, ICIJ Offshore Leaks match, and more). Exports as JSON/JSONL/XML/ZIP. Phase 50 of active development. Uses `bods-mapper` and `bods-dagre`.

**bods-mapper** (https://github.com/StephenAbbott/bods-mapper) is the shared canonical library for mapping UK Companies House PSC events to BODS v0.4. All 86 `natures_of_control` codes → BODS `interestType`. Handles cessation lifecycle (`recordStatus: "closed"`, `replacesStatements`). Used by both opencheck and bods-stream to ensure consistent mapping. Core function: `map_psc_event(event)`.

**bods-stream** is a live web application (not a library) — it consumes the Companies House PSC Streaming API and converts each filing event to BODS v0.4 statements in real time, displayed as BOVS diagrams. Live at https://bods-stream.onrender.com/. It demonstrates BODS's append-only change model (`recordStatus` new/updated/closed, `replacesStatements`) in action — the only public real-time beneficial ownership feed anywhere. Depends on [bods-mapper](https://github.com/StephenAbbott/bods-mapper), a shared CH → BODS v0.4 mapping library also used by [OpenCheck](https://github.com/StephenAbbott/opencheck).

**bods-fixtures** (`pip install bods-v04-fixtures`) — canonical BODS v0.4 test fixtures: curated statement bundles for direct ownership, circular ownership, and anonymous persons. Single source of truth across the BODS adapter ecosystem.

**pytest-bods-fixtures** (`pip install pytest-bods-v04-fixtures`) — pytest plugin wrapping the fixture pack as an auto-parametrized `bods_fixture`, running one test per conformance case without boilerplate.
