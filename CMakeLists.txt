cmake_minimum_required(VERSION 3.21..3.25)

option(SF_RELEASE "Builds as an official release" OFF)
option(SF_64_BITS "64-/32-bit operating system" ON)
option(SF_PREFETCH "Use prefetch asm-instruction" OFF) # ON??
option(SF_POPCNT "Use popcnt asm-instruction" OFF)
option(SF_PEXT "Use pext x86_64 asm-instruction" OFF)
option(SF_SSE "Use Intel Streaming SIMD Extensions" OFF)
option(SF_MMX "Use Intel MMX instructions" OFF)
option(SF_SSE2 "Use Intel Streaming SIMD Extensions 2" OFF)
option(SF_SSSE3 "Use Intel Supplemental Streaming SIMD Extensions 3" OFF)
option(SF_SSE41 "Use Intel Streaming SIMD Extensions 4.1" OFF)
option(SF_AVX2 "Use Intel Advanced Vector Extensions 2" OFF)
option(SF_AVXVNNI "Use Intel Vector Neural Network Instructions AVX" OFF)
option(SF_AVX512 "Use Intel Advanced Vector Extensions 512" OFF)
option(SF_VNNI256 "Use Intel Vector Neural Network Instructions 512 with 256bit operands" OFF)
option(SF_VNNI512 "Use Intel Vector Neural Network Instructions 512" OFF)
option(SF_NEON "Use ARM SIMD architecture" OFF)
option(SF_DOTPROD "Use ARM advanced SIMD Int8 dot product instructions" OFF)
set(SF_NNUE "" CACHE STRING "Path to the NNUE database to use. Leave empty if unsure.")

set(CMAKE_CXX_STANDARD 17 CACHE STRING "C++ Standard")
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_OSX_DEPLOYMENT_TARGET 10.14)

# The default NNUE database to use. Is downloaded automatically if missing.
set(SF_DEFAULT_NNUE "nn-8d69132723e2.nnue")

project(Stockfish VERSION 15.2
    DESCRIPTION "UCI chess engine"
    HOMEPAGE_URL https://stockfishchess.org/
    LANGUAGES CXX)

# Create the actual executable
add_executable(stockfish)
add_subdirectory(src)

# Download the latest NNUE if necessary.
if(NOT SF_NNUE)
    set(SF_NNUE_DB "${CMAKE_BINARY_DIR}/${SF_DEFAULT_NNUE}")

    # Only download the database if it doesn't exit.
    if(NOT EXISTS "${SF_NNUE_DB}")
        file(DOWNLOAD "https://tests.stockfishchess.org/api/nn/${SF_DEFAULT_NNUE}" "${SF_NNUE_DB}"
            STATUS SF_NNUE_DOWNLOAD_STATUS
            SHOW_PROGRESS)
        # TODO If the download failed for whatever reason, try using the GitHub mirror.
    endif()
else()
    if(NOT EXISTS "${SF_NNUE}")
        message(FATAL_ERROR "NNUE database path doesn't exist. Does the file exist?")
    endif()
    set(SF_NNUE_DB "${SF_NNUE}")
endif()

# Determine the correct version string.
if(SF_RELEASE)
    set(SF_VERSION "${CMAKE_PROJECT_VERSION}")
else()
    execute_process(COMMAND git rev-parse --short HEAD
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        OUTPUT_VARIABLE SF_HASH
        OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND git show -s --date=format:%Y%m%d --format=%cd HEAD
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        OUTPUT_VARIABLE SF_DATE
        OUTPUT_STRIP_TRAILING_WHITESPACE)
    set(SF_VERSION "dev-${SF_DATE}-${SF_HASH}")
endif()

configure_file(src/nnue.h.in "${CMAKE_SOURCE_DIR}/src/nnue.h" @ONLY)
configure_file(src/version.h.in "${CMAKE_SOURCE_DIR}/src/version.h" @ONLY)