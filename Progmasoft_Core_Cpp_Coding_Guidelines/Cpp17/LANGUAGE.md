<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# C++17

C++17 is for a library meant for other people's projects, because C++17 is still the widest common target and code
written for it also works in every later standard. A C++20 library would leave those users out.

A C++17 project follows every chapter of [Common](../Common/PRINCIPLES.md) unchanged. It uses no C++20 feature, even
where a compiler accepts one; a feature of a later standard is used only through a library that provides it, such as
`llvm::Expected` in place of `std::expected`.

## Building

- The compiler flag is `-std=c++17`, and `/std:c++17` with `clang-cl`, next to the [flags](../Common/TOOLING.md#flags)
  every project uses.
- The [clang-format profile](../Common/FORMATTING.md#the-profile) says `Standard: c++17`.
- No compiler extensions: `-Wpedantic` is on.

## The C++17 form of the C++20 rules

Where the [C++20 page](../Cpp20/LANGUAGE.md) uses a C++20 feature, a C++17 project writes this instead:

| C++20 | C++17 |
| --- | --- |
| Concepts and `requires` | a named trait, `kIsInteger<Type>`, used by `std::enable_if_t` in a defaulted template parameter; `static_assert` when no overload depends on it |
| `operator==` and `operator<=>` `= default` | written out over `std::tie` of all fields, with a test that fails when a field is left out |
| Designated initializers | default construct, then assign the fields; a factory function when this repeats |
| `std::span` | `const std::vector<T> &`, or a view such as `llvm::ArrayRef` where the project has one |
| `std::ranges` | the iterator algorithms: `std::sort(items.begin(), items.end())` |
| `std::bit_cast` | `static_cast` between integers, `std::memcpy` into a value otherwise |
| `consteval`, `constinit` | `constexpr` |
| Template lambdas | a function template in a `Detail` namespace |
| `std::erase_if` | erase-remove: `items.erase(std::remove_if(...), items.end())` |
| `contains`, `starts_with`, `ends_with` | `find`, `compare` or `substr` |
| `std::remove_cvref_t`, `std::identity` | one small header of the project, with a comment naming what it stands for |
| `std::format` | `fmt`, or streaming |
| `std::jthread` | `std::thread`, joined explicitly |
| `using enum` | qualified enumerators |
| `[[likely]]`, `[[unlikely]]` | nothing |

## Checks for later standards

Code written for C++17 is promised to later standards too, so its checks have to reach them:

- **CI builds and tests in C++17 and in the newest standard the project supports.** A rule of C++20 can reject C++17 code,
  such as the rewritten comparison operators.
- **CI builds with each compiler the project supports, not only Clang.** Clang accepts designated initializers in C++17
  and warns; MSVC rejects them. Clang's `-Wc++20-extensions` shows most of these before CI does.
- **Headers compile under the default modes of the compilers.** MSVC is permissive in C++17 unless `/permissive-` is
  given, and a user of the library may not give it. A non-capturing lambda that reads a local `constexpr`, for instance,
  is an error there.
