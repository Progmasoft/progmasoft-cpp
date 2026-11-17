<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# Changelog

A revision of Progmasoft C/C++ Styles is named by its date. A change that requires existing code to change says so.

## Unreleased

Progmasoft Core C Coding Guidelines cover GNU99 next to C11. The chapters that apply to both moved to
`Progmasoft_Core_C_Coding_Guidelines/Common/`; `C11/LANGUAGE.md` and `Gnu99/LANGUAGE.md` hold what applies to one
version, and Language (chapter 9) says how a project chooses. GNU99 is `-std=gnu99` with a closed list of GNU
extensions (attributes, `__typeof__` and statement expressions inside macros, overflow built-ins, `__builtin_expect`,
inline assembly in `static inline` functions), no variable-length arrays, and a GNU99 form of each C11 feature. No C11
rule changed; links to `Progmasoft_Core_C_Coding_Guidelines/C11/` move to `Common/`, except `C11/LANGUAGE.md`.

Progmasoft Core C Coding Guidelines are written, for C11 with LLVM 23, in
`Progmasoft_Core_C_Coding_Guidelines/C11/`: eleven chapters, a `.clang-format` and a `.clang-tidy` profile, and an
`index.json`. They are a style of their own; the C interfaces of C++ libraries keep following the C++ guidelines.

`index.json` names its JSON Schema,
[schemastore.progmasoft.com/json/progmasoft-cpp-index.json](https://schemastore.progmasoft.com/json/progmasoft-cpp-index.json),
in `$schema`; editors check the file against it.

The former C++17 component guidelines folder is removed.

Naming 3.4 no longer allows a short lowercase namespace for a heavily used data model: every namespace is
`PascalCase` under the project's root. A project that keeps such a namespace records it as a deviation (Adopting, 3).
The examples name Catch3's namespaces.

The repository is named Progmasoft C/C++ Styles, and its C++ guide Progmasoft Core C++ Coding Guidelines; both
were Progmasoft C++ Style.

The guidelines moved to `Progmasoft_Core_Cpp_Coding_Guidelines/`. `Common/` holds the thirteen chapters, which apply to
both versions; `Cpp20/` and `Cpp17/` hold what applies to one version: the rules that use C++20 features, and their
C++17 form with the checks a C++17 library needs. `index.json` lists every document with its chapter and versions. No
rule changed; links to `Guide/` move to `Progmasoft_Core_Cpp_Coding_Guidelines/Common/`.

The guidelines cover C++17 as well as C++20. A project picks one; C++20 stays the default, and C++17 is for libraries
meant for other people's projects. [The C++17 page](Progmasoft_Core_Cpp_Coding_Guidelines/Cpp17/LANGUAGE.md) lists the
C++17 form of each C++20 feature the guidelines use. Existing C++20 projects need no change.

## 2026-10-18

The first revision of Progmasoft C++ Style. It writes down the conventions of the Progmasoft C++ projects and decides
what they left open:

- thirteen chapters, from principles to tooling, with the `.clang-format` and `.clang-tidy` profiles and the compiler
  flags in full;
- guidance for adopting the guide and recording a project's deviations;
- the text under CC BY 4.0, the configuration and code examples under CC0 1.0.
