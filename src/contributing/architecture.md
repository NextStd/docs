# Architecture Overview

`NextStd` is not a traditional C library. It is a hybrid architecture designed
to seamlessly bridge the gap between two radically different languages.

To a developer using the library, `NextStd` looks and feels exactly like modern
C. They include a header, call a macro, and their code compiles cleanly. Under
the hood, however, there is almost no C execution happening. The C code acts
entirely as a lightweight routing layer that safely passes data across a Foreign
Function Interface (FFI) boundary into a highly optimized, memory-safe Rust
engine.

## The Three Pillars of NextStd

Understanding how data flows through the library requires looking at its three
distinct layers.

### 1. The C Front-End (`include/nextstd/`)

This is the only part of the library the end-user ever sees. It consists
entirely of header files containing:

* **Memory-Layout Structs:** C structs (like `ns_string` or `ns_vec`) that
  define the exact byte layout of the data so the C compiler knows how much
  memory to reserve on the stack.
* **`_Generic` Macros:** The secret weapon of `NextStd`. Because C does not
  support function overloading, we use C11 `_Generic` macros to inspect the type
  of a variable at compile time and automatically route it to the correct
  underlying function (e.g., routing an `int` to `ns_print_int` and an
  `ns_string` to `ns_print_ns_string`).
* **Error Handling Macros:** The `NS_TRY` and `NS_EXCEPT` macros that evaluate
  the `ns_error_t` enums returned by the backend.

### 2. The FFI Boundary

The Foreign Function Interface (FFI) is the bridge between C and Rust. Because
Rust and C manage memory differently, crossing this bridge requires strict
rules:

* **No C++ Magic:** Everything must use standard C ABI (Application Binary
  Interface).
* **Pointer Passing:** When C needs Rust to modify a struct (like allocating
  heap memory for a new string), C passes a raw pointer (`ns_string*`) across
  the boundary.
* **Universal Error Currency:** Rust functions never crash or "panic" across the
  boundary. Every fallible operation returns an `ns_error_t` enum back to C.

### 3. The Rust Back-End (`crates/`)

This is where the actual logic lives. The Rust backend exposes functions marked
with `#[unsafe(no_mangle)] pub extern "C"`.

* **The "Unsafe" Gateway:** When Rust receives a raw pointer from C, it must
  enter an `unsafe {}` block to dereference it.
* **The Safe Execution:** Once the raw C data is converted into safe Rust types
  (like turning a `*const c_char` into a `CStr`), the rest of the execution
  happens in 100% safe Rust. Rust handles the dynamic memory allocation, the
  string formatting, and the bounds checking.
* **The Clean Return:** Rust updates the memory at the C pointer address,
  converts any internal Rust errors into an `ns_error_t`, and cleanly returns
  control to C.

## The Lifecycle of a Function Call

To picture how this all comes together, let's trace exactly what happens when a
user calls `ns_read(&my_text)`:

1. **Compile Time (C Front-End):** The C preprocessor evaluates
   `ns_read(&my_text)`. It sees that `&my_text` is an `ns_string*` type. Using
   the `_Generic` macro, it replaces the code with a call to
   `ns_read_ns_string(&my_text)`.
2. **Execution (Crossing the FFI):** The compiled C program jumps to the memory
   address of `ns_read_ns_string`.
3. **Rust Takes Over (Back-End):** The Rust function
   `pub extern "C" fn ns_read_ns_string(ptr: *mut NsString)` wakes up. It safely
   reads dynamic input from the terminal using Rust's `std::io`.
4. **Memory Allocation:** Rust calculates the exact size of the input, allocates
   the necessary heap memory, and carefully writes the pointer addresses and
   capacity sizes directly into the `ptr` struct.
5. **Return:** Rust returns `NsError::Success` (which maps perfectly to
   `NS_ERROR_SUCCESS` in C). Control is handed back to the C program.

By keeping the C layer incredibly thin and offloading all the complex logic to
Rust, `NextStd` guarantees memory safety without sacrificing the ease of use of
a C library.
