# Building ISA-L with CMake

This document describes how to build the Intel(R) Intelligent Storage Acceleration Library (ISA-L) using CMake for x86-64 Linux environments.

## Prerequisites

*   **CMake**: Version 3.12 or higher.
*   **C Compiler**: A C compiler that supports C99 or later (e.g., GCC, Clang).
*   **NASM**: Netwide Assembler, version 2.14.01 or later, for assembling x86-64 specific routines.

You can install these tools using your distribution's package manager. For example, on Debian/Ubuntu:

```bash
sudo apt-get update
sudo apt-get install cmake gcc nasm
```

On Fedora/CentOS:

```bash
sudo yum install cmake gcc nasm
```

If your distribution provides an older version of NASM, you might need to build it from source. Refer to the [NASM website](https://www.nasm.us/) or the main `build.md` for instructions.

## Build Steps

1.  **Clone the ISA-L repository** (if you haven't already):

    ```bash
    git clone https://github.com/intel/isa-l.git
    cd isa-l
    ```

2.  **Create a build directory**: It's good practice to create a separate build directory.

    ```bash
    mkdir build
    cd build
    ```

3.  **Run CMake to configure the project**:

    ```bash
    cmake ..
    ```

    You can specify the installation prefix if needed:

    ```bash
    cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
    ```

4.  **Compile the library**:

    ```bash
    make
    ```

    You can use parallel builds if you have multiple cores:

    ```bash
    make -j$(nproc)
    ```

5.  **(Optional) Run tests**:

    ```bash
    ctest
    ```

    This will run the example test program (if enabled and configured during the CMake setup).

6.  **(Optional) Install the library**:

    ```bash
    sudo make install
    ```

    This will install the library, header files, and pkg-config file to the locations specified by `CMAKE_INSTALL_PREFIX` (or system defaults).

## Library Components

The CMake build will produce:

*   `libisal.so`: The shared library.
*   Header files: Installed to `<prefix>/include/isa-l.h` and `<prefix>/include/isa-l/` for supporting headers.
*   `libisal.pc`: A pkg-config file installed to `<prefix>/lib/pkgconfig` or `<prefix>/share/pkgconfig`.

You can then link against ISA-L in your projects using:

```bash
gcc my_program.c $(pkg-config --cflags --libs libisal) -o my_program
```

Or by finding the library with CMake in your own `CMakeLists.txt`:

```cmake
find_package(isal REQUIRED)
# ...
target_link_libraries(my_target PRIVATE isal)
```
(Note: `find_package(isal)` would require a `isal-config.cmake` or `Findisal.cmake` file to be generated and installed by this build system. This was not part of the initial request but is a common CMake practice for libraries. The current build provides `libisal.pc` which is a common way for non-CMake build systems and some CMake setups to find libraries).
