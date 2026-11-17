<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 9. Language

The guidelines cover two versions of C. A project names its version in its contributing guide; without one, it is C11.

- **[C11](../C11/LANGUAGE.md)** is the default, for every C project: ISO C11, with no language extensions.
- **[GNU99](../Gnu99/LANGUAGE.md)** is for code that joins a codebase built as GNU99, such as firmware or kernel code,
  or that needs one of the GNU extensions its page lists. It is C99 with a short, closed list of GNU extensions.

The rules below hold in both. A version's page says how it is built and which features it uses.

## 9.1 What every version allows

- **Two constructs outside ISO C are part of every header and are allowed:** `#pragma once`, and `extern "C"` inside
  `#ifdef __cplusplus`, which a C compiler never sees
  ([Files and headers](FILES-AND-HEADERS.md#42-the-start-of-a-header)).
- **An empty initializer is `{ 0 }`.** `{}` is not C11, and GNU99 accepts it only as an extension.

## 9.2 Features of every version

- **`bool`, `true` and `false` come from `<stdbool.h>`** and should be used for every truth value.
- **`restrict` may be used only where the aliasing contract has been checked**; in a public interface, the
  documentation states what it forbids the caller.
- **`volatile` does not synchronize threads.** Shared state between threads uses the atomics of the version
  ([C11](../C11/LANGUAGE.md#features), [GNU99](../Gnu99/LANGUAGE.md#the-gnu99-form-of-the-c11-features)) or the
  threads of the platform.
- **A variable that is atomic or local to a thread is named like any other variable**; its storage adds no prefix.
- **`auto` and `register` should not be written.**
