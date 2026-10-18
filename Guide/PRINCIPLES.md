<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 1. Principles

Every rule in this guide serves one of the principles below. When the guide says nothing about a case, decide by them,
in this order.

## 1.1 Correct before fast, and both before clever

Code that is wrong quickly is still wrong. Write the obvious correct version first, measure, and optimize where a
measurement says so. When an optimization makes code harder to read, the comment next to it says what it gains and how
that was measured.

A clever construct that saves three lines but costs every later reader a minute is a bad trade. Prefer the form that
a reader new to the file understands on the first pass.

## 1.2 The reader comes first

Code is read far more often than it is written. Names say what a thing is for; types say who owns what; a function does
one thing that its name states. A reviewer should be able to judge a change from the diff and the code around it,
without reconstructing intent from history.

## 1.3 Make invalid states unrepresentable

Prefer types that cannot hold a wrong value over checks that reject one. An `enum class` instead of an `int` code, a
strong identifier type instead of a bare `std::uint32_t`, `std::optional<T>` instead of a sentinel, a factory that
validates instead of a constructor that cannot. Each check that a type makes unnecessary is a check nobody can forget.

## 1.4 Failure is part of the interface

A function that can fail says so in its signature: it returns `Expected<T>` or `Error`, and the caller must look at the
result. A precondition that a caller can violate is either checked and reported, or documented and asserted. Nothing
fails silently, and nothing fails by surprise.

## 1.5 Determinism

The same input gives the same output on every machine, every run. Output that depends on hash order, pointer values,
thread timing, the locale or the time of day is a defect unless it is the point of the program. Sort before you print;
iterate in an order you chose.

## 1.6 Tools enforce what tools can

Formatting, naming of members, include order and a long list of bug patterns are checked by `clang-format`,
`clang-tidy` and the compiler, with warnings as errors. A review then spends its attention on what tools cannot judge:
design, names, ownership, and whether the change does what it says.

## 1.7 Consistency within a project beats consistency with the guide

A project that adopts this guide over existing code converges gradually. A file follows the guide where it is touched;
a mechanical sweep is its own change, never mixed into a behavior change. Where a project deliberately deviates, it
writes the deviation down once (see [Adopting the guide](../ADOPTING.md)), and then the project's rule wins in that
project.

## 1.8 Small, explicit dependencies

Every dependency is a cost: build time, security surface, and a second set of conventions. Prefer the standard library,
then one well-chosen library per problem. A build target names the libraries it uses directly instead of relying on
what another target happens to bring in.
