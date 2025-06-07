cmake_minimum_required(VERSION 3.12)
project(isal VERSION 2.31.1 LANGUAGES C ASM_NASM)

# --- Versioning ---
# Using project version directly for now. Autotools version:
# LIBISAL_CURRENT=2
# LIBISAL_REVISION=31
# LIBISAL_AGE=0
# CMake SOVERSION will be CURRENT - AGE = 2. VERSION will be C.R.A (2.31.0 for libisal.so.2.31.0)
# For libisal.so.2, this means SOVERSION = 2.
set(LIBISAL_CURRENT 2)
set(LIBISAL_REVISION 31) # Corresponds to minor version in Autotools
set(LIBISAL_AGE 0)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# --- Compiler Flags ---
# Flags from configure.ac: -Wall -Wchar-subscripts -Wformat-security -Wnested-externs -Wpointer-arith -Wshadow -Wstrict-prototypes -Wtype-limits -fstack-protector -D_FORTIFY_SOURCE=2
set(COMMON_COMPILE_FLAGS "-Wall -Wchar-subscripts -Wformat-security -Wnested-externs -Wpointer-arith -Wshadow -Wstrict-prototypes -Wtype-limits -fstack-protector -D_FORTIFY_SOURCE=2")
add_compile_options(${COMMON_COMPILE_FLAGS})

# For x86_64 specific flags if needed later
if(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "x86_64" OR CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64")
    add_compile_options(-Dx86_64)
    # NASM specific options for x86-64
    # From configure.ac: asm_args="-f elf64" for Linux
    # For Windows: asm_args="-f win64"
    # For Darwin: asm_args="-f macho64 --prefix=_ "
    if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
        set(CMAKE_ASM_NASM_OBJECT_FORMAT elf64)
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
        set(CMAKE_ASM_NASM_OBJECT_FORMAT macho64)
        set(CMAKE_ASM_NASM_FLAGS "${CMAKE_ASM_NASM_FLAGS} --prefix=_")
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
        set(CMAKE_ASM_NASM_OBJECT_FORMAT win64)
    endif()
else()
    message(FATAL_ERROR "This CMake build currently only supports x86_64 architecture.")
endif()

# --- Include Directories ---
include_directories(include) # Global include directory

# --- Generate isa-l.h ---
# Based on Makefile.am rule for isa-l.h
set(ISA_L_H_INPUT_HEADERS
    include/crc.h
    include/crc64.h
    include/erasure_code.h
    include/gf_vect_mul.h
    include/igzip_lib.h
    include/mem_routines.h
    # include/memcpy.asm # This is an .asm file, not a header for isa-l.h
    # include/multibinary.asm # This is an .asm file
    include/raid.h
    # include/reg_sizes.asm # This is an .asm file
    include/test.h # This is pkginclude_HEADERS in Makefile.am, but also listed in isa-l.h generation
    include/unaligned.h
    # Headers from submodules if they have their own specific public part,
    # but current isa-l.h seems to only list files directly in include/ or isa-l/
)

# The Makefile.am script sorts them and then adds #include <isa-l/header.h>
# For CMake, we can collect them, sort, and then generate the content.

# Get a list of base names
set(ISA_L_H_BASE_HEADERS "")
foreach(header_file ${ISA_L_H_INPUT_HEADERS})
    get_filename_component(header_basename ${header_file} NAME)
    list(APPEND ISA_L_H_BASE_HEADERS ${header_basename})
endforeach()
list(SORT ISA_L_H_BASE_HEADERS)

set(ISA_L_H_CONTENT "")
string(APPEND ISA_L_H_CONTENT "/**\n *  @file isa-l.h\n *  @brief Include for ISA-L library\n */\n\n")
string(APPEND ISA_L_H_CONTENT "#ifndef _ISAL_H_\n#define _ISAL_H_\n\n")
string(APPEND ISA_L_H_CONTENT "#define ISAL_MAJOR_VERSION ${PROJECT_VERSION_MAJOR}\n")
string(APPEND ISA_L_H_CONTENT "#define ISAL_MINOR_VERSION ${PROJECT_VERSION_MINOR}\n")
string(APPEND ISA_L_H_CONTENT "#define ISAL_PATCH_VERSION ${PROJECT_VERSION_PATCH}\n")
string(APPEND ISA_L_H_CONTENT "#define ISAL_MAKE_VERSION(maj, min, patch)  ((maj) * 0x10000 + (min) * 0x100 + (patch))\n")
string(APPEND ISA_L_H_CONTENT "#define ISAL_VERSION ISAL_MAKE_VERSION(ISAL_MAJOR_VERSION, ISAL_MINOR_VERSION, ISAL_PATCH_VERSION)\n\n")

foreach(base_header ${ISA_L_H_BASE_HEADERS})
    string(APPEND ISA_L_H_CONTENT "#include <isa-l/${base_header}>\n")
endforeach()
string(APPEND ISA_L_H_CONTENT "\n#endif //_ISAL_H_\n")

set(ISA_L_H_GENERATED_FILE ${CMAKE_CURRENT_BINARY_DIR}/isa-l.h)
file(WRITE ${ISA_L_H_GENERATED_FILE} "${ISA_L_H_CONTENT}")

# Add the directory containing the generated isa-l.h to include path
include_directories(${CMAKE_CURRENT_BINARY_DIR})


# --- Subdirectories ---
# These will be added as modules are created
add_subdirectory(crc)
add_subdirectory(igzip)
add_subdirectory(erasure_code)
add_subdirectory(raid)
add_subdirectory(mem)
# add_subdirectory(examples) # For later, if we add example builds
# add_subdirectory(tests) # For later, if we add test builds


# --- Final Library ---
# This will collect all module static libraries into the main shared library
# The actual add_library for isal (shared) will be here,
# target_link_libraries will link crc, igzip etc.
# For now, create a placeholder for the library target that sub-modules can link to.
# We will define it properly after all sub-modules are processed.
add_library(isal_core INTERFACE) # Placeholder for components to link against initially

# --- Final Library Definition ---
# Retrieve the list of module libraries collected from subdirectories
get_property(ISAL_MODULE_LIBRARIES GLOBAL PROPERTY ISAL_MODULE_LIBRARIES)
if(NOT ISAL_MODULE_LIBRARIES)
    message(WARNING "No module libraries found for ISAL. Final library will be empty or may fail to link.")
endif()

message(STATUS "Collected ISAL module libraries: ${ISAL_MODULE_LIBRARIES}")

# Add the shared library 'isal' compiled from the module libraries
add_library(isal SHARED "") # No sources directly in this target, it's a collection of other libs

# Link all the collected module static libraries to the main 'isal' shared library.
# Also link isal_core to propagate its usage requirements (like include directories from generated isa-l.h)
# to the final shared library and anything that links to it.
target_link_libraries(isal PUBLIC ${ISAL_MODULE_LIBRARIES} isal_core)

# Set library version properties
# LIBISAL_CURRENT=2 (affects SOVERSION)
# LIBISAL_REVISION=31 (affects filename and VERSION field)
# LIBISAL_AGE=0 (affects SOVERSION)
# Result: libisal.so.2.31.x (PROJECT_VERSION_PATCH)
# SOVERSION = LIBISAL_CURRENT - LIBISAL_AGE (e.g. 2 - 0 = 2)
# VERSION = ${LIBISAL_CURRENT}.${LIBISAL_REVISION}.${PROJECT_VERSION_PATCH} (e.g. 2.31.1 based on project version)
# However, CMake's VERSION property expects major.minor.patch.
# Let's use PROJECT_VERSION for the VERSION property, which is already 2.31.1
# SOVERSION should be LIBISAL_CURRENT (which is 2).
set_target_properties(isal PROPERTIES
    VERSION ${PROJECT_VERSION} # Full version like 2.31.1
    SOVERSION ${LIBISAL_CURRENT}  # API version, results in libisal.so.2
    OUTPUT_NAME "isal" # Ensures the library is named libisal.so
)

# The isal_core INTERFACE library was used by modules to access global include directories
# like the one for the generated isa-l.h. By linking isal_core to 'isal' PUBLICLY,
# these usage requirements are propagated. The original add_library(isal_core INTERFACE)
# defined before add_subdirectory() calls, served its purpose for the modules and
# does not need to be an alias or modified further.

message(STATUS "Project Name: ${PROJECT_NAME}")
message(STATUS "Project Version: ${PROJECT_VERSION}")
message(STATUS "Compiler: ${CMAKE_C_COMPILER}")
message(STATUS "C Flags: ${CMAKE_C_FLAGS} ${COMMON_COMPILE_FLAGS}")
message(STATUS "NASM Flags: ${CMAKE_ASM_NASM_FLAGS}")
message(STATUS "Generated isa-l.h at: ${ISA_L_H_GENERATED_FILE}")

# --- Testing ---
# Enable testing for the project
enable_testing()
add_subdirectory(examples/cmake_test)

# --- Installation Rules ---
include(GNUInstallDirs)

# Install the shared library and any runtime components
install(TARGETS isal
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT libisal_runtime
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT libisal_libraries
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT libisal_devel # For static lib or import lib on Windows
)

# Install the generated public API header (isa-l.h)
install(FILES ${ISA_L_H_GENERATED_FILE}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    RENAME isa-l.h # Ensure it's named isa-l.h in the install include dir
    COMPONENT libisal_devel
)

# Install the other public headers from include/isa-l/
# These are the headers that isa-l.h includes.
# They should be installed into an isa-l subdirectory of the main include directory.
install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/isa-l
    FILES_MATCHING PATTERN "*.h"
    COMPONENT libisal_devel
    # Exclude .asm files if any were in include/ (they are not in ISA-L's include/)
)

# --- Pkg-config file ---
# Use relative paths for libdir and includedir from CMAKE_INSTALL_PREFIX
# CMAKE_INSTALL_LIBDIR_REL and CMAKE_INSTALL_INCLUDEDIR_REL are defined by GNUInstallDirs
# and are relative to CMAKE_INSTALL_PREFIX (e.g., "lib" or "include")
set(PC_FILE_CONTENT "prefix=${CMAKE_INSTALL_PREFIX}\n"
"exec_prefix=\${prefix}\n"
"libdir=\${exec_prefix}/${CMAKE_INSTALL_LIBDIR_REL}\n"
"includedir=\${prefix}/${CMAKE_INSTALL_INCLUDEDIR_REL}\n"
"\n"
"Name: libisal\n"
"Description: Intel(R) Intelligent Storage Acceleration Library\n"
"Version: ${PROJECT_VERSION}\n" # Use the full project version
"Libs: -L\${libdir} -lisal\n"
"Cflags: -I\${includedir} -I\${includedir}/isa-l\n" # Added isa-l subdir for Cflags
)

# Write the libisal.pc file directly to the build directory
set(LIBISAL_PC_FILE ${CMAKE_CURRENT_BINARY_DIR}/libisal.pc)
file(WRITE ${LIBISAL_PC_FILE} "${PC_FILE_CONTENT}")

install(FILES ${LIBISAL_PC_FILE}
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig # Standard path for pkgconfig files
    COMPONENT libisal_devel
)

message(STATUS "Installation prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "Install Runtimes to: ${CMAKE_INSTALL_BINDIR} (libisal_runtime component)")
message(STATUS "Install Libraries to: ${CMAKE_INSTALL_LIBDIR} (libisal_libraries component)")
message(STATUS "Install Development files (headers, pkgconfig) to: ${CMAKE_INSTALL_INCLUDEDIR}, ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig (libisal_devel component)")
