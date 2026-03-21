# Contributing to NextStd

Welcome, and thank you for your interest in contributing to `NextStd`!

Standard C is foundational to modern computing, but its lack of memory safety
continues to be one of the largest sources of vulnerabilities in software today.
By contributing to this project, you are helping bridge the gap between C's
unmatched portability and Rust's airtight memory guarantees.

Because `NextStd` is a hybrid project—combining C11 `_Generic` macros on the
front-end with an optimized Rust Foreign Function Interface (FFI) on the
back-end—working on it is a unique and highly rewarding engineering challenge.

## What's in this section?

Whether you want to fix a bug, optimize an existing FFI boundary, or build an
entirely new memory-safe data structure, these guides will get you up to speed
on how the codebase operates:

* **[Architecture Overview](architecture.md):** Understand the data flow between
  the C headers, the `_Generic` router macros, and the Rust `unsafe extern "C"`
  backend.
* **[Adding New Modules](new-modules.md):** A step-by-step guide on how to
  conceptualize, write, and safely expose a new Rust-backed feature to the C
  front-end.

## Prerequisites for Developing

To compile the codebase and run the test suites locally, you will need:

* A standard C compiler (`gcc` or `clang`)
* The Rust toolchain (`cargo` and `rustc`)
* `make` (for executing the build and linking scripts)

We are thrilled to have you on board. Let's make C safer, together!
