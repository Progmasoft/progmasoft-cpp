<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# Contributing

Progmasoft C++ Style changes when a rule turns out to be wrong, unclear or missing. Proposals are welcome from anyone,
whether or not they work on a Progmasoft project.

## Proposing a change

Open an issue first for anything beyond a correction of wording, a typo or a broken link. Describe:

- **the rule**, as it reads now or as you would write it;
- **the problem**: code that the current rule makes worse, a bug it fails to prevent, or a case it does not cover;
- **the cost of the change** for projects that already follow the guide: how much code would have to change, and whether
  a tool can do it.

A rule earns its place by preventing a real class of mistakes or by removing a choice that would otherwise be argued
about in every review. "Another style guide does it" is a starting point, not a reason.

## Writing the change

- Keep the shape of the guide: a rule in bold that says what to do, then the reason, then an example when it helps.
- Examples follow every rule of the guide, including formatting by the profile and the 80-column limit.
- A change to `.clang-format` or `.clang-tidy` says which clang version it was tested with and what it changes in
  formatted code.
- Write in US English, in the present tense and the active voice.
- Note the change under `## Unreleased` in [CHANGELOG.md](CHANGELOG.md), saying whether existing code must change.

## License of contributions

By contributing you agree that your text is licensed under [CC BY 4.0](LICENSE.md) and that your configuration and code
examples are dedicated to the public domain under [CC0 1.0](LICENSES/CC0-1.0.md), like the rest of the guide.
