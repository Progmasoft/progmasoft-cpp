<!-- SPDX-FileCopyrightText: 2026 Progmasoft <support@progmasoft.com> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# 8. Memory and ownership

None of this is formatting, and no tool checks all of it: review does.

## 8.1 Ownership

- **Every dynamic object has a contract**: who owns it, when ownership moves, how long a borrowed pointer stays valid,
  and what state a failure leaves behind. The function's documentation states it.
- **The names say it too**: `Create` returns an object the caller owns and `Destroy` frees, `Init` and `Deinit` work on
  memory the caller already has ([Naming](NAMING.md#36-what-function-names-mean)).
- **Ordinary allocation failures should be reported to the caller.** `abort` and `exit` may end the program only in a
  documented case that cannot be recovered; a library does not end the process of its caller by surprise.
- **A wrapper allocator may be used**; without a need for one, the standard functions are called directly.

## 8.2 Allocating

- **`calloc` should be used when every bit must start at zero, and `malloc` otherwise.** All-bits-zero is not the zero
  value of every C type, such as a pointer or a floating-point number on every platform.
- **The size should come from the target**: `malloc(sizeof(*pValue))`, not `malloc(sizeof(Type))`, so that the size
  follows a change of the pointer's type.
- **The result of `malloc`, `calloc` and `realloc` is not cast.**
- **Nothing allocates zero bytes**: no `malloc(0)`, `calloc(0, size)`, `calloc(count, 0)` or `realloc(pBlock, 0)`.
  Their results differ between implementations. A request for zero elements is an empty result or an invalid argument,
  as the contract says.
- **A size computation cannot overflow**: a multiplication or an addition is checked against `SIZE_MAX` first.

## 8.3 Growing and shrinking

- **`realloc` writes to a temporary pointer, which is checked before it replaces the old one.** On failure the old
  block is still valid and still owned; writing the result straight back would lose it.
- **After a successful `realloc`, the old pointer and every pointer into the old block are invalid**, even when the
  address did not change.
- **New memory is initialized before it is read**, and not zeroed when it will be written first anyway.
- **A general-purpose dynamic array should grow geometrically.** A failed growth leaves the pointer, the capacity and
  the size as they were.
- **Removing elements should only lower the logical size.** Memory is given back when the caller asks; an array that
  shrinks by itself does so with hysteresis, so that adding and removing one element does not reallocate each time.
  What happens at zero elements is part of the contract.

```c
bool
Example_List_Append(Example_List_tList* aInOut_pList, int aValue)
{
        if(aInOut_pList == NULL)
                return (false);

        if(aInOut_pList->count == aInOut_pList->capacity)
        {
                size_t newCapacity;

                if(aInOut_pList->capacity == 0U)
                        newCapacity = 8U;
                else
                {
                        if(aInOut_pList->capacity > SIZE_MAX / 2U)
                                return (false);

                        newCapacity = aInOut_pList->capacity * 2U;
                }

                if(newCapacity > SIZE_MAX / sizeof(*aInOut_pList->pValues))
                        return (false);

                int* pTemporary = realloc(aInOut_pList->pValues, newCapacity * sizeof(*pTemporary));

                if(pTemporary == NULL)
                        return (false);

                aInOut_pList->pValues  = pTemporary;
                aInOut_pList->capacity = newCapacity;
        }

        aInOut_pList->pValues[ aInOut_pList->count ] = aValue;
        aInOut_pList->count++;

        return (true);
}
```

## 8.4 Freeing

- **`free(NULL)` does nothing, so no check is added only to protect a `free`.**
- **A pointer that stays in scope after its block is freed should be set to `NULL`** when it could be used again.

```c
void
Example_List_Deinit(Example_List_tList* aInOut_pList)
{
        if(aInOut_pList == NULL)
                return;

        free(aInOut_pList->pValues);
        aInOut_pList->pValues  = NULL;
        aInOut_pList->count    = 0U;
        aInOut_pList->capacity = 0U;
}
```

The check of `aInOut_pList` protects the member accesses, not the `free`.
