# Building a CLI Menu

One of the most common tasks in systems programming is building an interactive
Command Line Interface (CLI) menu. However, doing this in standard C often
introduces a severe bug: **the infinite input loop**.

If you build a menu using a standard `while` loop and ask for the user's choice
using `scanf("%d", &choice)`, your program expects an integer. If the user
accidentally types a letter (like "A"), `scanf` fails to read it, but it
*leaves the letter in the input buffer*. The loop repeats, `scanf` sees the same
letter, fails again, and your terminal is instantly flooded with an infinite
loop of repeated prompts.

`NextStd` completely eliminates this class of bugs.

## The NextStd Approach

By combining `ns_read`, our `ns_color` macros, and `NS_TRY`/`NS_EXCEPT` blocks,
you can build a menu that is completely bulletproof.

When you use `ns_read(&op)`, the Rust backend safely reads the input line. If
the user types "A" instead of a number, it catches the parsing error, flushes
the bad input, and cleanly throws an `NS_ERROR_INVALID_INPUT` error. Your
program can catch it, print a polite warning, and safely wait for the next
input.

## The Complete Example

Here is a fully functional "System Control" menu. Try running this and
intentionally typing words instead of numbers—notice how it handles the errors
gracefully without ever crashing or looping infinitely!

```c
#include <nextstd/ns.h>
#include <nextstd/ns_color.h>
#include <stdbool.h>

int main(void)
{
  int op = 0;     // Option
  ns_error_t err;
  bool continue_loop = true;

  while (continue_loop) {
    ns_println("Choose any of the below options: ");
    ns_println("1. First Option\n2. Second Option \n3. Third Option \n4. Exit");
    ns_print("Enter your option: ");
    NS_TRY(err, ns_read(&op)) {
      if (op == 1) {
        ns_println(NS_COLOR_GREEN "You chose option 1" NS_COLOR_RESET);
      }
      else if (op == 2) {
        ns_println(NS_COLOR_BLUE "You chose option 2" NS_COLOR_RESET);
      } else if (op == 3) {
        ns_println(NS_COLOR_MAGENTA "You chose option 3" NS_COLOR_RESET);
      } else {
        ns_println(NS_COLOR_YELLOW "Exiting...." NS_COLOR_RESET);
        continue_loop = false;
      }
    }
    NS_EXCEPT(err, NS_ERROR_INVALID_INPUT) {
      ns_println(NS_COLOR_RED "Wrong input" NS_COLOR_RESET);
      ns_println(ns_error_message(err));
    }
  }
}
```

### Why this works so well

* **State Management:** The `continue_loop` boolean gives us a clean way to
  break out of the infinite `while` loop when the user selects the Exit option.
* **Aesthetic Feedback:** The `ns_color` macros provide instant visual feedback.
  Successes are green, information is blue/magenta, warnings are yellow, and
  errors are red.
* **Absolute Stability:** The `NS_TRY` block isolates the unpredictable nature
  of human input, guaranteeing the program state remains valid.
