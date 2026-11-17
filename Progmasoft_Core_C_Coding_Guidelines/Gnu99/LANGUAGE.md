<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# GNU99

GNU99 is for code that joins a codebase built as GNU99, such as firmware or kernel code, or that needs one of the GNU
extensions below. It is C99 with a short, closed list of GNU extensions; every extension not listed here is forbidden,
as in C11.

A GNU99 project follows every chapter of [Common](../Common/PRINCIPLES.md) unchanged and uses the forms below where
the [C11 page](../C11/LANGUAGE.md) uses a C11 feature.

## Building

- **Code compiles with `-std=gnu99`**, next to the warnings of [Tooling](../Common/TOOLING.md#112-commands).
  `-pedantic-errors` is not used, because it rejects the allowed extensions; review checks that only the listed ones
  appear ([What review checks](../Common/TOOLING.md#113-what-review-checks)).
- **`clang-tidy` runs with the same flag**: `clang-tidy Example.List.c -- -std=gnu99 -I.`
- **Clang and GCC both build it.** Every extension below is one that both compilers support, so the code does not
  depend on which of them a codebase uses.
- **Extensions are written in their double-underscore form**, `__attribute__`, `__typeof__`, `__asm__`, which also
  compiles under `-std=c99` and `-std=c11`, where `typeof` and `asm` are not keywords.

## The extensions it allows

- **Function and variable attributes**, `__attribute__((...))`, with these names: `noreturn`, `format`, `nonnull`,
  `warn_unused_result`, `unused`, `aligned`, and `packed` only for a layout that an external format or device fixes.
- **`__typeof__` and statement expressions, `({ ... })`, only inside a macro**, so that the macro evaluates each
  parameter once ([Preprocessor](../Common/PREPROCESSOR.md#72-macros)). The variables of a statement expression carry
  the name of the macro, so they cannot hide a variable of the caller.
- **`__builtin_add_overflow`, `__builtin_sub_overflow` and `__builtin_mul_overflow` for size computations**, which the
  [check against `SIZE_MAX`](../Common/MEMORY-AND-OWNERSHIP.md#82-allocating) may use instead of a division.
- **`__builtin_expect` only where a measurement shows that it helps**, with the measurement named next to it.
- **Inline assembly, `__asm__ __volatile__`, only inside a `static inline` function** whose name says what the
  instruction does, in a header of its own for each architecture.

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

## What it does not allow

- **No other extension**: no nested functions, zero-length arrays (C99 has the flexible array member), case ranges,
  labels as values, `__label__`, the conditional with an omitted middle operand (`a ?: b`), or anonymous structures
  and unions.
- **Variable-length arrays must not be used.** Their size is not checked against the stack, and C11 makes them
  optional, so the code keeps building where it moves to C11. A buffer whose size is known only at run time is
  allocated.

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

## Public headers

A public header of a GNU99 library is included by C11 and C++ code. It uses no extension but `__attribute__` with the
names above, which Clang and GCC accept in C11 with `-pedantic-errors` and in C++, and it declares nothing that only
GNU99 can compile.
