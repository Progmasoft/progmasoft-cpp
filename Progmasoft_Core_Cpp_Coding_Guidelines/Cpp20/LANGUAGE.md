<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# C++20

C++20 is the default version: for applications and for libraries used inside one product. A C++20 project follows every
chapter of [Common](../Common/PRINCIPLES.md) and the rules below, which use what C++20 adds. The
[C++17 page](../Cpp17/LANGUAGE.md) gives the C++17 form of each of them.

## Building

- The compiler flag is `-std=c++20`, and `/std:c++20` with `clang-cl`, next to the [flags](../Common/TOOLING.md#flags)
  every project uses.
- The [clang-format profile](../Common/FORMATTING.md#the-profile) says `Standard: c++20`.
- No compiler extensions: `-Wpedantic` is on.

## Initialization

**Designated initializers** name the fields of an aggregate: `Edge{.from = a, .to = b}`. Use them whenever an aggregate
has more than two fields or two fields of the same type.

## Compile-time code

`consteval` when a function must only run at compile time, and `constinit` for a static variable that must be
initialized before the program starts. Everything else known at compile time is `constexpr`, as
[Language](../Common/LANGUAGE.md#54-const-and-constexpr) says.

## Casts

`std::bit_cast` reinterprets the bits of a trivially copyable value. `reinterpret_cast` stays for byte and ABI
boundaries.

## Templates and concepts

- **Constrain every template** with a concept or a `requires` clause instead of SFINAE: `template<std::integral Id>`.
  The error then names the requirement, not a substitution failure deep inside.
- **Name a concept for a requirement used more than once.** Concepts are `PascalCase` nouns or adjectives:
  `Hashable`, `ByteRange`.
- **Template lambdas** where a generic lambda needs to name its parameter type.

## Comparisons

`operator==` and, when there is an order, `operator<=>`, both `= default` where memberwise comparison is right. The
other rules of [Operators](../Common/TYPES-AND-CLASSES.md#66-operators) hold as they are.

## Ranges, views and algorithms

- **Ranges and algorithms** where they state the intent better than a loop: `std::ranges::sort(items)`,
  `std::ranges::any_of(...)`. A plain loop is better than a pipeline of five views that needs a comment to explain.
- **`std::span`** for a parameter that reads a contiguous sequence it does not own.
- `std::erase_if`, and `contains`, `starts_with` and `ends_with` of the standard containers and strings.

## Formatting text

`std::format` and its relatives, never `sprintf` into a fixed buffer.

## Smaller features

- `std::jthread` for a thread that stops and joins when it goes out of scope.
- `using enum` inside a function or a `switch` that names many enumerators of one enumeration.
- `std::remove_cvref_t` and `std::identity` from the standard library.
- **`[[likely]]` and `[[unlikely]]`** only where a measurement showed a difference.

## Not used

| Feature | Instead |
| --- | --- |
| C++20 modules | headers; build-tool support is not yet uniform across the supported compilers |
| Coroutines | explicit state machines or callbacks, until a project adopts a coroutine library deliberately |
