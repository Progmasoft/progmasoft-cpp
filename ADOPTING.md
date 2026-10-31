<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# Adopting the guide

A project adopts Progmasoft C++ Style by pointing to it, copying its two tool profiles and writing down, once, where it
deviates.

## 1. Point to the guide

Add a short section to the project's contributing guide:

```markdown
## C++ style

C++ code follows [Progmasoft C++ Style](https://github.com/Progmasoft/progmasoft-cpp).
`clang-format` and `clang-tidy` enforce what they can; review covers the rest.
Deviations of this project are listed below.
```

Do not copy the rules into the project. A copy drifts from the guide, and two documents that disagree are worse than
one.

State the language version in the same section, C++20 or C++17; [the guide says how to choose](Guide/LANGUAGE.md).

## 2. Copy the profiles

| File | From |
| --- | --- |
| `.clang-format` | [Formatting: the profile](Guide/FORMATTING.md#the-profile) |
| `.clang-tidy` | [Tooling: the clang-tidy profile](Guide/TOOLING.md#132-the-clang-tidy-profile), with the path filters set to the project's directories |
| compiler flags | [Tooling: flags](Guide/TOOLING.md#flags), in the build files of the project |

The profiles are in the public domain (see [LICENSE.md](LICENSE.md)), so they can be copied under the project's own
license. Note the guide revision they come from in a comment at the top of each file, so a later update can be compared.

## 3. Record the deviations

Every project has a few rules of its own: a namespace root, an include prefix, a test framework, a component that must
use exceptions because its host does. List them in the contributing guide, each with its reason:

```markdown
### Deviations

- The intermediate representations use the lowercase namespaces `tala::core`, `tala::xpp` and `tala::xmm`, because
  they are qualified on almost every line of the compiler (Naming, 3.4).
- Assertion failures are reported by throwing, as the Catch2 API requires; exceptions stay inside the library
  (Errors, 7.1).
```

Within the project, its documented rule wins over the guide. An undocumented difference is not a deviation; it is code
that has not caught up yet.

## 4. Converge gradually

Existing code is not rewritten in one sweep. A file follows the guide where it is changed. A purely mechanical change,
such as applying `clang-format` to a directory or renaming a member convention, is its own pull request, never mixed
with a change in behavior, so that review and `git blame` stay useful.

## Projects that follow the guide

| Project | Deviations recorded in |
| --- | --- |
| [Progmasoft/tala](https://github.com/Progmasoft/tala) | `Documents/Contributing/STYLE-AND-LICENSING.md` |
| [Progmasoft/catch3](https://github.com/Progmasoft/catch3) | the Progmasoft-authored sources under `src/Progmasoft/`; code derived from Catch2 keeps its upstream style |
