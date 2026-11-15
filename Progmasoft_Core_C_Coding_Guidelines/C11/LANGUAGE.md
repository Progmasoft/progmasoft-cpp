<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 9. Language

The guidelines are written for ISO C11: code compiles with `-std=c11 -pedantic-errors`.

## 9.1 Extensions

- **No language extensions.** Two constructs that are not ISO C11 are part of every header and are allowed:
  `#pragma once`, and `extern "C"` inside `#ifdef __cplusplus`, which a C compiler never sees
  ([Files and headers](FILES-AND-HEADERS.md#42-the-start-of-a-header)).
- **`{}` is not C11**: an empty initializer is `{ 0 }`.

## 9.2 C11 features

- **`bool`, `true` and `false` come from `<stdbool.h>`** and should be used for every truth value.
- **`restrict` may be used only where the aliasing contract has been checked**; in a public interface, the
  documentation states what it forbids the caller.
- **An `_Atomic` variable is named like any other variable.** `_Atomic(Type)` should be preferred to the qualifier form.
- **`_Thread_local` adds no prefix to a name.**
- **`_Alignas`, `_Alignof`, `_Static_assert`, `_Generic` and `_Noreturn` are used where they mean something.**
  `_Static_assert` always has a message that says what failed; `_Noreturn` marks only functions that never return.
- **`volatile` does not synchronize threads.** Shared state between threads uses `_Atomic` or the functions of
  `<threads.h>` or the platform.
- **`auto` and `register` should not be written.**

```c
_Static_assert(EXAMPLE_LIST_MAX_VALUES <= SIZE_MAX / sizeof(int),
               "EXAMPLE_LIST_MAX_VALUES must fit in a size_t of bytes.");
```
