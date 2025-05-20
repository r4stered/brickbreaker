cmake_minimum_required(VERSION 3.25)
project(brickbreaker VERSION 0.1.0 LANGUAGES CXX)

# Require C++23
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Generate compile_commands.json for tooling
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Options
option(ENABLE_WARNINGS "Enable warnings" ON)
option(ENABLE_WERROR "Treat warnings as errors" OFF)
option(ENABLE_TESTING "Enable testing" ON)
option(ENABLE_SANITIZER_ADDRESS "Enable address sanitizer" OFF)
option(ENABLE_SANITIZER_UNDEFINED "Enable undefined behavior sanitizer" OFF)
option(ENABLE_SANITIZER_THREAD "Enable thread sanitizer" OFF)

# Set default build type if not specified
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "Build type" FORCE)
endif()

find_package(raylib QUIET)
if(NOT raylib_FOUND)
    include(FetchContent)
    FetchContent_Declare(
        raylib
        GIT_REPOSITORY https://github.com/raysan5/raylib.git
        GIT_TAG
            c1ab645 # 5.5
        GIT_SHALLOW 1
        SYSTEM
    )
    FetchContent_MakeAvailable(raylib)
endif()

find_package(raylib_cpp QUIET)
if(NOT raylib_cpp_FOUND)
    include(FetchContent)
    FetchContent_Declare(
        raylib_cpp
        GIT_REPOSITORY https://github.com/RobLoach/raylib-cpp.git
        GIT_TAG
            6d9d02c # 5.5
        SYSTEM
    )
    FetchContent_MakeAvailable(raylib_cpp)
endif()

FetchContent_Declare(
    doctest
    GIT_REPOSITORY https://github.com/doctest/doctest
    GIT_TAG v2.4.11
)
FetchContent_MakeAvailable(doctest)
include("${doctest_SOURCE_DIR}/scripts/cmake/doctest.cmake")

# Add executable
add_executable(${PROJECT_NAME} src/main.cpp)

# Include directories
target_include_directories(
    ${PROJECT_NAME}
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# Set properties
set_target_properties(
    ${PROJECT_NAME}
    PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)

# Warnings and compiler flags
if(ENABLE_WARNINGS)
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        target_compile_options(
            ${PROJECT_NAME}
            PRIVATE
                -Wall
                -Wextra
                -Wpedantic
                -Wshadow
                -Wconversion
                -Wsign-conversion
        )
        if(ENABLE_WERROR)
            target_compile_options(${PROJECT_NAME} PRIVATE -Werror)
        endif()
    elseif(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
        target_compile_options(${PROJECT_NAME} PRIVATE /W4)
        if(ENABLE_WERROR)
            target_compile_options(${PROJECT_NAME} PRIVATE /WX)
        endif()
    endif()
endif()

# Sanitizers
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    if(ENABLE_SANITIZER_ADDRESS)
        target_compile_options(
            ${PROJECT_NAME}
            PRIVATE -fsanitize=address -fno-omit-frame-pointer
        )
        target_link_options(${PROJECT_NAME} PRIVATE -fsanitize=address)
    endif()

    if(ENABLE_SANITIZER_UNDEFINED)
        target_compile_options(${PROJECT_NAME} PRIVATE -fsanitize=undefined)
        target_link_options(${PROJECT_NAME} PRIVATE -fsanitize=undefined)
    endif()

    if(ENABLE_SANITIZER_THREAD)
        target_compile_options(${PROJECT_NAME} PRIVATE -fsanitize=thread)
        target_link_options(${PROJECT_NAME} PRIVATE -fsanitize=thread)
    endif()
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE raylib raylib_cpp)

# Testing
if(ENABLE_TESTING)
    enable_testing()

    # Add executable
    add_executable(${PROJECT_NAME}_tests src/test/tests.cpp)

    target_link_libraries(${PROJECT_NAME}_tests PRIVATE doctest::doctest)

    doctest_discover_tests(${PROJECT_NAME}_tests ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
endif()

# Install rules
install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)
