# Capturing Stdout & Stderr

The fatal flaw of standard C's `system()` function is that it blindly dumps the
command's output directly to the user's terminal. If your C program actually
needs to *read* that output to parse a version number, check a status, or log an
error, `system()` is completely useless.

You are usually forced to use `popen()`, which requires manually allocating
buffers, reading chunks in a `while` loop, and praying you don't trigger a
buffer overflow.

NextStd completely eliminates this friction.

## The `ns_cmd_output` Struct

When you run a command using `ns_cmd_run`, the entire lifecycle of the process
is captured and packed into a single, clean C struct:

```c
typedef struct ns_cmd_output {
    ns_string stdout_data;
    ns_string stderr_data;
    int exit_code;
} ns_cmd_output;
```

### 1. The Output Streams (`stdout_data` & `stderr_data`)

Instead of raw `char*` arrays that you have to manually resize, NextStd uses the
underlying Rust `std::process::Command` engine to capture the raw bytes of the
streams and convert them directly into memory-safe `ns_string` structs.

**The SSO Advantage:** Because these streams are backed by NextStd's Small
String Optimization (SSO), if your command outputs less than 24 bytes (like
simply printing `"OK"` or an exit status string),
**zero heap allocation occurs.** The output is stored directly inside the struct
on the stack, making it incredibly fast.

### 2. The Exit Code

NextStd captures the exact integer status code returned by the shell. You no
longer have to mess with C's cryptic `WEXITSTATUS` macros just to find out if
your command succeeded. If the process exits cleanly, `exit_code` will be `0`.

## Example: Parsing Success vs. Failure

Because NextStd strictly separates standard output from standard error, your C
program can easily build branching logic based on whether a command failed.

```c
#include <nextstd/ns.h>
#include <nextstd/ns_cmd.h>

int main(void) {
    ns_autocmd ns_cmd_output out = {0};

    // Attempting to read a file that doesn't exist
    ns_cmd_run("cat /path/to/missing_file.txt", &out);

    if (out.exit_code != 0) {
        ns_println("Command failed with code: {}", out.exit_code);
        // We can safely read the exact error message the shell generated
        ns_println("Error Reason: {}", out.stderr_data);
    } else {
        ns_println("File Contents: {}", out.stdout_data);
    }

    return 0;
}
```

By default, these strings are dynamically allocated (if over 23 bytes) and
require cleanup. However, thanks to a powerful compiler extension, you rarely
ever need to free them manually.
