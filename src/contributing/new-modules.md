# Adding New Modules

Because `NextStd` relies on a hybrid architecture, adding a new feature isn't as
simple as just writing a new C function. You are essentially building a bridge:
constructing the heavy machinery in Rust, and then creating a clean, easy-to-use
control panel for it in C.

This guide will walk you through the standard workflow for contributing a new
module to the library.

## Step 1: Build the Rust Backend

Every new feature starts in the `crates/` directory. You will either add a new
file to an existing crate (like `ns_io` or `ns_string`) or create an entirely
new crate for a major feature.

When writing your Rust logic, keep the FFI (Foreign Function Interface) boundary
in mind:

* **Use `#[unsafe(no_mangle)]`:** This prevents the Rust compiler from renaming
  your function, ensuring the C linker can find it by its exact name.
* **Use `extern "C"`:** This forces Rust to use the C Application Binary
  Interface (ABI).
* **Handle Pointers Safely:** C will pass raw pointers (`*mut T` or `*const T`).
  You must check them for null before dereferencing them inside an `unsafe {}`
  block.

```rust
// Example: crates/ns_math/src/lib.rs

use ns_error::NsError;
use std::os::raw::c_int;

#[unsafe(no_mangle)]
pub unsafe extern "C" fn ns_math_add(a: c_int, b: c_int, out: *mut c_int) -> NsError {
    if out.is_null() {
        return NsError::Any; // Never trust the C pointer!
    }

    // Perform the safe Rust logic
    let result = a.saturating_add(b);

    // Write back to the C pointer safely
    unsafe { *out = result; }

    NsError::Success
}
```

## Step 2: Define the C Interface

Once the Rust engine is built, you need to expose it to the C developers. This
happens in the `include/nextstd/` directory.

Create a new header file (e.g., `ns_math.h`). Inside, you will:

1. Declare the external function exactly as it is named in Rust.
2. Create a user-friendly macro to hide the pointer logic and error handling if
   necessary.

```c
// Example: include/nextstd/ns_math.h

#ifndef NS_MATH_H
#define NS_MATH_H

#include "ns_error.h"

#ifdef __cplusplus
extern "C" {
#endif

// 1. Expose the Rust function
ns_error_t ns_math_add(int a, int b, int* out);

// 2. Create a clean macro for the user
#define ns_add(a, b, out_ptr) ns_math_add((a), (b), (out_ptr))

#ifdef __cplusplus
}
#endif

#endif // NS_MATH_H
```

## Step 3: Integrate `_Generic` (If Applicable)

If your new module handles multiple data types (like printing or reading), you
should use C11 `_Generic` macros to route the data automatically.

For example, if you added `ns_math_add_float` in Rust, your C macro would look
like this:

```c
#define ns_add(a, b, out_ptr) _Generic((out_ptr), \
    int*: ns_math_add, \
    float*: ns_math_add_float \
)(a, b, out_ptr)
```

## Step 4: Hook up the Build System

Finally, ensure your new Rust crate is included in the Cargo workspace (if it's
a new crate) and that the compiled static or dynamic library is properly linked
in the project's `Makefile`.

Run `make rust` to compile your new backend, write a quick test script in
`main.c`, and verify that your C macros successfully trigger your safe Rust
code!
