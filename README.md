# MAXI Schema

![MAXI Logo](./assets/maxi-logo.svg)

**MAXI** – **M**inimal **A**PI E**x**change **I**nterface  
Compact by design. Readable by default.

This repository contains the specification for the **MAXI** data format and schema language.

---

## What is MAXI?

MAXI is a compact, schema-driven serialization format. Type definitions are declared once in a schema; records carry only positional values with no repeated field names or verbose delimiters. This makes it efficient wherever the same structure repeats across many records and the schema can be shared or declared once.

Typical use cases:

- **LLM contexts**: minimize token usage when passing structured datasets to or from language models
- **REST APIs**; reduce payload size for high-throughput endpoints; schemas can be hosted externally and cached by clients
- **WebSocket & RPC**: a human-readable alternative to binary formats like Protocol Buffers, still readable in logs and debuggers without tooling
- **Data exports & bulk datasets**: schema agreed upon once, data stream carries only compact positional records
- **Streaming pipelines**: schema-first design allows consumers to begin processing before the full dataset arrives

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
@maxi:1.0.0

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

The spec is currently **1.0.0-alpha.2**.  
Feedback, issues, and suggestions are welcome via GitHub issues and pull requests.

---

## Ecosystem

This specification is the foundation for a broader MAXI ecosystem.

- **Test data & conformance repository** [maxi-testdata](https://github.com/maxi-format/maxi-testdata)  
  Canonical example `.maxi`/`.mxs` files, edge cases, malformed inputs, and expected
  parse/validation results. Used by all language libraries to verify conformance.

- **CLI converter**  
  [maxi-converter](https://github.com/maxi-format/maxi-go/tree/main/cmd/maxi-converter) - CLI tool (part of `maxi-go`) for converting between MAXI and JSON. Install via `go install` or download a pre-built binary from [releases](https://github.com/maxi-format/maxi-go/releases).

- **Language libraries**  
  - [Go - maxi-go](https://github.com/maxi-format/maxi-go)
  - [Java - maxi-java](https://github.com/maxi-format/maxi-java)
  - [JavaScript - maxi-javascript](https://github.com/maxi-format/maxi-javascript)
  - [PHP - maxi-php](https://github.com/maxi-format/maxi-php)
  - [Python - maxi-python](https://github.com/maxi-format/maxi-python)

  All libraries cover parsing, validation, schema handling, and (de-)serialization and pass the shared conformance suite.
  Add your own library for your language of choice!

- **Examples**
  [maxi-examples](https://github.com/maxi-format/maxi-examples) - cross-language demo with servers and CLI clients in Go, Java, JavaScript, PHP, and Python all exchanging `application/maxi` over HTTP. Includes a browser client and a Docker Compose setup to run everything locally.

---

## License

Released under the [MIT License](./LICENSE).
