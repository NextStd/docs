# Auto-Cleanup (RAII in C)

One of the biggest pain points in standard C is manual memory management. If a
library dynamically allocates memory to capture a string or a file, the
developer is strictly responsible for calling `free()` when they are done.

If you forget, your program leaks memory. If you call it twice, your program
crashes.

Languages like C++ and Rust solved this decades ago using **RAII** (Resource
Acquisition Is Initialization) and destructors. When a variable goes out of
scope (reaches the closing brace `}` of its block), the compiler automatically
inserts the cleanup code.

With NextStd, you get that exact same magic in C.

## The `ns_autocmd` Macro

NextStd leverages a powerful, widely-supported GCC/Clang compiler extension
called `__attribute__((cleanup))`. We wrap this extension in a simple, ergonomic
macro called `ns_autocmd`.

When you prefix your `ns_cmd_output` struct with `ns_autocmd`, you are attaching
a hidden destructor to the variable. The moment that variable falls out of
scope, the compiler automatically intercepts it and safely frees any internal
`ns_string` heap allocations.

```c
#include <nextstd/ns.h>
#include <nextstd/ns_cmd.h>

void run_status_check() {
    // 1. Declare the struct with the auto-cleanup macro
    ns_autocmd ns_cmd_output out = {0}; 

    // 2. Run the command
    ns_cmd_run("uptime", &out);

    // 3. Print the result
    ns_println("System Uptime: {}", out.stdout_data);

    // 4. End of scope. The compiler automatically calls 
    // ns_cmd_output_free(&out) right here! No memory leaks!
}
```

## ⚠️ The Golden Rule: Zero-Initialization

There is one critical rule when using `ns_autocmd`:
**You must always zero-initialize your struct (`= {0};`).**

```c
// ❌ DANGEROUS: Contains random stack garbage memory
ns_autocmd ns_cmd_output bad_out; 

// ✅ SAFE: Memory is explicitly zeroed out
ns_autocmd ns_cmd_output good_out = {0}; 
```

**Why is this required?** In C, uninitialized stack variables contain random
"garbage" data left over in RAM.

If `ns_cmd_run` were to fail *before* it could successfully populate your struct
(for example, if the system couldn't spawn a shell), the destructor would
eventually run and try to `free()` that random garbage memory. This results in a
guaranteed segmentation fault.

By zero-initializing the struct (`= {0};`), you guarantee that if the command
fails early, the destructor simply sees `NULL` pointers and lengths of `0`, and
safely does nothing.

## Manual Cleanup

If you are compiling on a highly restrictive embedded compiler that does not
support GCC/Clang extensions, or if you simply prefer manual memory management,
you can still declare the struct normally and use the manual free function:

```c
ns_cmd_output out = {0};
ns_cmd_run("ls", &out);
ns_println("Output: {}", out.stdout_data);

// Manually free the internal strings
ns_cmd_output_free(&out);
```
