<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 7. Preprocessor

## 7.1 Conditions

- **`#ifdef` and `#ifndef` should be used only to ask whether a name is defined.** A combined or numeric condition may
  use `#if`.
- **The whole condition of `#if` and `#elif` is in parentheses**: `#if (EXAMPLE_LIST_LEVEL > 1)`.
- **Nested directives keep `#` in the first column and indent after it by four spaces per level.**

```c
#if (EXAMPLE_LIST_LEVEL > 1)
#    if defined(EXAMPLE_LIST_TRACE)
#        define EXAMPLE_LIST_LOG_LEVEL 2U
#    endif
#endif
```

## 7.2 Macros

- **The values of consecutive parameterless macros line up.**
- **The `\` of a macro over several lines stand in one column**, and nothing follows a `\`, not even a space.
- **`#` is separated from its parameter by one space, and `##` has one space on each side**: `# dA_Value`,
  `dA_First ## dA_Second`.
- **A macro of several statements uses `do`, `{` on its own line, and `} while(0)`**, so that it is one statement where
  it is used. An operation that is an ordinary function should be a `static inline` function instead.
- **A macro evaluates each parameter at most once at run time.** `EXAMPLE_MAX(a, b)` that evaluates the larger one
  twice runs a side effect twice.
- **Each use of a parameter is parenthesized where precedence could change its meaning.**

```c
#define EXAMPLE_LIST_MAX_VALUES 1024U
#define EXAMPLE_LIST_MIN_VALUES 16U

#define EXAMPLE_LIST_RESET(dA_Value) \
        do                           \
        {                            \
                (dA_Value) = 0;      \
        } while(0)
```

[Naming](NAMING.md#34-functions-types-constants-and-macros) gives the names of macros and their parameters; every macro
carries the module prefix ([Modules](NAMING.md#35-modules)).
