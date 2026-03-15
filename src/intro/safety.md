# Safety Guarantees

When we say NextStd is "safe," we aren't just talking about compiler warnings or
best practices. We are talking about architectural guarantees enforced at the
Foreign Function Interface (FFI) boundary between C and Rust.

Here are the core vulnerability classes that NextStd entirely eliminates from
your C codebase.

## 1. Null Pointer Immunity (No More Segfaults)

In standard C, passing a `NULL` pointer to `strlen()` or `strcpy()` results in
immediate Undefined Behavior, typically manifesting as a fatal Segmentation
Fault.

NextStd actively intercepts bad memory. Every single FFI boundary function in
the library checks for `NULL` before performing any operations. If a `NULL`
pointer is detected, the operation is safely aborted, and an `NS_ERROR_ANY` code
is returned for your `NS_TRY` block to catch gracefully.

## 2. Out-of-Memory (OOM) Protection

Standard C's `malloc` returns `NULL` when the heap is exhausted—a condition
developers frequently forget to check. Standard Rust, on the other hand, will
panic and instantly crash the program when an allocation fails.

NextStd takes the safe middle path. It uses Rust's `try_reserve` API for all
dynamic heap allocations. If the system runs out of memory (a critical threat in
embedded and constrained environments), NextStd safely catches the failure and
returns an `NS_ERROR_STRING_ALLOC` code without crashing your process.

## 3. Buffer Overflow Prevention

C strings are notoriously dangerous null-terminated (`\0`) arrays. If you forget
the terminator, or use `strcat` into a buffer that is even one byte too small,
you silently overwrite adjacent memory.

NextStd uses a length-prefixed struct combined with Small String Optimization
(SSO). The exact length of the string is always known. If a string needs to grow
beyond its 24-byte inline capacity, NextStd automatically and safely requests
exactly the right amount of heap memory. Buffer overflows are mathematically
impossible by design.

## 4. Format String Vulnerabilities

Using `printf("%d", "text")` forces the C compiler to trust the developer
blindly. If the format specifier doesn't match the variable type, the program
reads garbage memory from the stack or crashes entirely.

NextStd leverages C11 `_Generic` macros to completely eliminate this class of
bugs by routing types at compile time:

```c
int age = 21;
ns_string name; // Assume initialized

ns_println(age);  // Automatically routes to ns_print_int
ns_println(name); // Automatically routes to ns_print_string
```

Because there are no format strings, format string vulnerabilities simply cannot
exist.
