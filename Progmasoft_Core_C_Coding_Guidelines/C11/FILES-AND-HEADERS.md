<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 4. Files and headers

## 4.1 File and directory names

- **A file is named after its module**, the parts of the module name in `PascalCase` joined by dots:
  `Example.List.h`, `Example.List.Builder.h`.
- **A directory is optional; when there is one, it is the first part of the module name in lowercase**, with no
  separators: `example/Example.List.h`. Both `Example.List.h` and `example/Example.List.h` are right;
  `example/Other.Core.h` is wrong, because the directory and the module disagree.
- **Names are compared without regard to case.** Two files or directories whose names differ only in case collide on
  some file systems, so a project must not have them.
- **An umbrella header, which includes the whole module, should end in `.Foundation.h`**:
  `example/Example.Foundation.h`.

## 4.2 The start of a header

- **Every header starts with `#pragma once`.** Include guard macros are not used.
- **A public header wraps its declarations in a conditional `extern "C"` block**, so that a C++ caller links against
  the C names. The block is active only when a C++ compiler reads the header.

```c
#pragma once

#include <stdbool.h>
#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif

/* The declarations of the module. */

#ifdef __cplusplus
}
#endif
```

## 4.3 Includes

- **The groups come in a fixed order**: the C standard library, then third-party libraries, then the project's own
  headers. One blank line separates the groups, and each group is sorted alphabetically.
- **Standard and third-party headers are included with `<...>`**; the project's own with `"..."`, by the path of the
  module: `"example/Example.List.h"`.

```c
#include <stdint.h>
#include <stdlib.h>

#include <zlib.h>

#include "example/Example.List.h"
```

The profile in [Formatting](FORMATTING.md#the-profile) sorts and groups includes this way.
