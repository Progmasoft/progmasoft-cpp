<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 4. Files and headers

## 4.1 Extensions

| Extension | Holds |
| --- | --- |
| `.hpp` | a C++ header |
| `.cpp` | a C++ translation unit |
| `.h` | a C11 header for a [C interface](C-INTERFACES.md), and nothing else |
| `.inc` | a fragment that is included more than once on purpose, such as an X-macro table |

A C++-only header never uses `.h`: the extension tells a reader, and a build tool, which language the file is.

## 4.2 The layout of a header

```cpp
// SPDX-FileCopyrightText: 2026 Example <contact@example.com>
// SPDX-License-Identifier: MPL-2.0
#pragma once

#include <cstdint>
#include <vector>

#include <llvm/Support/Error.h>

#include "Example/Graph/BlockId.hpp"

namespace Example::Graph
{
    /// ...
} // namespace Example::Graph
```

1. The license header, as two SPDX comment lines, when the project uses them.
2. `#pragma once`. Every major compiler supports it, and it cannot collide the way two include guards with the same
   macro name can.
3. The includes (see below).
4. One namespace block. `clang-format` adds the closing comment.

A translation unit has the same shape without `#pragma once`.

## 4.3 Includes

- **Include what you use.** A file includes the header of every name it uses directly, even when another include
  happens to bring it in. A header that compiles only because of what was included before it is broken.
- **Every header compiles on its own.** A build that compiles each public header in a translation unit of its own
  catches a missing include before a user does.
- **Angle brackets for code you do not own, quotes for code you do.** `<vector>`, `<llvm/ADT/StringRef.h>` and the
  headers of every dependency use `<>`; the project's own headers use `""`. `clang-format` then puts the two groups
  in that order and sorts each.
- **Spell a project include from the root of its include path**: `"Example/Graph/BlockId.hpp"`, never a relative path
  such as `"../BlockId.hpp"`. The spelling then names the same file from every directory.
- **Public headers live under a directory named for the project**, such as `Headers/Example/Graph/`, so that the
  include spelling starts with the project name and cannot clash with another library's headers.
- **Forward-declare** a project class in a header that only needs its name for a pointer or reference, to keep build
  dependencies small. Do not forward-declare names from the standard library or from another project; include their
  header instead.
- **No `using namespace`**, in a header or a translation unit, at any scope. A `using` declaration of one name, a type
  alias or a namespace alias is fine inside a function or inside the project's own namespace in a `.cpp`:

  ```cpp
  namespace
  {
      namespace Graph = Example::Graph;
  } // namespace
  ```

## 4.4 What goes in a header

A header declares the interface of a component: types, function declarations, constants, and the inline functions and
templates that must be visible to callers. Everything else goes in the `.cpp`.

- A function is defined in the header only when it is a template, `constexpr`, or a one-line accessor that the type
  definition reads better with.
- Helpers used by one translation unit go in an unnamed namespace in that `.cpp`, giving them internal linkage. Do not use
  `static` for that purpose at namespace scope.
- A header defines no non-`inline` variable. A constant in a header is `inline constexpr`.
- A header does not depend on a macro being defined before it is included, except for the configuration macros the
  project documents.

## 4.5 The order inside a class

Order a class by what a reader of its interface needs first:

1. `public`, then `protected`, then `private`.
2. Within each: types and aliases, then constants, then factories and constructors, then the destructor, then the special
   members, then other member functions, then data members.

Data members are always last and always `private` in a class (see [Types and classes](TYPES-AND-CLASSES.md)).

## 4.6 Size

A file is at most 1500 lines; 750 or fewer is the target. The limit is not a reason to cut a coherent unit at an
arbitrary line. Split by responsibility instead, such as model, verifier, lowering, diagnostics, I/O. A file that grows
past the target is a hint that it does two things.

A function should fit on a screen, about 60 lines. One that does not usually has steps that deserve names of their own.

There is no `Utils.hpp`, `Common.hpp` or `Helpers.cpp`. A helper belongs with the code that uses it, or in a component
named for what the helper does.

## 4.7 Generated and vendored code

Generated sources are written into the build output, not into the source tree, and are never edited by hand. Vendored
code lives under one directory, such as `third_party/`, keeps its own license, formatting and style, and is excluded
from formatting and linting. A change to vendored code is made upstream, or recorded as a patch with its reason.
