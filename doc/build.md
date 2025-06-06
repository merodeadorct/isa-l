# Building ISA-L

ISA-L can be built using Autotools on Linux and macOS, or using Nmake on Windows with MSVC.
This document also describes the newer CMake-based build system, which is cross-platform.

## Prerequisites

*   **For Autotools (Linux/macOS):**
    *   Autoconf, Automake, Libtool
    *   GCC or Clang compiler
    *   NASM assembler (version 2.14 or later recommended for x86_64)
*   **For Nmake (Windows):**
    *   Microsoft Visual Studio (for `cl.exe` compiler and `nmake.exe`)
    *   NASM assembler (version 2.14 or later recommended)
*   **For CMake (Linux, macOS, Windows):**
    *   CMake (version 3.12 or later)
    *   A suitable C compiler (GCC, Clang, MSVC)
    *   NASM assembler (version 2.14 or later recommended for x86_64 builds)
    *   Ninja or Make (for Linux/macOS), Visual Studio or Ninja (for Windows)

## Building with Autotools (Linux/macOS)

1.  **Generate configure script:**
    ```sh
    ./autogen.sh
    ```

2.  **Run configure script:**
    ```sh
    ./configure
    ```
    Common options:
    *   `--prefix=/path/to/install` (default is /usr/local)
    *   `--enable-debug` (to build with debug symbols)

3.  **Build the library:**
    ```sh
    make
    ```

4.  **Run tests (optional):**
    ```sh
    make check
    ```

5.  **Install the library:**
    ```sh
    make install
    ```

## Building with Nmake (Windows)

1.  **Open a command prompt** that has `cl.exe`, `nmake.exe`, and `nasm.exe` in the PATH.
    (e.g., "x64 Native Tools Command Prompt for VS")

2.  **Build the library:**
    ```sh
    nmake -f Makefile.nmake
    ```
    This typically builds both static and shared libraries.

3.  **To build tests or examples:**
    Refer to targets in `Makefile.nmake` (e.g., `nmake -f Makefile.nmake checks`).

## Building with CMake (Cross-Platform)

The CMake build system allows for building ISA-L on various platforms including Linux, macOS, and Windows with different generator backends (Makefiles, Ninja, Visual Studio, etc.).

### General CMake Steps:

1.  **Create a build directory:**
    It's recommended to build ISA-L out-of-source.
    ```sh
    mkdir build
    cd build
    ```

2.  **Configure with CMake:**
    Run CMake from the build directory, pointing it to the root of the ISA-L source tree.
    ```sh
    # For Linux/macOS (using Makefiles generator by default)
    cmake ..

    # For Linux/macOS (using Ninja generator)
    cmake -G Ninja ..

    # For Windows (using Visual Studio generator, e.g., VS 2019)
    # Open "x64 Native Tools Command Prompt for VS" or ensure MSVC vars are set
    cmake -G "Visual Studio 16 2019" -A x64 ..

    # For Windows (using Ninja generator with MSVC compiler)
    # Open "x64 Native Tools Command Prompt for VS"
    cmake -G Ninja ..
    ```

    **Common CMake Options:**
    You can pass options to CMake using `-D<option_name>=<value>`.
    *   `-DCMAKE_INSTALL_PREFIX=/path/to/install`: Specify the installation directory (e.g., `C:/libs/isa-l` on Windows).
    *   `-DCMAKE_BUILD_TYPE=Release`: Build type (Debug, Release, RelWithDebInfo, MinSizeRel). Defaults to Debug if not specified by some generators.
    *   `-DBUILD_SHARED_LIBS=ON`: Build shared libraries in addition to static ones (default is OFF).
    *   `-DBUILD_TESTS=ON`: Build test programs (default is ON).
    *   `-DBUILD_EXAMPLES=ON`: Build example programs (default is ON).
    *   `-DENABLE_DEBUG=ON`: Enable internal debug messages in the library (default is OFF).
    *   `-DENABLE_AVX512=ON`: Enable AVX512 compilation for EC and RAID modules on x86_64 (default is ON, requires compatible compiler).

3.  **Build the library:**
    ```sh
    # If using Makefiles or Ninja
    cmake --build .

    # Or directly with make/ninja
    # make
    # ninja

    # If using Visual Studio generator, you can open the .sln file in the build directory
    # or use msbuild:
    # msbuild ISA-L.sln /p:Configuration=Release
    ```

4.  **Run tests (if `BUILD_TESTS` was ON):**
    CTest is used to run the tests.
    ```sh
    # From the build directory
    ctest
    # For verbose output
    ctest -V
    ```

5.  **Install the library:**
    ```sh
    # If using Makefiles or Ninja
    cmake --build . --target install

    # Or directly with make/ninja (if an install target is generated)
    # make install
    # ninja install

    # If using Visual Studio, the INSTALL project can be built from the IDE or msbuild
    # msbuild INSTALL.vcxproj /p:Configuration=Release
    ```

### Example: Release build on Linux with Ninja

```sh
mkdir build && cd build
cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=~/isa-l_install ..
ninja
ninja test  # Or ctest
ninja install
```

### Example: Release build on Windows with Visual Studio 2019

Ensure you are in an "x64 Native Tools Command Prompt for VS 2019".
```bat
mkdir build
cd build
cmake -G "Visual Studio 16 2019" -A x64 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="C:/Program Files/isa-l" ..
cmake --build . --config Release
ctest -C Release
cmake --build . --target INSTALL --config Release
```

This provides a comprehensive guide for developers wanting to build ISA-L using either the existing systems or the new CMake system.
