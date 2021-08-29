cmake_minimum_required(VERSION 3.21)

project(CXX20examples CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_VERBOSE_MAKEFILE 1)

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(GCC 1)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(CLANG 1)
endif()

if (GCC OR CLANG)
    set(CMAKE_CXX_FLAGS "-fmodules-ts")
elseif(MSVC)
    set(CMAKE_CXX_FLAGS "/experimental:module")
endif()

function(add_module name)
    set(bin_dir ${CMAKE_BINARY_DIR}/modules/${name})
    set(sources)

    if (GCC)
        # GCC doesn't understand how to compile .ixx files
        foreach(arg IN LISTS ARGN)
            if (arg MATCHES .*\\.ixx)
                get_filename_component(rel_dir ${arg} DIRECTORY)
                get_filename_component(file_name ${arg} NAME_WLE)
                set(final_file_directory ${bin_dir}/${rel_dir})
                set(final_file_path ${final_file_directory}/${file_name}.cpp)
                file(MAKE_DIRECTORY ${final_file_directory})
                file(COPY_FILE ${arg} ${final_file_path})
                list(APPEND sources ${final_file_path})
            endif()
        endforeach()
    elseif(MSVC)
        # MSVC only understands .ixx files when building c++20 modules
        set(sources ${ARGN})
    else()
        message(FATAL_ERROR "Unsupported compiler: ${CMAKE_CXX_COMPILER_ID}")
    endif()

    add_library(${name} STATIC ${sources})
endfunction()


add_module(math_module src/math.ixx)
add_executable(main_app src/main.cpp)
target_link_libraries(main_app PRIVATE math_module)
