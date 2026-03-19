# Installation & Linking

NextStd is designed to be installed globally on your Unix-like system, allowing
you to include its memory-safe data structures and I/O functions in any C
project just like the standard library.

Here is how to get your environment set up in under two minutes.

## Prerequisites

Before installing, ensure your system has the required build tools:

1. **Rust & Cargo**: To compile the safe backend.

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. **GCC or Clang**: For the C front-end and macro expansion.
3. **Make**: For build automation.

## Step 1: Clone the Repository

Grab the latest version of the NextStd source code from GitHub:

```bash
git clone https://github.com/NextStd/NextStd.git
cd NextStd
```

## Step 2: System-Wide Installation (Recommended)

To make NextStd available to all your C projects, run the install command. This
builds the highly optimized Rust release binaries and copies them, along with
the C headers, into your system's standard `/usr/local` directories.

```bash
sudo make install
```

**What is happening behind the scenes?**

* The C headers (`ns.h`, `ns_data.h`, etc.) are securely placed in
  `/usr/local/include/nextstd/`.
* The compiled Rust static archives (`libns_io.a`, `libns_data.a`, etc.) are
  placed in `/usr/local/lib/`.

*(To completely remove the library from your system later, simply run `sudo make
uninstall` from the repository root).*

## Step 3: Linking in Your Projects

Now that NextStd is installed globally, you can include it in any standalone C
file across your machine using standard angle brackets:

```c
#include <nextstd/ns.h>
#include <nextstd/ns_data.h>
```

When compiling your standalone program, you need to tell GCC to link the
specific NextStd modules you are using, as well as the standard system libraries
that the Rust standard library depends on.

Because the library is in `/usr/local/lib`, you no longer need complex `-L`
paths. A typical compilation command looks like this:

```bash
gcc main.c -lns_data -lns_io -lns_string -lns_error -lpthread -ldl -lm -o my_safe_program
```

* **`-lns_*`**: Links your modular NextStd archives.
* **`-lpthread`, `-ldl`, `-lm`**: Standard system libraries required by the Rust
  backend on standard Linux/macOS environments.

## Alternative: Local Testing

If you want to test the library, develop new modules, or run the built-in
examples without installing it system-wide, you can build the backend locally:

```bash
make rust
```

Then, run one of the pre-configured examples (do not include the `.c`
extension):

```bash
make 01_print_integer
```

This commands your compiler to grab the example, link it against your local
`target/release/` directory, and execute the resulting binary.
