# Creating & Freeing Strings

Unlike standard C strings, which are often just raw pointers to arbitrary blocks
of memory, an `ns_string` is a fully managed data structure.

To guarantee memory safety and prevent undefined behavior, every `ns_string`
must go through a strict, three-step lifecycle: **Initialization**, **Usage**,
and **Destruction**.

## 1. Initialization (`ns_string_new`)

Before you can use an `ns_string`, it must be initialized. You do this by
passing a pointer to your uninitialized `ns_string` struct and the initial
standard C string (or `""` for an empty string) to the `ns_string_new` function.

This is where the Rust backend takes over. It calculates the length of your
input, decides whether to store it on the stack (via Small String Optimization)
or allocate memory on the heap, and populates your struct safely.

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_string my_text;

    // 1. Initialize the string
    NS_TRY(err, ns_string_new(&my_text, "Welcome to NextStd!")) {
        ns_println("String initialized successfully.");
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_println("Failed to allocate string.");
        return 1;
    }

    // ... usage and destruction ...
    return 0;
}
```

## 2. Usage

Once initialized, your `ns_string` struct contains safely managed properties.
Because it utilizes Small String Optimization (SSO), the underlying text data is
safely tucked away inside a union.

Fortunately, you don't have to manually extract it. Because `ns_print` and
`ns_println` handle `ns_string` objects natively via C11 `_Generic` macros, you
can simply pass the entire struct directly to print your managed string!

You can also print struct properties like `len` seamlessly since `ns_io`
natively supports C's `size_t` type.

```c
    // 2. Use the string
    ns_print("Message: ");
    ns_println(my_text); // No need to access inner pointers!

    ns_print("Length: ");
    ns_println(my_text.len); // Prints size_t seamlessly
```

## 3. Destruction (`ns_string_free`)

Standard C requires you to manually call `free()` on any memory you allocate. If
you forget, your program leaks memory. If you call it twice, your program
crashes (a "double free" vulnerability).

When you are finished with an `ns_string`, you **must** pass its pointer to
`ns_string_free`.

```c
    // 3. Destroy the string
    ns_string_free(&my_text);
```

The Rust backend handles the deallocation process intelligently. If the string
was utilizing Small String Optimization (stored entirely on the stack),
`ns_string_free` does practically nothing, safely avoiding an invalid heap
operation. If it was allocated on the heap, Rust cleanly returns the memory to
the system and nullifies the pointers to prevent accidental use-after-free
vulnerabilities.

---

### The Complete Example

Putting it all together, here is the complete, memory-safe lifecycle of an
`ns_string`:

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_string greeting;

    // 1. Initialization
    NS_TRY(err, ns_string_new(&greeting, "Hello, memory safety!")) {

        // 2. Usage
        ns_print("The string is: ");
        ns_println(greeting); 

        ns_print("Length: ");
        ns_println(greeting.len);

    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_print("Error: ");
        ns_println(ns_error_message(err));
        return 1;
    }

    // 3. Destruction (Always clean up!)
    ns_string_free(&greeting);

    return 0;
}
```
