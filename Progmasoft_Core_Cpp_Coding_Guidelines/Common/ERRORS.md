<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 7. Errors

A failure is one of three things, and each has one way of being handled.

| Kind | Example | Handled by |
| --- | --- | --- |
| **Recoverable failure** | a malformed input file, a missing path, a value out of range | returning `Expected<T>` or `Error` |
| **Broken invariant** | a verifier that accepted an IR it must reject, an index the caller guaranteed in range | an assertion, or a fatal error in release builds |
| **Unrecoverable environment** | out of memory, a corrupted heap | termination |

## 7.1 No exceptions in libraries and tools

Code is compiled with `-fno-exceptions` (`/clang:-fno-exceptions` with `clang-cl`). There is no `throw`, `try` or
`catch` in first-party code, including tests, fuzzers and benchmarks.

The reasons are practical: an exception is invisible in a signature, so a caller cannot see that a call can fail; it
cannot cross a C interface; and its cost and its interaction with destructors are hard to reason about in code that also
runs under sanitizers and fuzzers.

**A project may opt in** when its host model depends on exceptions, as a C++ test framework that reports a failed
assertion by throwing does. The project says so in its documentation, keeps exceptions inside its own code, and still
never lets one cross a C interface or escape a destructor. Third-party code keeps its own compiler options either way.

## 7.2 `Expected<T>` and `Error`

A function that produces a value or fails returns `Expected<T>`; one that only succeeds or fails returns `Error`. The guide
does not prescribe the library. `llvm::Expected<T>` and `llvm::Error` suit code that already depends on LLVM; a
backport of `std::expected` (such as `tl::expected`) or a small type of the project's own suit the rest, and C++23's
`std::expected` replaces them once a project moves to it. What matters is the discipline below: the failure is in the
signature, it cannot be ignored, and it carries a message.

The examples use LLVM's types.

```cpp
[[nodiscard]] auto
ParseHeader(llvm::ArrayRef<std::uint8_t> bytes) -> llvm::Expected<Header>
{
    if (bytes.size() < kHeaderSize)
    {
        return llvm::createStringError(
            "header is %zu bytes; at least %zu are required",
            bytes.size(),
            kHeaderSize);
    }
    // ...
}

[[nodiscard]] auto
LoadModule(const std::filesystem::path &path) -> llvm::Expected<Module>
{
    auto bytes = ReadFile(path);
    if (!bytes)
    {
        return bytes.takeError();
    }
    auto header = ParseHeader(*bytes);
    if (!header)
    {
        // path::c_str() is wide on Windows; string() is the narrow form.
        return llvm::joinErrors(
            llvm::createStringError("%s: not a module",
                                    path.string().c_str()),
            header.takeError());
    }
    return Module{*header, std::move(*bytes)};
}
```

- **Every result is inspected exactly once.** Prefer a type that enforces this: an `llvm::Error` that is destroyed
  unchecked aborts a debug build, and `[[nodiscard]]` catches the rest. Propagate the error, handle it, or consume
  it with a comment saying why it may be dropped.
- **Add context on the way up**, so the final message says what was being done: the file, the record, the field. Do not
  wrap an error without adding information.
- **Messages are lower-case, without a trailing period**, and name the offending value: `entry 'Demo.Main' names no
  class`. They are written for the person who has to fix the input.
- **A typed error** (an error class or an error enumeration, such as a class derived from `llvm::ErrorInfo`) when a
  caller needs to react to one kind differently, for example to retry. Otherwise an error with a message is enough.
- **Do not use `std::error_code` as the interface** of new code. At the boundary of an API that reports through
  `std::error_code`, such as the non-throwing `std::filesystem` overloads, convert it into the project's error
  type (`llvm::errorCodeToError` with LLVM).
  Always call the non-throwing overload; code built without exceptions cannot catch the other one.

## 7.3 Invariants

An invariant is something the program's own logic guarantees. A broken invariant is a bug in the program, not in its
input, and there is no meaningful way to continue.

- **`assert`** checks an invariant that is cheap to state, in debug builds. It never checks input, never has a side
  effect, and its message is a condition a reader understands: `assert(index < blocks_.size() && "block id out of range")`.
- **An always-on check** (a project macro that prints and calls `std::abort`, or `llvm::report_fatal_error`) guards
  an invariant whose
  violation would corrupt memory or produce wrong output in a release build, such as a verifier disagreeing with
  itself. It prints a message and terminates.
- **An unchecked unwrap** (`llvm::cantFail`, or `.value()` behind an assertion) takes the value of an `Expected` whose
  success the caller has just established, such as an
  operation on two sets that were created with the same size a line above. It is never a shortcut for input
  validation.
- **An unreachable marker** (a project macro, `llvm_unreachable`, or C++23's `std::unreachable` behind an assertion)
  follows a `switch` that handles every enumerator; it is not a way to silence a missing case.

The line between the two first kinds is the source of the value: **anything that comes from outside the process**
(a file, the network, the command line, the environment, another component's serialized output) is validated and
reported as a recoverable failure, never asserted.

## 7.4 Fatal defects in tests and fuzzers

A fuzz oracle that detects a defect, such as a round trip that changes a value, terminates with a fatal error so that
the fuzzer keeps the input. A benchmark whose fixture is corrupt terminates as well; it must not time a broken
workload. A test that cannot set up its fixture fails with a message, not a crash.

## 7.5 Destructors and cleanup

A destructor never fails. Cleanup that can fail, such as flushing a file, is a member function that returns `Error`
and that the owner calls before the object goes away. The destructor then only releases what remains, silently.

## 7.6 Diagnostics are not errors

A compiler or a linter that reports problems in its input produces **diagnostics**: data with a severity, a location
and a message, collected and printed by a sink. Producing a diagnostic is normal operation and does not use
`Expected`. `Expected` is for the failure of an operation itself, such as being unable to read the input at all.
