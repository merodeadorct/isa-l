# mem/CMakeLists.txt

set(MEM_SOURCES
    # Common C files for mem module (if any beyond dispatcher/aliases)
    # From mem/Makefile.am, mem_zero_detect_base.c is a common source.
    mem_zero_detect_base.c
)

set(MEM_SOURCES_X86_64
    mem_multibinary.asm # Contains CPUID dispatchers etc.
    mem_zero_detect_sse.asm
    mem_zero_detect_avx.asm
    mem_zero_detect_avx2.asm
    mem_zero_detect_avx512.asm
    # Potentially other x86_64 specific mem functions if added
)

set(MEM_SOURCES_AARCH64
    aarch64/mem_aarch64_dispatcher.c
    aarch64/mem_multibinary_arm.S
    aarch64/mem_zero_detect_neon.S
)

set(MEM_SOURCES_RISCV64
    riscv64/mem_multibinary_riscv64_dispatcher.c
    riscv64/mem_multibinary_riscv64.S
)

set(MEM_SOURCES_RISCV64_RVV
    riscv64/mem_zero_detect_rvv.S
)

# Add the library target for the mem module
add_library(mem OBJECT ${MEM_SOURCES})

target_include_directories(mem PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> # Should not be needed for mem if all headers are in global include
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_include_directories(mem PRIVATE ${CMAKE_SOURCE_DIR}/include)

if(TARGET_ARCH STREQUAL "x86_64")
    target_sources(mem PRIVATE ${MEM_SOURCES_X86_64})
    target_sources(mem PRIVATE mem_zero_detect_base_aliases.c) # From lsrc_base_aliases
elif(TARGET_ARCH STREQUAL "aarch64")
    target_sources(mem PRIVATE ${MEM_SOURCES_AARCH64})
    target_compile_definitions(mem PRIVATE ISA_L_AARCH64)
elif(TARGET_ARCH STREQUAL "riscv64")
    target_sources(mem PRIVATE ${MEM_SOURCES_RISCV64})
    if(HAVE_RVV)
        target_sources(mem PRIVATE ${MEM_SOURCES_RISCV64_RVV})
        target_compile_definitions(mem PRIVATE HAVE_RVV)
    endif()
    target_compile_definitions(mem PRIVATE ISA_L_RISCV64)
# No PPC64LE specific sources for mem in Makefile.am
else()
    # Fallback for undefined or generic architecture
    target_sources(mem PRIVATE mem_zero_detect_base_aliases.c)
endif()

# Handle .S files for AArch64 and RISC-V (compile as C for preprocessing)
if(TARGET_ARCH STREQUAL "aarch64" OR TARGET_ARCH STREQUAL "riscv64")
    set(ARCH_S_FILES "")
    if(TARGET_ARCH STREQUAL "aarch64")
        list(APPEND ARCH_S_FILES ${MEM_SOURCES_AARCH64})
    else() # riscv64
        list(APPEND ARCH_S_FILES ${MEM_SOURCES_RISCV64})
        if(HAVE_RVV)
            list(APPEND ARCH_S_FILES ${MEM_SOURCES_RISCV64_RVV})
        endif()
    endif()

    foreach(S_FILE ${ARCH_S_FILES})
        if (S_FILE MATCHES "\\.S$")
            set_source_files_properties(${S_FILE} PROPERTIES LANGUAGE C)
            if(TARGET_ARCH STREQUAL "aarch64")
                 set_source_files_properties(${S_FILE} PROPERTIES COMPILE_DEFINITIONS "__ASSEMBLY__")
            endif()
        endif()
    endforeach()
endif()

# Create mem_zero_detect_base_aliases.c if it doesn't exist
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/mem_zero_detect_base_aliases.c")
    file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/mem_zero_detect_base_aliases.c" "// Placeholder for mem_zero_detect_base_aliases.c
#include <mem_routines.h>
// Add minimal required symbols or functions if any
")
endif()
