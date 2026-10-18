<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 8. Ownership and memory

Every object has exactly one owner, and the type says who it is. A reader should be able to tell from a declaration
alone whether a function keeps, borrows or gives away what it is handed.

## 8.1 What owns, what borrows

| Type | Meaning |
| --- | --- |
| `T` (a value) | owns |
| `std::unique_ptr<T>` | owns a heap object; ownership moves, never copies |
| `std::vector<T>`, `std::string`, other containers | own their elements |
| `T &`, `const T &` | borrows, never null, for the duration of the call |
| `T *`, `const T *` | borrows, may be null; never owns |
| `std::span<T>`, `std::string_view`, `llvm::ArrayRef<T>`, `llvm::StringRef` | borrows a range for the duration of the call |
| `std::shared_ptr<T>` | shared ownership; needs a reason, see below |

- **No raw `new` or `delete`.** Create a heap object with `std::make_unique`; a container or an RAII type releases it.
- **No owning raw pointers**, not even briefly. A C API that returns an owned pointer is wrapped immediately in a
  `std::unique_ptr` with a deleter (see [C interfaces](C-INTERFACES.md)).
- **`std::shared_ptr` only for genuinely shared lifetime**, where no single owner can be named, such as a cache entry
  used by several threads. Shared ownership hides who keeps an object alive and makes destruction order hard to see.
  Never use it to avoid deciding who the owner is.
- **Pass ownership by value**: a function that takes ownership of a `std::unique_ptr` takes it by value, and the caller
  writes `std::move`. A function that only uses the object takes a reference.

## 8.2 Views and their lifetimes

A view does not own what it points to. It is valid only while its source lives and is not modified.

- **A view is a parameter or a local, not a member**, unless the class documents that it is itself a view and states
  what must outlive it.
- **Never return a view into a temporary or into a local**, and never keep one past a call that can reallocate the
  source (`push_back`, `insert`, `resize`).
- **`llvm::Twine` is consumed in the expression that creates it.** Never store a `Twine`, never name one with `auto`.
- **A view of a `std::string` that may be modified** is taken again after the modification.

```cpp
// Avoid: the view outlives the string it refers to.
auto
Name(const Symbol &symbol) -> std::string_view
{
    return symbol.Qualified() + ".type"; // a temporary std::string
}
```

## 8.3 RAII for every resource

A file descriptor, a lock, a temporary directory, a C library handle and a registered callback are each owned by an
object whose destructor releases them. Cleanup never depends on reaching the end of a function. When a resource has no
dedicated type, use a scope guard (`llvm::make_scope_exit` or an equivalent) right after acquiring it.

## 8.4 Containers

- **`std::vector` is the default sequence.** Reserve when the final size is known. Do not use `std::list`; a vector of
  indices is almost always faster and simpler.
- **Lookups**: a hash map when order does not matter, a sorted vector or `std::map` when it does. In code built on LLVM,
  `llvm::DenseMap`, `llvm::StringMap` and `llvm::SmallVector` fit compiler-style workloads (small, dense, short-lived).
  Keep them out of public, library-independent interfaces, which use standard types.
- **Never iterate a hash map to produce output or to make a decision that depends on order.** The order differs between
  runs, libraries and platforms. Sort the keys first, or use an insertion-ordered map such as `llvm::MapVector`.
- **Know the sentinels of a container.** `llvm::DenseMap` reserves two key values for empty and deleted buckets; a map
  whose keys come from input must use a key type that cannot collide with them.
- **Do not hold an iterator, reference or pointer to an element across an operation that may invalidate it.**

## 8.5 Allocation

- **Allocate less.** Prefer values and inline storage (`std::array`, a small-vector type) for small, bounded data.
- **Sizes from input are checked before allocating.** A length field in a file that claims four gigabytes is rejected,
  not passed to `resize`.
- **No custom allocators** without a measurement that shows the default is the bottleneck.
- **Out-of-memory is not a recoverable failure.** Do not try to report it through `Expected`; let the allocation
  terminate the process.

## 8.6 Undefined behavior

The code must be free of undefined behavior, not merely free of crashes. In particular:

- no reads of uninitialized memory, no out-of-bounds access, no use after free or after move except to assign or destroy;
- no signed overflow, no shift by a negative amount or by the width of the type or more;
- no type punning through a `union` or a `reinterpret_cast`ed pointer; use `std::bit_cast` or `std::memcpy`;
- no data race (see [Concurrency](CONCURRENCY.md)).

AddressSanitizer, UndefinedBehaviorSanitizer and ThreadSanitizer run in CI to find what review misses (see
[Testing](TESTING.md#123-sanitizers)). Passing them is necessary, not sufficient: they only see the paths the tests run.
