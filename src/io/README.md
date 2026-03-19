# Input & Output (`ns_io`)

In standard C, interacting with the terminal is a minefield.

Functions like `printf` and `scanf` rely on format strings (e.g., `%d`, `%s`,
`%f`). If you accidentally mismatch the format specifier with the actual
variable type, the compiler often won't stop you. At best, you print garbage
memory to the screen. At worst, you introduce severe security vulnerabilities
(like Format String Attacks) or trigger buffer overflows that crash your entire
system.

Furthermore, `scanf` is notorious for leaving dangling newlines in the input
buffer, causing subsequent reads to randomly skip or fail.

## The NextStd Approach

`NextStd` completely eliminates format strings from your codebase.

By leveraging C11's powerful `_Generic` keyword, the `ns_io` module inspects
your variables at compile time. It automatically routes your data to the
correct, memory-safe Rust backend function.

If you try to print or read a data type that the library doesn't support, your
code simply won't compile—catching the error before the program ever runs.

## What's in this chapter?

This chapter covers how to safely interact with the terminal without ever typing
a `%` symbol again:

- **[Safe Printing (`ns_print`)](printing.md):** Learn how to output text and
  variables to the console seamlessly.
- **[Safe Reading (`ns_read`)](reading.md):** Discover how to take user input
  safely, without worrying about buffer overflows, invalid data types, or
  leftover newlines.
- **[Terminal Colors](colors.md):** Make your CLI applications beautiful with
  cross-platform, macro-driven terminal colors and styles.

With `ns_io`, you get the convenience and safety of a high-level language's
`print()` and `input()` functions, running at the blistering speed of native C.
