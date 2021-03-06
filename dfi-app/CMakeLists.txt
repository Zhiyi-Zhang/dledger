cmake_minimum_required(VERSION 3.14)

# download wasm
if (NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/wasmtime-c-api)
    if (APPLE)
        # download
        file(DOWNLOAD
                https://github.com/bytecodealliance/wasmtime/releases/download/v0.20.0/wasmtime-v0.20.0-x86_64-macos-c-api.tar.xz
                ${CMAKE_CURRENT_SOURCE_DIR}/wasmtime-c-api.tar.xz)
        execute_process(COMMAND tar -xJvf -
                 INPUT_FILE ${CMAKE_CURRENT_SOURCE_DIR}/wasmtime-c-api.tar.xz
                 WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
        file(RENAME wasmtime-v0.20.0-x86_64-macos-c-api wasmtime-c-api)
    elseif(UNIX)
        # linux download
        file(DOWNLOAD
                https://github.com/bytecodealliance/wasmtime/releases/download/v0.20.0/wasmtime-v0.20.0-x86_64-linux-c-api.tar.xz
                ${CMAKE_CURRENT_SOURCE_DIR}/wasmtime-c-api.tar.xz)
        execute_process(COMMAND tar -xJvf -
                INPUT_FILE ${CMAKE_CURRENT_SOURCE_DIR}/wasmtime-c-api.tar.xz
                WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
        file(RENAME wasmtime-v0.20.0-x86_64-linux-c-api wasmtime-c-api)
    endif()
    file(REMOVE ${CMAKE_CURRENT_SOURCE_DIR}/wasmtime-c-api.tar.xz)
endif()

IF(APPLE)
    set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -D_GNU_SOURCE")
endif()

set(CMAKE_CXX_STANDARD 14)

find_package(PkgConfig REQUIRED)
pkg_check_modules(NDN_CXX REQUIRED libndn-cxx)

# include
include_directories(${NDN_CXX_INCLUDE_DIRS})

# link
link_directories(${NDN_CXX_LIBRARY_DIRS})

include_directories(wasmtime-c-api/include)
link_directories(wasmtime-c-api/lib)

add_executable(ledger-dfi dynamic-function-runner.cpp dfi-ledger.cpp)
target_compile_options(ledger-dfi PUBLIC ${NDN_CXX_CFLAGS})
target_link_libraries(ledger-dfi PUBLIC ${NDN_CXX_LIBRARIES} wasmtime dledger)

add_executable(dfi-test dynamic-function-runner.cpp dfi-test.cpp)
target_compile_options(dfi-test PUBLIC ${NDN_CXX_CFLAGS})
target_link_libraries(dfi-test PUBLIC ${NDN_CXX_LIBRARIES} wasmtime)

if (BUILD_EXAMPLES)
    add_subdirectory(c-test1)
    add_subdirectory(c-filter1)
endif(BUILD_EXAMPLES)