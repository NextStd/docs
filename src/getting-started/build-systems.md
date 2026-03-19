# Build System Integration

Because NextStd installs its headers and compiled archives directly into your
system's standard `/usr/local/` directories, integrating it into your own
projects is as simple as linking any other C library.

The only special requirement is that because NextStd's backend is powered by
Rust, you must link a few standard system libraries (`pthread`, `dl`, and `m`)
alongside the NextStd modules.

Here is how to configure the most popular build systems and command runners to
work with NextStd.

## GNU Make (`Makefile`)

If you are using a standard `Makefile`, you just need to append the NextStd
libraries to your `LDFLAGS` (Linker Flags) variable.

Here is a minimal, robust `Makefile` for a NextStd project:

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -O2
# Link your required NextStd modules and the Rust system dependencies
LDFLAGS = -lns_data -lns_io -lns_string -lns_error -lpthread -ldl -lm

TARGET = my_app
SRC = main.c

all: $(TARGET)

$(TARGET): $(SRC)
 $(CC) $(CFLAGS) $(SRC) -o $(TARGET) $(LDFLAGS)

clean:
 rm -f $(TARGET)
```

**Tip:** Order matters in GCC! Always put your source files (`main.c`) *before*
your `-l` linker flags, or the compiler might complain about undefined
references.

## Just (`justfile`)

If you prefer using [just](https://github.com/casey/just) (a modern, handy
command runner highly popular in the Rust ecosystem), configuring it is
incredibly clean.

Here is a ready-to-use `justfile` that handles building, running, and cleaning
your NextStd project:

```justfile
compiler := "gcc"
cflags := "-Wall -Wextra -O2"
libs := "-lns_data -lns_io -lns_string -lns_error -lpthread -ldl -lm"
target := "my_app"
src := "main.c"

# Default recipe: Build the application
@build:
    {{compiler}} {{cflags}} {{src}} -o {{target}} {{libs}}

# Build and immediately run the application
@run: build
    ./{{target}}

# Clean up compiled artifacts
@clean:
    rm -f {{target}}
```

To build and run your safe C code, you simply type `just run` in your terminal!

## CMake (`CMakeLists.txt`)

If you are using CMake, the process is just as easy. You use the
`target_link_libraries` directive to tell CMake exactly what needs to be bundled
into your executable.

Here is a complete `CMakeLists.txt` file:

```cmake
cmake_minimum_required(VERSION 3.10)

# Define the project and enable C
project(NextStdApp C)

# Add your executable
add_executable(my_app main.c)

# Link the NextStd archives and the required system libraries
target_link_libraries(my_app
    ns_data
    ns_io
    ns_string
    ns_error
    pthread
    dl
    m
)
```

## Which Modules Should I Link?

In the examples above, we linked all four core NextStd modules (`ns_data`,
`ns_io`, `ns_string`, `ns_error`).

NextStd is highly modular. If you are writing a small script that only uses
`ns_print` and doesn't use vectors, hashmaps, or strings, you can safely omit
`-lns_data` and `-lns_string` from your build configuration to slightly reduce
your final binary size.

However, `-lns_error` is required by almost all modules, so it should generally
always be included.
