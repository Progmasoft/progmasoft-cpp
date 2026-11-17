<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# C11

C11 is the default, for every C project. A C11 project follows every chapter of [Common](../Common/PRINCIPLES.md)
unchanged and uses the features below.

## Building

- **Code compiles with `-std=c11 -pedantic-errors`**, next to the warnings of
  [Tooling](../Common/TOOLING.md#112-commands).
- **No language extensions** but the two that every header uses
  ([Language](../Common/LANGUAGE.md#91-what-every-version-allows)).

## Features

- **`_Atomic(Type)` should be preferred to the `_Atomic` qualifier**, and `<stdatomic.h>` gives the operations.
- **`_Thread_local` makes a variable local to a thread.**
- **`_Alignas`, `_Alignof`, `_Static_assert`, `_Generic` and `_Noreturn` are used where they mean something.**
  `_Static_assert` always has a message that says what failed; `_Noreturn` marks only functions that never return.
- **Threads and locks come from `<threads.h>` or the platform.**

```c
_Static_assert(EXAMPLE_LIST_MAX_VALUES <= SIZE_MAX / sizeof(int),
               "EXAMPLE_LIST_MAX_VALUES must fit in a size_t of bytes.");
```
