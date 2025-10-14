cmake_minimum_required(VERSION 3.20)
project(dmercator Fortran CXX)

get_filename_component(PARENT_DIR ../ ABSOLUTE)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wshadow -pedantic")

# Find OpenMP - with macOS/Homebrew support
if(APPLE)
    # Try to find libomp from Homebrew
    execute_process(COMMAND brew --prefix libomp OUTPUT_VARIABLE LIBOMP_PREFIX OUTPUT_STRIP_TRAILING_WHITESPACE ERROR_QUIET)
    if(LIBOMP_PREFIX)
        set(OpenMP_C_FLAGS "-Xpreprocessor -fopenmp -I${LIBOMP_PREFIX}/include" CACHE STRING "" FORCE)
        set(OpenMP_C_LIB_NAMES "omp" CACHE STRING "" FORCE)
        set(OpenMP_CXX_FLAGS "-Xpreprocessor -fopenmp -I${LIBOMP_PREFIX}/include" CACHE STRING "" FORCE)
        set(OpenMP_CXX_LIB_NAMES "omp" CACHE STRING "" FORCE)
        set(OpenMP_omp_LIBRARY "${LIBOMP_PREFIX}/lib/libomp.dylib" CACHE PATH "" FORCE)
    endif()
endif()

find_package(OpenMP)
if(OpenMP_CXX_FOUND)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O2 ${OpenMP_CXX_FLAGS}") # with -O3 compiler changes the code...
else()
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O2") # with -O3 compiler changes the code...
endif()

set(CMAKE_Fortran_FLAGS "-c -fpic -Wall -O3")

include_directories("${PARENT_DIR}/include")
include_directories("${PARENT_DIR}/include/*")
add_library(trapecis_Sd "${PARENT_DIR}/include/trapecis_Sd.f")

set(source_dir "${PARENT_DIR}/src/")
set(source_files "${source_dir}/embeddingSD_unix.cpp")

set(include_dir "${PARENT_DIR}/include/")
file(GLOB include_files "${include_dir}/*.hpp")

add_subdirectory(pybind11)
pybind11_add_module(dmercator "${include_files}" python_bind.cpp)

# add_executable(mercator ${source_files} ${include_files})
target_link_libraries(dmercator PRIVATE trapecis_Sd)

if(OpenMP_CXX_FOUND)
    target_link_libraries(dmercator PRIVATE OpenMP::OpenMP_CXX)
endif()
