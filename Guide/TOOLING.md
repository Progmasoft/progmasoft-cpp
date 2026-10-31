<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 13. Tooling

The guide relies on tools for everything that can be checked mechanically. A project that adopts it turns them all on,
with warnings as errors, in CI and in the local build alike.

## 13.1 Compiler

The warning set below is given for Clang, and for `clang-cl` on Windows; `clang-tidy` and the thread-safety analysis
need Clang in any case. A project that also builds with GCC or MSVC enables their equivalent warnings, with warnings
as errors, in a CI job of its own.

### Flags

```text
-std=c++20    # -std=c++17 in a C++17 project
-fno-exceptions
-Wall -Wextra -Wpedantic -Werror
-Wshadow
-Wformat=2
-Wcast-align -Wcast-qual
-Wundef
-Wnull-dereference
-Wimplicit-fallthrough
-Wnon-virtual-dtor -Woverloaded-virtual
-Wextra-semi
-Wmissing-field-initializers
-Wdouble-promotion
-Wimplicit-int-conversion -Wimplicit-float-conversion
-Wdeprecated
-Wunreachable-code-aggressive
-Wvla
-Wthread-safety
-Wcomma
```

With `clang-cl`, each flag is passed as `/clang:<flag>`, and the language standard as `/std:c++20` or `/std:c++17`.

These flags apply to first-party code only. Third-party code is compiled with its own options; its headers are included
as system headers (`-isystem`, or the build tool's equivalent), so their warnings do not fail the build.

### Optimized and debug builds

Debug builds keep assertions on and use `-O0` or `-Og`. Release builds use `-O2` or `-O3` with `NDEBUG`. CI builds both,
so that code that only compiles with assertions, or only without, is caught.

## 13.2 The clang-tidy profile

Copy this file to `.clang-tidy` at the root of the project and adjust the two path filters at the end. Every enabled
check is an error.

```yaml
---
Checks: >
  -*,
  clang-analyzer-*,
  bugprone-*,
  concurrency-*,
  performance-*,
  portability-*,
  modernize-loop-convert,
  modernize-make-unique,
  modernize-redundant-void-arg,
  modernize-type-traits,
  modernize-use-equals-delete,
  modernize-use-nullptr,
  modernize-use-override,
  modernize-use-using,
  misc-throw-by-value-catch-by-reference,
  readability-duplicate-include,
  readability-else-after-return,
  readability-implicit-bool-conversion,
  readability-inconsistent-declaration-parameter-name,
  readability-make-member-function-const,
  readability-misleading-indentation,
  readability-redundant-access-specifiers,
  readability-redundant-casting,
  readability-redundant-control-flow,
  readability-redundant-declaration,
  readability-redundant-function-ptr-dereference,
  readability-redundant-inline-specifier,
  readability-redundant-smartptr-get,
  readability-redundant-string-cstr,
  readability-redundant-string-init,
  readability-redundant-typename,
  readability-simplify-boolean-expr,
  readability-static-accessed-through-instance,
  readability-uniqueptr-delete-release,
  -bugprone-easily-swappable-parameters,
  -bugprone-exception-escape,
  -portability-avoid-pragma-once,
  -modernize-use-auto,
  -modernize-use-emplace,
  -modernize-use-trailing-return-type,
  -readability-braces-around-statements,
  -readability-function-size,
  -readability-magic-numbers,
  -readability-qualified-auto,
  -readability-redundant-member-init

WarningsAsErrors: '*'
HeaderFileExtensions:
  - h
  - hpp
ImplementationFileExtensions:
  - c
  - cpp
# First-party headers only; adjust to the project's source directories.
HeaderFilterRegex: '(^|.*/)(Source|Headers|Tests)/.*'
ExcludeHeaderFilterRegex: '(^|.*/)(build|third_party|external)/.*'
SystemHeaders: false
FormatStyle: file

CheckOptions:
  readability-implicit-bool-conversion.AllowPointerConditions: false
  readability-implicit-bool-conversion.AllowIntegerConditions: false
  modernize-use-using.IgnoreExternC: true
  modernize-use-using.IgnoreMacros: true
  modernize-loop-convert.MinConfidence: reasonable
...
```

### Why some checks are off

| Check | Reason |
| --- | --- |
| `bugprone-easily-swappable-parameters` | flags most functions with two parameters of one type; [strong types](TYPES-AND-CLASSES.md#67-strong-types) address the real risk |
| `bugprone-exception-escape` | meaningless when exceptions are disabled |
| `portability-avoid-pragma-once` | the guide uses `#pragma once` on purpose |
| `modernize-use-auto` | the guide writes types that carry information ([Language](LANGUAGE.md#52-auto)) |
| `modernize-use-emplace` | `push_back` of a constructed value is clearer and equally fast after a move |
| `modernize-use-trailing-return-type` | the guide uses trailing return types, but not for `void` functions, which the check would rewrite |
| `readability-braces-around-statements` | braces are required by the guide; the check's fix-its fight `clang-format` |
| `readability-function-size`, `readability-magic-numbers` | judged in review; the fixed thresholds produce more noise than findings |
| `readability-qualified-auto` | conflicts with writing `auto` for pointers returned by APIs whose type is obvious |
| `readability-redundant-member-init` | aggregate members keep an explicit `{}` so aggregate initialization that omits trailing fields stays clean under `-Wmissing-field-initializers` |

### Naming

Naming is checked in review, not by `readability-identifier-naming`. The check can enforce casing, but it cannot tell
a constant that deserves `kPascalCase` from an immutable local, and its exceptions for third-party callbacks grow into a
second configuration. A project may enable it once its code is consistent.

## 13.3 Other checks

| Tool | Checks |
| --- | --- |
| `clang-format --dry-run --Werror` | every file is formatted by the profile |
| Doxygen, warnings as errors | every public declaration is documented |
| A header self-containment build | every public header compiles on its own |
| The C interface test | each C header compiles as C11 and is called from C |
| Sanitizer and fuzzer jobs | see [Testing](TESTING.md) |
| The Clang Static Analyzer | the `clang-analyzer-*` checks over every translation unit |

## 13.4 Versions

Pin the exact versions of Clang, `clang-format` and `clang-tidy` that CI uses, and use the same versions locally. A
different version can format a file differently or report a check the CI version does not have. A version upgrade is
its own change: it updates the pin, reformats or fixes what the new version reports, and nothing else.
