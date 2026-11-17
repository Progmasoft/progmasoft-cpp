<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# GNU99

GNU99 is C99 with the GNU extensions. It is for code that joins a codebase built as GNU99, such as firmware or kernel
code, or that uses GNU extensions.

A GNU99 project follows every chapter of [Common](../Common/PRINCIPLES.md) unchanged and uses the forms below where
the [C11 page](../C11/LANGUAGE.md) uses a C11 feature.

## Building

- **Code compiles with `-std=gnu99`**, next to the warnings of [Tooling](../Common/TOOLING.md#112-commands).
  `-pedantic-errors` is not used, because it rejects the extensions.
- **`clang-tidy` runs with the same flag**: `clang-tidy Example.List.c -- -std=gnu99 -I.`

## Extensions

- **GNU extensions may be used.** Attributes, `__typeof__`, statement expressions, the built-in functions, inline
  assembly and the other extensions of GNU C are part of the language of a GNU99 project.
- **An extension follows the chapters of Common like any other code**: a statement expression in a macro still
  evaluates each parameter once ([Preprocessor](../Common/PREPROCESSOR.md#72-macros)), and its variables carry the name
  of the macro, so they cannot hide a variable of the caller.

```c
#define EXAMPLE_LIST_MAX(dA_First, dA_Second)                             \
        ({                                                                \
                __typeof__(dA_First) exampleListMaxFirst = (dA_First);    \
                __typeof__(dA_Second) exampleListMaxSecond = (dA_Second); \
                exampleListMaxFirst > exampleListMaxSecond                \
                        ? exampleListMaxFirst                             \
                        : exampleListMaxSecond;                           \
        })

size_t bytes = 0;
if(__builtin_mul_overflow(aCount, sizeof(*pValues), &bytes))
        return (false);
```

## The GNU99 form of the C11 features

| C11 | GNU99 |
| --- | --- |
| `_Static_assert(condition, "message")` | the same: Clang and GCC accept it in GNU99, and it keeps its message |
| `_Noreturn` | `__attribute__((noreturn))` |
| `_Alignas(N)` | `__attribute__((aligned(N)))` |
| `_Alignof(Type)` | `__alignof__(Type)` |
| `_Thread_local` | `__thread` |
| `_Atomic` and `<stdatomic.h>` | the `__atomic_*` built-ins on an integer or a pointer, each with its memory order (`__ATOMIC_ACQUIRE`, ...) written out |
| `_Generic` | none: a function for each type |
| `<threads.h>` | the threads of the platform |
