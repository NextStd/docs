# Key-Value Maps (`ns_hashmap`)

If you want to associate a string with an integer, or an ID with a user profile,
standard C leaves you entirely on your own. There is no built-in dictionary or
hash map data structure. Developers are usually forced to build complex,
error-prone linked lists or pull in massive third-party dependencies just to map
a key to a value.

NextStd solves this with `ns_map`: a fast, memory-safe key-value store powered
by Rust's highly optimized standard library hash map, brought to C with a clean
and type-safe macro API.

## 1. Initialization (`ns_map_new`)

Because `ns_map` can store any data type, you need to tell it the size of the
keys and the values it will be holding when you create it. You do this by
passing `sizeof(your_key_type)` and `sizeof(your_value_type)` into the
`ns_map_new` initialization function.

```c
#include <nextstd/ns.h>

int main() {
    ns_error_t err;
    ns_map student_grades;

    // 1. Initialize a map with `int` keys (Student ID) and `float` values (Grade)
    NS_TRY(err, ns_map_new(&student_grades, sizeof(int), sizeof(float))) {
        ns_println("Map initialized successfully!");
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_println("Failed to allocate map.");
        return 1;
    }

    // ...
    return 0;
}
```

## 2. Inserting Data (`ns_map_put`)

In standard C, passing arbitrary generic types into a function is a nightmare of
`void*` casting.

NextStd abstracts all of this away with the `ns_map_put` macro. You simply
provide the map pointer, the type of the key, the key itself, the type of the
value, and the value itself. The macro safely handles the memory address
referencing behind the scenes.

```c
    // 2. Insert key-value pairs
    // Usage: ns_map_put(&map, key_type, key_data, val_type, val_data)

    NS_TRY(err, ns_map_put(&student_grades, int, 101, float, 95.5f)) {} 
    NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    NS_TRY(err, ns_map_put(&student_grades, int, 102, float, 88.0f)) {} 
    NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }
```

## 3. Retrieving Data (`ns_map_at`)

To retrieve a value, you use the `ns_map_at` macro. You provide the key you are
looking for, and a pointer to a variable where NextStd should write the
resulting value.

Because hash map lookups can fail (e.g., if you ask for a key that does not
exist), `ns_map_at` returns an error code. This forces you to handle missing
keys gracefully instead of accidentally reading uninitialized memory.

```c
    float grade;

    // 3. Retrieve a value by its key
    // Usage: ns_map_at(&map, key_type, key_data, &output_variable)

    NS_TRY(err, ns_map_at(&student_grades, int, 101, &grade)) {
        ns_print("Student 101 Grade: ");
        ns_println_float(grade); // Prints 95.5
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_println("Student not found in the map.");
    }
```

## 4. Destruction (`ns_map_free`)

Like all dynamic NextStd structures, `ns_map` allocates memory on the heap. When
you are finished with the map, you **must** free it to return that memory to the
operating system.

```c
    // 4. Clean up the map
    ns_map_free(&student_grades);
```

---

### The Complete Example

Here is a complete, memory-safe workflow demonstrating how to map integer IDs to
float values using `ns_map`.

```c
#include <nextstd/ns.h>
#include <nextstd/ns_hashmap.h> // Ensure you include the correct header

int main() {
    ns_error_t err;
    ns_map prices;

    // Initialize: int keys (Item ID), float values (Price)
    NS_TRY(err, ns_map_new(&prices, sizeof(int), sizeof(float))) {
    } NS_EXCEPT(err, NS_ERROR_ANY) { return 1; }

    // Insert data
    NS_TRY(err, ns_map_put(&prices, int, 1, float, 19.99f)) {} NS_EXCEPT(err, NS_ERROR_ANY) {}
    NS_TRY(err, ns_map_put(&prices, int, 2, float, 5.49f)) {} NS_EXCEPT(err, NS_ERROR_ANY) {}

    // Print the size of the map
    ns_print("Total items in map: ");
    ns_println(prices.len);

    // Retrieve data safely
    float result;
    NS_TRY(err, ns_map_at(&prices, int, 2, &result)) {
        ns_print("Price for Item 2: $");
        ns_println_float(result);
    } NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_println("Item not found!");
    }

    // Always clean up!
    ns_map_free(&prices);

    return 0;
}
```
