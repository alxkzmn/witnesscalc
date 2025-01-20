cmake_minimum_required(VERSION 3.5)

include(cmake/platform.cmake)

# Read CIRCUIT_NAMES from an environment variable
if (DEFINED ENV{CIRCUIT_NAMES})
    set(CIRCUIT_NAMES $ENV{CIRCUIT_NAMES})
else()
    message(FATAL_ERROR "Environment variable CIRCUIT_NAMES is not set.")
endif()

if (NOT TARGET_PLATFORM MATCHES "android_x86_32")
    set(USE_ASM ON CACHE BOOL "Use asm implementation for Fr and Fq")
endif()

project(witnesscalc LANGUAGES CXX ASM)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

message("USE_ASM=" ${USE_ASM})
message("CMAKE_CROSSCOMPILING=" ${CMAKE_CROSSCOMPILING})

message("GMP_PREFIX=" ${GMP_PREFIX})
message("GMP_INCLUDE_DIR=" ${GMP_INCLUDE_DIR})
message("GMP_LIB_DIR=" ${GMP_LIB_DIR})

if (NOT EXISTS ${GMP_INCLUDE_FILE_FULLPATH})
    message("WARNING: ${GMP_INCLUDE_FILE_FULLPATH} is not found and so system ${GMP_INCLUDE_FILE} is used.")
endif()

if (NOT EXISTS ${GMP_LIB_FILE_FULLPATH})
    message("WARNING: ${GMP_LIB_FILE_FULLPATH} is not found and so system ${GMP_LIB_FILE} is used.")
    set(GMP_LIB gmp)
endif()


include_directories(BEFORE ${GMP_INCLUDE_DIR})

add_subdirectory(src)


# Split CIRCUIT_NAMES into a list
string(REPLACE ";" ";" CIRCUIT_NAMES "${CIRCUIT_NAMES}")
message(STATUS "CIRCUIT_NAMES: ${CIRCUIT_NAMES}")

foreach(CIRCUIT_NAME ${CIRCUIT_NAMES})
    message(STATUS "Configuring targets for circuit: ${CIRCUIT_NAME}")

    install(TARGETS
        ${CIRCUIT_NAME}
        witnesscalc_${CIRCUIT_NAME}
        witnesscalc_${CIRCUIT_NAME}Static
        RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
        BUNDLE DESTINATION ${CMAKE_INSTALL_PREFIX}/app
        LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
endforeach()

install(TARGETS
    tests
    test_platform
    fr
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
    BUNDLE DESTINATION ${CMAKE_INSTALL_PREFIX}/app
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

install(FILES "${GMP_LIB_DIR}/${GMP_LIB_FILE}"
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)


foreach(CIRCUIT_NAME ${CIRCUIT_NAMES})
    message(STATUS "Configuring files for circuit: ${CIRCUIT_NAME}")

    install(FILES
        src/${CIRCUIT_NAME}.dat
        DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)
endforeach()


foreach(CIRCUIT_NAME ${CIRCUIT_NAMES})
    message(STATUS "Configuring headers for circuit: ${CIRCUIT_NAME}")

    install(FILES
        src/witnesscalc_${CIRCUIT_NAME}.h
        DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
endforeach()

install(FILES
    src/witnesscalc.h
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
