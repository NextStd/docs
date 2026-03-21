# Safe String Manipulation

In standard C, combining strings is one of the most dangerous and tedious
operations you can perform.

Because standard C strings are just primitive arrays of characters ending in a
null byte (`\0`), they have no concept of their own capacity. If you want to
combine two strings using `strcat()`, you must first manually calculate the
length of both strings, allocate a completely new buffer large enough to hold
them both, and carefully check for off-by-one errors. Forgetting just one of
these steps results in a buffer overflow that will silently overwrite adjacent
memory.

`NextStd` eliminates this entirely.

## The NextStd Solution: Safe Concatenation

Because every `ns_string` is a managed struct backed by Rust, it tracks its own
length, capacity, and allocation state (SSO vs. Heap).

When you want to combine two strings, you simply pass them to `ns_string_concat`
wrapped in an `NS_TRY` block. The Rust backend instantly calculates the required
capacity for the combined string, safely allocates a new block of memory (or
uses Small String Optimization if the combined length is short enough), and
writes the result into your destination struct—all behind the scenes.

## The Complete Example

Here is a practical example demonstrating how to safely combine two `ns_string`
objects. Notice how there are zero `malloc()`, `realloc()`, or `strlen()` calls!

```c
#include <nextstd/ns.h>
#include <nextstd/ns_string.h>

int main(void) 
{
    ns_error_t err;
    ns_string greeting;
    ns_string name;
    ns_string final_message;

    // 1. Initialize our starting strings
    NS_TRY(err, ns_string_new(&greeting, "Hello, ")) {
    } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    NS_TRY(err, ns_string_new(&name, "World!")) {
    } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    // 2. Safely concatenate them into final_message
    NS_TRY(err, ns_string_concat(&final_message, greeting, name)) {
    } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    // 3. Print the safely resized, concatenated string!
    ns_println(final_message);

    ns_print("Final String Length: ");
    ns_println(final_message.len);

    // 4. Always clean up all initialized strings
    ns_string_free(&greeting);
    ns_string_free(&name);
    ns_string_free(&final_message);

    return 0;
}
```

### Why this is completely safe

* **No Overflows:** The backend automatically calculates the exact memory
  required for `final_message`. If the combined string is over 24 bytes, it
  smoothly and invisibly transitions to a heap allocation.
* **Error Handling:** If the system is completely out of memory and cannot
  allocate the combined buffer, the `NS_TRY` block will gracefully catch the
  `NS_ERROR_ALLOCATION_FAILED` error instead of crashing the program with a
  segmentation fault.
* **Clean Deallocation:** Calling `ns_string_free` at the end drops the memory
  safely for all three structs.
