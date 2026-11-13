<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# Progmasoft C/C++ Styles

Progmasoft C/C++ Styles holds the coding guidelines of Progmasoft for C and C++. They say how code is named, laid out,
structured and checked, and why, so that a reader can move between files and projects without relearning the
conventions each time.

| Guidelines | Languages | State |
| --- | --- | --- |
| [Progmasoft Core C++ Coding Guidelines](#progmasoft-core-c-coding-guidelines) | C++17, C++20 | written |
| `Progmasoft_Core_C_Coding_Guidelines/` | C11 | not written yet |
| `Progmasoft_TalaComponents_Cpp_Coding_Guidelines/` | C++17 | not written yet |

They are written for anyone who wants a complete, opinionated rule set to adopt. Progmasoft's own C++ projects follow
them, among them [Catch3](https://github.com/Progmasoft/catch3) and [Tazelish](https://github.com/Progmasoft/tazelish).

## How to read them

Each rule is short and states what to do. Most come with the reason, because a rule whose reason is known can be
applied to a case it did not foresee. Examples show the preferred form first; a form marked **Avoid** shows what the
rule replaces.

The words follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) in spirit: **must** is a rule a review enforces,
**should** is the default that a good reason, written down next to the code, may override, and **may** is a choice
left to the author.

## Progmasoft Core C++ Coding Guidelines

The guidelines for C++17 and C++20, in `Progmasoft_Core_Cpp_Coding_Guidelines/`.

### Contents

| Chapter | Covers |
| --- | --- |
| [1. Principles](Progmasoft_Core_Cpp_Coding_Guidelines/Common/PRINCIPLES.md) | what the guidelines optimize for, and how to decide a case they do not cover |
| [2. Formatting](Progmasoft_Core_Cpp_Coding_Guidelines/Common/FORMATTING.md) | the `.clang-format` profile, and what formatting leaves to the author |
| [3. Naming](Progmasoft_Core_Cpp_Coding_Guidelines/Common/NAMING.md) | types, functions, variables, constants, namespaces, files, macros |
| [4. Files and headers](Progmasoft_Core_Cpp_Coding_Guidelines/Common/FILES-AND-HEADERS.md) | extensions, `#pragma once`, includes, file size, the order inside a file |
| [5. Language](Progmasoft_Core_Cpp_Coding_Guidelines/Common/LANGUAGE.md) | Language versions and features: `auto`, initialization, `const`, casts, integers, templates, lambdas |
| [6. Types and classes](Progmasoft_Core_Cpp_Coding_Guidelines/Common/TYPES-AND-CLASSES.md) | structs and classes, special members, inheritance, operators, enums |
| [7. Errors](Progmasoft_Core_Cpp_Coding_Guidelines/Common/ERRORS.md) | exception-free code, `Expected` and `Error`, invariants, fatal defects |
| [8. Ownership and memory](Progmasoft_Core_Cpp_Coding_Guidelines/Common/OWNERSHIP-AND-MEMORY.md) | RAII, smart pointers, views and their lifetimes, containers |
| [9. Comments and documentation](Progmasoft_Core_Cpp_Coding_Guidelines/Common/COMMENTS-AND-DOCUMENTATION.md) | Doxygen, what a comment is for, TODOs |
| [10. C interfaces](Progmasoft_Core_Cpp_Coding_Guidelines/Common/C-INTERFACES.md) | C11 headers, `extern "C"`, prefixes, ownership across the boundary |
| [11. Concurrency](Progmasoft_Core_Cpp_Coding_Guidelines/Common/CONCURRENCY.md) | threads, locks, atomics, thread-safety annotations |
| [12. Testing](Progmasoft_Core_Cpp_Coding_Guidelines/Common/TESTING.md) | unit tests, fuzzing, sanitizers, benchmarks |
| [13. Tooling](Progmasoft_Core_Cpp_Coding_Guidelines/Common/TOOLING.md) | compiler flags, the `.clang-tidy` profile, CI checks |

Each version has a page of its own with the rules that use its features:

| Version | Covers |
| --- | --- |
| [C++20](Progmasoft_Core_Cpp_Coding_Guidelines/Cpp20/LANGUAGE.md) | the default: concepts, designated initializers, defaulted comparisons, ranges, `std::span`, `std::format` |
| [C++17](Progmasoft_Core_Cpp_Coding_Guidelines/Cpp17/LANGUAGE.md) | for libraries used by other projects: the C++17 form of each C++20 rule, and the checks for later standards |

The guidelines are in `Progmasoft_Core_Cpp_Coding_Guidelines/`: `Common/` holds what applies to both versions, `Cpp20/`
and `Cpp17/` what applies to one. [`index.json`](Progmasoft_Core_Cpp_Coding_Guidelines/index.json) lists every document
with its chapter and the versions it applies to, for tools that show or check the guidelines.

[Adopting the guidelines](ADOPTING.md) explains how a project points to them, which files it copies and how it
records a deviation. [Contributing](CONTRIBUTING.md) explains how to propose a change to a rule.

### The short version

- C++17 or C++20, formatted by the [clang-format
  profile](Progmasoft_Core_Cpp_Coding_Guidelines/Common/FORMATTING.md#the-profile), checked by the [clang-tidy
  profile](Progmasoft_Core_Cpp_Coding_Guidelines/Common/TOOLING.md#132-the-clang-tidy-profile) with every warning an
  error.
- `PascalCase` for types, functions and namespaces; `camelCase` for variables and parameters; `name_` for private data
  members; `kPascalCase` for constants and enumerators; `UPPER_SNAKE_CASE` for macros.
- Four spaces, braces on their own line, 80 columns, the return type on its own line.
- `.hpp` and `.cpp`; `.h` only for a C interface. `#pragma once`. Standard and third-party headers in `<>`, the
  project's own in `""`.
- No `using namespace`. No exceptions in libraries and tools: recoverable failures return `Expected<T>` or `Error`.
- Ownership is visible in the type: values and `std::unique_ptr` own, references and views borrow.
- Every public declaration has a `///` comment, in US English.
- Tests sit next to the code they test; sanitizers and fuzzers run in CI. The guidelines prescribe no test, fuzzing or
  benchmark framework, and no error library: it prescribes how they are used.

## License

The text is licensed under [CC BY 4.0](LICENSE.md). The configuration files and code examples are dedicated to the
public domain under [CC0 1.0](LICENSES/CC0-1.0.md), so they can be copied into any project without credit.
