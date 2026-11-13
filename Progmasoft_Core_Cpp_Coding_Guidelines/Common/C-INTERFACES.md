<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 10. C interfaces

A C interface is the stable binary boundary of a library: what another language, another compiler or another version of
the library links against. It is written in C11 and kept small; everything behind it is C++.

## 10.1 The header

```c
// SPDX-FileCopyrightText: 2026 Example <contact@example.com>
// SPDX-License-Identifier: MPL-2.0
#ifndef EXAMPLE_PARSER_H
#define EXAMPLE_PARSER_H

#include <stddef.h>
#include <stdint.h>

#ifdef __cplusplus
extern "C" {
#endif

/// Version of this interface; bumped on every incompatible change.
#define EXAMPLE_PARSER_ABI_VERSION 3

/// Opaque parser state, owned by the library.
typedef struct example_parser example_parser;

/// Result of an operation of this interface.
typedef enum example_status
{
    EXAMPLE_STATUS_OK = 0,
    EXAMPLE_STATUS_INVALID_ARGUMENT = 1,
    EXAMPLE_STATUS_MALFORMED_INPUT = 2,
} example_status;

/// Create a parser. Release it with example_parser_destroy.
/// @param out Receives the parser on success; untouched on failure.
example_status
example_parser_create(example_parser **out);

/// Release a parser created by example_parser_create. Accepts NULL.
void
example_parser_destroy(example_parser *parser);

#ifdef __cplusplus
}
#endif

#endif // EXAMPLE_PARSER_H
```

- **`.h`, C11, and include guards.** A C header may be read by a C compiler that does not know `#pragma once`, so it uses
  a guard named after the file.
- **Every name has the project prefix**: functions and types `example_`, macros and enumerators `EXAMPLE_`. C has one
  namespace for the whole program.
- **`extern "C"`** guards wrap the declarations, so C++ callers link against the same symbols.
- **Opaque types**: callers see a forward-declared struct and a pointer, never the layout. The layout can then change
  without breaking callers.
- **Fixed-width types** (`uint32_t`, `size_t` for sizes) and no `bool` in a struct whose layout is part of the ABI; use
  `uint8_t` with documented values.
- **A version macro or a version function**, so a caller can check at build time or at run time that it talks to the
  interface it was written for.

## 10.2 Ownership across the boundary

- **Whoever allocates, frees.** A buffer the library returns is released by a function of the library, never by the
  caller's `free`, because the two may not share an allocator.
- **Borrowed data is copied before the call returns** when the library needs it longer. A callback that receives a
  pointer and a length may use them only until it returns; the library copies anything it keeps into memory it owns.
- **Document every pointer parameter**: may it be null, who owns it after the call, how long it must stay valid.
- **No C++ type crosses the boundary**: no `std::string`, no reference, no class with a constructor, no template.

## 10.3 Failure across the boundary

- **No exception crosses a C interface**, ever. With exceptions disabled this holds by construction; a project that
  enables them catches everything at the boundary.
- **Every fallible function returns a status** and writes its results through out-parameters, which it leaves untouched
  on failure. A function that cannot fail returns `void` or the value.
- **Detailed messages** are available through a separate call or an error object the caller owns and releases, not
  through a global "last error" that breaks with threads.

## 10.4 The C++ side

Behind the interface, the implementation is ordinary C++ that follows the rest of this guide:

```cpp
extern "C" auto
example_parser_create(example_parser **out) -> example_status
{
    if (out == nullptr)
    {
        return EXAMPLE_STATUS_INVALID_ARGUMENT;
    }
    auto parser = std::make_unique<example_parser>();
    *out = parser.release();
    return EXAMPLE_STATUS_OK;
}
```

C++ callers wrap a C handle in an RAII type right away:

```cpp
struct ParserDeleter final
{
    void
    operator()(example_parser *parser) const noexcept
    {
        example_parser_destroy(parser);
    }
};

using ParserHandle = std::unique_ptr<example_parser, ParserDeleter>;
```

## 10.5 Keeping it stable

- **Record the interface in one place**: a manifest or a header that is the single source of truth for every symbol,
  struct layout and magic number, checked in CI against the code.
- **Add, do not change.** A new function or a new enumerator is compatible; changing a signature, a layout or the meaning
  of a value is not, and bumps the interface version.
- **Test the interface from C**: a test translation unit compiled as C11 includes the header and calls every function,
  so that a C++-only construct in the header cannot slip in.
