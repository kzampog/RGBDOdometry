cmake_minimum_required(VERSION 2.6.0)
project(RGBDOdometry)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}")

find_package(CUDA REQUIRED)
find_package(Eigen3 REQUIRED)

include_directories(include)
include_directories(${CUDA_INCLUDE_DIRS})
include_directories(${EIGEN3_INCLUDE_DIRS})

file(GLOB srcs src/*.cpp)
file(GLOB cuda_utils src/cuda_utils/*.cu)
file(GLOB cuda_containers src/cuda_utils/containers/*.cpp)

set(CUDA_ARCH_BIN "30 35 50 52 61" CACHE STRING "Specify 'real' GPU arch to build binaries for, BIN(PTX) format is supported. Example: 1.3 2.1(1.3) or 13 21(13)")
set(CUDA_ARCH_PTX "" CACHE STRING "Specify 'virtual' PTX arch to build PTX intermediate code for. Example: 1.0 1.2 or 10 12")

include(CudaComputeTargetFlags.cmake)
append_target_arch_flags()

set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-Xcompiler;-fPIC;-std=c++11")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "--ftz=true;--prec-div=false;--prec-sqrt=false") 

cuda_compile(cuda_objs ${cuda_utils})

set(CMAKE_CXX_FLAGS ${ADDITIONAL_CMAKE_CXX_FLAGS} "-O3 -msse2 -msse3 -Wall -std=c++11")

add_library(rgbd_odometry SHARED ${srcs} ${cuda_utils} ${cuda_objs} ${cuda_containers})
target_link_libraries(rgbd_odometry ${CUDA_LIBRARIES})
