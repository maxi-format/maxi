# MAXI Specification v1.0.0 - DRAFT

**MAXI** - **M**inimal **A**PI E**x**change **I**nterface

*Maximum efficiency, minimum tokens*

---

## Table of Contents

1. [Overview](#1-overview)
    - 1.1 [Introduction](#11-introduction)
    - 1.2 [Design Principles](#12-design-principles)
    - 1.3 [Quick Example](#13-quick-example)
2. [File Structure & Directives](#2-file-structure--directives)
    - 2.1 [File Structure](#21-file-structure)
    - 2.2 [Delimiters](#22-delimiters)
    - 2.3 [Directives](#23-directives)
        - 2.3.1 [Version Directive](#231-version-directive)
        - 2.3.2 [Mode Directive](#232-mode-directive)
        - 2.3.3 [Schema Directive](#233-schema-directive)
    - 2.4 [Comments](#24-comments)
    - 2.5 [String Escaping](#25-string-escaping)
    - 2.6 [File Format and Extensions](#26-file-format-and-extensions)
3. [Type System](#3-type-system)
    - 3.1 [Primitive Types](#31-primitive-types)
    - 3.2 [Complex Types](#32-complex-types)
        - 3.2.1 [Arrays](#321-arrays)
        - 3.2.2 [Enums](#322-enums)
        - 3.2.3 [Maps](#323-maps)
    - 3.3 [Objects & Inheritance](#33-objects--inheritance)
        - 3.3.1 [Type Definitions](#331-type-definitions)
        - 3.3.2 [Type Name Naming Rules](#332-type-name-naming-rules)
        - 3.3.3 [Alias Naming Rules](#333-alias-naming-rules)
        - 3.3.4 [Alias-Only Syntax](#334-alias-only-syntax)
        - 3.3.5 [Type Definition](#335-type-definition)
        - 3.3.6 [Multi-line Definitions](#336-multi-line-definitions)
        - 3.3.7 [Type Inheritance](#337-type-inheritance)
    - 3.4 [Type Annotations](#34-type-annotations)
4. [Constraints](#4-constraints)
    - 4.1 [General Constraints](#41-general-constraints)
    - 4.2 [String Constraints](#42-string-constraints)
    - 4.3 [Numeric Constraints](#43-numeric-constraints)
        - 4.3.1 [Integer Constraints](#431-integer-constraints)
        - 4.3.2 [Decimal Constraints](#432-decimal-constraints)
        - 4.3.3 [Binary Constraints](#433-binary-constraints)
    - 4.4 [Array Constraints](#44-array-constraints)
    - 4.5 [Map Constraints](#45-map-constraints)
5. [Data Records](#5-data-records)
    - 5.1 [Basic Syntax](#51-basic-syntax)
        - 5.1.1 [Single-line Records](#511-single-line-records)
        - 5.1.2 [Multi-line Records](#512-multi-line-records)
    - 5.2 [Field Values & Null Semantics](#52-field-values--null-semantics)
        - 5.2.1 [Default Value Definition](#521-default-value-definition)
    - 5.3 [Objects (Inline vs Reference)](#53-objects-inline-vs-reference)
        - 5.3.1 [Object References](#531-object-references)
        - 5.3.2 [Inline Object Definitions](#532-inline-object-definitions)
        - 5.3.3 [Mixed Usage](#533-mixed-usage)
        - 5.3.4 [Nested Inline Objects](#534-nested-inline-objects)
        - 5.3.5 [When to Use References vs Inline](#535-when-to-use-references-vs-inline)
        - 5.3.6 [Object Identifiers](#536-object-identifiers)
        - 5.3.7 [Reference Resolution](#537-reference-resolution)
    - 5.4 [Arrays & Maps](#54-arrays--maps)
        - 5.4.1 [Arrays](#541-arrays)
        - 5.4.2 [Maps](#542-maps)
6. [Schema Management](#6-schema-management)
    - 6.1 [Imports](#61-imports)
    - 6.2 [Overrides](#62-overrides)
    - 6.3 [Circular Imports](#63-circular-imports)
    - 6.4 [Schema Resolution Algorithm](#64-schema-resolution-algorithm)
7. [Error Handling](#7-error-handling)
    - 7.1 [Required Parser Behaviors (MUST)](#71-required-parser-behaviors-must)
    - 7.2 [Recommended Parser Behaviors (SHOULD)](#72-recommended-parser-behaviors-should)
    - 7.3 [Optional Parser Behaviors (MAY)](#73-optional-parser-behaviors-may)
    - 7.4 [Handling Specific Error Cases](#74-handling-specific-error-cases)
        - 7.4.1 [Unknown Type Aliases](#741-unknown-type-aliases)
        - 7.4.2 [Missing Required Fields](#742-missing-required-fields)
        - 7.4.3 [Type Mismatches](#743-type-mismatches)
        - 7.4.4 [Invalid Constraint Values](#744-invalid-constraint-values)
        - 7.4.5 [Invalid Enum Values](#745-invalid-enum-values)
8. [Streaming](#8-streaming)
    - 8.1 [Overview](#81-overview)
    - 8.2 [Streaming Requirements](#82-streaming-requirements)
        - 8.2.1 [Two-Phase Processing](#821-two-phase-processing)
        - 8.2.2 [Schema-First Processing](#822-schema-first-processing)
- [Appendix A: Quick Reference](#appendix-a-quick-reference)
- [Appendix B: Error Codes](#appendix-b-error-codes)
- [Appendix C: ABNF (Schema Grammar)](#appendix-c-abnf-schema-grammar)
- [Appendix D: ABNF (Data Grammar)](#appendix-d-abnf-data-grammar)

---

## 1. Overview

### 1.1 Introduction

MAXI is a token-efficient data serialization format designed to minimize token usage in Large Language Model (LLM) contexts and API communications.

### 1.2 Design Principles

MAXI is built on the following core principles:

1. **Schema-based**: Define structure once, reuse for all records
   - Types are defined in a schema section or external `.mxs` files
   - Data records reference types by alias for maximum token efficiency

2. **Schema-optional**: Works with or without schema definitions
   - Pure data files (no schema) are valid MAXI (then the schema must be clear from context)
   - Inline schemas (in the same file) are supported
   - External schemas (imported via `@schema`) are supported

3. **Positional fields**: No repeated field names in data records
   - Field order is defined by the schema
   - Data records contain only values, separated by `|`
   - Reduces token count by eliminating field name repetition

4. **Minimal delimiters**: Use lightweight separators
   - Single-character delimiters: `|` `,` `:` `~`
   - Bracket pairs only when needed: `()` `[]` `{}`
   - Schema/data separator: `###`

5. **Type-optional**: Types can be specified or inferred
   - Default type is `str` if not specified
   - Explicit types provide validation and documentation
   - Parsers may infer types from values in lax mode

6. **Human-readable**: Maintains readability while optimizing for tokens
   - Clear syntax without excessive punctuation
   - Multi-line support for complex records
   - Comments allowed in schema sections

7. **Reference support**: Avoid duplicating nested objects
   - Object references by ID: `O(100|1|99.99)`
   - Inline object definitions: `O(100|(1|John|john@ex.com)|99.99)`
   - Mixed usage in the same dataset

8. **Smart defaults**: Sensible default values reduce verbosity
   - Fields can specify default values: `role=guest`
   - Omitted fields use defaults or become `null`
   - Explicit null override: `~`

9. **Strict and lax modes**: Flexible validation for different use cases
   - **Lax mode** (default): Permissive parsing with warnings
      - Allows type coercion (e.g., `"25"` → `25` for `int`)
      - Allows missing fields (filled with `null`)
      - Allows extra fields (ignored)
      - Issues warnings for schema deviations
   - **Strict mode** (`@mode:strict`): Enforces schema compliance
      - Type mismatches cause errors
      - Missing required fields cause errors
      - Extra fields cause errors
      - Forward references may be rejected (parser-dependent)

### 1.3 Quick Example

```maxi
U:User(id|name(!))
O:Order(id:int|user:U|total:decimal)
###
U(1|Julie Miller)
U(2|Matt Smith)
O(100|1|99.99)
O(101|2|149.50)
```

---

## 2. File Structure & Directives

### 2.1 File Structure

MAXI files consist of an optional schema section followed by a data section, separated by a `###` delimiter.

```maxi
[@version:X.Y.Z]
[@mode:strict]
[@schema:external.mxs]

[schema definitions]

###

[data records]
```

The schema section can contain directives and type definitions. The data section contains the data records.

The `###` separator is **required** if a schema section (containing directives or type definitions) is present.
If a file contains only data records, the separator can be omitted.

MAXI supports multiple formats:

**Schema-less (data-only file):**
```maxi
U(1|Julie|julie@maxi.org)
U(2|Matt|matt@maxi.org)
```

**Inline Schema:**
```maxi
U:User(id|name|email)
###
U(1|Julie|julie@maxi.org)
U(2|Matt|matt@maxi.org)
```

**External Schema Reference:**
```maxi
@schema:users.mxs
###
U(1|Julie|julie@maxi.org)
```

### 2.2 Delimiters

| Delimiter | Purpose                                              |
|-----------|------------------------------------------------------|
| `\|`      | Field separator (pipe)                               |
| `,`       | Array element separator (comma)                      |
| `()`      | Object wrapper, constraint definition                |
| `[]`      | Array wrapper                                        |
| `{}`      | Map wrapper                                          |
| `:`       | Type/field/constraint annotations                    |
| `<>`      | Inheritance indicator, type parameterization         |
| `###`     | Schema/Data separator                                |
| `~`       | Explicit null value                                  |
| `@`       | Directive prefix, type annotations (e.g., `str@date`)|

### 2.3 Directives

Directives provide metadata about the MAXI file and must appear before any type definitions or data records if used.
Directives are optional and can appear in any order, but they must precede schema definitions and data records.

#### 2.3.1 Version Directive

Specifies the MAXI version used in the file.

**Syntax:**
```maxi
@version:1.0.0
```

**Example:**
```maxi
@version:1.0.0
U:User(id|name|email)
###
U(1|Julie|julie@maxi.org)
```

#### 2.3.2 Mode Directive

Specifies the parsing mode for the MAXI file.

**Allowed modes:**
- `strict` - Enforces strict schema adherence.
- `lax` - Default mode, allows some schema deviations (like type coercion or extra fields) while issuing warnings.

**Lax mode allows:**
- Missing required fields (set to null if left empty), but a warning should be issued.
- Extra fields at the end of a record (ignored).
- Type mismatches, where the parser will attempt to coerce the value (e.g., a field `age:int` receiving a string value `"25"`).

**Syntax:**
```maxi
@mode:strict
```

**Example:**
```maxi
@mode:strict
U:User(id|name|age:int)
###
U(1|Anna|24)
```

#### 2.3.3 Schema Directive

Imports type definitions from external schema files. Multiple `@schema` directives can be specified, and they are processed in order.

**Syntax:**
```maxi
@schema:path/to/schema.mxs
@schema:https://example.com/schema.mxs
```

**Examples:**

Single Schema Import:
```maxi
@schema:users.mxs
###
U(1|Julie|julie@maxi.org)
```

Multiple Schema Imports:
```maxi
@schema:products.mxs
@schema:users.mxs
###
P(1|Widget|9.99)
U(1|Julie|julie@maxi.org)
```

Remote Schema Import:
```maxi
@schema:https://api.example.com/schemas/main.mxs
###
U(1|Julie|julie@maxi.org)
```

Inline Schema with Imports:
```maxi
@schema:common.mxs
# Define additional types inline
O:Order(id:int|user:User|total:decimal)
###
O(1|1|99.99)
```

**Note:** The intended pattern for APIs with multiple endpoints is to host shared schemas at a common URL (e.g., `https://api.maxi.org/schemas/api_spec.mxs`) and have each endpoint schema import it. Clients can then cache the shared schema once and reuse it across multiple requests.

### 2.4 Comments

Comments start with `#` and extend to the end of the line.
**Comments are only allowed in schema sections (before the `###` delimiter) and in `.mxs` schema files.**
Data records do not support comments for token efficiency.

**Schema files (`.mxs`) or schema sections - Comments allowed:**
```maxi
# Define user type
U:User(id:int|name|email)

# Define order type with reference to user
O:Order(id:int|user:User|total:decimal)
```

```maxi
U:User(id:int|name|email)
# ALLOWED: This comment is in the schema section.
###
U(1|Julie|julie@maxi.org)  # INVALID: The parser treats this as part of the data, not a comment.
U(2|Matt|matt@maxi.org)
```

If you need to annotate data records, use a dedicated field in your schema (for example, `notes` or `metadata`).

### 2.5 String Escaping

MAXI uses standard escape sequences within quoted strings. **All strings are UTF-8 encoded**.

#### 2.5.1 Escape Sequences

```txt
\"  - Double quote
\\  - Backslash
\n  - Newline (line feed)
\r  - Carriage return
\t  - Tab
```

#### 2.5.2 When to Quote Strings

Use double quotes when strings contain:
- Pipe `|`
- Comma `,`
- Parentheses `(` and `)`
- Brackets `[` and `]`
- Braces `{` and `}`
- Tilde `~`
- Leading or trailing whitespace
- Escape sequences
- Non-ASCII UTF-8 characters (optional; quoting is recommended only if combined with other special characters)

#### 2.5.3 Examples

```maxi
U(1|"First line\nSecond line"|email@maxi.org)
U(2|"She said \"Hello\""|email@maxi.org)
U(3|"Path: C:\\Users\\Julie"|email@maxi.org)
U(4|"Tab\tseparated"|email@maxi.org)
U(5|"Café"|restaurant)
U(6|München|city)
```

#### 2.5.4 UTF-8 Support

MAXI fully supports UTF-8 characters in both quoted and unquoted strings:

```maxi
# Unquoted UTF-8 (allowed if no special characters)
U(1|José|employee)
U(2|北京|city)
U(3|Москва|location)

# Quoted UTF-8 (always safe)
U(1|"José"|employee)
U(2|"北京"|city)
U(3|"Москва"|location)
```

#### 2.5.5 Unquoted Strings

Simple strings without special characters don't need quotes. **All leading and trailing whitespace is stripped from unquoted values:**

```maxi
U(1|Julie|julie@maxi.org|NYC)
U(1|  Julie  |email)           # name = "Julie" (whitespace stripped)
U(1|"  Julie  "|email)         # name = "  Julie  " (whitespace preserved, quoted)
U(1 | Julie | email )          # name = "Julie" (whitespace stripped)
```

### 2.6 File Format and Extensions

#### 2.6.1 File Extensions

- **`.maxi`** - MAXI data files (data with optional inline schema)
- **`.mxs`** - MAXI schema files (type definitions and meta-schema)

#### 2.6.2 Content Types

- **`application/maxi`** - MAXI data format
- **`application/maxi-schema`** - MAXI schema format (`.mxs` files)

#### 2.6.3 File Encoding

All MAXI files (`.maxi` and `.mxs`) **MUST** be encoded in UTF-8.

---

## 3. Type System

### 3.1 Primitive Types

MAXI supports the following primitive types, which are intrinsic to the format and do not require explicit definition:

| Type      | Description                                         | Example Values            |
|-----------|-----------------------------------------------------|---------------------------|
| `int`     | Integer numbers                                     | `42`, `-17`, `0`          |
| `decimal` | Decimal numbers                                     | `3.14`, `-0.5`, `100.00`  |
| `str`     | UTF-8 strings                                       | `"Hello"`, `World`, `""`  |
| `bool`    | Boolean values                                      | `true`, `false`, `1`, `0` |
| `bytes`   | Binary data (base64-encoded by default)             | `SGVsbG8=`                |

**Note:** `str` is the default type if no type annotation is provided.

**Note:** For `bool` you can either use the literals `true`/`false` (lowercase), or `1`/`0` in data records. `1` and `0` are preferred for token efficiency.

**Note:** For `bytes`, the default encoding format is `@base64`.

### 3.2 Complex Types

#### 3.2.1 Arrays

Arrays are defined using the `type[]` syntax.

**Syntax:**
```maxi
fieldName:type[]
```

**Examples:**
```maxi
U:User(
  id:int|
  name|
  tags:str[]|           # Array of strings
  scores:int[]          # Array of integers
)
```

**Data representation:**
```maxi
U(1|Julie|[tag1,tag2,tag3]|[95,87,92])
U(2|Matt|[]|[88])                        # Empty array for tags
```

#### 3.2.2 Enums

Enums restrict a field to a predefined set of values.

**Syntax:**
```maxi
fieldName:enum<type>[val1,val2,val3]
```

**Note:** `enum<str>` can be shortened to `enum` (string is the default enum type).

**Examples:**
```maxi
U:User(
  id:int|
  name|
  role:enum[admin,user,guest]|        # String enum (default)
  status:enum<int>[0,1,2]              # Integer enum
)
```

**Data representation:**
```maxi
U(1|Julie|admin|1)
U(2|Matt|user|0)
```

#### 3.2.3 Maps

Maps are key-value pairs defined using the `map<keyType,valueType>` syntax.
Map keys must be `int` or `str`.

**Syntax:**
```maxi
fieldName:map<keyType,valueType>
```

**Note:** Default key and value type is `str`, so `map<str,str>` can be shortened to `map`.
Also, `map<str,int>` can be shortened to `map<int>`.

**Examples:**
```maxi
C:Config(
  id:int|
  settings:map<str,str>| # String to string map
  scores:map<str,int>|   # String to integer map
  metadata:map           # Shorthand for map<str,str>
)
```

**Data representation:**
```maxi
C(1|{key1:value1,key2:value2}|{math:95,science:87}|{author:Julie,version:1.0})
C(1|{"key:with:colons":"value,with,commas",normal:value})
```

### 3.3 Objects & Inheritance

#### 3.3.1 Type Definitions

**Basic Syntax:**
```maxi
Alias:TypeName(field1|field2|field3)
```

**Example:**
```maxi
U:User(id:int|name|email|age:int)
```

The alias (in this case `U`) is used in data records for token efficiency.
The full type name (in this case `User`) is optional and provides documentation and clarity.

#### 3.3.2 Type Name Naming Rules

Type Names must follow these rules:
- Start with a letter (`a-z`, `A-Z`)
- Followed by zero or more letters, digits (`0-9`), underscores `_`, or hyphens `-`
- Pattern: `[a-zA-Z][a-zA-Z0-9_\-]*`
- Case-sensitive (for example, `User`, `user`, and `USER` are different)

**Valid Examples:**
```maxi
User        # Word
UserAccount # CamelCase
user_data   # With underscore
user-data   # With hyphen
User1       # With digits (but not starting with digit)
```

**Invalid Examples:**
```
1User       # ERROR: Starts with digit
2ndUser     # ERROR: Starts with digit
```

**Note:** Type names are optional in type definitions. If omitted, only the alias is used.
Type names are primarily for documentation and clarity.

#### 3.3.3 Alias Naming Rules

Aliases must follow these rules:
- Start with a letter (`a-z`, `A-Z`) or underscore `_`
- Followed by zero or more letters, digits (`0-9`), underscores `_`, or hyphens `-`
- Pattern: `[a-zA-Z_][a-zA-Z0-9_\-]*`
- Case-sensitive (for example, `User`, `user`, and `USER` are different).

**Valid Examples:**
```maxi
U           # Single letter
User        # Word
u_d         # With underscore
u-d         # With hyphen
_priv       # Starting with underscore
U1          # With digits
U_2_D       # Multiple underscores and digits
```

**Invalid Examples:**
```
1           # ERROR: Starts with digit
1U          # ERROR: Starts with digit
2nd         # ERROR: Starts with digit
```

#### 3.3.4 Alias-Only Syntax

For maximum brevity, the full type name can be omitted:
```maxi
U(id|name|email|age)
```

#### 3.3.5 Type Definition

```maxi
U:User(field:type|field:type|field:type)
```

Or without the full type name:

```maxi
U(field:type|field:type|field:type)
```

**Example:**
```maxi
U:User(id:int|name|email|age:int)
# Equivalent to:
U(id:int|name|email|age:int)
```

#### 3.3.6 Multi-line Definitions

For readability, type definitions can span multiple lines:

```maxi
O:Order(
  id:int|
  user:User|
  items:Item[]|
  total:decimal|
  status|
  createdAt
)
```

Within the parentheses `()`, line breaks and indentation are ignored. The parser treats this the same as a single-line definition.

**Whitespace Handling in Schema:**

All whitespace (spaces, tabs, newlines) inside type definition parentheses `()` is ignored and stripped away:

```maxi
# These three definitions are functionally identical:

# Single-line
O:Order(id:int|user:User|items:Item[]|total:decimal|status|createdAt)

# OR Multi-line with indentation
O:Order(
  id:int|
  user:User|
  items:Item[]|
  total:decimal|
  status|
  createdAt
)

# OR Mixed formatting
O:Order(id:int|user:User|
  items:Item[]|
  total:decimal|status|
  createdAt)
```

#### 3.3.7 Type Inheritance

MAXI supports single and multiple inheritance. A type can inherit fields from one or more parent types using the `<>` syntax.

**Syntax:**
```maxi
Child:ChildType<Parent1,Parent2>(own_field1|own_field2)
```

**Field Merging Rules:**

1. Inherited fields appear first, in the order of inheritance declaration
2. Own fields are appended after inherited fields
3. If a field name appears in multiple inherited types, the first occurrence (leftmost parent) takes precedence
4. If a field name is redefined in the child type, it overwrites the inherited version

**Example with Single Inheritance:**

```maxi
P:Person(id:int|name|email)
U:User<P>(role|status)
###
U(1|Julie|julie@maxi.org|admin|active)
U(2|Matt|matt@maxi.org|user|inactive)
```

Field order for `User`: `id` (inherited) | `name` (inherited) | `email` (inherited) | `role` (own) | `status` (own)

**Example with Multiple Inheritance:**

```maxi
TS:TimeStamp(createdAt|updatedAt)
P:Person(id:int|name|email)
U:User<P,TS>(role)
###
U(1|Julie|julie@maxi.org|2024-01-15|2024-11-20|admin)
U(2|Matt|matt@maxi.org|2024-02-10|2024-11-21|user)
```

Field order for `User`: `id` (from P) | `name` (from P) | `email` (from P) | `createdAt` (from TS) | `updatedAt` (from TS) | `role` (own)

```maxi
P:Person(id:int|name|status=active)
U:User<P>(status=admin)
###
U(1|Julie)           # status defaults to "admin" (overridden in User)
```

When `User` redefines `status`, it replaces the inherited version entirely (including any constraints or defaults from `Person`).

**Example with Conflicting Fields (Multiple Inheritance):**

```maxi
A:Animal(name|type)
C:Creature(name|age:int)
D:Dog<A,C>(breed)
###
D(dog_name|animal|5|labrador)
```

Field order for `Dog`: `name` (from A, first parent takes precedence) | `type` (from A) | `age` (from C) | `breed` (own)

The `name` field from `A` is used; the `name` from `C` is ignored due to inheritance order.

**Important Notes:**

- Parent types must be resolvable during the schema resolution phase (they may be defined earlier in the same file or in imported schemas; forward references are allowed as long as no circular inheritance occurs)
- Circular inheritance is not allowed (A inherits B, B inherits A)
- Field count and order must be consistent across all records of the same type
- When inheriting, all parent constraints (`id`, `!`, `<10`, etc.) are inherited unless overridden

### 3.4 Type Annotations

MAXI supports type annotations to provide additional metadata about field values.

**Syntax:**
```maxi
fieldName:type@annotation
```

**Supported Annotations:**

| Annotation   | Description                            | Valid Types |
|--------------|----------------------------------------|-------------|
| `@base64`    | Base64 encoding (default for `bytes`)  | `bytes`     |
| `@hex`       | Hexadecimal encoding                   | `bytes`     |
| `@timestamp` | Unix timestamp (seconds since epoch)   | `int`       |
| `@date`      | Date in `YYYY-MM-DD` format (ISO 8601) | `str`       |
| `@datetime`  | Date-time in ISO 8601 format           | `str`       |
| `@time`      | Time in `HH:MM:SS` format (ISO 8601)   | `str`       |
| `@email`     | Email address                          | `str`       |
| `@url`       | URL                                    | `str`       |
| `@uuid`      | UUID                                   | `str`       |

**Example:**
```maxi
U:User(
  id:int|
  name|
  email:str@email|
  createdAt:str@datetime|
  birthDate:str@date|
  avatar:bytes@base64
)
```

**Note:** The `@base64` annotation for `bytes` fields is optional and represents the default encoding.

---

## 4. Constraints

### 4.1 General Constraints

Constraints are specified in type definitions after the type annotation using the following syntax:

```maxi
fieldName:type(constraint1,constraint2,...)
```

**Base Constraint:**

| Constraint | Description                                      |
|------------|--------------------------------------------------|
| `!`        | Field is required (must be present and non-null) |

**Example:**

```maxi
U:User(
  name(!)|       # Required field
  email|         # Optional field
  role=guest     # Default value "guest" but not required, can be set to null by using `~`
)
```

**Note on Required Fields with Defaults:**
- A field marked with `!` (required) AND having a default value means:
   - The field must always have a value (never null)
   - If omitted, the default is used
   - Explicit null (`~`) is **not allowed** and will cause a validation error
- A field with only a default (no `!`) means:
   - If omitted, the default is used
   - Explicit null (`~`) is **allowed** and overrides the default

### 4.2 String Constraints

The following constraints apply to `str` fields:

| Constraint          | Description                                                              |
|---------------------|--------------------------------------------------------------------------|
| `>=N`               | Minimum length of the string (N is a non-negative integer)               |
| `>N`                | Minimum length greater than N (N is a non-negative integer)              |
| `<=N`               | Maximum length of the string (N is a non-negative integer)               |
| `<N`                | Maximum length less than N (N is a non-negative integer)                 |
| `pattern:<PATTERN>` | Regular expression pattern that the string must match (quoted if needed) |

**Example:**

```maxi
U:User(
  username(>=3,<=20,pattern:^[a-zA-Z0-9_]+$)| # Alphanumeric with underscores, 3-20 chars
  email:str@email(!)                          # Valid email, required
)
```

### 4.3 Numeric Constraints

#### 4.3.1 Integer Constraints

The following constraints apply to `int` fields:

| Constraint | Description                      |
|------------|----------------------------------|
| `>=N`      | Minimum value (N is an integer)  |
| `>N`       | Greater than N (N is an integer) |
| `<=N`      | Maximum value (N is an integer)  |
| `<N`       | Less than N (N is an integer)    |

**Example:**

```maxi
P:Product(
  id:int|                        # Unique identifier
  name(!)|                       # Required name
  stock:int(>=0)                 # Stock must be non-negative
)
```

#### 4.3.2 Decimal Constraints

The following constraints apply to `decimal` fields:

| Constraint | Description                                                                                                                                           |
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| `>=N`      | Minimum value (N is a decimal number)                                                                                                                 |
| `>N`       | Greater than N (N is a decimal number)                                                                                                                |
| `<=N`      | Maximum value (N is a decimal number)                                                                                                                 |
| `<N`       | Less than N (N is a decimal number)                                                                                                                   |
| `M:N.X:Y`  | Precision and scale (M is the min digits before decimal, N is max digits before decimal, X is min digits after decimal, Y is max digits after decimal) |
| `M:N.X`    | Precision and scale shorthand (M is the min digits before decimal, N is max digits before decimal, X is exact digits after decimal)                     |
| `N.X:Y`    | Precision and scale shorthand (N is max digits before decimal, X is min digits after decimal, Y is max digits after decimal)                           |
| `N.X`      | Precision and scale shorthand (N is max digits before decimal, X is exact digits after decimal)                                                        |
| `.X:Y`     | Scale only (X is min digits after decimal, Y is max digits after decimal)                                                                             |
| `.X`       | Scale only (X is exact digits after decimal)                                                                                                          |
| `M:N.`     | Precision only (M is min value before decimal, N is max value before decimal)                                                                         |
| `N.`       | Precision only (N is max value before decimal)                                                                                                        |

**Examples:**

Basic example showing various decimal constraints:

```maxi
I:Invoice(
  id:int|                         # Unique identifier
  total:decimal(>=0,0:10.2)|      # Total must be non-negative, max 10 digits before decimal, 2 digits after
  tax:decimal(>=0,0:10.0:2)       # Tax must be non-negative, up to 10 digits before decimal, between 0 and 2 digits after decimal
)
P:Product(
  price:decimal(>=0,0:7.2)|       # Price must be non-negative, max 7 digits before decimal, 2 digits after
  discount:decimal(>=0,0:5.2)     # Discount must be non-negative, max 5 digits before decimal, 2 digits after
)
```

More detailed examples due to the complexity of decimal constraints:

```maxi
DE:DecimalExamples(
  price:decimal(5.2)|       # Max 99999.99 (5 digits before, 2 after)
  tax:decimal(0:100.2)|     # 0.00 to 100.00
  ratio:decimal(.2:4)|      # 0.01 to 0.9999 (min 2, max 4 decimal places), undefined integer part
  count:decimal(1:999.)     # 1 to 999, undefined decimal places
)
```

#### 4.3.3 Binary Constraints

The following constraints apply to `bytes` fields:

| Constraint  | Description                                                                                                                                 |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| `>=N`       | Minimum length of the byte array (N is a non-negative integer)                                                                              |
| `>N`        | Minimum length greater than N (N is a non-negative integer)                                                                                 |
| `<=N`       | Maximum length of the byte array (N is a non-negative integer)                                                                              |
| `<N`        | Maximum length less than N (N is a non-negative integer)                                                                                    |
| `mime:type` | MIME type of the binary data (e.g., `mime:image/png`); supports wildcards like `mime:image/*` and a comma-separated list surrounded by `[]` |

**Note:** Binary encoding format is specified via type annotation (`@base64`, `@hex`).

**Example:**

```maxi
F:File(
  filename(!)|                                       # Required filename
  data:bytes(>=1,mime:application/pdf)|              # Non-empty PDF file (base64 by default)
  thumbnail:bytes@base64(mime:[image/png,image/jpg]) # PNG/JPG thumbnail, explicit base64 format
)
```

**Data representation:**

```maxi
F(
  report.pdf|
  SGVsbG8gV29ybGQh|
  iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg==
)
```

### 4.4 Array Constraints

The following constraints apply to array fields (`type[]`):

For arrays, the constraints can be defined for the array itself and for the individual elements.

**Syntax:**
```maxi
fieldName:elementType(elementConstraints)[](arrayConstraints)
```

**Array Constraints:**

| Constraint | Description                                                             |
|------------|-------------------------------------------------------------------------|
| `>=N`      | Minimum number of elements in the array (N is a non-negative integer)   |
| `>N`       | Minimum number of elements greater than N (N is a non-negative integer) |
| `<=N`      | Maximum number of elements in the array (N is a non-negative integer)   |
| `<N`       | Maximum number of elements less than N (N is a non-negative integer)    |
| `=N`       | Exact number of elements in the array (N is a non-negative integer)     |

**Examples:**

```maxi
U:User(
  id:int|                             # Unique identifier
  name(!)|                            # Required name
  tags:str[](>=1,<=5)|                # Array of strings with 1 to 5 elements
  scores:int(>=0,<=100)[]             # Array of integers between 0 and 100
)
```

```maxi
P:Product(
  # An array of 1 to 10 tags. Each tag must be a string of 3 to 20 chars.
  tags:str(>=3,<=20)[](>=1,<=10)
)
```

### 4.5 Map Constraints

The following constraints apply to map fields (`map<keyType,valueType>`):

| Constraint | Description                                                                    |
|------------|--------------------------------------------------------------------------------|
| `>=N`      | Minimum number of key-value pairs in the map (N is a non-negative integer)     |
| `>N`       | Minimum number of key-value pairs greater than N (N is a non-negative integer) |
| `<=N`      | Maximum number of key-value pairs in the map (N is a non-negative integer)     |
| `<N`       | Maximum number of key-value pairs less than N (N is a non-negative integer)    |
| `=N`       | Exact number of key-value pairs in the map (N is a non-negative integer)       |

**Syntax for constrained maps:**
```maxi
map<keyType(keyConstraints),valueType(valueConstraints)>(mapConstraints)
```

**Example:**

```maxi
C:Config(
  settings:map<str(>=1,<=50),int(>=0,<=100)>(>=1,<=10) # Map with string keys (1-50 chars) and integer values (0-100), 1-10 entries
)
```

---

## 5. Data Records

### 5.1 Basic Syntax

#### 5.1.1 Single-line Records

The simplest form uses a single line per record:

```maxi
U:User(id:int|name|email)
###
U(1|Julie|julie@maxi.org)
U(2|Matt|matt@maxi.org)
```

Every data record must be prefixed with its type alias.

#### 5.1.2 Multi-line Records

For records with long or complex field values, use multi-line format:

```maxi
U:User(id:int|name|email|bio)
###
U(
1|Julie|
julie@maxi.org|
"Software engineer with 10+ years experience"
)
U(
2|Matt|
matt@maxi.org|
"Product designer and UX specialist"
)
```

The type alias must still be present, followed by a parenthesized list of field values (one per line or multiple per line).

**Note:** Values can appear on the same line or on separate lines; all whitespace between `(` and `)` is normalized to field separators `|`.

### 5.2 Field Values & Null Semantics

The meaning of a field's value depends on how it is provided in a data record.

- **Value Provided (`...|value|...`)**: The field is set to the provided value. For unquoted values, leading and trailing whitespace is stripped. For quoted values, whitespace is preserved.
- **Empty (`...||...`)**: The field is set to null or the default value if one is defined in the schema.
- **Explicit Null (`...|~|...`)**: The field is explicitly set to `null`. This is primarily used to override a default value and set the field to null instead.
- **Omitted (at the end of a record)**: If a record ends before all fields are specified, the remaining fields are considered "omitted".
    - If an omitted field has a `default` value, it takes that value.
    - If an omitted field has no `default` value, it is set to `null`.

**Default Behavior:**

By default, **all fields are `null`** unless:
1. A value is explicitly provided
2. A `default` value is defined in the schema
3. The field is omitted and a default exists

**Null vs Default:**

| Syntax      | Meaning                                       | When to Use                                                   |
|-------------|-----------------------------------------------|---------------------------------------------------------------|
| `\|value\|` | Field has a value                             | Normal case                                                   |
| `\|\|`      | Field is null or the default value if defined | When you need an empty string you need to add a default `=""` |
| `\|~\|`     | Field is null (overrides default)             | When you want null despite having a default value             |
| *(omitted)* | Field is null OR uses default                 | When field can be inferred from schema defaults               |
**Examples:**

```maxi
U:User(id:int|name|role|bio)
###
U(1|Julie|admin|"Developer")   # All fields provided
U(2|Matt||"Designer")          # id=2, name="Matt", role=null, bio="Designer"
U(3|Anna)                      # id=3, name="Anna", role=null, bio=null
```

**Examples with defaults:**

```maxi
U:User(id:int|name=John|role=user|status)
###
U(1)                 # id=1, name="John" (default), role="user" (default), status=null
U(2|Matt)            # id=2, name="Matt", role="user" (default), status=null
U(3|~|admin|active)  # id=3, name=null (explicit), role="admin", status="active"
```

**Examples Empty Strings vs Null:**

```maxi
# Schema with default empty string
U:User(id:int|name="")

# Data
U(1|John)        # name = "John"
U(2)             # name = "" (uses default)
U(3|~)           # name = null (explicit override)
U(4|"")          # name = "" (explicit empty string, even without default)
```

**Key Points:**

- `null` is the default state for all fields
- `~` is **only needed** when you want to set a field to `null` despite having a `default` value defined
- Trailing omitted fields automatically use their defaults or become `null`

### 5.2.1 Default Value Definition

It is possible to define default values for fields in type definitions.
Default values are used when a field is omitted or left empty in a data record.

**Syntax:**
```maxi
fieldName:type=defaultValue
```

**Example:**

```maxi
U:User(
  name|           # Optional string field
  role(!)=member  # Default value "member", field is required
)
```

**Note:** Default values must be compatible with the field's type and constraints.

**Note:** Default values are defined after the constraints (if any).

## 5.3 Objects (Inline vs Reference)

MAXI supports both **inline object definitions** and **object references** for fields that are typed as other objects. This provides flexibility to avoid duplication while allowing full object embedding when needed.

### 5.3.1 Object References

When a field is defined as another type, you can reference an existing object by its identifier instead of embedding the full object.

**Example:**

```maxi
U:User(id:int|name|email)
O:Order(id:int|user:U|total:decimal)
###
U(1|Julie|julie@maxi.org)
U(2|Matt|matt@maxi.org)
O(100|1|99.99)           # References User with id=1
O(101|2|149.50)          # References User with id=2
```

In this example, `O(100|1|99.99)` references the User object with `id=1`.

### 5.3.2 Inline Object Definitions

Instead of referencing an existing object, you can define the object inline by wrapping it in parentheses.

**Syntax:**
```maxi
TypeAlias(field1|(inlineField1|inlineField2|...)|field3)
```

**Example:**

```maxi
U:User(id:int|name|email)
O:Order(id:int|user:U|total:decimal)
###
U(1|Julie|julie@maxi.org)
O(100|1|99.99)                              # References User id=1
O(101|(2|Matt|matt@maxi.org)|149.50)        # Inline User definition
```

In the second Order record, `(2|Matt|matt@maxi.org)` defines a User object inline.

### 5.3.3 Mixed Usage

You can mix references and inline definitions in the same dataset:

**Example:**

```maxi
U:User(id:int|name|email)
A:Address(id:int|street|city|zip)
O:Order(id:int|user:U|shipTo:A|total:decimal)
###
U(1|Julie|julie@maxi.org)
A(1|123 Main St|NYC|10001)
O(100|1|1|99.99)                                         # References both User and Address
O(101|(2|Matt|matt@maxi.org)|1|149.50)                   # Inline User, reference Address
O(102|1|(2|456 Oak Ave|LA|90001)|199.99)                   # Reference User, inline Address
O(103|(3|Anna|anna@maxi.org)|(3|789 Elm|SF|94102)|249.99)  # Both inline
```

### 5.3.4 Nested Inline Objects

Inline definitions can be nested to any depth:

**Example:**

```maxi
C:Company(id:int|name)
U:User(id:int|name|email|company:C)
O:Order(id:int|user:U|total:decimal)
###
C(1|ACME Corp)
O(100|(1|Julie|julie@maxi.org|1)|99.99)           # User inline, Company ref
O(101|(2|Matt|matt@maxi.org|(2|Beta Inc))|149.50) # User inline, Company inline
```

### 5.3.5 When to Use References vs Inline

**Use References When:**
- The same object appears in multiple records (avoid duplication)
- Object data is defined separately from its usage
- Building normalized data structures
- Object is complex and frequently reused

**Use Inline When:**
- Object is used only once
- Data is naturally hierarchical (e.g., order items within an order)
- Embedding improves readability
- Avoiding forward reference complexity

### 5.3.6 Object Identifiers
Identifiers are defined implicitly by the attribute name `id` with type `int` or `str`, or can be explicitly marked with the `id` constraint:

```maxi
P:Product(id:int|name|price:decimal)                                       # id is implicit identifier
U:User(id|name|email)                                                      # id is implicit identifier
O:Order(order_id:int(id)|user:U|total:decimal)                             # order_id is explicit identifier
PV:ProductVariant(sku(id)|product:P|color|size)                            # sku is explicit identifier
OI:OrderItem(id:int|order_item_id:int(id)|product_variant:PV|quantity:int) # order_item_id is the identifier
```

**Rules for Object Identifiers:**
- When a field is designated as an object identifier, it is automatically required
- If a type has a field named `id` but you want another field to be the identifier, you must explicitly mark the desired field with the `id` constraint.
- Only one field per type can be marked as an identifier.
- If multiple objects of the same type use the same identifier value, the parser raises a `DuplicateIdentifierError`.

### 5.3.7 Reference Resolution

**Parser Behavior:**

1. **Immediate Resolution** (preferred): When a reference is encountered (e.g., `O(100|1|99.99)`), the parser looks up the referenced object by ID.
2. **Forward References**: If the referenced object hasn't been defined yet, parsers **MAY** support forward references by deferring resolution until all records are loaded.
3. **Strict Mode**: In `@mode:strict`, unresolved references **MUST** trigger an error.

**Example of Forward Reference:**

```maxi
U:User(id:int|name|email)
O:Order(id:int|user:U|total:decimal)
###
O(100|1|99.99)           # Forward reference to User id=1
U(1|Julie|julie@maxi.org)  # User defined after being referenced
```

Parsers **SHOULD** support this pattern in lax mode, but **MAY** reject it in strict mode.

### 5.4 Arrays & Maps

#### 5.4.1 Arrays

Arrays are represented using brackets `[]` with comma-separated values.

**Example:**

```maxi
U:User(id:int|name|tags:str[]|scores:int[])
###
U(1|Julie|[tag1,tag2,tag3]|[95,87,92])
U(2|Matt|[]|[88])                        # Empty array for tags
```

#### 5.4.2 Maps

Maps are represented using braces `{}` with comma-separated key:value pairs.

**Example:**

```maxi
C:Config(id:int|settings:map<str,str>|scores:map<str,int>)
###
C(1|{key1:value1,key2:value2}|{math:95,science:87})
C(2|{}|{})                                              # Empty maps
```

---

## 6. Schema Management

### 6.1 Imports

Schema files can import other schema files using the `@schema` directive. Multiple schemas can be imported, and they are processed in order.

**Example:**

File: `base.mxs`
```maxi
B:Base(id:int|name)
```

File: `common.mxs`
```maxi
@schema:base.mxs
C:Common<B>(description)
```

File: `main.maxi`
```maxi
@schema:common.mxs
# Both Base and Common are now available
###
B(1|BaseItem)
C(2|CommonItem|Description)
```

**Processing Order:**

1. `main.maxi` requests `common.mxs`
2. `common.mxs` requests `base.mxs`
3. Parser loads `base.mxs` first (dependency)
4. Parser loads `common.mxs` (can now resolve `B`)
5. Parser processes `main.maxi` (has access to both `B` and `C`)

### 6.2 Overrides

When multiple schemas define the same type alias, **the last definition wins** for the current file's scope. However, within each imported schema file, the definitions remain as originally defined.

**Example:**

File: `products.mxs`
```maxi
TS:TimeStampable(createdAt:str@date|updatedAt:str@date)
P:Product<TS>(id:int|name|price:decimal)
# P inherits from the local TS (with createdAt and updatedAt)
```

File: `users.mxs`
```maxi
TS:TimeStampable(createdAt:str@datetime)  # Different definition
U:User<TS>(id:int|name)
# U inherits from the local TS (with createdAt)
```

File: `main.maxi`
```maxi
@schema:products.mxs
@schema:users.mxs

# Define your own TS, overriding both imports
TS:TimeStampable(createdAt:int@timestamp)
###
P(2025-06-11|2025-12-17|1|Gadget|49.99)
U(2025-10-26T16:47:30+00:00|42|Alice)
TS(1234567890)
```

**Override Behavior:**

1. **Import Order Matters**: Schemas are loaded in declaration order. Later imports override earlier ones.
2. **Local Definitions Win**: Type definitions in the current file override all imported definitions with the same alias.
3. **File-Local Scope**: Within an imported file (e.g., `products.mxs`), its own internal definitions are used, not overrides from other files.
4. **Explicit Override**: You can redefine any imported type in your own schema to customize it.

### 6.3 Circular Imports

Circular imports are permitted and handled correctly by the parser.

**Example:**

File: `users.mxs`
```maxi
@schema:orders.mxs
U:User(id:int|name|orders:Order[])
```

File: `orders.mxs`
```maxi
@schema:users.mxs
O:Order(id:int|user:User|total:decimal)
```

**Parser Behavior:**

1. When loading `users.mxs`, the parser encounters `@schema:orders.mxs`
2. While loading `orders.mxs`, it encounters `@schema:users.mxs` (circular reference)
3. Parser detects the cycle and skips re-loading `users.mxs`
4. Both type definitions are registered in the schema registry
5. Forward references are resolved after all schemas are loaded

**Important Notes:**
- Type resolution happens after all schemas in the dependency graph are loaded

- Parsers **MUST** track which files are currently being loaded to detect cycles
- Parsers **MUST** allow forward references within circular imports
- Circular imports do **NOT** cause infinite loops

**Circular Import Error Case:**

Circular imports are fine, but circular **inheritance** is not:

```maxi
# users.mxs
@schema:orders.mxs
U:User<O>(id:int|name)  # ERROR: Circular inheritance

# orders.mxs
@schema:users.mxs
O:Order<U>(id:int|total:decimal)  # ERROR: Circular inheritance
```

This will trigger: `CircularInheritanceError: Circular inheritance detected: U -> O -> U`

### 6.4 Schema Resolution Algorithm

Parsers **MUST** follow this algorithm:

1. **Initialization**:
    - Create an empty schema registry
    - Create an empty "loading stack" to track circular imports

2. **For each `@schema` directive**:
    - Resolve the file path (relative or absolute URL)
    - Check if the file is already in the "loading stack" (circular import)
        - If yes: Skip loading (already being processed)
        - If no: Continue
    - Add the file to the "loading stack"
    - Load and parse the schema file recursively
    - Process all `@schema` directives in the loaded file (recursive)
    - Register all type definitions from the loaded file
    - Remove the file from the "loading stack"

3. **Process inline schema**:
    - Register all type definitions from the current file
    - These override any imported definitions with the same alias

4. **Resolve inheritance**:
    - After all schemas are loaded, resolve parent types for inherited types
    - Detect circular inheritance and fail if found

5. **Build final schema**:
    - The final schema registry contains all types, with local definitions overriding imports

---

## 7. Error Handling

This section defines how parsers should handle errors and malformed data. Compliance levels are defined as:

- **MUST**: Required behavior; all compliant parsers must implement it
- **SHOULD**: Recommended behavior; parsers should implement it unless there is a specific reason not to
- **MAY**: Optional behavior; parsers can choose to implement it for better user experience

### 7.1 Required Parser Behaviors (MUST)

Parsers **MUST** reject (fail-fast) in these cases:

- **Unsupported @version**: File specifies a version the parser doesn't support
    - Error: `UnsupportedVersionError: Parser supports v1.0.0, file requires v2.0.0`

- **Duplicate type aliases in schema**: Same alias defined multiple times in a single schema
    - Error: `DuplicateTypeError: Type alias 'U' defined multiple times`

- **Invalid type reference**: A field references a type that doesn't exist in the schema
    - Error: `UnknownTypeError: Type 'NonExistent' referenced in field 'user' but not defined`

- **Malformed records with @mode:strict**: When strict mode is enabled:
    - Record has wrong number of fields (too many or too few without defaults)
    - Field value type doesn't match schema type (e.g., string value for int field)
    - Forward references cannot be resolved
    - Error: `SchemaValidationError: Record 'U(2)' missing required fields: name, email`

- **Circular inheritance**: Type A inherits from B, which inherits from A
    - Error: `CircularInheritanceError: Circular inheritance detected: A -> B -> A`

- **Undefined parent type**: After schema resolution, a child type still refers to a parent that does not exist
     - Error: `UndefinedParentError: Type 'User' inherits from 'Person', but 'Person' is not defined in any loaded schema`

- **Invalid constraint syntax**: Malformed constraint (for example, missing closing parenthesis)
    - Error: `ConstraintSyntaxError: Invalid constraint '(>=3' in field 'age'`

- **Invalid array syntax**: Arrays with unmatched brackets or invalid element syntax
    - Error: `ArraySyntaxError: Malformed array '[a,b,c' in record`

### 7.2 Recommended Parser Behaviors (SHOULD)

Parsers **SHOULD** implement these behaviors for better diagnostics:

- **Line number in error messages**: Include file line and column information
    - Example: `SchemaValidationError at line 5, column 12: Type alias 'U' already defined`

- **Warn on unknown directives**: Unknown `@` directives should generate a warning but not fail
    - Warning: `UnknownDirective: Directive '@debug' is not recognized; ignoring`

- **Validate reference targets in non-strict mode**: Even without @mode:strict, warn when:
    - A referenced ID doesn't exist in the data
    - Field type doesn't match the referenced type
    - Warning: `UnresolvedReference: Record references user ID '999', but no such user found`

- **Suggest corrections**: When type alias is not found, suggest similar aliases
    - Error: `UnknownTypeError: Type 'User' not found. Did you mean 'U'?`

- **Validate constraints**: Check that constraint values are valid
    - Error: `InvalidConstraintValueError: Constraint '(>=-5)' invalid for field 'age' (expected non-negative for unsigned types)`

- **Type coercion warnings**: Warn when a value can be coerced but might lose data
    - Warning: `TypeCoercion: String '123.45' coerced to int, fractional part lost`

### 7.3 Optional Parser Behaviors (MAY)

Parsers **MAY** implement these for enhanced user experience:

- **Best-effort parsing**: Continue parsing after recoverable errors (missing optional fields, unknown directives)
    - Collect all errors and report at end instead of failing immediately

- **Lax mode helpers**: Provide tooling to assist development/testing
    - Allow undefined types
    - Allow extra fields beyond schema
    - Allow type mismatches with warnings

- **Performance warnings**: Warn about performance issues
    - Warning: `PerformanceWarning: Large array with 10000 elements in single record`

- **Compatibility mode**: Support legacy MAXI versions with automatic conversion
    - Note: Conversion should be explicit and documented

### 7.4 Handling Specific Error Cases

#### 7.4.1 Unknown Type Aliases

**In Strict Mode (@mode:strict):**
- **MUST** reject record with unknown type alias
- **SHOULD** provide suggestion if similar alias exists
- Example: `UnknownTypeError: Type 'Usr' not found. Did you mean 'U'?`

**In Lax Mode (default):**
- **MAY** attempt to parse the record anyway
- **SHOULD** warn about unknown type
- Example: `Warning: Unknown type 'Usr', attempting best-effort parsing`

#### 7.4.2 Missing Required Fields

**In Strict Mode (@mode:strict):**
- **MUST** reject record
- Example: `SchemaValidationError: Required field 'email' missing in record`

**In Lax Mode (default):**
- **MAY** use default value if defined
- **MAY** set field to null if not defined
- **SHOULD** warn about missing field
- Example: `Warning: Required field 'email' missing, setting to null`

#### 7.4.3 Type Mismatches

**In Strict Mode (@mode:strict):**
- **MUST** reject record if field value doesn't match declared type
- Example: `TypeMismatchError: Field 'age' expects int, got string 'twenty-five'`

**In Lax Mode (default):**
- **MAY** attempt type coercion
- **SHOULD** warn on coercion
- Example: `Warning: Coercing string '25' to int for field 'age'`

#### 7.4.4 Invalid Constraint Values

Parsers encounter invalid constraints when:
- Constraint value is invalid for its type (e.g., `(>=abc)` where an integer is expected)
- A type annotation contradicts the field's base type (e.g., `@email` on an `int` field)
- Conflicting constraints are applied (e.g., `(>=10,<=5)`)

**In Both Modes:**
- **MUST** reject the schema during schema validation phase
- **SHOULD** provide specific error with context

**Examples:**
- Error: `ConstraintSyntaxError: Constraint value 'abc' is not a valid integer in '(>=abc)'`
- Error: `InvalidConstraintError: Type annotation '@email' cannot be applied to 'int' field`
- Error: `ConstraintConflictError: Constraint '(>=10)' conflicts with '(<=5)' in field 'count'`

#### 7.4.5 Invalid Enum Values

**In Strict Mode:**
- **MUST** reject record with value not in enum definition
- Example: `EnumValidationError: Value 'superadmin' not in enum [admin,user,guest] for field 'role'`

**In Lax Mode:**
- **MAY** accept the value with a warning
- **SHOULD** warn about invalid enum value
- Example: `Warning: Value 'superadmin' not in defined enum for field 'role'`
---

## 8. Streaming

### 8.1 Overview

MAXI is designed to support streaming parsing for large files, enabling efficient processing of datasets that exceed available memory. In streaming mode, parsers process records one at a time without loading the entire file into memory.

**Key Benefits:**
- **Memory Efficiency**: Process arbitrarily large files with constant memory usage
- **Latency Reduction**: Begin processing records before entire file is received
- **Network Efficiency**: Process data as it arrives over network connections
- **Scalability**: Handle multi-gigabyte datasets on resource-constrained devices

### 8.2 Streaming Requirements

Parsers that implement streaming **MUST** adhere to the following requirements:

#### 8.2.1 Two-Phase Processing

1. **Schema Phase**:
    - Parse and fully resolve all directives (`@version`, `@mode`, `@schema`)
    - Load and process all imported schema files
    - Parse all inline type definitions
    - Build complete schema registry
    - Validate schema consistency (no circular inheritance, all types defined)

2. **Data Phase**:
    - Process records one at a time
    - Emit parsed records via callback/iterator
    - Validate each record against schema (if strict mode enabled)
    - Resolve object references incrementally

#### 8.2.2 Schema-First Processing

The `###` delimiter marks the transition from schema to data. Parsers **MUST**:
- Complete all schema loading before processing any data records
- Buffer or block data records until the schema phase is complete
- Fail fast if schema phase encounters errors

**Example:**

```maxi
@version:1.0.0
@schema:https://api.example.com/schemas/users.mxs
U:User(id:int|name|email)
###
U(1|Julie|julie@maxi.org)
U(2|Matt|matt@maxi.org)
... (millions more records)
```

Parser behavior:
1. Parse `@version` directive
2. Fetch and parse remote schema `users.mxs`
3. Parse inline `User` type definition
4. Encounter `###` delimiter → schema phase complete
5. Begin streaming data records

---

## Appendix A: Quick Reference

### Directives
```maxi
@version:1.0.0
@mode:strict              # or @mode:lax, or omit (lax is default)
@schema:path/to/file.mxs
@schema:https://example.com/schema.mxs
```

### File Structure
```maxi
[directives]
[schema definitions]
###
[data records]
```

### Delimiters
```
|       Field separator
,       Array/map element separator
()      Object wrapper, constraints
[]      Array wrapper
{}      Map wrapper
:       Type annotation, map key-value separator
<>      Inheritance, type parameters
###     Schema/data separator
~       Explicit null
@       Directive prefix, type annotation
```

### Primitive Types
```maxi
int                      # Integer
decimal                  # Decimal number
str                      # String (default if no type)
bool                     # Boolean (true/false or 1/0)
bytes                    # Binary data (base64 by default)
```

### Type Annotations
```maxi
str@email               # Email address
str@url                 # URL
str@uuid                # UUID
str@date                # YYYY-MM-DD
str@datetime            # ISO 8601 datetime
str@time                # HH:MM:SS
int@timestamp           # Unix timestamp
bytes@base64            # Base64 encoding (default)
bytes@hex               # Hexadecimal encoding
```

### Type Definitions
```maxi
# Basic
U:User(id|name|email)

# With types
U:User(id:int|name|email)

# Alias only (no full type name)
U(id:int|name)

# With constraints and defaults
U:User(
  id:int(!)|                    # Required
  name(>=3,<=50)|           # Length constraints
  email:str@email(!)|           # Required email
  age:int(>=0,<=120)|           # Range constraints
  role=guest|               # Default value
  status:enum[active,inactive]  # Enum
)

# Multi-line
O:Order(
  id:int|
  user:User|
  total:decimal
)
```

### Inheritance
```maxi
# Single inheritance
P:Person(id:int|name)
U:User<P>(role)

# Multiple inheritance
TS:TimeStamp(createdAt|updatedAt)
U:User<P,TS>(role)
U:User<P>(status=registered)
# Field override
P:Person(status=active)
U:User<P>(status=admin)
```

### Arrays
```maxi
# Schema
tags:str[]                      # String array
scores:int(>=0,<=100)[]         # Array of constrained ints
items:Item[]                    # Array of objects
matrix1:int[][](>=2,<=10)       # Outer array has 2-10 elements (each element is int[])
matrix2:int[](>=2,<=10)[]       # Each inner array has 2-10 elements (int values)

# Data
U(1|[tag1,tag2,tag3])
U(2|[])                         # Empty array
```

### Maps
```maxi
# Schema
settings:map                    # map<str,str> (shorthand)
settings:map<str,str>           # String to string
scores:map<str,int>             # String to int (or map<int>)
data:map<int,str>               # Int to string
config:map<map>                 # Map of maps(str keys/values) with str keys
meta:map(>=1,<=10)              # Size constraints

# Data
C(1|{key1:value1,key2:value2})
C(2|{})                         # Empty map
C(3|{"key:with:colon":"value,with,comma"})
```

### Enums
```maxi
# Schema
role:enum[admin,user,guest]              # String enum (default)
status:enum<int>[0,1,2]                  # Integer enum
priority:enum<str>[low,medium,high]      # Explicit string enum

# Data
U(1|admin)
U(2|1)
```

### Constraints

**General:**
```maxi
field:type(!)           # Required
field:type=default      # Default value
field:type(!,>=3)       # Multiple constraints
```

**Strings:**
```maxi
name(>=3,<=50)                    # Length 3-50
username(pattern:^[a-z0-9]+$)     # Pattern match
```

**Numbers:**
```maxi
age:int(>=0,<=120)                    # Range
price:decimal(>=0,0:10.2)             # Min + precision (max 10 digits before, 2 digits after)
ratio:decimal(.2:4)                   # Scale only (2-4 digits after decimal), undefined integer part
count:decimal(1:999.)                 # 1 to 999, undefined decimal places
```

**Arrays:**
```maxi
tags:str[](>=1,<=5)                   # 1-5 elements
scores:int(>=0,<=100)[](=3)           # Exactly 3 elements, each 0-100
```

**Maps:**
```maxi
settings:map(>=1,<=10)                           # 1-10 entries
data:map<str(>=1,<=50),int(>=0)>(>=1)            # Constrained keys and values
```

**Binary:**
```maxi
file:bytes(>=1,mime:application/pdf)             # PDF file, non-empty
image:bytes(mime:[image/png,image/jpg])          # PNG or JPG
avatar:bytes@hex(<=1024)                         # Hex encoded, max 1KB
```

### Data Records

**Basic:**
```maxi
U(1|Julie|julie@maxi.org)
```

**Multi-line:**
```maxi
U(
  1|
  Julie Miller|
  julie@maxi.org
)
```

**Objects - Reference vs Inline:**
```maxi
# By reference (ID)
O(100|1|99.99)                              # user:U references User id=1

# Inline definition
O(101|(2|Matt|matt@maxi.org)|149.50)        # user:U defined inline

# Nested inline
O(102|(3|Anna|anna@maxi.org|(1|ACME))|199) # user:U inline, company:C inline
```

### String Escaping
```maxi
# Unquoted (simple strings)
U(1|Julie|NYC)

# Quoted (special characters or whitespace)
U(1|"First|Last"|"New York, NY")
U(2|"Line 1\nLine 2"|"Path: C:\\Users")
U(3|"She said \"Hello\""|email)
U(1|José|México)

# UTF-8
U(1|José|employee)
U(2|"北京"|China)
```

### Comments
```maxi
# Only in schema definitions (.mxs), NOT in data sections

# Define user type
U:User(id:int|name)

###
# This comment would be INVALID here (data section)
U(1|Julie)
```

### Common Patterns
```maxi
# Integer ID
id:int

# Custom identifier
sku(id)
order_id:int(id)

# Timestamps
createdAt:str@datetime
updatedAt:int@timestamp

# Email validation
email:str@email(!)

# Price
price:decimal(>=0,0:10.2)

# Status enum
status:enum[active,inactive,pending]

# Tags
tags:str(>=3,<=20)[](>=1,<=10)

# Metadata
metadata:map<str,str>

# Soft delete
deletedAt:str@datetime
```

---

## Appendix B: Error Codes

```
E001 - UnsupportedVersionError
E002 - DuplicateTypeError
E003 - UnknownTypeError
E004 - UnknownDirectiveError
E005 - InvalidSyntaxError
E006 - SchemaMismatchError
E007 - TypeMismatchError
E008 - ConstraintViolationError
E009 - UnresolvedReferenceError
E010 - CircularInheritanceError
E011 - MissingRequiredFieldError
E012 - InvalidConstraintValueError
E013 - UndefinedParentError
E014 - ConstraintSyntaxError
E015 - ArraySyntaxError
E016 - DuplicateIdentifierError
E017 - UnsupportedBinaryFormatError
E018 - InvalidDefaultValueError
E019 - StreamError
E020 - SchemaLoadError
```

---

## Appendix C: ABNF (Schema Grammar)

This appendix provides an **ABNF** (RFC 5234) grammar for the **schema portion** of MAXI (`.mxs` files and the schema section of `.maxi` files before `###`).  
It is intended to make parser implementations easier and more consistent.

**Scope (syntax only):**
- Directives (`@name:value`)
- Comments (`# ...`)
- Type definitions (alias, optional type name, optional inheritance, field list)
- Field type expressions (primitives, arrays, enums, maps, type references)
- Type annotations (e.g., `str@email`, `bytes@base64`)
- Constraint list syntax (tokens only; semantic validation is defined in the main spec)

> Note: This is a **syntax grammar**. Rules like "aliases must be unique", "constraints must match the base type", and inheritance resolution are **semantic** and must be enforced separately.

```abnf
; ============================================================================
; MAXI Schema ABNF (RFC 5234) - Syntax Grammar
; ============================================================================
; Scope: Schema section of .maxi files and .mxs schema files (not data)
; Note: Semantic validation (type compatibility, constraint validity) is 
;       specified in the normative text, not in this syntax grammar.
; ============================================================================

schema-file     = *schema-element

schema-element  = directive / type-def / comment / blank-line

; Schema/data separator (only appears in .maxi files, not .mxs)
schema-sep      = "###" line-end

; ============================================================================
; Directives
; ============================================================================

directive       = "@" directive-name ":" directive-value line-end

directive-name  = "version" / "mode" / "schema"

directive-value = 1*vchar-printable
                ; version: semver (e.g., "1.0.0")
                ; mode: "strict" / "lax"
                ; schema: file path or URL

; ============================================================================
; Comments and Whitespace
; ============================================================================

comment         = "#" *vchar-printable line-end

blank-line      = *WSP line-end

; ============================================================================
; Type Definitions
; ============================================================================
; Examples:
;   U:User(id:int|name(!)|email:str@email)
;   U(id|name|email)
;   U:User<Person,Timestamped>(role=admin)

type-def        = *WSP alias 
                  [ ":" type-name ] 
                  [ inheritance ] 
                  *WSP "(" *WSP field-list *WSP ")" *WSP 
                  line-end

inheritance     = "<" *WSP type-ref *( *WSP "," *WSP type-ref ) *WSP ">"

field-list      = [ field *( *WSP "|" *WSP field ) ]

field           = field-name  [ ":" type-expr [ type-annotation ] ]
                  [ constraints ] [ default-clause ]

; ============================================================================
; Type Expressions
; ============================================================================

type-expr       = primitive-type 
                / array-type 
                / enum-type 
                / map-type 
                / type-ref

primitive-type  = "int" / "decimal" / "str" / "bool" / "bytes"

array-type      = element-type "[]"

element-type    = primitive-type [ constraints ] 
                / type-ref [ constraints ]

enum-type       = "enum" [ "<" enum-base ">" ] "[" enum-values "]"

enum-base       = "str" / "int"

enum-values     = *WSP enum-value *( *WSP "," *WSP enum-value ) *WSP

enum-value      = identifier / integer / quoted-string

map-type        = "map" [ "<" map-params ">" ]

map-params      = map-value-type 
                / ( map-key-type "," map-value-type )

map-key-type    = primitive-type [ constraints ] 
                / type-ref

map-value-type  = primitive-type [ constraints ] 
                / type-ref

type-ref        = identifier

; ============================================================================
; Type Annotations
; ============================================================================
; Examples: @email, @url, @datetime, @base64, @hex, @timestamp

type-annotation = "@" annotation-name

annotation-name = "email" / "url" / "uuid" / "date" / "datetime" / "time"
                / "timestamp" / "base64" / "hex"

; ============================================================================
; Constraints
; ============================================================================
; Syntax: (constraint1,constraint2,...)
; Semantic validation depends on field type (specified in main spec)

constraints     = "(" *WSP constraint-expr 
                  *( *WSP "," *WSP constraint-expr ) *WSP ")"

constraint-expr = required-marker
                / comparison-constraint
                / pattern-constraint
                / mime-constraint
                / decimal-precision
                / identifier-marker
                / exact-length

; Individual constraint patterns

required-marker    = "!"

identifier-marker  = "id"

comparison-constraint = comparison-op number

comparison-op      = ">=" / ">" / "<=" / "<" / "="

number             = integer / decimal-number

integer            = [ "-" ] 1*DIGIT

decimal-number     = [ "-" ] 1*DIGIT "." 1*DIGIT

; Pattern constraint for strings
; Example: pattern:^[a-z0-9]+$
pattern-constraint = "pattern:" pattern-value

pattern-value      = 1*pattern-char

pattern-char       = %x21-2B / %x2D-7E  ; Printable except comma

; MIME constraint for bytes
; Examples: mime:image/png, mime:[image/png,image/jpg], mime:image/*
mime-constraint    = "mime:" mime-spec

mime-spec          = mime-single / mime-list

mime-single        = mime-type

mime-list          = "[" *WSP mime-type *( *WSP "," *WSP mime-type ) *WSP "]"

mime-type          = 1*mime-char [ "/" 1*mime-char ]

mime-char          = ALPHA / DIGIT / "-" / "+" / "." / "*"

; Decimal precision constraint
; Examples: 5.2, 0:10.2, .2:4, 1:999., 0:100.0:2
decimal-precision  = precision-only / scale-only / precision-and-scale

precision-only     = [ min-digits ":" ] max-digits "."

scale-only         = "." scale-spec

precision-and-scale = [ min-digits ":" ] max-digits "." scale-spec

min-digits         = 1*DIGIT

max-digits         = 1*DIGIT

scale-spec         = exact-scale / min-max-scale

exact-scale        = 1*DIGIT

min-max-scale      = 1*DIGIT ":" 1*DIGIT

exact-length       = "=" 1*DIGIT

; ============================================================================
; Default Values
; ============================================================================
; Examples: =guest, =0, ="default value", =true

default-clause  = "=" default-value

default-value   = unquoted-default / quoted-string / boolean-literal

unquoted-default = 1*unquoted-char

unquoted-char    = ALPHA / DIGIT / "-" / "_" / "."
                 ; Simple unquoted defaults (no spaces, no special chars)

boolean-literal  = "true" / "false" / "0" / "1"

; ============================================================================
; Identifiers and Names
; ============================================================================

alias           = identifier

type-name       = identifier

field-name      = identifier

identifier      = ident-start *ident-char

ident-start     = ALPHA / "_"

ident-char      = ALPHA / DIGIT / "_" / "-"

; ============================================================================
; Quoted Strings (for defaults and enum values)
; ============================================================================
; Supports escape sequences: \", \\, \n, \r, \t

quoted-string   = DQUOTE *string-content DQUOTE

string-content  = unescaped / escape-seq

unescaped       = %x20-21 / %x23-5B / %x5D-7E / %x80-10FFFF
                ; Any char except DQUOTE (") and backslash (\)

escape-seq      = "\" ( DQUOTE / "\" / "n" / "r" / "t" )

DQUOTE          = %x22

; ============================================================================
; Whitespace and Line Endings
; ============================================================================

line-end        = [ CR ] LF

vchar-printable = %x20-7E / %x80-10FFFF
                ; Printable characters including UTF-8

WSP             = SP / HTAB

SP              = %x20

HTAB            = %x09

CR              = %x0D

LF              = %x0A

ALPHA           = %x41-5A / %x61-7A  ; A-Z / a-z

DIGIT           = %x30-39            ; 0-9
```

---

## Appendix D: ABNF (Data Grammar)

This appendix provides an **ABNF** (RFC 5234) grammar for the **data section** of MAXI (the section after `###` in `.maxi` files, or standalone data-only files).
It complements Appendix C (Schema Grammar) and is intended to make parser implementations consistent.

**Scope (syntax only):**
- Data records (type alias + parenthesized field values)
- Field values (primitives, strings, null, arrays, maps, inline objects, references)
- Multi-line record formatting
- Quoted strings with escape sequences
- Nested structures (arrays of objects, maps with complex values)

**Note This is a syntax grammar:** Semantic rules like "field count must match schema", "reference must resolve to existing object", and type validation are enforced separately by the parser based on schema and mode.

```abnf
; ============================================================================
; MAXI Data ABNF (RFC 5234) - Syntax Grammar
; ============================================================================
; Scope: Data section of .maxi files (after ###) and data-only files
; Note: Semantic validation (schema matching, reference resolution) is
;       specified in the normative text, not in this syntax grammar.
; ============================================================================

data-section    = *data-element

data-element    = data-record / blank-line

; ============================================================================
; Data Records
; ============================================================================
; Examples:
;   U(1|Julie|julie@maxi.org)
;   O(100|1|99.99)
;   U(1|Julie|(123 Main St|NYC|10001))

data-record     = *WSP type-alias *WSP "(" *WSP field-values *WSP ")" *WSP line-end

type-alias      = identifier

field-values    = [ field-value *( *WSP "|" *WSP field-value ) ]

; ============================================================================
; Field Values
; ============================================================================
; A field value can be:
;   - Empty (null or default)
;   - Explicit null (~)
;   - Primitive value (string, number, boolean)
;   - Array
;   - Map
;   - Object reference (identifier/number)
;   - Inline object

field-value     = explicit-null
                / array-value
                / map-value
                / inline-object
                / primitive-value
                / empty-value

empty-value     = ""

explicit-null   = "~"

; ============================================================================
; Primitive Values
; ============================================================================

primitive-value = quoted-string
                / boolean-value
                / decimal-value
                / integer-value
                / unquoted-string

boolean-value   = "true" / "false" / "1" / "0"

integer-value   = [ "-" ] 1*DIGIT

decimal-value   = [ "-" ] 1*DIGIT "." 1*DIGIT

; Unquoted strings: any sequence of characters not containing special delimiters
; Leading/trailing whitespace is stripped
unquoted-string = 1*unquoted-char

unquoted-char   = %x21-27          ; ! " # $ % & '
                / %x2A-2B          ; * +
                / %x2D-3B          ; - . / 0-9 : ;
                / %x3D             ; =
                / %x3F-5A          ; ? @ A-Z
                / %x5E-7A          ; ^ _ ` a-z
                / %x80-10FFFF      ; UTF-8 extended
                ; Excludes: ( ) [ ] { } | , ~ < > SP HTAB

; ============================================================================
; Quoted Strings
; ============================================================================
; Used when values contain special characters, whitespace, or escape sequences
; Supports: \", \\, \n, \r, \t

quoted-string   = DQUOTE *string-content DQUOTE

string-content  = unescaped / escape-seq

unescaped       = %x20-21          ; SP and !
                / %x23-5B          ; # through [
                / %x5D-7E          ; ] through ~
                / %x80-10FFFF      ; UTF-8 extended
                ; Excludes: DQUOTE (") and backslash (\)

escape-seq      = "\" escape-char

escape-char     = DQUOTE           ; \"
                / "\"              ; \\
                / "n"              ; \n (newline)
                / "r"              ; \r (carriage return)
                / "t"              ; \t (tab)

DQUOTE          = %x22

; ============================================================================
; Arrays
; ============================================================================
; Examples:
;   [tag1,tag2,tag3]
;   [95,87,92]
;   []
;   [(1|Item1),(2|Item2)]

array-value     = "[" *WSP [ array-elements ] *WSP "]"

array-elements  = array-element *( *WSP "," *WSP array-element )

array-element   = explicit-null
                / array-value
                / map-value
                / inline-object
                / primitive-value

; ============================================================================
; Maps
; ============================================================================
; Examples:
;   {key1:value1,key2:value2}
;   {math:95,science:87}
;   {}
;   {"key:with:colon":"value,with,comma"}

map-value       = "{" *WSP [ map-entries ] *WSP "}"

map-entries     = map-entry *( *WSP "," *WSP map-entry )

map-entry       = map-key *WSP ":" *WSP map-entry-value

map-key         = quoted-string
                / integer-value
                / map-key-unquoted

map-key-unquoted = 1*map-key-char

map-key-char    = ALPHA / DIGIT / "_" / "-" / "."
                ; Simple unquoted keys (no colons, commas, or braces)

map-entry-value = explicit-null
                / array-value
                / map-value
                / inline-object
                / primitive-value

; ============================================================================
; Inline Objects
; ============================================================================
; Inline object definitions within data records
; Examples:
;   (2|Matt|matt@maxi.org)
;   (1|ACME Corp)
;   (456 Oak Ave|LA|90001)

inline-object   = "(" *WSP inline-fields *WSP ")"

inline-fields   = [ inline-field-value *( *WSP "|" *WSP inline-field-value ) ]

inline-field-value = explicit-null
                   / array-value
                   / map-value
                   / inline-object
                   / primitive-value
                   / empty-value

; ============================================================================
; Object References
; ============================================================================
; Object references are represented as primitive values (int or str)
; The parser determines if a value is a reference based on schema context
; Examples:
;   1         (integer reference to User id=1)
;   user_001  (string reference)

; Note: Object references are syntactically identical to primitive values.
; The distinction is semantic and determined by the field's type in the schema.

; ============================================================================
; Identifiers
; ============================================================================

identifier      = ident-start *ident-char

ident-start     = ALPHA / "_"

ident-char      = ALPHA / DIGIT / "_" / "-"

; ============================================================================
; Multi-line Records
; ============================================================================
; Records can span multiple lines within the parentheses
; Whitespace (including newlines) inside parentheses is normalized
;
; Example:
;   U(
;     1|
;     Julie Miller|
;     julie@maxi.org
;   )
;
; Note: The grammar above already handles this via *WSP which includes
; optional whitespace. Parsers should treat all whitespace between
; ( and ) as equivalent to field separators.

; ============================================================================
; Whitespace and Line Endings
; ============================================================================

blank-line      = *WSP line-end

line-end        = [ CR ] LF

WSP             = *( SP / HTAB / CR / LF )
                ; Within data records, all whitespace is normalized

SP              = %x20

HTAB            = %x09

CR              = %x0D

LF              = %x0A

ALPHA           = %x41-5A / %x61-7A  ; A-Z / a-z

DIGIT           = %x30-39            ; 0-9
```
