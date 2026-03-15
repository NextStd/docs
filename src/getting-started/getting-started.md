# Getting Started with NextStd

Welcome to the practical side of NextStd. Up to this point, we have covered the
philosophy and the safety guarantees. Now, it is time to actually write some
code.

Integrating a Rust-backed static or dynamic library into a C project might sound
intimidating, but standard tooling makes the process incredibly straightforward.
Because NextStd exposes a standard Foreign Function Interface (FFI) boundary,
your C compiler treats it exactly like any other legacy C library.

## The 3-Step Integration Workflow

Using NextStd in any project always follows the same three basic steps:

1. **Build the Backend:** Use Rust's package manager (`cargo`) to compile the
   NextStd source code into a static archive (`.a`) or dynamic shared object
   (`.so` / `.dylib` / `.dll`).
2. **Include the Headers:** Drop the `include/` directory containing `ns.h`,
   `ns_error.h`, and `ns_string.h` into your C project so your compiler knows
   the function signatures and macros.
3. **Link and Compile:** Run `gcc` or `clang` on your C files, passing the
   NextStd library file to the linker.

## Prerequisites

Before moving forward with the installation, ensure your development environment
has the required tools installed:

* **A C Compiler:** `gcc` or `clang` (or a cross-compiler if you are targeting
  embedded systems).
* **The Rust Toolchain:** You need `cargo` and `rustc` installed to build the
  backend. (Available via [rustup.rs](https://rustup.rs/)).
* **A Build System (Recommended):** While you can compile by typing raw `gcc`
  commands into the terminal, we highly recommend using `Make` or `CMake` to
  automate the compilation of both the Rust backend and the C frontend
  simultaneously.

## What's Next?

Use the sidebar to navigate through the setup process:

* **[Installation & Linking](installation.md):** Step-by-step instructions for
  compiling the backend and connecting it to your C project.
* **[Your First Program](hello-world.md):** Write, compile, and run your first
  memory-safe NextStd application.
* **[Build System Integration](build-systems.md):** Learn how to seamlessly
  integrate NextStd into your `Makefile` or `CMakeLists.txt` for automated
  builds.
