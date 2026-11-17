<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 1. Principles

Progmasoft Core C Coding Guidelines are for projects written in C. They are a style of their own, not the C subset of
[Progmasoft Core C++ Coding Guidelines](../../Progmasoft_Core_Cpp_Coding_Guidelines/Common/PRINCIPLES.md): names,
layout and comments differ on purpose. A C interface that a C++ library exposes follows the C++ guidelines
([C interfaces](../../Progmasoft_Core_Cpp_Coding_Guidelines/Common/C-INTERFACES.md)).

## 1.1 The language and the tools

- **The language is ISO C11**, or GNU99 for code that joins a codebase built as GNU99 or uses GNU extensions.
  [Language](LANGUAGE.md) says how to choose, and the [C11](../C11/LANGUAGE.md) and [GNU99](../Gnu99/LANGUAGE.md)
  pages what each version uses.
- **The toolchain is LLVM 23**: Clang compiles, and `clang-format` and `clang-tidy` 23 run the profiles in
  [Formatting](FORMATTING.md#the-profile) and [Tooling](TOOLING.md#111-the-clang-tidy-profile).
- **C11 uses no language extensions**, with two exceptions that every header uses: `#pragma once`, and the
  conditional `extern "C"` block of a public header ([Files and headers](FILES-AND-HEADERS.md)). GNU99 may use the GNU
  extensions ([GNU99](../Gnu99/LANGUAGE.md#extensions)).

## 1.2 How the rules are written

Each rule says what to do. **Must** is a rule a review enforces, and **must not** forbids; an exception exists only
where the rule names one. **Should** is the default, which a technical reason, written next to the code, may override.
**May** leaves the choice to the author.

## 1.3 Names carry the kind of a thing

A C program has one namespace and no types that a tool shows on hover in every reader's editor. So a name says what it
is: its prefixes give the scope, the storage and the pointer depth (`gspUser`), the direction of a parameter
(`aOut_pResult`) and the module it belongs to (`Example_List_Create`). A reader knows from the name alone whether a
line changes a global, writes through an out parameter or calls into another module.

## 1.4 Ownership is a contract

C has no destructors. Every function that allocates, frees, borrows or hands over memory says so, in its name
([Naming](NAMING.md#36-what-function-names-mean)) and in its documentation, and leaves a defined state behind when it
fails ([Memory and ownership](MEMORY-AND-OWNERSHIP.md)).

## 1.5 Tools do what they can; review does the rest

`clang-format` lays out the code, `clang-tidy` and the compiler catch a set of bugs. Neither checks the naming scheme,
ownership, Doxygen content or every initializer rule. [Tooling](TOOLING.md#113-what-review-checks) lists what review
covers, and where the formatter and the guidelines disagree, the guidelines are the rule.
