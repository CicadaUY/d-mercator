cmake_minimum_required(VERSION 3.20)
project(mercator Fortran CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wshadow -pedantic")

# Find OpenMP
find_package(OpenMP)
if(OpenMP_CXX_FOUND)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O2 ${OpenMP_CXX_FLAGS}") # with -O3 compiler changes the code...
else()
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O2") # with -O3 compiler changes the code...
endif()

set(CMAKE_Fortran_FLAGS "-c -fpic -Wall -O3")

include_directories(include)
include_directories(include/*)
add_library(trapecis_Sd include/trapecis_Sd.f)
# add_library(trapecis_Sd_expected include/trapecis_Sd_expected.f)

set(source_dir "${PROJECT_SOURCE_DIR}/src/")
set(source_files "${source_dir}/embeddingSD_unix.cpp")

set(include_dir "${PROJECT_SOURCE_DIR}/include/")
file(GLOB include_files "${include_dir}/*.hpp")

add_executable(mercator ${source_files} ${include_files})
target_link_libraries(mercator trapecis_Sd)

if(OpenMP_CXX_FOUND)
    target_link_libraries(mercator OpenMP::OpenMP_CXX)
endif()
