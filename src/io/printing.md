# Safe Printing (`ns_print`)

In standard C, the `printf` function is inherently unsafe because it parses
format strings (like `%d` or `%s`) at runtime. If the format string doesn't
perfectly match the variables you pass to it, the compiler might not catch it,
leading to garbled output or severe security vulnerabilities.

`NextStd` replaces `printf` with two type-safe macros:

- `ns_print(val)`: Prints the value to the standard output without a newline.
- `ns_println(val)`: Prints the value and automatically appends a newline
  (`\n`).

## Basic Usage

You never have to memorize format specifiers again. Simply pass your variable or
literal directly into the macro, and NextStd will figure out how to print it.

```c
#include <nextstd/ns.h>
#include <stdbool.h>

int main() {
    int age = 42;
    double pi = 3.14159;
    bool is_active = true;

    // Printing strings directly
    ns_println("--- User Profile ---");

    // Printing integers
    ns_print("Age: ");
    ns_println(age);

    // Printing floating-point numbers
    ns_print("Pi approximation: ");
    ns_println(pi);

    // Printing booleans (automatically formats as "true" or "false")
    ns_print("Account Active: ");
    ns_println(is_active);

    return 0;
}
```

## How It Works Under the Hood

If C doesn't support function overloading like C++ or Rust, how does
`ns_println` know what type of data you are passing to it?

NextStd leverages a feature introduced in the C11 standard called `_Generic`
selections. Think of it as a `switch` statement that evaluates data types at
compile time rather than values at runtime.

When you call `ns_print(val)`, the C preprocessor expands it into something like
this:

```c
// Simplified representation of the NextStd printing macro
#define ns_print(X) _Generic((X), \
    int: ns_print_int, \
    double: ns_print_double, \
    bool: ns_print_bool, \
    char*: ns_print_str, \
    const char*: ns_print_str \
)(X)
```

### The Safety Guarantee

Because this evaluation happens strictly at compile time:

1. **Zero Runtime Overhead:** There is no format string to parse while your
   program is running. It is as fast as calling the specific print function
   directly.
2. **Compile-Time Errors:** If you try to pass a data type that NextStd doesn't
   know how to print yet (like a raw `struct` or a multidimensional array), the
   compilation will fail immediately. It is impossible to accidentally trigger
   undefined behavior.
