# The Try/Except Macros

Standard C doesn't have `try` and `catch` blocks. Instead, developers are forced
to write endless `if (result != 0)` checks, which clutters the "happy path" of
the code and makes the logic hard to read.

`NextStd` solves this by providing the `NS_TRY` and `NS_EXCEPT` macros. These
give you the clean, readable structure of modern exception handling without any
performance overhead—it all compiles down to standard C `if/else` statements
under the hood.

## The Basic Syntax

To use the macros, you first declare an `ns_error_t` variable to hold the return
state. Then, you pass that variable and the function you want to execute into
`NS_TRY`.

Here is a standard example of safely initializing an `ns_string` *(Note: We will
cover strings in depth in the **Strings** chapter, but they make a perfect
example here!)*:

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_string my_text;

    // Try to safely allocate memory for the string
    NS_TRY(err, ns_string_new(&my_text, "Hello, memory-safe world!")) {
        // This block ONLY runs if the function returns NS_SUCCESS
        ns_println("String initialized successfully:");
        ns_println(my_text);
    } 
    NS_EXCEPT(err, NS_ERROR_ANY) {
        // This block runs if an error occurred (e.g., out of memory)
        ns_println("Critical failure: Could not allocate string.");
        return 1;
    }

    ns_string_free(&my_text);
    return 0;
}
```

## How It Works

`NS_TRY` evaluates the function you pass to it. If the function returns
`NS_SUCCESS` (which equals `0`), the code inside the `{}` brackets executes.

If the function returns anything else, it skips the `NS_TRY` block and evaluates
the `NS_EXCEPT` block. The `err` variable stores the exact error code returned
by the function, so you can inspect it or log it inside the exception block.

## The "Macro Comma" Gotcha

As you progress to advanced NextStd features (like Vectors and HashMaps later in
this book), you might run into a specific C preprocessor quirk.

If the function you are passing into `NS_TRY` is *itself* a macro containing
commas (such as `ns_map_at`), the C compiler will get confused. It sees the
commas inside your function and mistakenly thinks you are passing too many
arguments to `NS_TRY`.

**The Problem:**

```c
// COMPILER ERROR: Too few arguments to function call
NS_TRY(err, ns_map_at(&map, ns_string, key, &val)) { ... }
```

**The Solution:** Wrap the inner macro call in an extra set of parentheses. This
"hides" the commas from the outer `NS_TRY` macro, allowing the preprocessor to
expand it correctly.

```c
// SUCCESS: Notice the extra parentheses around the map function
NS_TRY(err, (ns_map_at(&map, ns_string, key, &val))) { 
    ns_print("Value retrieved successfully.");
} NS_EXCEPT(err, NS_ERROR_ANY) {
    ns_println("Key not found in map.");
}
```

Alternatively, you can evaluate the macro first, store its result in your error
variable, and pass that variable into the try block:

```c
err = ns_map_at(&map, ns_string, key, &val);
NS_TRY(err, err) {
    ns_print("Value retrieved successfully.");
}
```
