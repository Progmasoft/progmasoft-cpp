<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 2. Formatting

`clang-format` with the profile below does most of the layout. The rules in this chapter are the layout itself; the
profile applies them where `clang-format` 23 can, and [What the tool leaves to you](#what-the-tool-leaves-to-you) lists
the rest.

## The shape of the code

```c
int
CalculateResult(int aFirstValue, int aSecondValue)
{
        if(aFirstValue < 0)
                return(-1);

        return(aFirstValue + aSecondValue);
}

void
NoOperation(void)
{ }
```

## 2.1 Indentation and lines

- **Indent by eight spaces**, never by tabs.
- **Code and comments end by column 100.**
- **A continuation line is indented by eight spaces.**
- **Exactly one blank line between two functions.**

## 2.2 Functions and braces

- **The return type stands on its own line**, above the function name, in declarations and definitions.
- **The opening brace of a function, `struct`, `union`, `enum`, `if`, `for`, `while`, `switch` and `do` stands on its
  own line.**
- **`else` and the `while` of a `do` follow the closing brace**: `} else`, `} else if(`, `} while(`.
- **An empty body is `{ }` on the line after its head**: `NoOperation(void)` above, `for(;;)` then `{ }`, and `do`,
  then `{ } while(condition);`.

## 2.3 Bodies with one statement

- **A body of one statement has no braces**, and the statement stands on the next line. A body of more than one
  statement has braces.
- **The `if` and the `else` of a statement are judged separately**: one may have braces while the other has none.

```c
if(aIsReady)
{
        values[ 0 ] = aValue;
        gsCounter++;
} else if(aValue > 0)
        values[ 1 ] = aValue;
else
        values[ 2 ] = 0;
```

## 2.4 Parentheses and spaces

- **No space between `if`, `for`, `while` or `switch` and its `(`**: `if(`, `else if(`, `switch(state)`.
- **`return` with a value writes it in parentheses**: `return(result);`. A `void` function may return early with
  `return;`.
- **One space after each comma.**
- **Binary arithmetic, bitwise, comparison and assignment operators have a space on each side.** A unary operator (`!`,
  `~`, `+`, `-`, `*`, `&`, `++`, `--`) sticks to its operand.
- **A cast is `(Type) expression`**, with one space; `sizeof` is `sizeof(expression)`. A compound literal has no space
  between `)` and `{`: `(Example_tUserData){ 0 }`.
- **No space around `.` and `->`.**
- **Square brackets have one space inside**, in declarations and in indexing: `values[ 10 ]`, `values[ index ]`.
- **The `*` of a pointer sticks to the type**: `int* pValue`.

## 2.5 Breaking long lines

- **Parameters and arguments that do not fit on one line each get a line of their own**, the first one right after the
  `(` and the others in the column of the first. The closing `)` should stay on the line of the last one.
- **A binary operator that continues an expression starts the new line**: `=`, `+=`, `&&`, `+`, `|`, `==` and the
  others.
- **In a long ternary, `?` and `:` start their lines.**
- **In an expression over several lines, `(` stays on the line of the first operand and `)` on the line of the last.**
  Parentheses are added only where they make the meaning clearer.

```c
Example_List_CopyRange(aInOut_pDestinationList,
                       aIn_pSourceList,
                       aFirstIndex,
                       aCount,
                       aOut_pCopiedCount);

bool isWithinLimits = aIn_pList->count + aAdditionalCount <= EXAMPLE_LIST_MAX_VALUES
        && aIn_pList->capacity <= SIZE_MAX / sizeof(*aIn_pList->pValues);
```

## 2.6 Alignment

Consecutive lines line up, so a block of declarations reads as a table:

- the names and the `=` of declarations in one group, and consecutive assignments, compound ones included;
- the member names of a `struct` or `union`, and the `:` of bit-fields;
- the `=` of enumerators;
- the values of consecutive parameterless macros, and the `\` of a macro over several lines.

```c
typedef struct Example_Header
{
        unsigned int kind  : 4;
        unsigned int flags : 4;
        size_t       size;
} Example_tHeader;
```

## The profile

Copy this file to `.clang-format` at the root of the project. It needs `clang-format` 23; pin one exact version in CI,
because two versions can format the same file differently.

```yaml
---
Language: Cpp
BasedOnStyle: LLVM

IndentWidth: 8
ContinuationIndentWidth: 8
TabWidth: 8
UseTab: Never
ColumnLimit: 100

BreakAfterReturnType: All
BreakBeforeBraces: Custom
BraceWrapping:
  AfterFunction: true
  AfterStruct: true
  AfterUnion: true
  AfterEnum: true
  AfterCaseLabel: true
  AfterControlStatement: Always
  BeforeElse: false
  BeforeWhile: false
  AfterExternBlock: false
  IndentBraces: false
  SplitEmptyFunction: false
  SplitEmptyRecord: false

AllowShortFunctionsOnASingleLine: Empty
AllowShortBlocksOnASingleLine: Empty
AllowShortIfStatementsOnASingleLine: Never
AllowShortLoopsOnASingleLine: false
AllowShortCaseLabelsOnASingleLine: false
SpaceInEmptyBraces: Always

SpaceBeforeParens: Never
SpacesInParens: Never
SpacesInSquareBrackets: true
SpaceBeforeSquareBrackets: false
SpaceAfterCStyleCast: true
SpaceBeforeAssignmentOperators: true
SpaceBeforeCaseColon: false
BitFieldColonSpacing: Both
PointerAlignment: Left
DerivePointerAlignment: false

AlignAfterOpenBracket: true
AlignOperands: DontAlign
BreakBeforeBinaryOperators: All
BreakAfterOpenBracketFunction: false
BreakBeforeCloseBracketFunction: false
AllowAllArgumentsOnNextLine: false
AllowAllParametersOfDeclarationOnNextLine: false
PackArguments:
  BinPack: OnePerLine
PackParameters:
  BinPack: OnePerLine
PenaltyBreakAssignment: 100

AlignConsecutiveAssignments:
  Enabled: true
  AcrossEmptyLines: false
  AcrossComments: false
  AlignCompound: true
  PadOperators: false
  EnumAssignments: true
AlignConsecutiveDeclarations:
  Enabled: true
  AcrossEmptyLines: false
  AcrossComments: false
  AlignFunctionDeclarations: false
AlignConsecutiveBitFields: Consecutive
AlignConsecutiveMacros: Consecutive
AlignEscapedNewlines: Left

IndentCaseLabels: false
IndentCaseBlocks: false
IndentGotoLabels: NoIndent
IndentPPDirectives: AfterHash
PPIndentWidth: 4

Cpp11BracedListStyle: Block
BracedInitializerIndentWidth: 8
EnumTrailingComma: Insert

NumericLiteralCase:
  ExponentLetter: Lower
  HexDigit: Upper
  Prefix: Lower
  Suffix: Upper

SortIncludes:
  Enabled: true
  IgnoreCase: false
IncludeBlocks: Regroup
IncludeIsMainRegex: '$^'
IncludeCategories:
  - Regex: '^<(assert|complex|ctype|errno|fenv|float|inttypes|iso646|limits|locale|math|setjmp|signal|stdalign|stdarg|stdatomic|stdbool|stddef|stdint|stdio|stdlib|stdnoreturn|string|tgmath|threads|time|uchar|wchar|wctype)\.h>$'
    Priority: 1
  - Regex: '^<'
    Priority: 2
  - Regex: '^"'
    Priority: 3

MaxEmptyLinesToKeep: 1
KeepEmptyLines:
  AtStartOfBlock: false
  AtStartOfFile: false
  AtEndOfFile: false
ReflowComments: false
...
```

- `PenaltyBreakAssignment: 100` keeps a call on the line of its `=` and puts one argument on each line when the
  arguments do not fit; with the LLVM default, the formatter breaks before the `=` first.
- `IncludeIsMainRegex: '$^'` stops the formatter from moving a source file's own header to the top: the groups of
  [Files and headers](FILES-AND-HEADERS.md#43-includes) decide the order.
- `EnumTrailingComma: Insert` puts the comma after the last enumerator. LLVM documents that the option can change
  code in ways that need review, so a change it makes is reviewed like any other.

## What the tool leaves to you

`clang-format` 23 cannot express every rule above. These are checked in review, and the formatter's output is
corrected where it differs:

- **`return(value)`**: `clang-format` puts a space after `return` whatever the profile says, and writes
  `return (value);`.
- **Compound literals**: `SpaceAfterCStyleCast` also puts a space before the `{` of a compound literal,
  `(Example_tUserData) { 0 }`.
- **An empty `do` body**: `do`, then `{ } while(condition);` is split into `{` and `} while(condition);`.
- **One statement per body without braces** (2.3), the order of declaration groups and their blank lines, the blank
  lines around `case` groups and `goto` labels, and the columns of `@param` lines.

A part of a file the formatter must not touch may be wrapped in `/* clang-format off */` and `/* clang-format on */`,
as narrowly as the rule needs.
