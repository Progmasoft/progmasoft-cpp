<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 5. Language

The guide covers two language versions, and a project picks one:

- **[C++20](../Cpp20/LANGUAGE.md)** is the default, for applications and for libraries used inside one product.
- **[C++17](../Cpp17/LANGUAGE.md)** is for a library meant for other people's projects, because C++17 is still the
  widest common target and code written for it also works in every later standard.

A project builds with exactly its version and no compiler extensions. Its deviations do not change the version. This
chapter holds the rules of both versions; the page of each version adds what uses its own features.

## 5.1 Functions

- **Trailing return types for functions that return a value**: `auto Size() const -> std::size_t`. Return types then
  line up, and a long return type does not push the name to the right. A function that returns nothing is declared
  `void`, which reads better than `auto … -> void`.
- **`[[nodiscard]]` on every function whose result matters**, and always on one that returns `Expected`, `Error`, an
  owning pointer or a status. Ignoring such a result is then a compile error under `-Werror`.
- **Parameters** take an input by value when the function keeps a copy or the type is cheap to copy (scalars,
  `std::string_view`, `std::span`, small trivially copyable structs); by `const &` otherwise; by `&` only for an output
  or an in-out parameter, and prefer returning a value over an output parameter.
- **Default arguments** are allowed on non-virtual functions and constructors. A virtual function does not take them,
  because the default would depend on the static type of the call.
- **Overloads** do the same thing for different types. When two functions do different things, give them different
  names.
- **No function-style "flag" parameters** that select behavior: `Emit(module, true)` is unreadable at the call. Use an
  `enum class` or two functions.
- **`noexcept`** goes on move constructors, move assignment, `swap`, destructors (implicitly) and functions that by
  contract cannot fail. In exception-free code it documents that a function has no failure path; do not add it to
  everything.

## 5.2 `auto`

Use `auto` when the type is obvious from the right side, or long and uninformative:

```cpp
// The type is on the right.
auto parser = Parser::Create(source);
// The iterator type adds nothing.
auto iterator = blocks.find(id);
// A structured binding.
const auto &[name, value] = *entry;
```

Write the type when it carries information the reader needs, especially for numbers:

```cpp
// The width and the signedness matter.
std::uint32_t offset = header.Offset();
std::size_t count = Count(items);
```

Never `auto` for a variable that is meant to be a different type from the initializer, such as a proxy object
(`std::vector<bool>::reference`) or an expression template.

## 5.3 Initialization

- **Initialize every variable where it is declared.** An uninitialized variable is a bug waiting for a path that reads
  it. Declare a variable at the point where it gets its value, in the smallest scope.
- **Brace-initialize data members** with a default: `std::uint32_t count_{};`, `bool available{};`. Every object then
  starts in a defined state, and `-Wmissing-field-initializers` stays quiet for aggregate initialization that omits
  trailing fields.
- **Braces for initialization, parentheses for a constructor that braces would hide.** `std::vector<int> values(3, 0)`
  makes three zeros; `std::vector<int> values{3, 0}` makes the list `{3, 0}`. Use parentheses only where braces would
  select an `std::initializer_list` constructor you do not want.
- **Aggregates with many fields** are initialized so that each value names its field: designated initializers in
  [C++20](../Cpp20/LANGUAGE.md#initialization), assignments after default construction in
  [C++17](../Cpp17/LANGUAGE.md#the-c17-form-of-the-c20-rules).
- **No narrowing.** Brace initialization refuses it; with `=` and parentheses, write the conversion out.

## 5.4 `const` and `constexpr`

- **`const` by default.** A local that does not change is `const`; a member function that does not change the object is
  `const`; a pointer or reference parameter that is only read points to `const`.
- **`constexpr` for every value known at compile time**, and for functions that can be evaluated at compile time without
  contortion. C++20 adds [`consteval` and `constinit`](../Cpp20/LANGUAGE.md#compile-time-code).
- **`inline constexpr`** for a constant in a header. `constexpr` on a static data member already implies `inline`.
- Do not `const`-qualify a by-value parameter in a declaration; it means nothing to the caller. It may be `const` in the
  definition.
- **No `const_cast`** to write through a pointer that refers to a `const` object. Removing `const` is allowed only to call
  a C API that takes a non-`const` pointer it is documented not to modify, with a comment saying so.

## 5.5 Casts and conversions

- **No C-style casts and no functional casts** for anything but constructing a value of class type. Use
  `static_cast` for a value conversion, the version's way to reinterpret the bits of a trivially copyable value
  ([C++20](../Cpp20/LANGUAGE.md#casts), [C++17](../Cpp17/LANGUAGE.md#the-c17-form-of-the-c20-rules)), and
  `reinterpret_cast` only at a byte or ABI boundary, with a comment saying why the cast is valid.
- **No implicit conversion between integer widths or signedness.** `-Wimplicit-int-conversion` and the clang-tidy
  profile reject them; write the `static_cast` and make sure it cannot lose a value, or check that it cannot.
- **No implicit conversion to `bool`.** Write `pointer != nullptr`, `count != 0U`, `!text.empty()`.
- **`explicit`** on every constructor that can be called with one argument and on every conversion operator, unless the
  implicit conversion is the point of the type, as it is for a view.

## 5.6 Integers and arithmetic

- **Sizes and indices into memory are `std::size_t`.** Counts that come from or go to a file, a protocol or an ABI use
  the fixed-width types of `<cstdint>`, such as `std::uint32_t`. Do not use `long`, `unsigned long` or `short`, whose
  widths differ between platforms.
- **Unsigned for bits and sizes, signed for arithmetic that can go below zero.** Do not use unsigned to say "never
  negative" for a value that is computed by subtraction.
- **Integer literals carry their type**: `0U`, `1ULL << 40U`, `std::size_t{0}`.
- **Overflow of an untrusted value is checked**, before the operation, or with `__builtin_add_overflow` and its
  siblings, or with a checked-arithmetic helper. Signed overflow is undefined behavior; unsigned wrap-around is defined
  but almost never what a size computation means.
- **Floating point** compares with a tolerance that the code names, never with `==` on computed values.

## 5.7 Strings and text

- Text is UTF-8 in `std::string`. Take a read-only string parameter as `std::string_view` (or `llvm::StringRef` in code
  built on LLVM). Do not use `std::wstring`, `wchar_t`, or `std::u8string` in an interface.
- Convert to a platform encoding only at the platform boundary, such as a Windows wide-character API.
- Format with a formatting library ([`std::format` in C++20](../Cpp20/LANGUAGE.md#formatting-text), `fmt` in C++17) or
  `llvm::formatv`, or by streaming into a string builder; never with `sprintf` into a fixed buffer.
- A view parameter must not be stored. A member that keeps text owns a `std::string`.

## 5.8 Enumerations

Every enumeration is an `enum class`. Give it an explicit underlying type when its values are stored, sent or matched
against an ABI:

```cpp
enum class Opcode : std::uint8_t
{
    kLoad = 1,
    kStore = 2,
};
```

A `switch` over an enumeration lists every enumerator and has no `default`, so that `-Wswitch` reports a case that a new
enumerator needs. When a value may come from outside the program, convert it to the enumeration with a function that
validates it.

## 5.9 Templates

- **Constrain every template**: with a concept in [C++20](../Cpp20/LANGUAGE.md#templates-and-concepts), with a named
  trait in [C++17](../Cpp17/LANGUAGE.md#the-c17-form-of-the-c20-rules). The error then names the requirement.
- **Keep templates thin.** Move the part that does not depend on the template parameters into a non-template function,
  to keep compile times and code size down.
- **No template metaprogramming where `constexpr` code would do.** An `if constexpr` branch or a `constexpr` function is
  easier to read than a recursive specialization.

## 5.10 Lambdas

- **Capture explicitly.** `[&]` is allowed for a lambda that does not outlive the statement it is written in, such as a
  callback passed to an algorithm. A lambda that is stored, returned or run later names each capture, and captures by
  value whatever it must keep.
- Never capture `this` implicitly. Write `[this]` so the lifetime dependency is visible.
- A lambda longer than a few lines, or used twice, becomes a named function.

## 5.11 Control flow

- **Return early** for errors and edge cases; keep the main path unindented.
- **No `goto`.**
- **Range-based `for`** whenever the index is not needed. When it is, iterate with `std::size_t`.
- **`[[fallthrough]]`** marks every intended fall-through in a `switch`; `-Wimplicit-fallthrough` rejects the rest.
- **Algorithms** where they state the intent better than a loop; C++20 adds
  [ranges](../Cpp20/LANGUAGE.md#ranges-views-and-algorithms). A plain loop is better than a chain that needs a comment
  to explain.

## 5.12 Features not used

| Feature | Instead |
| --- | --- |
| RTTI: `dynamic_cast`, `typeid` | `std::variant` and `std::visit`, a kind enumeration, or LLVM-style `isa`/`cast`/`dyn_cast` |
| Exceptions | see [Errors](ERRORS.md) |
| `std::bind` | a lambda |
| Variable-length arrays | `std::array`, `std::vector`, or a small-vector type |
| Non-`const` global and static variables | state passed explicitly, owned by an object with a clear lifetime |
| `alloca`, `setjmp`/`longjmp` | ordinary allocation and returns |
| User-defined literals | named functions or constructors, except the standard ones such as `std::chrono` suffixes |

The features C++20 adds that the guide does not use are on the [C++20 page](../Cpp20/LANGUAGE.md#not-used).
