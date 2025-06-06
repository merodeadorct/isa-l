# erasure_code/CMakeLists.txt

# List of common source files for the erasure_code module
set(EC_SOURCES
    ec_base.c
)

# List of source files for x86_64 architecture
set(EC_SOURCES_X86_64
    ec_highlevel_func.c
    gf_vect_mul_sse.asm
    gf_vect_mul_avx.asm
    gf_vect_dot_prod_sse.asm
    gf_vect_dot_prod_avx.asm
    gf_vect_dot_prod_avx2.asm
    gf_2vect_dot_prod_sse.asm
    gf_3vect_dot_prod_sse.asm
    gf_4vect_dot_prod_sse.asm
    gf_5vect_dot_prod_sse.asm
    gf_6vect_dot_prod_sse.asm
    gf_2vect_dot_prod_avx.asm
    gf_3vect_dot_prod_avx.asm
    gf_4vect_dot_prod_avx.asm
    gf_5vect_dot_prod_avx.asm
    gf_6vect_dot_prod_avx.asm
    gf_2vect_dot_prod_avx2.asm
    gf_3vect_dot_prod_avx2.asm
    gf_4vect_dot_prod_avx2.asm
    gf_5vect_dot_prod_avx2.asm
    gf_6vect_dot_prod_avx2.asm
    gf_vect_mad_sse.asm
    gf_2vect_mad_sse.asm
    gf_3vect_mad_sse.asm
    gf_4vect_mad_sse.asm
    gf_5vect_mad_sse.asm
    gf_6vect_mad_sse.asm
    gf_vect_mad_avx.asm
    gf_2vect_mad_avx.asm
    gf_3vect_mad_avx.asm
    gf_4vect_mad_avx.asm
    gf_5vect_mad_avx.asm
    gf_6vect_mad_avx.asm
    gf_vect_mad_avx2.asm
    gf_2vect_mad_avx2.asm
    gf_3vect_mad_avx2.asm
    gf_4vect_mad_avx2.asm
    gf_5vect_mad_avx2.asm
    gf_6vect_mad_avx2.asm
    ec_multibinary.asm
    # AVX512/GFNI specific files (conditionally added below)
)

set(EC_SOURCES_X86_64_AVX512
    gf_vect_mad_avx2_gfni.asm
    gf_2vect_mad_avx2_gfni.asm
    gf_3vect_mad_avx2_gfni.asm
    gf_4vect_mad_avx2_gfni.asm
    gf_5vect_mad_avx2_gfni.asm
    gf_vect_dot_prod_avx512.asm
    gf_2vect_dot_prod_avx512.asm
    gf_3vect_dot_prod_avx512.asm
    gf_4vect_dot_prod_avx512.asm
    gf_5vect_dot_prod_avx512.asm
    gf_6vect_dot_prod_avx512.asm
    gf_vect_dot_prod_avx512_gfni.asm
    gf_vect_dot_prod_avx2_gfni.asm
    gf_2vect_dot_prod_avx2_gfni.asm
    gf_3vect_dot_prod_avx2_gfni.asm
    gf_2vect_dot_prod_avx512_gfni.asm
    gf_3vect_dot_prod_avx512_gfni.asm
    gf_4vect_dot_prod_avx512_gfni.asm
    gf_5vect_dot_prod_avx512_gfni.asm
    gf_6vect_dot_prod_avx512_gfni.asm
    gf_vect_mad_avx512.asm
    gf_2vect_mad_avx512.asm
    gf_3vect_mad_avx512.asm
    gf_4vect_mad_avx512.asm
    gf_5vect_mad_avx512.asm
    gf_6vect_mad_avx512.asm
    gf_vect_mad_avx512_gfni.asm
    gf_2vect_mad_avx512_gfni.asm
    gf_3vect_mad_avx512_gfni.asm
    gf_4vect_mad_avx512_gfni.asm
    gf_5vect_mad_avx512_gfni.asm
    gf_6vect_mad_avx512_gfni.asm
)

# List of source files for aarch64 architecture
set(EC_SOURCES_AARCH64
    aarch64/ec_aarch64_dispatcher.c
    aarch64/ec_aarch64_highlevel_func.c
    aarch64/ec_multibinary_arm.S
    aarch64/gf_2vect_dot_prod_neon.S
    aarch64/gf_2vect_dot_prod_sve.S
    aarch64/gf_2vect_mad_neon.S
    aarch64/gf_2vect_mad_sve.S
    aarch64/gf_3vect_dot_prod_neon.S
    aarch64/gf_3vect_dot_prod_sve.S
    aarch64/gf_3vect_mad_neon.S
    aarch64/gf_3vect_mad_sve.S
    aarch64/gf_4vect_dot_prod_neon.S
    aarch64/gf_4vect_dot_prod_sve.S
    aarch64/gf_4vect_mad_neon.S
    aarch64/gf_4vect_mad_sve.S
    aarch64/gf_5vect_dot_prod_neon.S
    aarch64/gf_5vect_dot_prod_sve.S
    aarch64/gf_5vect_mad_neon.S
    aarch64/gf_5vect_mad_sve.S
    aarch64/gf_6vect_dot_prod_sve.S # Note: No gf_6vect_dot_prod_neon.S in original listing
    aarch64/gf_6vect_mad_neon.S
    aarch64/gf_6vect_mad_sve.S
    aarch64/gf_7vect_dot_prod_sve.S # Note: No gf_7vect_dot_prod_neon.S in original listing
    aarch64/gf_8vect_dot_prod_sve.S # Note: No gf_8vect_dot_prod_neon.S in original listing
    aarch64/gf_vect_dot_prod_neon.S
    aarch64/gf_vect_dot_prod_sve.S
    aarch64/gf_vect_mad_neon.S
    aarch64/gf_vect_mad_sve.S
    aarch64/gf_vect_mul_neon.S
    aarch64/gf_vect_mul_sve.S
)

# List of source files for ppc64le architecture
set(EC_SOURCES_PPC64LE
    ppc64le/ec_ppc64le_dispatcher.c
    ppc64le/ec_multibinary_ppc.S
    ppc64le/gf_vect_dot_prod_altivec.S
    ppc64le/gf_vect_mad_altivec.S
    ppc64le/gf_vect_mul_altivec.S
    # Add other ppc64le specific files from erasure_code/ppc64le/Makefile.am if any
)

# List of source files for riscv64 architecture
set(EC_SOURCES_RISCV64
    riscv64/ec_riscv64_dispatcher.c
    riscv64/ec_multibinary_riscv64.S
    # Add other riscv64 specific files from erasure_code/riscv64/Makefile.am
)
set(EC_SOURCES_RISCV64_RVV # Specific files if RVV is present
    riscv64/gf_vect_dot_prod_rvv.S
    riscv64/gf_vect_mad_rvv.S
    riscv64/gf_vect_mul_rvv.S
)


# Add the library target for the erasure_code module
add_library(erasure_code OBJECT ${EC_SOURCES}) # OBJECT library for combining later

target_include_directories(erasure_code PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_include_directories(erasure_code PRIVATE ${CMAKE_SOURCE_DIR}/include)


# Conditionally add architecture-specific sources
if(TARGET_ARCH STREQUAL "x86_64")
    target_sources(erasure_code PRIVATE ${EC_SOURCES_X86_64})
    # Check for AVX512 support more reliably here if possible, e.g. via compiler defines or try_compile
    # For now, assume if compiler supports AVX512 flags, we can add these sources.
    # This might need a proper check_cxx_compiler_flag or similar.
    # As a placeholder, we'll rely on a CMake option or an environment variable if available.
    # Or, we can assume that if the compiler is new enough, it supports AVX512.
    # For simplicity, let's add an option, though autotools checks this.
    option(ENABLE_AVX512 "Enable AVX512 compilation for EC" ON)
    if(ENABLE_AVX512)
        target_sources(erasure_code PRIVATE ${EC_SOURCES_X86_64_AVX512})
        target_compile_definitions(erasure_code PRIVATE HAVE_AVX512) # Define for C/C++ code if needed
    endif()
    target_sources(erasure_code PRIVATE ec_base_aliases.c) # Part of lsrc_base_aliases in autotools for undefined CPU, but seems relevant here too for x86 functions
elif(TARGET_ARCH STREQUAL "aarch64")
    target_sources(erasure_code PRIVATE ${EC_SOURCES_AARCH64})
    target_compile_definitions(erasure_code PRIVATE ISA_L_AARCH64)
elif(TARGET_ARCH STREQUAL "ppc64le")
    target_sources(erasure_code PRIVATE ${EC_SOURCES_PPC64LE})
    target_compile_definitions(erasure_code PRIVATE ISA_L_PPC64LE)
elif(TARGET_ARCH STREQUAL "riscv64")
    target_sources(erasure_code PRIVATE ${EC_SOURCES_RISCV64})
    if(HAVE_RVV)
        target_sources(erasure_code PRIVATE ${EC_SOURCES_RISCV64_RVV})
        target_compile_definitions(erasure_code PRIVATE HAVE_RVV)
    endif()
    target_compile_definitions(erasure_code PRIVATE ISA_L_RISCV64)
else()
    # Fallback for undefined or generic architecture (CPU_UNDEFINED in autotools)
    target_sources(erasure_code PRIVATE ec_base_aliases.c)
endif()

# Add this module's object files to the main isal library source list
# This approach might change if we decide to make isal a SHARED/STATIC library directly
# For now, we assume the parent CMakeLists.txt will handle collecting OBJECT libraries.
# A common pattern is to use target_link_libraries with INTERFACE or to collect sources.
# Let's ensure this library is added to a property or list that the main CMake can use.
# Or, the main 'isal' library will explicitly link against 'erasure_code'.
# The root CMakeLists.txt already links 'isal' INTERFACE library to 'erasure_code'.
# So, 'erasure_code' should be a normal STATIC or SHARED library, or an OBJECT library
# that gets linked into a larger library. Let's stick to OBJECT for now.

# If specific ASM flags are needed for this module beyond global ones:
# set_source_files_properties(
#     ${EC_SOURCES_X86_64}
#     PROPERTIES COMPILE_FLAGS "-your-specific-asm-flags"
# )
# For .S files (usually GNU AS syntax), CMake treats them as ASM language.
# If they need to be compiled with C compiler (e.g. for preprocessor directives),
# you might need to adjust settings or use add_custom_command.
# The current autotools setup uses AM_CCASFLAGS for .S files on AArch64/RISC-V,
# implying they are processed by the C compiler.
# For NASM, .asm files are handled by ASM_NASM.
# For .S files on ARM/RISCV, ensure they are compiled correctly.
# CMake's default handling for .S files might be via CMAKE_ASM_COMPILER.
# If C preprocessor is needed for .S files:
# set_source_files_properties(
#    aarch64/ec_multibinary_arm.S # and other .S files
#    PROPERTIES LANGUAGE C) # This tells CMake to use C compiler for these, which runs preprocessor.
# This might be needed for ARM and RISC-V .S files. Let's test this assumption.

if(TARGET_ARCH STREQUAL "aarch64" OR TARGET_ARCH STREQUAL "riscv64" OR TARGET_ARCH STREQUAL "ppc64le")
    set(ARCH_S_FILES "")
    if(TARGET_ARCH STREQUAL "aarch64")
        list(APPEND ARCH_S_FILES ${EC_SOURCES_AARCH64})
    elseif(TARGET_ARCH STREQUAL "ppc64le")
        list(APPEND ARCH_S_FILES ${EC_SOURCES_PPC64LE})
    else() # riscv64
        list(APPEND ARCH_S_FILES ${EC_SOURCES_RISCV64})
        if(HAVE_RVV)
            list(APPEND ARCH_S_FILES ${EC_SOURCES_RISCV64_RVV})
        endif()
    endif()

    foreach(S_FILE ${ARCH_S_FILES})
        if (S_FILE MATCHES "\\.S$")
            # Ensure .S files are compiled with C compiler to handle preprocessing
            set_source_files_properties(${S_FILE} PROPERTIES LANGUAGE C)
            # Add specific flags if necessary, e.g. -D__ASSEMBLY__ if not globally defined
            # In root CMakeLists.txt, -D__ASSEMBLY__ is defined for riscv64.
            # For aarch64, the autotools setup does `as_filter = $(CC) -D__ASSEMBLY__`
            # So, ensure this definition is present for aarch64 .S files
            if(TARGET_ARCH STREQUAL "aarch64")
                 set_source_files_properties(${S_FILE} PROPERTIES COMPILE_DEFINITIONS "__ASSEMBLY__")
            endif()
        endif()
    endforeach()
endif()

# Ensure header files from this module are accessible if needed by other modules
# (though typically only `include` dir is for public headers)
# target_include_directories(erasure_code PRIVATE .) # If there are internal headers in erasure_code/

# Add dependency on header generation if any specific to this module
# add_dependencies(erasure_code generate_ec_headers) # Example
