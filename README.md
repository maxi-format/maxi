# MAXI Schema

![MAXI Logo](./assets/maxi-logo.svg)

**MAXI** – **M**inimal **A**PI E**x**change **I**nterface  
Maximum efficiency, minimum tokens.

This repository contains the specification for the **MAXI** data format and schema language.

---

## What is MAXI?

MAXI is a compact, schema-driven serialization format designed for:

- **LLM-friendly APIs** – minimize token usage in prompts and responses
- **High-throughput services** – low overhead, easy to parse
- **Human readability** – plain text, predictable structure
- **Strong typing** – optional but well-defined schema and constraints

Key ideas:

- **Positional fields**: field order is defined by the schema; field names are not repeated in data records
- **Short aliases**: types use short aliases in data (e.g., `U` for `User`)
- **Optional schema**: schema can be inline or external, but can also be omitted entirely when it’s clear from context which schema/types apply (e.g., by API endpoint/contract/version)
- **References & inheritance**: reuse types and avoid duplication
- **Streaming-friendly**: schema first, then record stream

---

## Quick Example

Schema + data in a single `.maxi` file:

```maxi
@version:1.0.0
@mode:strict

U:User(id:int|name:str(!)|email:str@email)
O:Order(id:int|user:U|total:decimal)

###
U(1|Julie Miller|julie@maxi.org)
U(2|Matt Smith|matt@maxi.org)
O(100|1|99.99)
O(101|2|149.50)
```

- `U` and `O` are **type aliases**
- Fields are positional: `U(id|name|email)`
- The `###` line separates **schema** from **data**

---

## Specification

The full specification is in [`SPEC.md`](./SPEC.md):

- File structure & directives
- Type system (primitives, arrays, maps, enums, objects & inheritance)
- Constraints (strings, numbers, binary, arrays, maps)
- Data records & null semantics
- Schema management & imports
- Error handling & error codes
- Streaming behavior

For implementers: `SPEC.md` also includes **Appendix C (ABNF schema grammar)** and **Appendix D (ABNF data grammar)** to help build parsers and serializers consistently (including schema-less/data-only files where the applicable schema is implied by context).

---

## File Types & MIME Types

- `.maxi` – data files (with optional inline schema)
- `.mxs` – schema files (type definitions only)

Suggested content types:

- `application/maxi`
- `application/maxi-schema`

---

## Status

The spec is currently **v1.0.0 – DRAFT**.  
Feedback, issues, and suggestions are welcome via GitHub issues and pull requests.

---

## Ecosystem & Roadmap

This specification is the foundation for a broader MAXI ecosystem.

Planned companion projects:

- **Test data & conformance repository**  
  A separate repository containing:
  - Canonical example `.maxi` and `.mxs` files
  - Edge cases and malformed inputs
  - Expected parse/validation results  
  
  This will help implementers build and verify MAXI parsers and serializers.

- **Language libraries**  
  Libraries for working with MAXI in various languages (for example):
  - Python
  - JavaScript/TypeScript
  - ...
  
  These libraries will cover parsing, validation, schema handling, and serialization.

---

## License

Released under the [MIT License](./LICENSE).
