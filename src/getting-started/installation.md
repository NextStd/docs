# Installation & Linking

Since NextStd is currently in active Alpha, the easiest and most reliable way to
use the library is to clone the repository directly and work within its provided
directory structure. Our build system handles the complexities of cross-language
linking for you.

Here is how to get your environment set up in under two minutes.

## Step 1: Clone the Repository

First, grab the latest version of the NextStd source code from GitHub:

```bash
git clone https://github.com/NextStd/NextStd.git
cd NextStd
```

## Step 2: Build the Rust Backend

NextStd's core logic is written in Rust. Before you can compile any C code that
uses the library, you need to compile the Rust source into a static C library
archive (`.a`).

We have provided a unified `Makefile` at the root of the project to handle this.
Simply run:

```bash
make rust
```

**What is happening behind the scenes?** This command invokes Cargo (Rust's
package manager) to build the workspace in release mode. The Rust compiler
(`rustc`) takes all the modules in the `crates/` directory, heavily optimizes
them, and outputs a highly performant static archive file.

## Step 3: Verify the Installation

To ensure everything compiled correctly and your C compiler can successfully
link against the new Rust backend, let's run one of the pre-configured examples.

Type the following command into your terminal:

```bash
make 01_hello_world
```

*(Note: Do not include the `.c` extension when using the make command!)*

If everything is configured correctly, your C compiler will grab
`examples/01_hello_world.c`, link it against the Rust backend, and execute the
resulting binary. You should see a safely printed greeting in your terminal!

## Advanced: Standalone Linking (Manual)

While we highly recommend working inside the cloned repository during the Alpha
phase, you *can* use NextStd in an external, standalone project.

If you choose to do this, you must manually copy the `include/` directory and
the `libns.a` file to your project. Furthermore, because `libns.a` contains Rust
code, you must explicitly link the C system libraries that the Rust standard
library depends on.

A typical GCC compilation command for a standalone project looks like this:

```bash
gcc main.c -L/path/to/libns_directory -lns -lpthread -ldl -lm -o my_safe_program
```

* **`-L`**: Tells the compiler where to find the library.
* **`-lns`**: Links the NextStd archive.
* **`-lpthread`, `-ldl`, `-lm`**: Standard system libraries required by the Rust
  backend. *(Note: These specific flags are for Linux; EndeavourOS, Arch, or
  other distributions will use these exact flags, while macOS or Windows MinGW
  may require slightly different ones).*
