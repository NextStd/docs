# Null Pointer Protection

In standard C, passing a `NULL` pointer into a standard library function (like
`strcpy` or `strlen`) is a fatal mistake. The library will blindly attempt to
read or write to memory address `0x0`, resulting in an immediate
**Segmentation Fault** and a hard crash of your entire application.

`NextStd` takes a drastically different approach to memory safety by leveraging
its Rust backend as a protective shield.

## The FFI Shield

Because NextStd's core logic is executed in Rust, every pointer passed from your
C code must cross the **Foreign Function Interface (FFI)** boundary.

Before NextStd ever attempts to read or write to a pointer you provide, the Rust
backend actively verifies that the pointer is not null (using Rust's
`.is_null()` checks).

If a `NULL` pointer is detected, the Rust backend immediately halts the
operation and returns a safe `ns_error_t` code (typically `NS_ERROR_ANY` or a
specific null-pointer error code) back to C.

## Graceful Failure in Action

Because NextStd intercepts the `NULL` pointer and translates it into an error
code, your application continues running safely. You can then use your `NS_TRY`
and `NS_EXCEPT` macros to handle the mistake gracefully.

Here is an example of attempting to initialize a string, but accidentally
passing `NULL` instead of a valid memory address:

```c
#include <nextstd/ns.h>
#include <stddef.h> // For NULL

int main() {
    ns_error_t err;

    // We maliciously pass NULL instead of a valid &ns_string pointer
    NS_TRY(err, ns_string_new(NULL, "This would normally crash!")) {

        ns_println("This block will never execute.");

    } 
    NS_EXCEPT(err, NS_ERROR_ANY) {

        // NextStd caught the NULL pointer and routed us here safely!
        ns_println("Crisis averted! NextStd refused to dereference the NULL pointer.");

    }

    return 0;
}
```

## Why This Matters

By neutralizing `NULL` pointers at the library boundary, NextStd drastically
reduces the debugging time spent hunting down mysterious segfaults. It
transforms catastrophic system crashes into manageable, predictable error states
that you can catch and log.
