<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 10. Comments and documentation

## 10.1 Comments

- **Comments should be `/* ... */`.** `//` is not forbidden, but should not be the choice.
- **A comment over several lines has a `*` at the start of each line.**
- **A comment stands above the code it explains, at the same indentation.** Comments at the end of a line are not used.
- **The project chooses the language of its comments and documentation**, and its own markers such as `TODO` and
  `FIXME`.

## 10.2 Doxygen

- **Every function is documented with Doxygen**: public, internal and `static` ones. A function that has a declaration
  and a definition is documented at both.
- **Commands start with `@`**, never `\`: `@brief`, not `\brief`.
- **`@brief` is one short sentence.** More detail, when needed, goes into a paragraph of its own.
- **Parameters are documented in the order of the signature**, with their direction: `@param[in]`, `@param[out]`,
  `@param[in,out]`. A plain value parameter is `@param[in]`. The names and the descriptions of the `@param` lines line
  up.
- **A function that returns a value documents it** with `@return`, or `@retval` for each value that has a meaning of
  its own. A `void` function has neither.
- **Every sentence of the documentation ends with a full stop.**
- **`@note` is information without risk, `@remark` a behavior that matters, `@warning` a serious risk.**
- **`@pre`, `@post`, `@see` and `@todo` may be used where they help; a deprecated interface has `@deprecated`.**
- **Code in the documentation**: a C block over several lines should be `@code{.c}` ... `@endcode`; a short piece may be
  `@c`.
- **Not used**: `@since`, `@version`, `@author`, `@file`, `@ingroup`, `@details`.

## 10.3 The order of a comment

The parts come in this order; a part that is not needed is left out:

1. `@brief`
2. the paragraph with the details
3. `@param`
4. `@return` or `@retval`
5. `@pre` and `@post`
6. `@note`, `@remark` and `@warning`
7. `@todo`
8. `@deprecated`
9. `@see`

`@param` and `@return` lines stand together, with no blank line between them; other parts may be separated by an empty
` *` line.

```c
/**
 * @brief Copies values into a destination buffer.
 *
 * @param[out] aOut_pDestination Destination array.
 * @param[in]  aIn_pSource       Source array.
 * @param[in]  aCount            Number of elements.
 * @return true on success, false otherwise.
 *
 * @pre The source and the destination do not overlap.
 * @see Example_List_Append()
 */
bool
Example_CopyValues(int* aOut_pDestination, const int* aIn_pSource, size_t aCount);
```
