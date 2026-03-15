# Your First Program

Now that your environment is set up and the Rust backend is compiled, let's
write your first memory-safe C program using NextStd.

We are going to replace the traditional, unsafe `printf` function with NextStd's
type-safe `ns_println` macro.

## The Basic Program

If you are working inside the cloned repository, create a new file named
`hello.c` inside the `examples/` directory.

Add the following code to print basic text and variables:

```c
#include "../include/ns.h"

int main() {
    // 1. Printing a standard string
    ns_println("Hello, World! Welcome to NextStd.");

    // 2. Printing an integer safely without format specifiers
    int version = 1;
    ns_print("NextStd Version: ");
    ns_println(version); 

    // 3. Printing a floating-point number
    double pi = 3.14159;
    ns_print("Value of Pi: ");
    ns_println(pi);

    return 0;
}
```

To compile and execute this program, open your terminal in the root of the
`NextStd` repository and run:

```bash
make hello
```

## Adding Terminal Colors

NextStd also provides a dedicated, cross-platform color module to help you build
beautiful CLI tools. You don't need to remember ANSI escape codes; you just
include the header and use the macros.

Create a second file named `hello_color.c` in your `examples/` directory:

```c
#include "../include/ns.h"
#include "../include/ns_color.h" // Import the color macros

int main() {
    // Printing with a specific color and resetting it afterward
    ns_println(NS_COLOR_GREEN "Success: System initialized safely." NS_COLOR_RESET);

    // Combining styles like bold text with colors
    ns_println(NS_COLOR_CYAN NS_COLOR_BOLD "NextStd is running..." NS_COLOR_RESET);

    // Warning and Error colors
    ns_println(NS_COLOR_YELLOW "Warning: Low memory." NS_COLOR_RESET);
    ns_println(NS_COLOR_RED "Error: Connection lost." NS_COLOR_RESET);

    return 0;
}
```

Compile and run this exactly like the first one:

```bash
make hello_color
```

## How It Works

If you are coming from standard C, the code above might look like magic. How
does `ns_println` know whether to print a string, an integer, or a double
without you typing `%s`, `%d`, or `%f`?

NextStd leverages C11's `_Generic` keyword to inspect the type of the variable
at **compile time**. It then automatically routes your data to the correct,
memory-safe Rust backend function (e.g., `ns_print_int`, `ns_print_double`, or
`ns_print_str`).

Because there are no format strings, there is zero risk of mismatched types
causing undefined behavior or reading garbage memory from the stack. If you try
to print a type that NextStd doesn't support yet, the compiler will safely throw
an error before the program ever runs.
