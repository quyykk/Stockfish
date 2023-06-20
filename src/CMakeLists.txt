if(MSVC)
else()
    target_compile_options(stockfish PRIVATE -Wall -Wcast-qual -fno-exceptions
        -pedantic -Wextra -Wshadow -Wmissing-declarations)
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_compile_options(stockfish PRIVATE -Wconditional-uninitialized)
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "IntelLLVM")
        target_compile_options(stockfish PRIVATE --intel -Wconditional-uninitialized -Wabi -Wdeprecated)
    endif()

    if(NOT WIN32)
        target_compile_definitions(stockfish PRIVATE USE_PTHREADS)
    endif()
    if(CMAKE_SYSTEM_PROCESSOR STREQUAL "riscv64")
        target_link_libraries(stockfish PRIVATE atomic)
    endif()
    if(MINGW)
        target_link_options(stockfish PRIVATE static)
    endif()
    if(NOT MINGW AND (APPLE OR CMAKE_SYSTEM_NAME STREQUAL "FreeBSD" OR CMAKE_SYSTEM_NAME STREQUAL "OpenBSD"))
        # TODO compiler rt check
        target_link_libraries(stockfish PRIVATE atomic)
    endif()
endif()

target_sources(stockfish PRIVATE
    benchmark.cpp
    benchmark.h
    bitbase.cpp
    bitboard.cpp
    bitboard.h
    endgame.cpp
    endgame.h
    evaluate.cpp
    evaluate.h
    main.cpp
	material.cpp
    material.h
    misc.cpp
    misc.h
    movegen.cpp
    movegen.h
    movepick.cpp
    movepick.h
    pawns.cpp
    pawns.h
    position.cpp
    position.h
    psqt.cpp
    psqt.h
	search.cpp
    search.h
    thread_win32_osx.h
    thread.cpp
    thread.h
    timeman.cpp
    timeman.h
    tt.cpp
    tt.h
    tune.cpp
    tune.h
    uci.cpp
    uci.h
    ucioption.cpp
    syzygy/tbprobe.cpp
    syzygy/tbprobe.h
	nnue/evaluate_nnue.cpp
    nnue/evaluate_nnue.h
    nnue/nnue_accumulator.h
    nnue/nnue_architecture.h
    nnue/nnue_common.h
    nnue/nnue_feature_transformer.h
    nnue/features/half_ka_v2_hm.cpp
    nnue/features/half_ka_v2_hm.h
    nnue/layers/affine_transform.h
    nnue/layers/clipped_relu.h
    nnue/layers/simd.h
    nnue/layers/sqr_clipped_relu.h)

# Architecture options
if(SF_64_BITS)
    target_compile_definitions(stockfish PRIVATE IS_64BIT)
endif()
if(NOT SF_PREFETCH AND NOT SF_SSE)
    target_compile_definitions(stockfish PRIVATE NO_PREFETCH)
endif()
if(SF_POPCNT)
    target_compile_definitions(stockfish PRIVATE USE_POPCNT)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -msse3 -mpopcnt)
    endif()
endif()
if(SF_PEXT)
    target_compile_definitions(stockfish PRIVATE USE_PEXT)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -mbmi2)
    endif()
endif()
if(SF_SSE)
    target_compile_definitions(stockfish PRIVATE USE_SSE)
    target_compile_options(stockfish PRIVATE $<IF:$<BOOL:${MSVC}>,/arch:SSE,-msse>)
endif()
if(SF_MMX)
    target_compile_definitions(stockfish PRIVATE USE_MMX)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -mmmx)
    endif()
endif()
if(SF_SSE2)
    target_compile_definitions(stockfish PRIVATE USE_SSE2)
    target_compile_options(stockfish PRIVATE $<IF:$<BOOL:${MSVC}>,/arch:SSE2,-msse2>)
endif()
if(SF_SSSE3)
    target_compile_definitions(stockfish PRIVATE USE_SSSE3)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -mssse3)
    endif()
endif()
if(SF_SSE41)
    target_compile_definitions(stockfish PRIVATE USE_SSE41)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -msse4.1)
    endif()
endif()
if(SF_AVX2)
    target_compile_definitions(stockfish PRIVATE USE_AVX2)
    target_compile_options(stockfish PRIVATE $<IF:$<BOOL:${MSVC}>,/arch:AVX2,-mavx2 -mbmi>)
endif()
if(SF_AVXVNNI)
    target_compile_definitions(stockfish PRIVATE USE_VNNI USE_AVXVNNI)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -mavxvnni)
    endif()
endif()
if(SF_AVX512)
    target_compile_definitions(stockfish PRIVATE USE_AVX512)
    target_compile_options(stockfish PRIVATE $<IF:$<BOOL:${MSVC}>,/arch:AVX512,-mavx512f -mavx512bw>)
endif()
if(SF_VNNI256)
    target_compile_definitions(stockfish PRIVATE USE_VNNI)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -mavx512f -mavx512bw -mavx512vnni -mavx512dq -mavx512vl -mprefer-vector-width=256)
    endif()
endif()
if(SF_VNNI512)
    target_compile_definitions(stockfish PRIVATE USE_VNNI)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -mavx512f -mavx512bw -mavx512vnni -mavx512dq -mavx512vl -mprefer-vector-width=512)
    endif()
endif()
if(SF_NEON)
    target_compile_definitions(stockfish PRIVATE USE_NEON)
    if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
        target_compile_options(stockfish PRIVATE -mfpu=neon)
    endif()
endif()
if(SF_DOTPROD)
    target_compile_definitions(stockfish PRIVATE USE_NEON_DOTPROD)
    if(NOT MSVC)
        target_compile_options(stockfish PRIVATE -march=armv8.2-a+dotprod)
    endif()
endif()