cmake_minimum_required(VERSION 3.11...3.13)
if(${CMAKE_VERSION} VERSION_LESS 3.12)
    cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()

include(FetchContent)

if(${CMAKE_VERSION} VERSION_LESS 3.14)
    macro(FetchContent_MakeAvailable NAME)
        FetchContent_GetProperties(${NAME})
        if(NOT ${NAME}_POPULATED)
            FetchContent_Populate(${NAME})
            add_subdirectory(${${NAME}_SOURCE_DIR} ${${NAME}_BINARY_DIR})
        endif()
    endmacro()
endif()

FetchContent_Declare(
    sqlite_zstd_vfs
    GIT_REPOSITORY  https://github.com/mlin/sqlite_zstd_vfs.git
    GIT_TAG         f2cead5
)
FetchContent_MakeAvailable(sqlite_zstd_vfs)
include_directories(${sqlite_zstd_vfs_SOURCE_DIR}/src)

project(genomicsqlite VERSION 1.0
        DESCRIPTION "Genomics Extensions for SQLite"
        LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

execute_process(COMMAND git describe --tags --long --always
                OUTPUT_VARIABLE GIT_REVISION OUTPUT_STRIP_TRAILING_WHITESPACE)
                set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DGIT_REVISION=\"\\\"${GIT_REVISION}\\\"\"")

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
add_library(genomicsqlite SHARED src/genomicsqlite.cc include/genomicsqlite.h)
target_link_libraries(genomicsqlite PRIVATE SQLiteCpp zstd)

add_subdirectory(loaders)

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    include(CTest)
    enable_testing()
    add_test(NAME pytest COMMAND env PYTHONPATH=$PYTHONPATH:${CMAKE_CURRENT_SOURCE_DIR}/bindings/python LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${CMAKE_BINARY_DIR} python3 -m pytest -v -n 4 --tb=short ${CMAKE_CURRENT_SOURCE_DIR}/test)
endif()