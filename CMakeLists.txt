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
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_COMMON} ${CMAKE_C_FLAGS}")
set(CMAKE_C_FLAGS_DEBUG "-g")
set(CMAKE_C_FLAGS_RELEASE "-O3 -DNDEBUG")

if(ENABLE_DEBUG)
    add_definitions(-DENABLE_DEBUG)
endif()

# Add include directory
include_directories(include)

# Generate isa-l.h
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/include/isa-l.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/isa-l.h"
)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/isa-l.h" DESTINATION include)


# Configure pkg-config file
set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix \${prefix})
set(libdir \${exec_prefix}/${CMAKE_INSTALL_LIBDIR})
set(includedir \${prefix}/include)
configure_file(libisal.pc.in libisal.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libisal.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

# Add subdirectories for modules
add_subdirectory(erasure_code)
add_subdirectory(raid)
add_subdirectory(crc)
add_subdirectory(igzip)
add_subdirectory(mem)

# Add main library target (defined after modules)
add_library(isal INTERFACE) # Changed to INTERFACE, actual library will be an alias or a wrapper.
target_link_libraries(isal INTERFACE erasure_code raid crc igzip mem)
# It might be better to create a SHARED or STATIC library from the module object files.
# This will be refined in a later step.

install(TARGETS isal # This will need to be adjusted based on how isal is defined
    EXPORT isa-l-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(EXPORT isa-l-targets
    FILE isa-l-targets.cmake
    NAMESPACE ISA_L::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/isa-l
)

if(BUILD_TESTS)
    add_subdirectory(tests)
endif()

if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

add_subdirectory(programs)

# Generate isa-l.h from a template
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

#include <isa-l/crc.h>
#include <isa-l/crc64.h>
#include <isa-l/erasure_code.h>
#include <isa-l/gf_vect_mul.h>
#include <isa-l/igzip_lib.h>
#include <isa-l/mem_routines.h>
#include <isa-l/raid.h>
#include <isa-l/types.h> // Assuming a new types.h or similar might be needed

#endif //_ISAL_H_
"
)

# Create a placeholder for types.h if it doesn't exist, as isa-l.h.in references it.
# This might need to be adjusted based on actual header dependencies.
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/include/types.h")
    file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/include/types.h" "// Placeholder for ISA-L types
#ifndef _ISA_L_TYPES_H_
#define _ISA_L_TYPES_H_
#include <stdint.h>
#endif
")
endif()
