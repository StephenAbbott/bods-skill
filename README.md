# BODS skill for Claude

A Claude skill providing expert reference for the **Beneficial Ownership Data Standard (BODS) v0.4**, developed by [Open Ownership](https://www.openownership.org) and [Open Data Services](https://opendataservices.coop/). Part of the [BODS Interoperability Toolkit](https://github.com/StephenAbbott/bods-interoperability-toolkit).

## What this skill does

When installed, this skill gives Claude deep knowledge of the BODS specification — including schema structure, statement types, codelists, and implementation guidance. It triggers automatically when you ask Claude about:

- Structuring entity, person, or relationship statements
- BODS codelists and interest types
- Modelling beneficial ownership chains
- Migrating from BODS v0.3 to v0.4
- Validating or publishing a BODS dataset
- UBOs, beneficial ownership transparency, openownership.org

## Installation

1. Download `bods.skill` from [Releases](../../releases)
2. Open the Claude desktop app
3. Click **Save skill** when prompted, or drag the `.skill` file into Claude

## Skill contents

```
bods-skill/
├── SKILL.md              # Skill instructions + quick reference
└── references/
    ├── bods.md           # Full BODS v0.4 field-level schema reference
    └── index.md          # Reference file index
```

## What's covered

- All three statement types: Entity, Person, Relationship
- Complete field reference for each statement type
- Interest type codelist (v0.4 camelCase + v0.3 migration notes)
- `recordDetails`, `recordId`, `recordStatus` (new in v0.4)
- Identifier schemes (GB-COH, LEI, etc.)
- Source/provenance structure
- Full JSON examples: direct ownership, record updates, unknown beneficial owners
- Validation tooling references
- Primer, About and Governance pages (2026 documentation rewrite)
- BODS data model UML: core objects vs. Declaration/Record pseudo-objects

## Official BODS resources

> **Note:** the default and `latest` documentation URLs now resolve to the `main` branch, not the `0.4.0` release branch. The 2026 Primer/About/Governance rewrite was merged to `main` and not back-ported to `0.4.0` (which remains the frozen, translated v0.4 release).

- Standard (current — `main` branch): https://standard.openownership.org/en/main/
- v0.4 release branch (frozen, translated): https://standard.openownership.org/en/0.4.0/
- Schema reference: https://standard.openownership.org/en/main/standard/reference.html
- Primer: https://standard.openownership.org/en/main/primer/index.html
- Data model (UML): https://standard.openownership.org/en/main/primer/datamodel.html
- Governance and development: https://standard.openownership.org/en/main/about/governance.html
- BODS development handbook: https://openownership.github.io/bods-dev-handbook/
- GitHub: https://github.com/openownership/data-standard
- Data review tool: https://datareview.openownership.org/
- Visualisation library: https://github.com/openownership/visualisation-tool
- Analysis notebooks and dashboards: https://github.com/openownership/bodsanalysis
- Beneficial ownership data analysis tools: https://github.com/openownership/bodsdata
- RDF vocabulary: https://github.com/openownership/bodsld

## License

This skill is released under the [MIT License](LICENSE). The BODS specification itself is developed by Open Ownership and Open Data Services.
