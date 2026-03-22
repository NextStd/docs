# Command Execution (`ns_cmd`)

Executing shell commands in standard C usually involves reaching for the
`system()` function or wrestling with `popen()`.

Both approaches are fraught with legacy issues. `system()` offers absolutely no
way to capture the output of the command (it just prints directly to the
terminal), and `popen()` requires manually managing streams, allocating buffers,
and parsing raw bytes—often leading to memory leaks or buffer overflows.

The **`ns_cmd`** module completely replaces these legacy functions with a
modern, memory-safe, and highly ergonomic wrapper around Rust's
`std::process::Command`.

## The NextStd Solution

With `ns_cmd`, executing a command and capturing its exact output is safe and
synchronous.

* **Complete Stream Capture:** Both `stdout` and `stderr` are automatically
  captured and placed into NextStd's blazingly fast
  [Small String Optimized (`ns_string`)](../string/README.md) structs.
* **Exit Codes:** The exact exit code of the shell process is captured and
  returned.
* **Smart Routing:** Pass either a standard C string literal (`"uname -a"`) or a
  dynamically built `ns_string`. The macros figure it out automatically.
* **Zero Memory Leaks:** Thanks to the `ns_autocmd` macro, process output
  structs automatically free their own internal string buffers the moment they
  go out of scope.

## Quick Glance

Here is how simple it is to execute a command, read its output, and clean up the
memory—all in four lines of C:

```c
#include <nextstd/ns.h>
#include <nextstd/ns_cmd.h>

int main(void) {
    // 1. Initialize the output struct with the auto-cleanup macro
    ns_autocmd ns_cmd_output out = {0}; 

    // 2. Run the command safely
    ns_cmd_run("uname -a", &out);

    // 3. Use the captured data!
    ns_println("Status: {}", out.exit_code);
    ns_println("Output: {}", out.stdout_data);

    // 4. No free() required. Memory cleans itself up here!
    return 0;
}
```

## Deep Dive

Explore the specific features of the `ns_cmd` module:

* [Running Shell Commands](execution.md)
* [Capturing Stdout & Stderr](output-capturing.md)
* [Auto-Cleanup (RAII in C)](auto-cleanup.md)
