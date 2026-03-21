# Installation & Linking

NextStd is designed to be installed globally on your Unix-like system, allowing
you to include its memory-safe data structures and I/O functions in any C
project just like the standard library.

Here is how to get your environment set up in under two minutes.

## Prerequisites

Before installing, ensure your system has the required build tools:

1. **Rust & Cargo**: To compile the safe back end. Run the official installer in
   your terminal:

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

   *👉 **Note for beginners:** The script will pause. Press `1` and hit `Enter`
   to proceed with the default installation.*

   > [!IMPORTANT] > Once the installation finishes, you must tell your current
   terminal where Rust is installed. Run this command:

   ```bash
   source $HOME/.cargo/env
   ```

   *(Alternatively, you can just close your terminal and open a new one).*
2. **GCC or Clang**: For the C front-end and macro expansion.
3. **Make**: For build automation.

---

## Step 1: Clone the Repository

Grab the latest version of the NextStd source code from GitHub:

```bash
git clone https://github.com/NextStd/nextstd.git
cd nextstd
```

## Step 2: System-Wide Installation (Recommended)

To make NextStd available to all your C projects, you need to build the
optimized Rust binaries and install them globally.

First, build the core libraries (this runs safely as your normal user):

```bash
make rust
```

Then, install the headers and static archives into your system's standard
`/usr/local` directories (this requires root privileges to copy the files):

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

---

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

---

## Alternative: Local Testing

If you want to test the library, develop new modules, or run the built-in
examples without installing it system-wide, you can build the backend locally
without `sudo`:

```bash
make rust
```

Then, run one of the pre-configured examples (do not include the `.c`
extension):

To list all the examples use:

```bash
make list
```

And then you can use the example names as given below.

> [!NOTE]
> Note the absence of the `.c` extension

```bash
make 01_print_integer
```

This commands your compiler to grab the example, link it against your local
`target/release/` directory, and execute the resulting binary.
