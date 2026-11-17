<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 3. Naming

A name in C says what kind of thing it is, where it lives and, in a public interface, which module owns it. The
prefixes below carry that; the words after them say what the thing is for.

## 3.1 Variables

| Kind | Style | Example |
| --- | --- | --- |
| Local variable, `const` variable | `camelCase` | `userCount`, `maxCount` |
| Global variable | `gPascalCase` | `gMaxCount` |
| `static` variable in a function | `sPascalCase` | `sCallCount` |
| `static` variable at file scope | `gsPascalCase` | `gsCounter` |
| Pointer | `pPascalCase` | `pUser` |
| Pointer to a pointer | `ppPascalCase` | `ppUser` |
| Global pointer | `gpPascalCase` | `gpUser` |
| `static` pointer at file scope | `gspPascalCase` | `gspUser` |
| `static` pointer in a function | `spPascalCase` | `spUser` |

- **Each level of pointer adds one `p`**: `pItem`, `ppItem`, `pppItem`, and in the same way `gpp`, `gspp`, `spp`.
- **The prefixes come in a fixed order**: the scope (`g`), the storage (`s`), one `p` per pointer
  level, then the name in `PascalCase`.
- **`const` and `restrict` add no prefix.** A pointer to a function takes the pointer prefixes like any other pointer.
- **A Boolean says what it states**, with `is`, `did`, `can` or `has` where they fit: `isReady`, `hasError`. The scope
  and storage prefixes come first: `gIsReady`, `gsHasError`.

## 3.2 Global state

- **A public interface declares no global variable**, not even a `const` one.
- **An internal global that is not `static` must be `const`.** State that changes lives in a `static` variable at file
  scope or in an object that the caller passes in.
- **Constants of one file should be `static const`.** A constant that several files of the module share may be
  `extern const`. In C, `const` alone does not give a name internal linkage.

## 3.3 Parameters

| Kind | Style | Example |
| --- | --- | --- |
| Plain | `aPascalCase` | `aCount` |
| Input | `aIn_PascalCase` | `aIn_Count` |
| Output | `aOut_PascalCase` | `aOut_Result` |
| Input and output | `aInOut_PascalCase` | `aInOut_Buffer` |
| Input pointer | `aIn_pPascalCase` | `aIn_pData` |
| Output pointer to a pointer | `aOut_ppPascalCase` | `aOut_ppData` |
| Input and output, three pointer levels | `aInOut_pppPascalCase` | `aInOut_pppData` |

- **The direction comes before the pointer level**: `aOut_pResult`, never `apOut_Result`.
- **Only a plain scalar parameter may go without a direction**: `aCount`, `aValue`.
- **The direction in the name matches the direction in the documentation**: `aOut_pResult` is `@param[out]`.

## 3.4 Functions, types, constants and macros

- **Functions are `PascalCase`**, `static` and `static inline` ones too, with no prefix for either.
- **The tags of `struct`, `union` and `enum` are `PascalCase`.**
- **A `typedef` is `tPascalCase`**; a `typedef` of a function pointer is `tFn_PascalCase`.
- **Enumerators are `kPascalCase`.**
- **Macros are `UPPER_SNAKE_CASE`**, and the parameters of a function-like macro are `dA_PascalCase`.

## 3.5 Modules

**Every type, function and macro of a public interface carries the module's prefix**, the parts of the module name
joined by underscores. For a public enumerator, the module prefix comes before the `k`:

```c
typedef struct Example_List_List
{
        size_t count;
} Example_List_tList;

typedef enum Example_List_Status
{
        Example_List_kSuccess,
        Example_List_kFailure,
} Example_List_tStatus;

typedef void (*Example_List_tFn_Visit)(int aValue);

#define EXAMPLE_LIST_MAX_VALUES 1024U

Example_List_tList*
Example_List_Create(void);
```

- **Every macro carries the module prefix**, internal macros as well as public ones.
- **An internal enumerator should be plain `kPascalCase`**: `kReady`. It may take the module prefix where the short name
  could collide.
- **An internal function needs no module prefix.**

A module's name also names its files ([Files and headers](FILES-AND-HEADERS.md#41-file-and-directory-names)).

## 3.6 What function names mean

A verb at the start of a function name promises a behavior. A function keeps the promise of the verb it uses:

| Name | Meaning |
| --- | --- |
| `Create` / `Destroy` | Allocate an object the caller then owns / release it, memory included. |
| `Init` / `Deinit` | Set up an object whose memory already exists / release what it holds, but not its own memory. |
| `Open` / `Close` | Acquire a resource / release it; `Close` does not free the object's own memory. |
| `Is` | Ask a Boolean property; changes nothing. |
| `Did` | Ask whether an event happened or finished; changes nothing. |
| `Can` | Ask whether an operation is possible, without doing it. |
| `Has` | Ask about ownership, presence, containment or a feature; changes nothing. |
| `Set` | Change a value or a property. |
| `Try` | An operation for which failing is an ordinary outcome. Not every function that can fail needs it. |
| `Find` | Search; finding nothing is an ordinary outcome. |
| `Remove` | Take out of a collection or a relation. |
| `Delete` | Remove a lasting resource: a file, a record. |
| `Add` / `Insert` / `Append` | Add / put at a given position / put at the end. |
| `Clear` / `Reset` | Empty the contents / go back to the defined default state. |
| `Copy` / `Clone` / `Move` | Copy into an existing target / make an independent object / hand over ownership. |
| `Compare` / `Equals` | Order, as the sign of an `int` / equality, as a `bool`. |
| `Validate` / `Check` | Check against rules / check a condition or a state. |
| `Parse` / `Convert` | Read a syntax / change a representation or a type. |
| `Read` / `Write` | Move data from a source / to a target. |

- **A getter needs no `Get`**: it may be named for the property, `BufferSize()`.
- **A setter starts with `Set`.**
- **The contract of a function states who owns what, and what happens when it fails**; the name only announces it.
