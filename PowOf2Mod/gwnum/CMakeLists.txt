cmake_minimum_required(VERSION 3.16)
project(gwnum VERSION 30.19.0 LANGUAGES C CXX)

# Set C standard
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Compiler flags
set(CMAKE_C_FLAGS_RELEASE "-O3 -DNDEBUG")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
set(CMAKE_C_FLAGS_DEBUG "-g -O0 -DGDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0 -DGDEBUG")

# Platform-specific settings
if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64")
    set(ARCH_FLAGS "-march=native")
    add_definitions(-DX86_64)
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "i386|i686")
    set(ARCH_FLAGS "-march=native")
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|ARM64")
    set(ARCH_FLAGS "-march=native")
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "arm")
    set(ARCH_FLAGS "-march=native")
endif()

# Common flags
set(COMMON_FLAGS "${ARCH_FLAGS} -fPIC")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_FLAGS}")

# Enable optimizations
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -flto")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -flto")
endif()

# Include directories
include_directories(${CMAKE_SOURCE_DIR})

# Source files
set(GWNUM_SOURCES
    cpuid.c
    gwnum.c
    gwtables.c
    gwthread.cpp
    gwini.c
    gwbench.c
    gwutil.c
    gwdbldbl.cpp
    giants.c
    radix.c
    ecmstag1.c
)

set(POLYMULT_SOURCES
    polymult.c
)

# Create object files for different SIMD variants
add_custom_command(
    OUTPUT polymult_sse2.o
    COMMAND ${CMAKE_C_COMPILER} ${CMAKE_C_FLAGS} -DSSE2 -msse2 -c ${CMAKE_SOURCE_DIR}/polymult.c -o polymult_sse2.o
    DEPENDS polymult.c
    COMMENT "Building polymult_sse2.o"
)

add_custom_command(
    OUTPUT polymult_avx.o
    COMMAND ${CMAKE_C_COMPILER} ${CMAKE_C_FLAGS} -DAVX -mavx -c ${CMAKE_SOURCE_DIR}/polymult.c -o polymult_avx.o
    DEPENDS polymult.c
    COMMENT "Building polymult_avx.o"
)

add_custom_command(
    OUTPUT polymult_fma.o
    COMMAND ${CMAKE_C_COMPILER} ${CMAKE_C_FLAGS} -DFMA -mfma -c ${CMAKE_SOURCE_DIR}/polymult.c -o polymult_fma.o
    DEPENDS polymult.c
    COMMENT "Building polymult_fma.o"
)

# Create libraries
add_library(gwnum STATIC ${GWNUM_SOURCES})
add_library(polymult STATIC ${POLYMULT_SOURCES} polymult_sse2.o polymult_avx.o polymult_fma.o)

# Set library properties
set_target_properties(gwnum PROPERTIES
    OUTPUT_NAME gwnum
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
)

set_target_properties(polymult PROPERTIES
    OUTPUT_NAME polymult
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
)

# Link libraries
target_link_libraries(gwnum m)
target_link_libraries(polymult m)

# Installation
install(TARGETS gwnum polymult
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(FILES 
    gwnum.h
    gwcommon.h
    gwthread.h
    giants.h
    gwini.h
    gwutil.h
    gwdbldbl.h
    gwtables.h
    radix.h
    polymult.h
    DESTINATION include/gwnum
)

# Testing
enable_testing()
# Add tests here when implemented

# Package configuration
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/gwnum-config-version.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/gwnum-config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/gwnum-config.cmake"
    INSTALL_DESTINATION lib/cmake/gwnum
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/gwnum-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/gwnum-config-version.cmake"
    DESTINATION lib/cmake/gwnum
)
