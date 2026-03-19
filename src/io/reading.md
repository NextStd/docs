# Safe Reading (`ns_read`)

Taking user input in standard C is notoriously difficult to get right.

If you use `scanf("%s", buffer)`, you are immediately vulnerable to a
**buffer overflow** if the user types more characters than your buffer can hold.
Furthermore, `scanf` is famous for leaving dangling newline characters (`\n`) in
the standard input stream, which causes subsequent read attempts to mysteriously
skip or fail.

`NextStd` fixes all of this with a single, memory-safe macro: `ns_read`.

## Basic Usage

Just like `ns_print`, `ns_read` uses C11 `_Generic` routing to figure out
exactly what data type you are trying to read at compile time.

You simply pass a **pointer** (using the `&` address-of operator) to the
variable you want to populate. Because interacting with the terminal can fail
(e.g., the user types a word instead of a number), `ns_read` returns an
`ns_error_t` that you should wrap in an `NS_TRY` block.

```c
#include <nextstd/ns.h>

int main() {
  ns_error_t err;

  int age;
  ns_print("Enter your age: ");

  // Pass the memory address of 'age' so NextStd can populate it
  NS_TRY(err, ns_read(&age)) {
    ns_print("Success! You are ");
    ns_print(age);
    ns_println(" years old.");
  } 
  NS_EXCEPT(err, NS_ERROR_ANY) {
    ns_print("Failed to read input: ");
    ns_println(ns_error_message(err));
  }

  return 0;
}
```

## How NextStd Protects You

When you call `ns_read`, the Rust backend takes complete control of the input
stream and performs several safety checks before your C code ever sees the data:

1. **Type Validation:** If you call `ns_read(&age)` (expecting an `int`) and the
   user types `"hello"`, the Rust backend will safely reject the input and
   return `NS_ERROR_INVALID_INPUT`. Your program will not crash, and your `age`
   variable will remain untouched.
2. **Buffer Flushing:** NextStd automatically flushes the standard input buffer
   after every read. You will never have to write hacky
   `while ((c = getchar()) != '\n' && c != EOF)` loops to clear out dangling
   newlines ever again.
3. **Memory Bounding:** When reading strings (which we cover in the Strings
   chapter), the Rust backend dynamically allocates exactly the amount of memory
   needed to hold the user's input. Buffer overflows are architecturally
   impossible.

## Reading Multiple Types

Because `ns_read` infers the type from the pointer you pass it, you can reuse
the exact same macro for integers, floats, and booleans.

```c
#include <nextstd/ns.h>
#include <stdbool.h>

int main() {
  ns_error_t err;

  double price;
  ns_print("Enter the price: ");
  NS_TRY(err, ns_read(&price)) {
    ns_println(price);
  } NS_EXCEPT(err, NS_ERROR_ANY) {
    ns_println(ns_error_message(err));
  }

  bool is_student;
  ns_print("Are you a student? (true/false): ");
  NS_TRY(err, ns_read(&is_student)) {
    ns_println(is_student);
  } NS_EXCEPT(err, NS_ERROR_ANY) {
    ns_println(ns_error_message(err));
  }

  return 0;
}
```
