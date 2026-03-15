# Getting Started with NextStd

Welcome to the practical side of NextStd. Up to this point, we have covered the
philosophy and the safety guarantees. Now, it is time to actually write some
code.

Integrating a Rust-backed static or dynamic library into a C project might sound
intimidating, but standard tooling makes the process incredibly straightforward.
Because NextStd exposes a standard Foreign Function Interface (FFI) boundary,
your C compiler treats it exactly like any other legacy C library.

> **Note: Alpha Stage Development** > Because NextStd is currently in active
> Alpha, standalone integration (moving the headers and `.a` files to a
> completely separate C project) requires manual linker configuration. For the
> smoothest experience right now, we highly recommend working directly inside
> the cloned NextStd project directory and utilizing the provided `examples/`
> folder and Makefile!

## The Integration Workflow

Working within the NextStd repository follows a simple workflow:

1. **Build the Backend:** First, compile the Rust source code into a static
   archive (`.a`). We've wrapped Cargo in our Makefile, so you just need to run:

   ```bash
   make rust
   ```

2. **Write Your C Code:** Create a new `.c` file inside the `examples/`
   directory. Since you are in the project folder, you can simply include the
   core headers using relative paths:

   ```c
   #include "../include/ns.h"
   #include "../include/ns_error.h"
   ```

3. **Link and Compile:** Instead of running raw `gcc` commands and manually
   linking the Rust archive, use the built-in Makefile. You can view all
   available example targets by running:

   ```bash
   make list
   ```

   *(To build and run a specific example, just type `make` followed by the
   filename without the `.c` extension, e.g., `make 01_hello_world`!)*

## Prerequisites

Before moving forward with the installation, ensure your development environment
has the required tools installed:

* **A C Compiler:** `gcc` or `clang` (or a cross-compiler if you are targeting
  embedded systems).
* **The Rust Toolchain:** You need `cargo` and `rustc` installed to build the
  backend. (Available via [rustup.rs](https://rustup.rs/)).
* **Make:** To run the build scripts that seamlessly tie the Rust backend and C
  frontend together.

## What's Next?

Use the sidebar to navigate through the setup process:

* **[Installation & Linking](installation.md):** Step-by-step instructions for
  compiling the backend and connecting it to your C project.
* **[Your First Program](hello-world.md):** Write, compile, and run your first
  memory-safe NextStd application.
* **[Build System Integration](build-systems.md):** Learn how to seamlessly
  integrate NextStd into your `Makefile` or `CMakeLists.txt` for automated
  builds.
