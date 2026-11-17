<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 11. Tooling

The toolchain is LLVM 23. The names of the binaries differ between distributions, `clang-format-23` or
`clang-format`, so check the version before relying on a result.

## 11.1 The clang-tidy profile

Copy this file to `.clang-tidy` at the root of the project. It is deliberately narrow: checks that apply to C and find
real bugs. It does not check the naming scheme, ownership or Doxygen content, and it does not replace the compiler or
review.

```yaml
---
Checks: >-
  -*,
  clang-analyzer-core.*,
  clang-analyzer-deadcode.*,
  clang-analyzer-unix.Malloc,
  bugprone-macro-repeated-side-effects,
  bugprone-sizeof-expression,
  bugprone-suspicious-memset-usage
WarningsAsErrors: ''
HeaderFilterRegex: '.*'
FormatStyle: file
SystemHeaders: false
...
```

`bugprone-macro-repeated-side-effects` catches a macro that evaluates a parameter twice
([Preprocessor](PREPROCESSOR.md#72-macros)), `bugprone-sizeof-expression` a `sizeof` of the wrong thing in an
allocation. A project that adds checks chooses them for C: groups such as `modernize-*` are written for C++ and report
false positives on C code. Run `clang-tidy` with the real compiler arguments, preferably from `compile_commands.json`.

## 11.2 Commands

```sh
clang-format --version
clang-format --style=file --dry-run --Werror Example.List.c Example.List.h
clang-tidy --version
clang-tidy Example.List.c -- -std=c11 -I.
clang -std=c11 -pedantic-errors -Wall -Wextra -I. -fsyntax-only Example.List.c
```

The commands are those of a C11 project. A GNU99 project writes `-std=gnu99` in place of `-std=c11 -pedantic-errors`
([GNU99](../Gnu99/LANGUAGE.md#building)).

`clang-format --dry-run --Werror` reports the places where the formatter's output differs from the file, including the
few where the guidelines are right and the formatter is not ([Formatting](FORMATTING.md#what-the-tool-leaves-to-you)).

## 11.3 What review checks

- A comma after the last item of an initializer over several lines, and none in a list on one line.
- Doxygen: `@brief`, the direction of each parameter, documentation at both the declaration and the definition, the
  order of the parts.
- A valid value before the first read; each macro parameter evaluated once.
- `realloc` through a temporary pointer, no allocation of zero bytes, overflow checks on sizes, ownership.
- Module prefixes of public names, include paths, and file and directory names that match the module.
- In GNU99, only [the listed extensions](../Gnu99/LANGUAGE.md#the-extensions-it-allows), and none in a public header but
  the listed attributes.
- The line breaks the formatter cannot make, the blank lines around `goto` labels, and the forms in
  [What the tool leaves to you](FORMATTING.md#what-the-tool-leaves-to-you).

## 11.4 References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174): the
  meaning of must, should and may.
- [Clang-Format Style Options](https://releases.llvm.org/23.1.0/tools/clang/docs/ClangFormatStyleOptions.html), 23.1.0.
- [Clang-Tidy](https://releases.llvm.org/23.1.0/tools/clang/tools/extra/docs/clang-tidy/index.html) and [its
  checks](https://releases.llvm.org/23.1.0/tools/clang/tools/extra/docs/clang-tidy/checks/list.html), 23.1.0.
