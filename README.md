# BODS skill for Claude

A Claude skill providing expert reference for the **Beneficial Ownership Data Standard (BODS) v0.4**, developed by [Open Ownership](https://www.openownership.org) and [Open Data Services](https://opendataservices.coop/).

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

## Official BODS resources

- Standard: https://standard.openownership.org/en/0.4.0/
- Schema reference: https://standard.openownership.org/en/latest/standard/reference.html
- GitHub: https://github.com/openownership/data-standard

## License

This skill is released under the [MIT License](LICENSE). The BODS specification itself is developed by Open Ownership and Open Data Services.
