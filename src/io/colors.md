# Terminal Colors (`ns_color`)

When building a Command Line Interface (CLI), color is essential for user
experience. It helps users instantly distinguish between a routine success
message, a non-critical warning, and a fatal error.

In standard C, adding color means injecting raw ANSI escape codes directly into
your strings, which makes your code incredibly difficult to read:

```c
// Standard C: Hard to read and easy to mess up
printf("\033[1;31mError:\033[0m Could not open file.\n");
```

`NextStd` solves this by providing semantic, easy-to-read macros in the
`ns_color.h` header.

## Basic Usage

Because these macros resolve to standard string literals under the hood, you can
rely on the C preprocessor to automatically concatenate them with your text.

Just include `<nextstd/ns_color.h>`, place the color macro immediately before
your string, and **always** remember to append `NS_COLOR_RESET` at the end!

```c
#include <nextstd/ns.h>
#include <nextstd/ns_color.h>

int main() {
    // A standard success message
    ns_println(NS_COLOR_GREEN "Success: User profile updated." NS_COLOR_RESET);

    // A warning message
    ns_println(NS_COLOR_YELLOW "Warning: Disk space is running low." NS_COLOR_RESET);

    // A critical error message
    ns_println(NS_COLOR_RED "Error: Failed to connect to the database." NS_COLOR_RESET);

    return 0;
}
```

## Mixing Colors and Styles

You can stack multiple macros together to combine colors with text formatting,
such as making a string bold or underlining a URL. Just separate them with a
space.

```c
#include <nextstd/ns.h>
#include <nextstd/ns_color.h>

int main() {
    // Bold and Cyan text
    ns_println(NS_COLOR_BOLD NS_COLOR_CYAN "Starting NextStd Server..." NS_COLOR_RESET);

    // Underlined Magenta text
    ns_println(NS_COLOR_UNDERLINE NS_COLOR_MAGENTA "https://github.com/NextStd/nextstd" NS_COLOR_RESET);

    return 0;
}
```

## The Importance of Resetting

Terminal emulators are stateful. When you send a color code to the terminal, it
changes the color of **all subsequent text** until it receives a reset command.

If you forget to include `NS_COLOR_RESET` at the end of your `ns_println` call,
every single thing your program (and potentially your user's terminal prompt)
prints afterward will be stuck in that format!

## Available Macros Reference

Here is a list of the standard text styling macros available in `ns_color.h`:

**Colors:**

* `NS_COLOR_RED`
* `NS_COLOR_GREEN`
* `NS_COLOR_YELLOW`
* `NS_COLOR_BLUE`
* `NS_COLOR_MAGENTA`
* `NS_COLOR_CYAN`
* `NS_COLOR_WHITE`

**Styles:**

* `NS_COLOR_BOLD`
* `NS_COLOR_UNDERLINE`

**State Control:**

* `NS_COLOR_RESET` (Resets all colors and text formatting back to the terminal
  default)
