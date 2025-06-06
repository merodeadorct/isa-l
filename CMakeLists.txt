cmake_minimum_required(VERSION 3.12)
project(isa-l VERSION 2.31.1 LANGUAGES C ASM)

option(BUILD_TESTS "Build test programs" ON)
option(BUILD_EXAMPLES "Build example programs" ON)
option(ENABLE_DEBUG "Enable debug messages" OFF)

# Detect host architecture
if(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "x86_64" OR CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "amd64")
    set(TARGET_ARCH "x86_64")
elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "aarch64" OR CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "arm64")
    set(TARGET_ARCH "aarch64")
elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "ppc64le" OR CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "powerpc64le")
    set(TARGET_ARCH "ppc64le")
elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "riscv64")
    set(TARGET_ARCH "riscv64")
else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_HOST_SYSTEM_PROCESSOR}")
endif()

message(STATUS "Target architecture: ${TARGET_ARCH}")

if(TARGET_ARCH STREQUAL "x86_64")
    find_package(NASM)
    if(NOT NASM_FOUND)
        message(FATAL_ERROR "NASM not found. Please install NASM.")
    endif()
    # Add NASM build rules for .asm files
    enable_language(ASM_NASM)
    set(CMAKE_ASM_NASM_OBJECT_FORMAT elf64)
    if(WIN32)
        set(CMAKE_ASM_NASM_OBJECT_FORMAT win64)
    elseif(APPLE)
        set(CMAKE_ASM_NASM_OBJECT_FORMAT macho64)
        set(CMAKE_ASM_NASM_FLAGS_INIT "--prefix=_")
    endif()
    # Check for Intel CET
    include(CheckCSourceCompiles)
    check_c_source_compiles("#ifndef __CET__
#error CET is not enabled
#endif
int main() {return 0;}" INTEL_CET_ENABLED)
    if(INTEL_CET_ENABLED)
        add_definitions(-DINTEL_CET_ENABLED)
    endif()
elif(TARGET_ARCH STREQUAL "riscv64")
    # Check for RVV support
    include(CheckCSourceCompiles)
    check_c_source_compiles("int main() { __asm__ volatile (\".option arch, +v\\nvsetivli zero, 0, e8, m1, ta, ma\\n\"); return 0;}" HAVE_RVV_FLAG_SUPPORT)
    if(HAVE_RVV_FLAG_SUPPORT)
         set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=rv64gcv")
         set(HAVE_RVV TRUE)
         message(STATUS "RVV support detected and enabled.")
    else()
         set(HAVE_RVV FALSE)
         message(STATUS "RVV support not detected.")
    endif()
    add_definitions(-D__ASSEMBLY__) # Define __ASSEMBLY__ for RISC-V assembly files
endif()


# Set compiler flags
set(CMAKE_C_FLAGS_COMMON "-Wall -Wchar-subscripts -Wformat-security -Wnested-externs -Wpointer-arith -Wshadow -Wstrict-prototypes -Wtype-limits -fstack-protector -D_FORTIFY_SOURCE=2")
if(MSVC)
    set(CMAKE_C_FLAGS_COMMON "") # MSVC uses different flags
    add_compile_definitions(_USE_MATH_DEFINES)
    # Add common MSVC flags, e.g. /W4 for warning level 4
    # CMake default flags for MSVC are usually good. /Z7 and /Gy are often defaults or can be added.
    # For example: string(APPEND CMAKE_C_FLAGS_RELEASE " /Gy")
    # string(APPEND CMAKE_C_FLAGS_DEBUG " /Z7")
    # string(APPEND CMAKE_C_FLAGS_RELWITHDEBINFO " /Z7 /Gy")
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_COMMON} ${CMAKE_C_FLAGS}")
set(CMAKE_C_FLAGS_DEBUG "-g") # For GCC/Clang
set(CMAKE_C_FLAGS_RELEASE "-O3 -DNDEBUG") # For GCC/Clang

if(MSVC)
    set(CMAKE_C_FLAGS_DEBUG "/Od /Z7")
    set(CMAKE_C_FLAGS_RELEASE "/O2 /Z7 /Gy -DNDEBUG") # Example, adjust as needed
    set(CMAKE_C_FLAGS_RELWITHDEBINFO "/O2 /Z7 /Gy -DNDEBUG")
endif()

if(ENABLE_DEBUG)
    add_definitions(-DENABLE_DEBUG)
endif()

# Add include directory
include_directories(include) # This should be before subdirectories if they rely on it
# For generated isa-l.h, ensure binary dir is also available for sources that might include it during build
include_directories(${CMAKE_BINARY_DIR})


# Generate isa-l.h from a template
# (Keep existing isa-l.h.in generation logic here)
# Ensure isa-l.h.in is created before it's configured
file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/include/isa-l.h.in"
"
/**
 *  @file isa-l.h
 *  @brief Include for ISA-L library
 */

#ifndef _ISAL_H_
#define _ISAL_H_

#define ISAL_MAJOR_VERSION @PROJECT_VERSION_MAJOR@
#define ISAL_MINOR_VERSION @PROJECT_VERSION_MINOR@
#define ISAL_PATCH_VERSION @PROJECT_VERSION_PATCH@
#define ISAL_MAKE_VERSION(maj, min, patch)  ((maj) * 0x10000 + (min) * 0x100 + (patch))
#define ISAL_VERSION ISAL_MAKE_VERSION(ISAL_MAJOR_VERSION, ISAL_MINOR_VERSION, ISAL_PATCH_VERSION)

// Headers from 'include' directory, prefixed with isa-l/ by install rules or include structure
#include <isa-l/crc.h>
#include <isa-l/crc64.h>
#include <isa-l/erasure_code.h>
#include <isa-l/gf_vect_mul.h>
#include <isa-l/igzip_lib.h>
#include <isa-l/mem_routines.h>
#include <isa-l/raid.h>
// #include <isa-l/types.h> // Placeholder from earlier, review if needed. Standard types are usually sufficient.
// Let's assume types.h is not strictly needed for now or provided by individual module headers if necessary.

#endif //_ISAL_H_
"
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/include/isa-l.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/isa-l.h"
)
# This installs the generated isa-l.h into the final include directory.
# The headers from the 'include' directory itself also need to be installed.
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/isa-l.h" DESTINATION include/isa-l) # Install into 'isa-l' subdirectory
install(
    DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
    PATTERN "isa-l.h.in" EXCLUDE # Don't install the template
)


# Configure pkg-config file
# (Keep existing libisal.pc.in configuration and installation)
set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix \${prefix})
set(libdir \${exec_prefix}/${CMAKE_INSTALL_LIBDIR})
set(includedir \${prefix}/include) # pkg-config should point to the main include dir
configure_file(libisal.pc.in libisal.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libisal.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)


# Add subdirectories for modules that produce OBJECT libraries
add_subdirectory(erasure_code)
add_subdirectory(raid)
add_subdirectory(crc)
add_subdirectory(igzip)
add_subdirectory(mem)

# --- Define the main isal library ---
# Collect all sources from OBJECT libraries.
# Each module (erasure_code, raid, crc, igzip, mem) is an OBJECT library.
# These object files will be linked together into the static 'isal' library.
add_library(isal_static STATIC
    $<TARGET_OBJECTS:erasure_code>
    $<TARGET_OBJECTS:raid>
    $<TARGET_OBJECTS:crc>
    $<TARGET_OBJECTS:igzip>
    $<TARGET_OBJECTS:mem>
)
set_target_properties(isal_static PROPERTIES OUTPUT_NAME isal)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
if(BUILD_SHARED_LIBS)
    set(ISAL_SHARED_SOURCES
        $<TARGET_OBJECTS:erasure_code>
        $<TARGET_OBJECTS:raid>
        $<TARGET_OBJECTS:crc>
        $<TARGET_OBJECTS:igzip>
        $<TARGET_OBJECTS:mem>
    )
    if(MSVC AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/isa-l.rc")
        list(APPEND ISAL_SHARED_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/isa-l.rc")
    endif()
    add_library(isal_shared SHARED ${ISAL_SHARED_SOURCES})
    set_target_properties(isal_shared PROPERTIES OUTPUT_NAME isal)

    if(WIN32 AND NOT MINGW) # MSVC specific .def file handling
        if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/isa-l.def")
            # For MSVC, the DEF file is usually passed to the linker.
            # This can be done via source file properties or linker flags.
            # Adding it as a source file to the library target sometimes works for MSVC.
            # Or, more robustly:
            set_target_properties(isal_shared PROPERTIES LINK_FLAGS "/DEF:${CMAKE_CURRENT_SOURCE_DIR}/isa-l.def")
        endif()
    endif()
    # Set versioning for shared library (similar to autotools LIBISAL_CURRENT etc.)
    # LIBISAL_CURRENT=2, LIBISAL_REVISION=31, LIBISAL_AGE=0
    # Version: 2.31.0 => SOVERSION = 2.0.31 (approx) or just 2
    set_target_properties(isal_shared PROPERTIES VERSION "2.31.0" SOVERSION "2") # Adjust as per libtool versioning
endif()

# Alias library for simpler linking if only one type is built, or for users to choose 'isal'
# If both static and shared are built, this alias might not be strictly necessary,
# as users can link ISA_L::isal_static or ISA_L::isal_shared.
# For now, let's focus on installing the actual targets.

# --- Installation ---
# Install static library
install(TARGETS isal_static
    EXPORT isa-l-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    # PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/isa-l # Headers are installed separately now
)

# Install shared library if built
if(BUILD_SHARED_LIBS)
    install(TARGETS isal_shared
        EXPORT isa-l-targets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} # For .so/.dll
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} # For .dll on Windows
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} # For .lib import library on Windows
    )
endif()

# Install CMake package configuration files
install(EXPORT isa-l-targets
    FILE isa-l-config.cmake # Changed from isa-l-targets.cmake for convention (projectname-config.cmake)
    NAMESPACE ISA_L::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/isa-l
)

# Create isa-l-config-version.cmake
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/isa-l-config-version.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/isa-l-config-version.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/isa-l
)


# --- Subdirectories for tests, examples, programs ---
# These should link against 'isal_static' or 'isal_shared' (or an alias).
# To simplify, we can provide an alias that links to static by default, or shared if built.
# Let's define an alias 'isal' that points to the static library for now.
# If shared is built, it could point to shared.
# Or, downstream targets (tests/examples) can choose.
# For simplicity, let's assume they will link to isal_static for now.
# This means in tests/examples CMakeLists, they should link to 'isal_static' not 'isal'.
# This subtask is only for root CMakeLists.txt, so we'll assume that change is done separately or tests link to ISA_L::isal_static.

# The old INTERFACE library 'isal' is no longer needed.
# The 'target_link_libraries(isal INTERFACE erasure_code raid crc igzip mem)' is removed.

if(BUILD_TESTS)
    add_subdirectory(tests)
endif()

if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

add_subdirectory(programs)

# ... (rest of the file, e.g. CPack setup if any)
# The part generating include/isa-l.h.in and include/types.h was moved up
# to ensure isa-l.h.in is available before configure_file and
# to group header handling logic.
# The placeholder for types.h is removed based on the new isa-l.h.in content.
