# String Concatenation (`ns_string_concat`)

In standard C, appending text to a string using `strcat` is one of the most
dangerous operations you can perform.

If the destination buffer doesn't have enough pre-allocated space to hold both
the original string and the new text, `strcat` will silently overwrite adjacent
memory. This leads to data corruption, mysterious crashes, and severe security
vulnerabilities.

`NextStd` eliminates this entirely with memory-safe concatenation.

## Safe Concatenation

With `NextStd`, you never have to manually calculate buffer sizes. Instead of
mutating an existing string, the `ns_string_concat` function takes two source
`ns_string` objects and safely writes the combined text into a brand new
destination `ns_string`.

Because combining two strings might require allocating memory on the heap (which
can technically fail if the system is out of memory), it returns an `ns_error_t`
that you should wrap in an `NS_TRY` block.

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_string s1, s2, result;

    // 1. Initialize the source strings
    NS_TRY(err, ns_string_new(&s1, "Hello, ")) {
    } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    NS_TRY(err, ns_string_new(&s2, "memory safety!")) {
    } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    // 2. Safely concatenate them into the 'result' string
    NS_TRY(err, ns_string_concat(&result, s1, s2)) {
        ns_println("Concatenation successful!");
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_print("Failed to concatenate: ");
        ns_println(ns_error_message(err));
        return 1;
    }

    // 3. Print the combined result
    ns_print("Final message: ");
    ns_println(result);

    // 4. Always clean up all strings!
    ns_string_free(&s1);
    ns_string_free(&s2);
    ns_string_free(&result);

    return 0;
}
```

## How It Works (The SSO Transition)

When you call `ns_string_concat`, the Rust backend performs a series of safety
checks to determine exactly how the new `result` string should be constructed:

1. **Capacity Calculation:** It adds the `len` of `s1` and `s2` together to find
   the exact required size.
2. **The SSO Decision:** If the combined length is under 24 bytes, the `result`
   string will utilize **Small String Optimization (SSO)** and live entirely on
   the stack. Zero heap allocations occur!
3. **Dynamic Reallocation:** If the combined length is 24 bytes or larger, the
   Rust backend safely allocates a new buffer on the heap, copies the data from
   both source strings into it, and flags the `result` struct as `is_heap`.

This entire memory management process happens invisibly. From the C side, you
just call `ns_string_concat`, and `NextStd` guarantees it succeeds safely or
returns a catchable error!
