<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 3. Naming

A name is the shortest documentation a thing gets, and the one a reader sees most. The casing tells the reader what kind
of thing a name is before they look it up; the words tell them what it is for.

## 3.1 Casing

| Kind | Style | Example |
| --- | --- | --- |
| Namespace | `PascalCase` | `Progmasoft::Tala::Analysis` |
| Class, struct, union, enum, concept, type alias | `PascalCase` | `ControlFlowGraph`, `using BlockId = std::uint32_t;` |
| Template type parameter | `PascalCase` | `template<typename Value>` |
| Function, member function, static member function | `PascalCase` | `AnalyzeLiveness`, `graph.BlockCount()`, `Rectangle::Create` |
| Local variable, parameter | `camelCase` | `blockCount`, `const Node &node` |
| Public data member of a struct | `camelCase` | `point.x`, `edge.successor` |
| Private and protected data member | `camelCase_` | `bits_`, `blocks_` |
| Constant: `constexpr`, `const` at namespace scope, static constant member | `kPascalCase` | `kMaximumDepth` |
| Enumerator | `kPascalCase` | `Color::kRed` |
| Template value parameter | `kPascalCase` | `template<std::size_t kSize>` |
| Macro | `UPPER_SNAKE_CASE` with a project prefix | `EXAMPLE_ASSERT` |
| C function and type in a C interface | `snake_case` with a project prefix | `example_parser_create` |

The trailing underscore marks state that an object keeps for itself. A reader who sees `blocks_` inside a member
function knows it is the object's own field and not a local or a parameter, without any other prefix.

Public data members of a struct have no underscore: a struct is a plain bundle of values that callers read and write
directly (see [Types and classes](TYPES-AND-CLASSES.md#61-struct-or-class)).

## 3.2 Acronyms and abbreviations

An acronym is one word and is cased as one: `HttpClient`, `ParseUtf8`, `xmlReader`, `kMaxUrlLength`. Two capital
letters in a row are allowed only for a two-letter acronym that would be unreadable otherwise: `IOError` is wrong,
`IoError` is right, but `LLVM` stays `LLVM` in a namespace that names the library (`Backend::LLVM`), because that is
the product's own spelling.

Spell words out. `count`, `index`, `message`, `position` rather than `cnt`, `idx`, `msg`, `pos`. Abbreviations that are
more common than the word itself are fine: `id`, `url`, `utf8`, `min`, `max`, `ptr` only when it names a pointer whose
pointee is the subject.

## 3.3 Choosing the words

- **Types are nouns**: `Lexer`, `TokenStream`, `DiagnosticSink`.
- **Functions are verbs, or questions**: `Parse`, `EmitObject`, `IsEmpty`, `HasErrors`, `CanInline`.
- **A function that returns a property without work is named for the property**: `Size()`, `Name()`, `Entry()`. No
  `Get` prefix; reserve `Get` for a lookup that can fail, such as `GetSymbol(id) -> Expected<Symbol>`.
- **A setter says what it changes**: `SetName`, `Reset`, `Assign`.
- **Booleans read as statements**: `isVisible`, `hasBody`, `shouldRetry`, `available`. Avoid negatives that cause
  double negation: `isEnabled`, not `isNotDisabled`.
- **A factory that validates is `Create`**; one that reads from a source is `From…` or `Parse…`; one that converts is
  `To…`.
- **Units go in the name when the type does not carry them**: `timeoutMilliseconds`, `sizeBytes`, `widthPixels`.
  Prefer a type that carries the unit, such as `std::chrono::milliseconds`.
- **The scope decides the length.** A loop index over three lines may be `index`; a member that lives for the whole
  program needs a name that explains itself out of context.
- **Do not repeat the context.** Inside `class Parser`, the member is `Parse`, not `ParserParse`; in namespace
  `Diagnostics`, the type is `Sink`, not `DiagnosticsSink`, unless it is also used outside that namespace without a
  qualifier.

## 3.4 Namespaces

A project's code lives under one root namespace named for the organization or the project, then one level per product
and component: `Progmasoft::Tala::Backend::LLVM`. Use C++17 nested namespace definitions:
`namespace Progmasoft::Tala::Analysis`.

Each component owns one namespace and every name in it. Two components do not add declarations to the same namespace.

An implementation detail that must be visible in a header, such as a helper of a template, lives in a nested `Detail`
namespace, which callers must not use.

A project may keep a short lowercase namespace for a stable data model that is used heavily and qualified often, such
as an intermediate representation (`tala::core`, `tala::xpp`). This is a deliberate, documented exception, not a second
convention.

## 3.5 Files

A file is named after the main thing it declares, in the casing of that thing: `ControlFlowGraph.hpp` and
`ControlFlowGraph.cpp`. A file with several small related declarations is named after their common subject:
`Diagnostics.hpp`. A directory names a component: `Analysis/`, `Backend/`.

Test files add `Tests`: `ControlFlowGraphTests.cpp`. A benchmark adds `Benchmark`, a fuzz target `Fuzz`.

## 3.6 What not to name things

- Names that differ only in case: `Value` and `value` in the same scope.
- Names starting with an underscore followed by a capital letter, or containing a double underscore anywhere: the
  standard reserves them for the implementation.
- Hungarian notation and type prefixes: `pNode`, `strName`, `iCount`, `m_size`. The type is in the declaration, and
  `clang-tidy` and the editor show it.
- `Manager`, `Helper`, `Util`, `Data`, `Info` and `Handler` as the whole meaning of a name. Say what the thing manages
  or helps with.
- A name that lies. When the behavior of a function changes, its name changes with it.
