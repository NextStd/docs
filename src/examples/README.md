# Practical Examples

In the previous chapters, we explored the individual building blocks of
`NextStd`: handling errors gracefully, printing to the terminal, managing
dynamic memory, and safely storing data.

However, the true power of this library becomes apparent when you combine these
modules. When `ns_io`, `ns_error`, and `ns_string` work together, they transform
standard C from a language of constant manual memory vigilance into a modern,
safe, and highly productive tool.

## What's in this chapter?

This section provides complete, copy-pasteable programs that demonstrate how to
solve common C programming challenges without writing a single `malloc`, `free`,
or dangerous buffer manipulation.

- **[Safe User Input](safe-input.md):** Learn how to capture dynamic terminal
  input of any length safely, completely eliminating the buffer overflow
  vulnerabilities of `scanf` and `gets`.
- **[Building a CLI Menu](cli-menu.md):** Combine colored printing, safe integer
  reading, and error handling to create a robust interactive terminal
  application.
- **[String Manipulation](string-manipulation.md):** See how to safely build and
  combine dynamic strings without worrying about capacity calculations or
  `strcat` overwriting adjacent memory.

Whether you are building a small command-line utility or a larger system
application, these examples serve as a blueprint for writing memory-safe C code
with `NextStd`.
