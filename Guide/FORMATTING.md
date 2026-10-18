<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 2. Formatting

Formatting is mechanical, so a tool does it. Every C++ file is formatted by `clang-format` with the profile below, and
CI fails when a file is not. Nobody formats by hand, and nobody argues about a formatting choice in a review: a change
to the profile is a change to this guide.

## The shape of the code

```cpp
namespace Example::Geometry
{
    /// Axis-aligned rectangle with a non-negative size.
    class Rectangle final
    {
    public:
        /// Create a rectangle, or report why the size is not valid.
        /// @param width Width in pixels; must not be negative.
        /// @param height Height in pixels; must not be negative.
        /// @return The rectangle, or an error naming the invalid side.
        [[nodiscard]] static auto
        Create(std::int32_t width, std::int32_t height)
            -> llvm::Expected<Rectangle>;

        /// Area in square pixels.
        [[nodiscard]] auto
        Area() const noexcept -> std::int64_t
        {
            return static_cast<std::int64_t>(width_) * height_;
        }

    private:
        Rectangle(std::int32_t width, std::int32_t height)
            : width_{width}
            , height_{height}
        {
        }

        std::int32_t width_{};
        std::int32_t height_{};
    };
} // namespace Example::Geometry
```

- Four spaces, never tabs. Lines of at most 80 columns.
- Every brace that opens a namespace, class, function or control statement stands on its own line (Allman style).
- The contents of a namespace are indented.
- A function's return type, or `auto` with a trailing return type, stands on its own line above the name.
- When the parameters or arguments do not fit on one line, each gets a line of its own.
- A binary operator that starts a continuation line leads that line.
- Constructor initializers and base classes each start a line with their comma.
- `*` and `&` stick to the name: `const Node *node`, `std::string &text`.

## The profile

Copy this file to `.clang-format` at the root of the project. It needs a recent clang-format; pin one exact version in
CI, because two versions can format the same file differently.

```yaml
---
Language: Cpp
BasedOnStyle: LLVM
Standard: c++20

UseTab: Never
IndentWidth: 4
ContinuationIndentWidth: 4
ConstructorInitializerIndentWidth: 4
AccessModifierOffset: -4
IndentAccessModifiers: false
ColumnLimit: 80

BreakBeforeBraces: Custom
BraceWrapping:
  AfterCaseLabel: true
  AfterClass: true
  AfterControlStatement: Always
  AfterEnum: true
  AfterExternBlock: true
  AfterFunction: true
  AfterNamespace: true
  AfterStruct: true
  AfterUnion: true
  BeforeCatch: true
  BeforeElse: true
  BeforeLambdaBody: false
  BeforeWhile: false
  IndentBraces: false
  SplitEmptyFunction: false
  SplitEmptyRecord: false
  SplitEmptyNamespace: false

AllowShortBlocksOnASingleLine: Never
AllowShortCaseLabelsOnASingleLine: false
AllowShortEnumsOnASingleLine: false
AllowShortFunctionsOnASingleLine: None
AllowShortIfStatementsOnASingleLine: Never
AllowShortLambdasOnASingleLine: None
AllowShortLoopsOnASingleLine: false
AllowShortNamespacesOnASingleLine: false
AllowShortRecordOnASingleLine: Never

BreakAfterReturnType: All

PackArguments:
  BinPack: OnePerLine
PackParameters:
  BinPack: OnePerLine
AllowAllArgumentsOnNextLine: false
AllowAllParametersOfDeclarationOnNextLine: false

BreakBeforeBinaryOperators: All
BreakBeforeTernaryOperators: true
AlignOperands: Align

BreakConstructorInitializers: BeforeComma
PackConstructorInitializers: Never
BreakInheritanceList: BeforeComma

PointerAlignment: Right
DerivePointerAlignment: false
QualifierAlignment: Leave

SpaceBeforeParens: ControlStatements
SpaceBeforeAssignmentOperators: true
SpaceBeforeCaseColon: false
SpaceBeforeCtorInitializerColon: false
SpaceBeforeInheritanceColon: false
SpaceBeforeCpp11BracedList: false
SpacesInAngles: Never
SpacesInSquareBrackets: false
SpacesInParens: Never
SpaceInEmptyBraces: Never

Cpp11BracedListStyle: Block

SpaceAfterTemplateKeyword: false
BreakTemplateDeclarations: Yes

IndentCaseLabels: true
IndentCaseBlocks: false

NamespaceIndentation: All
FixNamespaceComments: true
ShortNamespaceLines: 0
CompactNamespaces: false

IndentPPDirectives: AfterHash
PPIndentWidth: 4

IncludeBlocks: Regroup
SortIncludes: CaseSensitive
IncludeIsMainRegex: '$^'
IncludeCategories:
  - Regex: '^<.*>$'
    Priority: 1
    SortPriority: 1
  - Regex: '^".*"$'
    Priority: 2
    SortPriority: 2

MaxEmptyLinesToKeep: 1
KeepEmptyLines:
  AtStartOfBlock: false

ReflowComments: true
SortUsingDeclarations: Lexicographic

LineEnding: DeriveLF
InsertNewlineAtEOF: true
...
```

### Why these choices

| Choice | Reason |
| --- | --- |
| 80 columns | Two files side by side, or a diff, fit on one screen without wrapping. |
| Four spaces | Nesting stays visible at a glance; deep nesting looks deep, which it should. |
| Braces on their own line | The opening and the closing brace of a block line up, and a block is never mistaken for a continuation. |
| Return type on its own line | Function names start at the same column, so a file can be scanned, and grepped with `^Name(`, for its functions. |
| One argument per line when they do not fit | A diff that adds an argument changes one line, and arguments are never hidden at the end of a long line. |
| No short forms on one line | Every body has the same shape, and a breakpoint can be set on any statement. |
| Leading binary operators | The operator that joins a line to the one above is the first thing a reader sees. |
| `IncludeIsMainRegex: '$^'` | No header is treated as the "main" header and moved to the top; the order is purely system, then project. |
| `DeriveLF` | A file keeps the line endings it has, so formatting never rewrites a whole file to change them. |

## What the tool leaves to you

`clang-format` lays out tokens; it does not decide what the code looks like at a larger scale.

- **Always use braces** for the body of `if`, `else`, `for`, `while` and `do`, even for one statement. A line added
  later then cannot fall outside the body.

  ```cpp
  if (items.empty())
  {
      return Error::Empty;
  }
  ```

- **One declaration per line.** `int width, height;` declares two things on a line that reads as one.
- **Blank lines separate steps.** A function that does three things in sequence shows three paragraphs. Do not open or
  close a block with a blank line.
- **Keep nesting shallow.** Return early for the error and edge cases, then write the main path at the outer level. More
  than three levels of nesting is a sign that a function should be split.
- **A long expression gets a name.** When a condition or a computation wraps over several lines, give its parts named
  variables instead of relying on the line breaks to explain it.
- **Do not align by hand.** Extra spaces to line up `=` or comments are undone by the next rename and make a diff wider
  than the change.
- **`// clang-format off`** is allowed only around a table whose layout carries meaning, such as a lookup table or a
  matrix, and is always closed again with `// clang-format on` within the same screen of code.

## Files that are not formatted

Vendored and generated code keeps the formatting of its origin. List those paths in `.clang-format-ignore`, so the
formatting check skips them, instead of reformatting a dependency.
