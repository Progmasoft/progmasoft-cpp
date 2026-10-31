<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 6. Types and classes

## 6.1 Struct or class

A **struct** is a bundle of values with no invariant: any combination of field values is valid. Its data members are
public, `camelCase`, and brace-initialized; it may have a defaulted comparison and small helpers that read its fields.

```cpp
/// One edge of a control-flow graph.
struct Edge final
{
    /// Block the edge leaves.
    BlockId from{};
    /// Block the edge enters.
    BlockId to{};

    [[nodiscard]] auto
    operator==(const Edge &) const -> bool = default;
};
```

A **class** keeps an invariant: some combinations of its state are invalid, and only its member functions may change it.
Its data members are `private` and end in `_`. A class has no `protected` data.

If a struct gains an invariant, it becomes a class. Do not put an invariant into a struct with a comment asking callers
to respect it.

## 6.2 `final` by default

Every class and struct is `final` unless it is designed as a base class. Designing for inheritance is a decision with
consequences (a virtual destructor, a stable protected interface, documentation of what an override may do), and
`final` makes the default decision visible.

## 6.3 Construction

- **A constructor establishes the invariant or does not exist.** When construction can fail, the constructor is private
  and a static `Create` returns `Expected<Type>`:

  ```cpp
  [[nodiscard]] static auto
  Create(std::size_t bitCount) -> llvm::Expected<BitSet>;
  ```

  A public constructor may still take values whose validity the caller guarantees by contract; it documents the
  precondition and asserts it.

- **`explicit`** on every single-argument constructor (see [Language](LANGUAGE.md#55-casts-and-conversions)).
- **Initialize members in the declaration** where a default exists, and in the constructor's initializer list otherwise,
  in declaration order. Do not assign in the constructor body what the initializer list can set.
- **No work that can fail, and no virtual calls, in a constructor or a destructor.**
- **No two-phase initialization** (`Init()` after construction): an object that exists is ready to use.

## 6.4 Special members: the rule of zero

Prefer members that manage themselves (values, `std::vector`, `std::unique_ptr`, an RAII handle), so that a class
declares none of the copy and move operations or the destructor, and the defaults are right.

When a class must manage a resource by hand, it declares all five: destructor, copy constructor, copy assignment, move
constructor and move assignment, each either implemented, `= default` or `= delete`. Move operations are `noexcept`.

```cpp
class FileHandle final
{
public:
    explicit FileHandle(int descriptor) noexcept;
    ~FileHandle();

    FileHandle(const FileHandle &) = delete;
    auto
    operator=(const FileHandle &) -> FileHandle & = delete;

    FileHandle(FileHandle &&other) noexcept;
    auto
    operator=(FileHandle &&other) noexcept -> FileHandle &;

private:
    int descriptor_{-1};
};
```

A type that must not be copied deletes the copy operations explicitly instead of inheriting from a "noncopyable" base.

## 6.5 Inheritance and polymorphism

- **Prefer composition, `std::variant` or a template over inheritance** for code reuse. Inherit to model "is a", for an
  interface that several unrelated implementations provide.
- **A polymorphic base** has a public virtual destructor, or a protected non-virtual one when objects are never deleted
  through the base. It has no data members unless every derived class needs them.
- **`override`** on every overriding function, and never `virtual` together with `override`. `final` on an override that
  must not be overridden further.
- **Public inheritance only.** Private inheritance is composition with a worse syntax.
- **No multiple inheritance of implementation.** A class may implement several pure interfaces.
- **Interfaces are small.** An interface with one implementation and no test double is not yet an interface.
- **No RTTI.** Dispatch with virtual functions, `std::visit`, or a kind tag and LLVM-style `isa`/`cast`/`dyn_cast`.

## 6.6 Operators

- Overload an operator only when it means what it means for built-in types: `+` adds, `==` compares for equality, `[]`
  indexes. A reader must never wonder what `a << b` does to `a`.
- **Default comparisons**: `operator==` and, when there is an order, `operator<=>`, both `= default` where memberwise
  comparison is right. [C++17 has neither](LANGUAGE.md#513-c17-projects).
- Binary operators that are symmetric are hidden friends or non-members, so both operands convert the same way.
- No overloads of `&&`, `||`, `,` or unary `&`.

## 6.7 Strong types

Wrap a value whose meaning a bare integer or string would lose:

```cpp
/// Identity of one block in a function; distinct from every other id kind.
struct BlockId final
{
    std::uint32_t value{};

    [[nodiscard]] auto
    operator<=>(const BlockId &) const = default;
};
```

A function that takes a `BlockId` and a `SymbolId` cannot be called with the two swapped, and the compiler, not a review,
catches it.

## 6.8 `std::optional`, `std::variant` and sentinels

- **`std::optional<T>`** for "a value or nothing", instead of a sentinel such as `-1`, `0` or an empty string.
- **`std::variant`** for "one of several kinds", visited with `std::visit` and an overload set, so a new alternative
  fails to compile where it is not handled.
- **A pointer that may be null** is fine for an optional reference to an existing object: `const Symbol *FindSymbol(...)`
  says "or nothing" just as well, without a copy.

## 6.9 Enumerations as flags

A set of flags is an `enum class` with the bit operators defined for it, or a `std::bitset` or a set type of the project.
Do not combine plain integer constants with `|`.
