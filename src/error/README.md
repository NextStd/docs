# Error Handling (`ns_error`)

In standard C, error handling is historically fragmented and highly error-prone.
Some functions return `-1`, others return `NULL`, some set `errno` as a global
state, and others simply crash the program if given invalid input.

This inconsistency makes it incredibly easy to forget to check an error
condition, leading to undefined behavior, silent failures, or segmentation
faults.

`NextStd` takes a modern approach. It unifies all error reporting into a single,
predictable type: `ns_error_t`.

Every `NextStd` operation that allocates memory, parses input, or interacts with
the Rust backend securely returns an `ns_error_t` code instead of returning
naked pointers or relying on global error states.

## What's in this chapter?

This chapter covers how NextStd handles errors safely and elegantly, allowing
you to write robust C code without the usual boilerplate:

- **[The Try/Except Macros](try-except.md):** Learn how to use `NS_TRY` and
  `NS_EXCEPT` to write clean, structured error handling that visually separates
  your "happy path" from your failure logic.
- **[Null Pointer Protection](null-safety.md):** Discover how the Rust FFI
  boundary safely intercepts and neutralizes `NULL` pointers before they can
  cause a system-level segfault.
- **[Error Codes Reference](codes.md):** A complete reference guide for all
  `ns_error_t` values (such as `NS_SUCCESS` and `NS_ERROR_ANY`) and how to
  handle them.

By enforcing a standard error type and structured control flow across the entire
library, NextStd ensures that your programs stay stable, predictable, and
memory-safe.
