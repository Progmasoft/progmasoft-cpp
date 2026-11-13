<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 12. Testing

A change that alters behavior comes with a test that fails without it. A bug fix comes with the test that reproduces the
bug. Tests are code: they follow this guide, they are reviewed, and a flaky test is a bug.

## 12.1 Unit tests

- **Tests live next to the component they test**, in a `Tests/` directory of that component, not in one tree for the
  whole repository. The test then moves, and is found, with its code.
- **One test file per unit under test**: `ControlFlowGraphTests.cpp` for `ControlFlowGraph.hpp`.
- **A test name is a sentence that states the behavior**: `"DenseIdMap keeps zero and the maximum id as distinct keys"`.
  When it fails, the name alone says what broke.
- **Arrange, act, assert**, separated by blank lines. One behavior per test; several assertions about that one behavior
  are fine.
- **Test the contract, not the implementation**: inputs and observable results, boundary values (zero, one, the maximum,
  one past it), the empty case, and every documented failure.
- **No shared mutable state between tests**, and no dependence on their order. A test creates what it needs, in a
  temporary directory when it touches files.
- **Deterministic**: no wall clock, no unseeded random numbers, no network. Property-based tests use a fixed or a
  recorded seed, so a failure can be reproduced.
- **Fast**: a unit test runs in milliseconds. A slow scenario is a separate, labeled test target.

The guide does not prescribe a test framework. Catch2, [Catch3](https://github.com/Progmasoft/catch3), GoogleTest,
doctest and others all fit; choose one per project and use it everywhere in that project. The example below uses the
Catch2-style API:

```cpp
TEST_CASE("BitSet union keeps the bits of both operands")
{
    auto left = llvm::cantFail(BitSet::Create(8U));
    auto right = llvm::cantFail(BitSet::Create(8U));
    left.Set(1U);
    right.Set(6U);

    // errorToBool consumes the error, so a failure is reported, not leaked.
    REQUIRE_FALSE(llvm::errorToBool(left.UnionWith(right)));

    CHECK(left.Test(1U));
    CHECK(left.Test(6U));
    CHECK(left.Count() == 2U);
}
```

Whatever the framework, use its two kinds of assertion for what they are for: one that stops a test whose further checks
would make no sense (`REQUIRE`, `ASSERT_*`), and one that reports and continues (`CHECK`, `EXPECT_*`), so one run shows
every failed expectation.

## 12.2 Fuzzing

Every parser of untrusted input, every decoder of a binary format and every round trip (encode, then decode) has a fuzz
target.

- **One target per entry point**, built with AddressSanitizer and UndefinedBehaviorSanitizer. Write it against the
  `LLVMFuzzerTestOneInput` entry point, which libFuzzer, AFL++ and honggfuzz all accept, so the engine can be chosen
  or changed without touching the target.
- **A seed corpus** of small valid inputs checked into the repository, and a dictionary of the format's keywords and
  magic numbers when it has them.
- **Bounded runs in CI** on every change, and longer campaigns on a schedule. A crash found by a fuzzer becomes a
  regression test with the minimized input.
- **An oracle beyond "it did not crash"**: a round trip compares the result with the input; a verifier and a
  transformation are checked against each other.

## 12.3 Sanitizers

CI builds and runs the tests under:

| Sanitizer | Finds |
| --- | --- |
| AddressSanitizer | out-of-bounds access, use after free, double free, leaks (with LeakSanitizer) |
| UndefinedBehaviorSanitizer | signed overflow, invalid shifts, misaligned access, invalid enum and bool values, null dereference |
| ThreadSanitizer | data races and lock-order inversions, for every test that runs threads |

UBSan runs with `-fno-sanitize-recover=all`, so the first finding fails the test instead of printing and continuing.
MemorySanitizer may be added where all dependencies can be built with it.

## 12.4 Benchmarks

Performance claims are measured. A change that is made for speed comes with a benchmark, and its pull request shows the
numbers before and after.

- **One benchmark binary per component**, next to its tests, written with the project's benchmark library (Google
  Benchmark, nanobench, the benchmarking support of the test framework, or another).
- **Benchmark realistic sizes**, and keep the setup outside the timed region.
- **Compare like with like**: the same machine, an optimized build, several repetitions, and the variance reported along
  with the mean.
- A benchmark whose fixture is invalid fails; it must not time a broken workload.

## 12.5 Coverage

Coverage is measured in CI and reported per component. It shows what is not tested; it is not a goal in itself, and a
line that is executed is not thereby tested. Review the uncovered branches of a change rather than chasing a percentage.
