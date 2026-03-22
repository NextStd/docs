# Running Shell Commands

The core of the `ns_cmd` module is the `ns_cmd_run` macro. Unlike standard C
functions that force you to strictly cast your types or use entirely different
function names for different inputs (like `print_int` vs `print_string`),
NextStd uses modern C11 features to do the heavy lifting for you.

## The `_Generic` Routing Magic

In standard C, if an execution function expected a dynamic string struct, you
would be forced to convert your simple string literals into structs every single
time you wanted to run a basic command.

NextStd solves this by turning `ns_cmd_run` into a `_Generic` macro wrapper.

When you call `ns_cmd_run`, the C compiler inspects the type of the argument you
passed and automatically routes it to the correct Rust FFI backend function:

* If you pass a `"string literal"`, it routes to `ns_cmd_run_cstr`.
* If you pass an `ns_string` struct, it routes to `ns_cmd_run_ns`.

### 1. Passing Standard C Strings

For 90% of your use cases, you will likely just pass a hardcoded string literal.
The macro accepts it natively:

```c
#include <nextstd/ns.h>
#include <nextstd/ns_cmd.h>

int main(void) {
    ns_autocmd ns_cmd_output out = {0}; 

    // The compiler sees a const char* and routes it automatically!
    ns_cmd_run("ls -la /var/log", &out);

    return 0;
}
```

### 2. Passing Dynamic `ns_string` Structs

If you are building a command dynamically based on user input or file parsing,
you can pass your NextStd `ns_string` directly into the exact same macro. No
unwrapping or `.ptr` access required:

```c
#include <nextstd/ns.h>
#include <nextstd/ns_cmd.h>
#include <nextstd/ns_string.h>

int main(void) {
    ns_autocmd ns_cmd_output out = {0}; 
    ns_string my_dynamic_cmd;

    // Safely build the command
    ns_string_new(&my_dynamic_cmd, "echo 'NextStd is awesome!'");

    // Pass the struct directly!
    ns_cmd_run(my_dynamic_cmd, &out);

    // Clean up the input string
    ns_string_free(&my_dynamic_cmd);

    return 0;
}
```

## Under the Hood: `sh -c`

When the Rust backend receives your command (regardless of how it was routed),
it securely spawns a new process using `sh -c "<your_command>"`.

This mimics the exact behavior of C's `system()`, meaning you have full access
to standard shell features like piping (`|`), redirecting (`>`), and environment
variable expansion (`$USER`), while still running entirely within Rust's
memory-safe execution sandbox.
