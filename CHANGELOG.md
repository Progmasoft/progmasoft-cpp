<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# Changelog

A revision of Progmasoft C/C++ Styles is named by its date. A change that requires existing code to change says so.

## Unreleased

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
