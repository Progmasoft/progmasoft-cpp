<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 5. Control flow

## 5.1 Conditions

- **A Boolean is tested directly**: `if(isReady)`, `if(!isReady)`; never `== true` or `== false`.
- **A pointer is compared with `NULL`**: `pData == NULL`, `pData != NULL`. The null pointer constant is `NULL`, never
  `0`.
- **A condition assigns nothing**: no `=`, `+=` or other assignment inside it.
- **The variable stands left of the constant**: `count == 0`, not `0 == count`.
- **A ternary is never nested.** A simple one may be used; when it does not fit on one line, `?` and `:` start their
  lines.

## 5.2 Returning

- **A function may return from several places.**
- **A returned value is written in parentheses**: `return (expression);`.
- **A `void` function may return early with `return;`**, but should not end with one.

## 5.3 Loops

- **A `for` loop should step with postfix `index++` or `index--`.** Elsewhere, choose prefix or postfix by meaning. An
  increment inside a larger expression should be its own statement.
- **An empty loop body is `{ }`**, never a bare `;`.
- **The empty parts of a `for` stay visible**: `for(; isRunning;)`, `for(;;)`.

## 5.4 `switch`

- **Every `switch` has a `default`, and it comes last.**
- **`case` and `default` start in the column of the `switch`.**
- **`case` labels that share a body may follow each other**, each on its own line.
- **No blank line follows a `case` label; one blank line separates two `case` groups.**
- **A `case` may have its own block**, `{ }`, when it needs local variables.
- **A deliberate fall-through is marked `/* Fall through. */`.** Adjacent labels that share a body need no mark.

```c
switch(state)
{
case kPending:
case kWaiting:
        ProcessPending();
        break;

case kRunning:
        ProcessEvents();
        break;

default:
        break;
}
```

## 5.5 `goto`

- **`goto` jumps only to the point that handles an error or releases resources.**
- **A label is `gotoPascalCase`**, starts in the first column, and has one blank line before it and none after it.

```c
int
FillBuffer(size_t aSize)
{
        int result = -1;

        int* pBuffer = malloc(aSize);

        if(pBuffer == NULL)
                goto gotoFailed;

        result = Fill(pBuffer, aSize);

gotoFailed:
        free(pBuffer);
        return (result);
}
```
