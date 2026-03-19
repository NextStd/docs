# Error Codes Reference

Under the hood, `ns_error_t` is a strict `enum` defined by the Rust backend and
exposed to C. Because it is a unified type, you can rely on these specific
return codes across all NextStd modules.

## Core Error Codes

When an operation fails, NextStd will return one of the following codes.

| C Macro / Rust Enum | Value | Description / Output |
| :--- | :---: | :--- |
| `NS_SUCCESS` | `0` | Success |
| `NS_ERROR_ANY` | `1` | Unknown Error. This acts as a catch-all (similar to Python's `except Exception`). |
| `NS_ERROR_IO_READ_FAILED` | `10` | I/O Error: Failed to read input. |
| `NS_ERROR_IO_WRITE_FAILED` | `11` | I/O Error: Failed to write input. |
| `NS_ERROR_INVALID_INPUT` | `12` | I/O Error: Invalid Input format. |
| `NS_ERROR_STRING_ALLOC_FAILED` | `20` | String Error: Memory Allocation failed. |
| `NS_ERROR_STRING_INVALID_UTF8` | `21` | String Error: Invalid UTF-8 sequence detected. |
| `NS_ERROR_INDEX_OUT_OF_BOUNDS` | `22` | Error: Index out of bounds. |

## Translating Errors to Strings (`ns_error_message`)

One of the most powerful features of NextStd's error handling is the
`ns_error_message` function.

Instead of forcing you to write massive `switch` statements in C to print the
correct error, the Rust backend provides a helper function that translates the
`ns_error_t` code directly into a readable C-string (`const char*`).

You can pass this directly into `ns_println` inside your exception blocks:

```c
#include <nextstd/ns.h>
#include <nextstd/ns_data.h>

int main() {
    ns_error_t err;
    ns_vec my_list;

    // Initialize list with some data...
    ns_vec_new(&my_list, sizeof(int));

    int result;
    // Attempting to read an index that doesn't exist
    NS_TRY(err, (ns_vec_get(&my_list, int, 99, &result))) {
        ns_println("Successfully retrieved item!");
    } 
    NS_EXCEPT(err, NS_ERROR_ANY) {
        ns_print("Operation failed: ");
        ns_println(ns_error_message(err)); 
    }

    ns_vec_free(&my_list);
    return 0;
}
```

By leveraging `ns_error_message(err)`, your CLI applications can provide
instantly readable, context-aware error messages to your users with zero extra C
code.
