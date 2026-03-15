# Introduction to NextStd

Welcome to the official documentation for **NextStd**—the safer,
modern alternative to the traditional C standard library.

For decades, C developers have relied on `<stdio.h>`, `<string.h>`,
and `<stdlib.h>`. While incredibly fast, these legacy libraries are
fundamentally unsafe. A single mismatched `%d` in a `printf`, a missing
`\0` in a string, or a silent `NULL` pointer can trigger catastrophic
Segmentation Faults, buffer overflows, and security vulnerabilities.

NextStd fixes this at the foundation.

By combining the elegant simplicity of C11 macros on the
frontend with the mathematically proven memory safety of Rust
on the backend, NextStd delivers a zero-compromise development experience.

## Core Pillars of NextStd

* **Type-Safe I/O:** Never write a format specifier again. NextStd uses C11
`_Generic` macros to automatically route data types at compile time.
* **Crash-Proof Control Flow:** Replace silent failures and dangerous `goto`
statements with Python-style `NS_TRY` and `NS_EXCEPT` macros.
* **Immunity to NULL:** Every FFI boundary rigorously checks for `NULL`
pointers. Passing bad memory to NextStd gracefully returns an error
code instead of killing your program.
* **Modern String Memory:** Strings are powered by Small String Optimization
(SSO), meaning short text costs zero heap allocations, and massive text safely
catches Out-Of-Memory (OOM) errors without panicking.

### Who is this for?

NextStd is built for C developers, systems programmers, and embedded engineers
who want the safety guarantees of a modern language without actually having to
rewrite their entire codebase in Rust. It compiles down to a standard static or
dynamic C library (`.a`, `.so`, `.dylib`, or `.dll`) and links identically t
o any standard C dependency.

