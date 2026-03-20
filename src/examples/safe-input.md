# Safe User Input

Taking text input from a user in standard C is notoriously dangerous.

Historically, functions like `gets()` were so prone to catastrophic buffer
overflows that they were completely removed from the C language standard. Even
the modern alternative, `scanf("%s", buffer)`, is risky. If you allocate a
`char buffer[50];` and the user types 100 characters, the program will silently
overwrite adjacent memory, leading to crashes or severe security
vulnerabilities.

To fix this in standard C, you have to write complex loops that read one
character at a time and manually reallocate heap memory on the fly. NextStd
handles all of this for you automatically.

## The NextStd Solution: Dynamic Reading

With NextStd, the philosophy is simple:
**You should never have to guess how much text a user is going to type.**

By passing an `ns_string` directly into the `ns_read` macro, the Rust backend
takes over. It safely reads the entire line from standard input and
automatically calculates the exact memory required.

* If the user types a short name (under 24 bytes), NextStd uses
  **Small String Optimization (SSO)** and stores it entirely on the stack.
* If the user pastes a 10,000-word essay into the terminal, NextStd seamlessly
  allocates the exact amount of heap space needed.

No buffers to manage, no arbitrary limits, and absolutely no memory corruption.

## Step-by-Step Walkthrough

Capturing user input dynamically requires just a few simple steps:

1. **Declare your string:** You only need an uninitialized `ns_string` struct.
2. **Prompt the user:** Ask your question using `ns_print`.
3. **Capture the input:** Pass a pointer to your string into `ns_read`, wrapping
   it in an `NS_TRY` block to safely catch any unexpected I/O failures.
4. **Use the data:** Print or manipulate the string safely.
5. **Clean up:** Always pass the string to `ns_string_free` when you are done to
   release any heap memory it may have allocated.

## The Complete Example

Here is a fully functional, memory-safe program that captures user input of any
length dynamically:

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_string user_input;

    // 1. Ask the user a question
    ns_print("Please enter your name (or paste a whole paragraph!): ");

    // 2. Read the input dynamically
    NS_TRY(err, ns_read(&user_input)) {

        // 3. Use the safely captured string
        ns_print("Hello, ");
        ns_println(user_input);

        ns_print("Your input was exactly ");
        ns_print(user_input.len);
        ns_println(" characters long.");

    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_print("Failed to read input: ");
        ns_println(ns_error_message(err));
        return 1;
    }

    // 4. Always clean up! 
    // (Safely frees heap memory, or does nothing if the string used SSO)
    ns_string_free(&user_input);

    return 0;
}
```

Because `ns_read` is heavily integrated with the NextStd `_Generic` macros, you
never have to specify format strings like `%s` or `%d`. The compiler inherently
knows you are reading into an `ns_string` and routes it to the exact memory-safe
Rust function required!
