<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# Progmasoft C++ Style

Progmasoft C++ Style is a style guide for C++20. It says how code is named, laid out, structured and checked, and why,
so that a reader can move between files and projects without relearning the conventions each time.

It is written for anyone who wants a complete, opinionated rule set to adopt. Progmasoft's own C++ projects follow it,
among them the [Tala compiler](https://github.com/Progmasoft/tala) and [Catch3](https://github.com/Progmasoft/catch3).

## How to read it

Each rule is short and states what to do. Most come with the reason, because a rule whose reason is known can be
applied to a case it did not foresee. Examples show the preferred form first; a form marked **Avoid** shows what the
rule replaces.

The words follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) in spirit: **must** is a rule a review enforces,
**should** is the default that a good reason, written down next to the code, may override, and **may** is a choice
left to the author.

## Contents

| Chapter | Covers |
| --- | --- |
| [1. Principles](Guide/PRINCIPLES.md) | what the guide optimizes for, and how to decide a case it does not cover |
| [2. Formatting](Guide/FORMATTING.md) | the `.clang-format` profile, and what formatting leaves to the author |
| [3. Naming](Guide/NAMING.md) | types, functions, variables, constants, namespaces, files, macros |
| [4. Files and headers](Guide/FILES-AND-HEADERS.md) | extensions, `#pragma once`, includes, file size, the order inside a file |
| [5. Language](Guide/LANGUAGE.md) | C++20 features: `auto`, initialization, `const`, casts, integers, templates, lambdas |
| [6. Types and classes](Guide/TYPES-AND-CLASSES.md) | structs and classes, special members, inheritance, operators, enums |
| [7. Errors](Guide/ERRORS.md) | exception-free code, `Expected` and `Error`, invariants, fatal defects |
| [8. Ownership and memory](Guide/OWNERSHIP-AND-MEMORY.md) | RAII, smart pointers, views and their lifetimes, containers |
| [9. Comments and documentation](Guide/COMMENTS-AND-DOCUMENTATION.md) | Doxygen, what a comment is for, TODOs |
| [10. C interfaces](Guide/C-INTERFACES.md) | C11 headers, `extern "C"`, prefixes, ownership across the boundary |
| [11. Concurrency](Guide/CONCURRENCY.md) | threads, locks, atomics, thread-safety annotations |
| [12. Testing](Guide/TESTING.md) | unit tests, fuzzing, sanitizers, benchmarks |
| [13. Tooling](Guide/TOOLING.md) | compiler flags, the `.clang-tidy` profile, CI checks |

[Adopting the guide](ADOPTING.md) explains how a project points to this guide, which files it copies and how it
records a deviation. [Contributing](CONTRIBUTING.md) explains how to propose a change to a rule.

## The short version

- C++20, formatted by the [clang-format profile](Guide/FORMATTING.md#the-profile), checked by the
  [clang-tidy profile](Guide/TOOLING.md#132-the-clang-tidy-profile) with every warning an error.
- `PascalCase` for types, functions and namespaces; `camelCase` for variables and parameters; `name_` for private data
  members; `kPascalCase` for constants and enumerators; `UPPER_SNAKE_CASE` for macros.
- Four spaces, braces on their own line, 80 columns, the return type on its own line.
- `.hpp` and `.cpp`; `.h` only for a C interface. `#pragma once`. Standard and third-party headers in `<>`, the
  project's own in `""`.
- No `using namespace`. No exceptions in libraries and tools: recoverable failures return `Expected<T>` or `Error`.
- Ownership is visible in the type: values and `std::unique_ptr` own, references and views borrow.
- Every public declaration has a `///` comment, in US English.
- Tests sit next to the code they test; sanitizers and fuzzers run in CI. The guide prescribes no test, fuzzing or
  benchmark framework, and no error library: it prescribes how they are used.

## License

The text is licensed under [CC BY 4.0](LICENSE.md). The configuration files and code examples are dedicated to the
public domain under [CC0 1.0](LICENSES/CC0-1.0.md), so they can be copied into any project without credit.
