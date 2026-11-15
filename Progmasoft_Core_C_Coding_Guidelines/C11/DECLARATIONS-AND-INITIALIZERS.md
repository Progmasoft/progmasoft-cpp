<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 6. Declarations and initializers

## 6.1 Declarations

- **Each pointer variable is declared on its own line.** Related variables that are not pointers may share a line;
  unrelated ones do not.
- **A variable should be declared where it is first used, and initialized there.** It should not be set to zero or
  `NULL` only to be overwritten; it must hold a valid value before it is first read.
- **Declarations are grouped by type**, with a blank line between two groups, and a blank line separates the
  declarations at the start of a block from the work that follows them.
- **A unary operation should be parenthesized where that reads more clearly**: `(-3) + (-2)`. A unary `+` that changes
  nothing is left out.

```c
size_t total = 0U;
size_t count = Example_List_Count(aIn_pList);

const int* pValues = Example_List_Values(aIn_pList);

for(size_t index = 0U; index < count; index++)
        total += (size_t) pValues[ index ];
```

## 6.2 Types

- **A `typedef` of a `struct`, `union` or `enum` keeps its tag**: `typedef struct Example_List_List { ... }
  Example_List_tList;`.
- **A forward declaration is `typedef struct Tag tType;`**: `typedef struct Example_List_List Example_List_tList;`.
- **The parameter names of a function pointer `typedef` are optional.** When they are written, they follow the rules for
  parameters ([Naming](NAMING.md#33-parameters)).
- **A function without parameters says `(void)`**, in its declaration, its definition and a function pointer type.

## 6.3 Literals

- **Hexadecimal literals have a lowercase `0x` and uppercase digits**: `0xFF00AAU`.
- **Suffixes are uppercase**: `U`, `L`, `LL`, `F`. A literal should carry the suffix of its real type: `0.5F` for a
  `float`, `1024U` for an unsigned value.
- **Scientific notation uses a lowercase `e`**: `1.5e+10`.
- **A decimal point has a digit on each side**: `0.5`, `2.0`, never `.5` or `2.`.
- **No octal literals**, and no leading zeros on decimal ones: `010` is eight, which nobody means.
- **Characters are escaped as `\n` and `\t`; the null character is `\x00`.**
- **A long string is split into adjacent literals**, lined up under the first one. A string never continues over a line
  with `\`.

```c
float        ratio     = 0.5F;
unsigned int flags     = 0xFF00AAU;
double       magnitude = 1.5e+10;
const char*  pText     = "First part "
                         "second part.";
```

## 6.4 Initializers

- **A short list may stand on one line**: `{ 1, 2, 3 }`.
- **A list over several lines opens with `{` on the line of the assignment**, indents its items by eight spaces and
  closes with `}` on a line of its own.
- **A list over several lines has a comma after its last item**: an `enum`, and an array, `struct`, `union` or
  compound literal initializer. A list on one line has none. The comma keeps the list over several lines when
  `clang-format` runs, and adding an item then changes one line of a diff, not two.
- **Designated initializers are used only where choosing the member is the point**, such as a `union` or a sparse
  table.
- **Zero initialization is `{ 0 }`.** `{}` is not C11.

```c
int values[ 3 ] = { 1, 2, 3 };

int numbers[] = {
        10,
        20,
        30,
};

Example_tUserData data = { 0 };
```

`clang-format` puts the comma after the last enumerator itself (`EnumTrailingComma: Insert` in
[the profile](FORMATTING.md#the-profile)); in an initializer, the author writes it.
