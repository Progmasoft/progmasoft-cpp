<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 11. Concurrency

Concurrent code is correct only when it is correct for every interleaving, and testing finds few of them. The rules
below make the shared state small and visible, so that a reader can check it.

## 11.1 Share less

- **Prefer no sharing**: give each thread its own data and combine the results at the end. A result that several
  threads compute is merged in a fixed order, so the output does not depend on scheduling.
- **Immutable data may be shared freely.** An object that is fully built before the threads start and never changed
  afterwards needs no lock.
- **Everything else that more than one thread touches** is protected by one mutex, or is an atomic, and the comment of
  the member says which.

## 11.2 Threads

- **`std::jthread`**, which joins on destruction and carries a stop token, instead of `std::thread`.
- **No detached threads.** Every thread has an owner that joins it.
- **A thread pool or an executor** for many short tasks, owned by the component that needs it, not a global.
- **Name long-lived threads** when the platform allows it, so they are recognizable in a debugger and a profiler.

## 11.3 Locks

- **`std::scoped_lock`** to lock, and `std::unique_lock` only when a condition variable or a deferred lock needs it.
  Never call `lock()` and `unlock()` by hand.
- **Hold a lock for as short as possible**, and never while calling code you do not control: a callback, a virtual
  function of an unknown implementation, I/O.
- **Lock several mutexes with one `std::scoped_lock`**, which avoids deadlock, or document a lock order and keep to it.
- **A condition variable waits with a predicate**: `condition.wait(lock, [&] { return ready; })`, never a bare `wait`,
  which wakes spuriously.

## 11.4 Thread-safety annotations

With Clang, annotate the shared state and the functions that need a lock, and build with `-Wthread-safety` as an error:

```cpp
class Cache final
{
public:
    void
    Insert(Key key, Value value) EXAMPLE_EXCLUDES(mutex_);

private:
    Mutex mutex_;
    std::unordered_map<Key, Value> entries_ EXAMPLE_GUARDED_BY(mutex_);
};
```

The annotation macros wrap Clang's `capability`, `guarded_by`, `requires_capability` and `locks_excluded` attributes,
and expand to nothing on other compilers. `Mutex` is a thin wrapper of `std::mutex` that carries the `capability`
attribute, because the standard type does not.

## 11.5 Atomics

- **An atomic is for one value**, such as a counter or a flag. Two atomics that must change together need a mutex.
- **Sequentially consistent by default**: `counter.fetch_add(1)`. A weaker memory order (`acquire`, `release`,
  `relaxed`) needs a comment that names the operation it synchronizes with and why the weaker order is enough.
- **No `volatile` for synchronization.** It is for memory-mapped hardware, not for threads.

## 11.6 Statics and globals

- A function-local `static` that is initialized once is thread-safe in C++; use it for a lazily built constant table.
  Do not use it for mutable state.
- No mutable global state. When a process-wide resource is unavoidable, such as a signal handler or a C library that
  must be initialized once, wrap it in one object with a documented lifetime.

## 11.7 Testing concurrent code

ThreadSanitizer runs the concurrent tests in CI (see [Testing](TESTING.md#123-sanitizers)). A test that relies on
`sleep` to order threads is wrong; it uses a latch, a barrier or a condition the threads wait for.
