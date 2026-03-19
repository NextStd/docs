# Your First Program

Now that your environment is set up and NextStd is installed globally on your
system, let's write your first memory-safe C program.

We are going to replace the traditional, unsafe `printf` function with NextStd's
type-safe `ns_println` macro.

## The Basic Program

Create a new file named `hello.c` anywhere on your computer.

Add the following code to print basic text and variables. Notice how we use
standard angle brackets for the includes now that NextStd is a system library:

```c
#include <nextstd/ns.h>

int main() {
    // 1. Printing a standard string
    ns_println("Hello, World! Welcome to NextStd.");

    // 2. Printing an integer safely without format specifiers
    int version = 2;
    ns_print("NextStd Version: ");
    ns_println(version); 

    // 3. Printing a floating-point number
    double pi = 3.14159;
    ns_print("Value of Pi: ");
    ns_println(pi);

    return 0;
}
```

To compile and execute this program, open your terminal and link the necessary
NextStd I/O and Error modules:

```bash
gcc hello.c -lns_io -lns_error -o hello
./hello
```

*(Note: If you are testing locally inside the cloned repository instead of using
the system-wide install, you can still just drop this in `examples/hello.c` and
run `make hello`)*

## Adding Terminal Colors

NextStd also provides a dedicated, cross-platform color module to help you build
beautiful CLI tools. You don't need to remember ANSI escape codes; you just
include the header and use the macros.

Create a second file named `hello_color.c`:

```c
#include <nextstd/ns.h>
#include <nextstd/ns_color.h> // Import the color macros

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
gcc hello_color.c -lns_io -lns_error -o hello_color
./hello_color
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
