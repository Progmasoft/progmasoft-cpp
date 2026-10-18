<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 9. Comments and documentation

Comments are written in US English, in full sentences that end with a period. They explain what the code cannot say for
itself: what an interface promises, why a choice was made, and what would break if someone changed it.

## 9.1 Documentation comments

Every public declaration in a header, meaning every type, function, enumerator, constant and data member that another
component can use, has a Doxygen comment written with `///`:

```cpp
/// Derive the blocks that are live on entry to each block of a function.
/// The result lists blocks in ascending id order, so it is stable across
/// runs and can be compared in tests.
/// @param graph Control-flow graph of the function; must be verified.
/// @param uses Variables read by each block.
/// @return Live-in sets, or an error when the graph and the use table
///         describe different numbers of blocks.
[[nodiscard]] auto
AnalyzeLiveness(const ControlFlowGraph &graph, const UseTable &uses)
    -> llvm::Expected<LivenessResult>;
```

- **The first sentence says what the declaration is or does**, as a summary that stands on its own in a list of
  declarations. Start a function's summary with a verb in the imperative: "Derive", "Return", "Create". Start a type's
  summary with a noun phrase.
- **Then the contract**: preconditions, what the result guarantees (order, ownership, lifetime of returned views), what
  happens on failure, and whether the function is thread-safe when that is not obvious.
- **`@param`** for every parameter whose meaning or constraints the name does not fully convey, and **`@return`** for
  every non-void function. `@pre`, `@post` and `@note` may be used where they help.
- **Do not document the implementation** in a header. A comment that says "uses a hash map internally" is wrong the day
  the implementation changes, and the caller should not depend on it.
- **A longer description** may use a `/** ... */` block with `@brief`, lists and examples, as Doxygen renders them.
- **Doxygen runs in CI** with warnings as errors for the public headers, so an undocumented or misdocumented declaration
  fails the build.

## 9.2 Implementation comments

Inside a function, comment **why**, not what:

```cpp
// The verifier already rejected unreachable blocks, so every block here has a
// predecessor and the entry block is the only root of the dominator tree.
```

```cpp
// Avoid: restates the code.
// Increment the counter.
++counter;
```

Comment especially:

- an invariant the code relies on and a later change could break: identity, ordering, ownership, a protocol bound;
- a workaround for a compiler, library or platform defect, with a link to the report and the condition for removing it;
- a choice that looks wrong but is deliberate, and why the obvious alternative does not work;
- the reason for every magic number that cannot be a named constant.

Keep a comment next to the code it explains, and change it in the same commit as the code.

## 9.3 TODOs

A `TODO` names the issue that tracks it: `// TODO(#123): accept signed offsets once the wire format has them.` A TODO
without an issue is either done now or not written. Do not leave commented-out code; version control keeps the history.

## 9.4 File comments

A file starts with its license header and has no other boilerplate: no author list, no creation date, no change log.
When a file needs an overview, such as a component's main header, write it as the documentation comment of the
namespace or of the main type.

## 9.5 Words

- Say what a thing is for, in the vocabulary of its users, not of its implementation.
- Prefer the active voice and the present tense: "The parser stops at the first error", not "The parsing will be
  stopped".
- Use the project's terms consistently. If the project calls something a "block", do not call it a "node" in one comment
  and a "basic block" in another.
- Do not use comments for jokes, blame or emotions. A comment is read years later, out of context, by someone in a hurry.
