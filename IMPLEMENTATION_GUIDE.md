# MAXI Implementation Guide

## 1. Introduction

This guide is for developers who are building a MAXI parser or library. It provides practical advice, clarifies ambiguities in the specification, and establishes conventions that ensure interoperability between different MAXI implementations.

This document is a companion to the formal [MAXI Specification (`SPEC.md`)](./SPEC.md). While the specification is the ultimate source of truth, this guide explains *how* to implement its rules, especially regarding parser configuration, error handling, and testing.

## 2. Parser Configuration

A compliant MAXI parser **must** support a set of independent configuration flags to control its behavior. The concept of a single "mode" (e.g., `lax` vs. `strict`) is deprecated and has been replaced by these granular options.

Callers configure the parser by passing these flags. If a flag is not provided, the parser **must** use its specified default value.

### Flag Defaults and Effects

| Flag | Values | Default | Description |
|---|---|---|---|
| `allowAdditionalFields` | `"ignore"` · `"warning"` · `"error"` | `"ignore"` | How to handle extra fields in a data record that are not defined in the schema. |
| `allowMissingFields` | `"null"` · `"warning"` · `"error"` | `"null"` | How to handle required fields that are missing from a data record. |
| `allowTypeCoercion` | `"coerce"` · `"warning"` · `"error"` | `"coerce"` | How to handle a value whose type does not match the schema (e.g., `string` for an `int` field). |
| `allowConstraintViolations` | `"warning"` · `"error"` | `"warning"` | How to handle a value that violates a schema constraint (e.g., `length`, `range`, `pattern`). |
| `allowForwardReferences` | `true` · `false` | `true` | Whether to allow a record to reference another record that appears later in the file. |
| `allowUnknownTypes` | `"ignore"` · `"warning"` · `"error"` | `"warning"` | How to handle a data record whose type alias does not correspond to any defined type. |

---

### 2.1. `allowAdditionalFields` Behavior

An "additional field" is a key-value pair in a data record where the key does not correspond to any field name defined for that type in the schema.

-   `"ignore"` (Default): The parser silently discards the additional field. The field and its value are not included in the parsed output.
-   `"warning"`: The parser includes the additional field in the parsed output and emits a `MaxiWarning` for each such field. The parse is considered successful.
-   `"error"`: The parser emits or throws a `MaxiError` (error code **E406**) as soon as it encounters an additional field. The parse fails.

### 2.2. `allowMissingFields` Behavior

A "missing field" is a field defined in the schema that is not present in a data record. This applies to fields marked as required (with `!`) and fields with a default value.

-   `"null"` (Default): The parser fills the missing field with `null` (or the language-equivalent `nil`/`None`). If the schema specifies a default value for the field, the default value is used instead. This action is performed silently.
-   `"warning"`: The parser behaves like `"null"` but also emits a `MaxiWarning` for each missing field that is filled.
-   `"error"`: The parser emits or throws a `MaxiError` (error code **E403**) for any missing required field. The parse fails.

### 2.3. `allowTypeCoercion` Behavior

Type coercion occurs when a scalar value in a data record has a different type than what the schema specifies for that field (e.g., providing the string `"123"` for a field defined as `int`).

-   `"coerce"` (Default): The parser attempts to convert the value to the target type (e.g., `"123"` → `123`). If coercion is successful, the result is used silently. If it fails, an error should be raised.
-   `"warning"`: The parser attempts to coerce the value and, if successful, emits a `MaxiWarning`.
-   `"error"`: The parser does not attempt coercion. It emits or throws a `MaxiError` (error code **E402**) for any type mismatch. The parse fails.

### 2.4. `allowConstraintViolations` Behavior

A constraint violation occurs when a value, while having the correct type, fails to meet a constraint defined in the schema. This includes:
- `string` length (`length`, `minLength`, `maxLength`) or `pattern`.
- `int` or `float` value (`range`, `min`, `max`).
- `array` or `map` size (`size`, `minSize`, `maxSize`).

-   `"warning"` (Default): The parser accepts the value but emits a `MaxiWarning` for each violation. The parse is successful.
-   `"error"`: The parser emits or throws a `MaxiError` (error code **E303**) as soon as it encounters a constraint violation. The parse fails.

### 2.5. `allowForwardReferences` Behavior

A forward reference is a reference (`@id`) to a record that has not yet been parsed (i.e., it appears later in the document or stream).

-   `true` (Default): The parser must defer the resolution of references. It should first parse all records, storing them in a map by their ID. After all records are loaded, it can resolve all references.
-   `false`: The parser attempts to resolve references immediately. If a reference's target ID has not yet been encountered, the parser emits or throws a `MaxiError` (error code **E204**). The parse fails.

### 2.6. `allowUnknownTypes` Behavior

An unknown type occurs when a data record uses a type alias that was not defined in the schema header.

-   `"ignore"`: The parser silently skips the record and its entire value. It is not included in the output.
-   `"warning"` (Default): The parser emits a `MaxiWarning` and attempts a best-effort parse of the record's fields without type or constraint information.
-   `"error"`: The parser emits or throws a `MaxiError` (error code **E201**) as soon as it encounters a record with an unknown type alias. The parse fails.

---

## 3. Type Name vs. Alias Validation

It is critical to distinguish between the validation rules for **Type Names** and **Type Aliases**.

-   **Type Names** (long form, e.g., `Person` in `P:Person(...)`): Must start with an uppercase or lowercase letter (`[a-zA-Z]`). See `SPEC.md` §3.3.2.
-   **Type Aliases** (short form, e.g., `P`): May start with a letter or an underscore (`_`). See `SPEC.md` §3.3.3.

For example, the schema declaration `_M:_Meta(...)` is **invalid** because the type name `_Meta` starts with an underscore. The testcase `9sqyp` validates that parsers correctly reject this.

## 4. Error and Warning Codes

Parsers should use the standardized error and warning codes defined in Appendix B of the `SPEC.md`. The flags introduced in this guide map to the following primary error codes when set to `"error"`:

| Flag | Error Code | Description |
|---|---|---|
| `allowAdditionalFields` | **E406** | An additional field was found in a record. |
| `allowMissingFields` | **E403** | A required field was missing from a record. |
| `allowTypeCoercion` | **E402** | A field value had a mismatched type. |
| `allowConstraintViolations` | **E303** | A field value violated a schema constraint. |
| `allowForwardReferences` | **E204** | A forward reference was encountered when disallowed. |
| `allowUnknownTypes` | **E201** | A record's type alias is not defined in the schema. |

## 5. Test Harness Integration

The [maxi-testdata](https://github.com/maxi-format/maxi-testdata) repository provides a shared suite of tests for ensuring parser conformance. Each testcase includes a `test.json` file that defines the expected outcome and necessary parser configuration.

### `parserOptions` in `test.json`

To run a test, a test runner **must**:
1. Read the `test.json` file.
2. Look for the `parserOptions` object. If it is absent, use an empty object.
3. For each of the six flags, use the value from `parserOptions` or the flag's specified default value if the key is not present.
4. Invoke the parser with these resolved flags.
5. Compare the parser's output (result, errors, and warnings) against the `expected.json` file for that testcase.

Example `test.json`:
```jsonc
{
  "id": "xyz12",
  "title": "Test with strict field checking",
  "category": "error",
  "parserOptions": {
    "allowAdditionalFields": "error",
    "allowMissingFields": "error"
  }
}
```
In this example, the runner would call the parser with `allowAdditionalFields` and `allowMissingFields` set to `"error"`. The other four flags would use their default values.

Example runners can be found in the official `maxi-javascript`, `maxi-php`, and `maxi-python` libraries.

## 6. Conformance Checklist

A fully conformant MAXI parser implementation must:

- [ ] Correctly parse all parts of the MAXI specification (`SPEC.md`).
- [ ] Support all six parser configuration flags defined in this guide, respecting their default values.
- [ ] Implement the behavior for each flag value (`ignore`/`warning`/`error`, etc.) as described.
- [ ] Emit correct error and warning codes as specified.
- [ ] Distinguish between valid type aliases and invalid type names (as per `9sqyp`).
- [ ] Pass all testcases in the `maxi-testdata` repository when run with the correct `parserOptions`.
