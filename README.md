# Semantic Config IDE

**Semantic Config IDE** is a DSL-powered browser IDE for designing, validating, and maintaining complex configuration systems.

It turns JSONC configuration files into a structured, semantic workspace with templates, references, validation, code generation, cross-file navigation, visual project trees, and advanced editor tooling.

Live demo: [faktordev.com/ide](https://faktordev.com/ide/)

---

## Why this project exists

Large configuration systems often start as simple JSON files and slowly become hard to maintain:

* duplicated structures;
* broken references;
* inconsistent enum values;
* missing required fields;
* unclear templates;
* no safe refactoring;
* weak validation;
* no connection between config data and generated code.

Semantic Config IDE solves this by adding a lightweight directive-based DSL on top of JSONC.
The goal is to keep configs readable for humans, while giving them enough semantic meaning for tooling, validation, navigation, and generation.

---

## Core idea

Instead of using plain JSON only, configs can contain semantic directives:

```jsonc
{
  //@config Items
  "Items": [
    {
      //@template
      //@primary_key
      "Id": "",
      "Name": "",

      //@enum[Common,Rare,Epic,Legendary]
      "Rarity": "Common",

      //@defaultValue 1
      "Weight": 1
    }
  ]
}
```

The IDE understands these directives and uses them to provide:

* schema generation;
* semantic validation;
* code generation;
* quick fixes;
* cross-file references;
* config graph analysis;
* editor hints and navigation.

---

## Features

### Semantic JSONC configs

Use JSONC files with comments and custom directives to describe structure, validation rules, relationships, templates, enums, defaults, and references.

Supported concepts include:

* `//@config`
* `//@template`
* `//@primary_key`
* `//@foreign_key`
* `//@enum[...]`
* `//@defaultValue`
* `//@optional`
* `//@nullable`
* `//@type`
* `//@clamp`
* `//@regex`
* `//@global_type`
* `//@local_type`
* `//@ref_type`
* `//@global_enum`
* `//@ref_enum`
* `//@union[...]`

---

### Template-driven configuration

Templates define the expected structure of runtime config entities.

The template is treated as the source of truth for:

* validation;
* generated schema;
* generated code;
* required fields;
* enum values;
* default values;
* reference rules.

---

### Semantic validation

The IDE validates configs beyond basic JSON syntax.

It can detect:

* invalid JSONC;
* missing required properties;
* invalid enum values;
* invalid types;
* broken foreign keys;
* duplicate primary keys;
* missing primary keys;
* constraint violations;
* directive errors;
* schema mismatches.

---

### Cross-file references

Configs can reference entities from other configs and files.

Example:

```jsonc
{
  //@config Recipes
  "Recipes": [
    {
      //@template
      //@primary_key
      "Id": "",

      //@foreign_key Items
      "ResultItemId": ""
    }
  ]
}
```

This allows the IDE to validate references and help maintain larger multi-file config systems safely.

---

### Monaco-powered editor tooling

The editor is built around Monaco and provides IDE-like language features for config files:

* diagnostics;
* hover information;
* completions;
* quick fixes;
* semantic highlighting;
* inlay hints;
* document symbols;
* go-to-definition;
* references;
* rename/refactor support.

---

### Quick fixes

The IDE can suggest and apply safe edits, such as:

* add missing required property;
* remove unsupported property;
* convert value type;
* add value to enum;
* apply directive transforms;
* fix schema-related issues.

---

### Code generation

Semantic Config IDE can generate typed code from configuration templates.

Current generation targets include:

* C# models;
* TypeScript models;
* combined export modes for configs, templates, schemas, and code.

This is useful for game development and data-driven applications where config structure must stay synchronized with application code.

---

### Visual project tree

The project tree is designed for working with many config files and folders.

It supports:

* file and folder creation;
* safe deletion dialogs;
* rename;
* drag-and-drop;
* custom colors;
* folder color scopes;
* semantic badges for entities, templates, warnings, and errors.

---

### Import / export workflow

Projects can be exported and imported as structured bundles.

Export modes can include:

* raw config files;
* templates;
* generated schemas;
* generated code;
* combined artifacts.

---

## Example use cases

Semantic Config IDE is especially useful for:

* game configuration systems;
* RPG items, units, skills, recipes, quests, environments;
* live-ops configuration;
* large data-driven applications;
* internal tools;
* configuration-heavy products;
* systems where JSON files need validation, references, and generated types.

---

## Tech stack

The project is built with:

* Next.js;
* React;
* TypeScript;
* Redux Toolkit;
* Monaco Editor;
* JSONC parser;
* AJV validation;
* IndexedDB / Dexie;
* Docker-based deployment;
* GitHub Actions CI/CD.

---

## Project status

The project is under active development.

Current focus areas:

* improving semantic validation;
* stabilizing quick fixes;
* improving SourceTree UX;
* polishing generated code output;
* expanding reusable global/local types;
* improving tests and CI/CD.

---

## Roadmap

Planned and ongoing improvements:

* better visual config graph;
* more advanced global type support;
* stronger rename/refactor tools;
* richer schema/code export modes;
* improved test coverage;
* better UI polish for large projects;
* local agent / external automation integration;
* support for more config formats in the future.

---

## Philosophy

Semantic Config IDE is built around one idea:

> Configuration files should remain simple text files, but the editor should understand their meaning.

The project keeps JSONC readable and lightweight, while adding enough semantic structure to make large configuration systems maintainable.

---

## License

License information will be added later.
