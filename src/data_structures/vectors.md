# Dynamic Arrays (`ns_vec`)

In standard C, arrays are rigid. Once you declare `int numbers[5];`, that array
can never hold a sixth number. If you need a collection that can grow or shrink
at runtime, you are forced to use raw pointers, `malloc`, and manual `realloc`
calls.

Worse, standard C arrays have **zero bounds checking**. If you try to read
`numbers[100]`, C will silently read garbage memory from that location,
potentially crashing your program or exposing a massive security vulnerability.

`NextStd` fixes this with `ns_vec`: a dynamically resizing, bounds-checked
vector managed safely by the Rust backend.

## 1. Initialization (`ns_vec_new`)

Creating a vector is as simple as creating a string. You declare your `ns_vec`
struct and pass a pointer to `ns_vec_new` to initialize it. The Rust backend
sets up the initial heap allocation and capacity tracking invisibly.

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_vec numbers;

    // 1. Initialize the vector
    NS_TRY(err, ns_vec_new(&numbers)) {
        ns_println("Vector created successfully!");
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_println("Failed to allocate vector.");
        return 1;
    }

    // ...
    return 0;
}
```

## 2. Adding Elements (`ns_vec_push`)

You don't need to keep track of memory capacity or calculate byte offsets to add
data. Just call `ns_vec_push`.

If the vector is full, the Rust backend will automatically and safely allocate a
larger block of memory on the heap, move the existing data over, and add your
new element. Because this reallocation can technically fail if the system runs
out of memory, it returns an `ns_error_t`.

```c
    // 2. Safely push data into the vector
    NS_TRY(err, ns_vec_push(&numbers, 10)) { } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }
    NS_TRY(err, ns_vec_push(&numbers, 20)) { } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }
    NS_TRY(err, ns_vec_push(&numbers, 30)) { } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }
```

## 3. Safe Access (`ns_vec_get`)

This is where `NextStd` truly shines. When you want to retrieve an element, you
pass the index and a pointer to the variable where you want the data stored.

The Rust backend performs a **strict bounds check**. If you ask for index 5 in a
vector that only has 3 elements, your program will not crash. Instead, Rust
safely blocks the memory access and returns an error!

```c
    int value;

    // 3. Retrieve an element safely
    NS_TRY(err, ns_vec_get(&numbers, 1, &value)) {
        ns_print("Value at index 1 is: ");
        ns_println(value); // Prints 20
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_print("Out of bounds: ");
        ns_println(ns_error_message(err));
    }
```

## 4. Destruction (`ns_vec_free`)

Just like strings, vectors manage dynamic heap memory. When you are done with
the vector, you **must** free it to prevent memory leaks.

```c
    // 4. Destroy the vector and free the heap memory
    ns_vec_free(&numbers);
```

---

### The Complete Example

Here is a complete, memory-safe workflow utilizing `ns_vec`. Notice how clean
the C code remains without a single `malloc` or `sizeof` in sight:

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_vec scores;

    // Initialize
    NS_TRY(err, ns_vec_new(&scores)) {
    } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    // Push values
    NS_TRY(err, ns_vec_push(&scores, 100)) {} NS_EXCEPT(err, NS_ERROR_ANY) {}
    NS_TRY(err, ns_vec_push(&scores, 95)) {} NS_EXCEPT(err, NS_ERROR_ANY) {}

    // Print the length
    ns_print("Total scores: ");
    ns_println(scores.len);

    // Safely retrieve a value
    int current_score;
    NS_TRY(err, ns_vec_get(&scores, 0, &current_score)) {
        ns_print("First score: ");
        ns_println(current_score);
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_println(ns_error_message(err));
    }

    // Always clean up!
    ns_vec_free(&scores);

    return 0;
}
```
