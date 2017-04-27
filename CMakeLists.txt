cmake_minimum_required(VERSION 2.6.0)

project(RGBDOdometry)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}")

# find_package(Pangolin 0.1 REQUIRED)
find_package(CUDA REQUIRED)
# find_package(SuiteSparse REQUIRED)

#remove this as soon as eigen is shipped with FindEigen.cmake
  get_filename_component(EIGEN_ROOT "/usr/include/local/eigen3" PATH)
  if(PKG_CONFIG_FOUND)
    pkg_check_modules(PC_EIGEN eigen3)
  endif(PKG_CONFIG_FOUND)
  find_path(EIGEN_INCLUDE_DIRS Eigen/Core
    HINTS ${PC_EIGEN_INCLUDEDIR} ${PC_EIGEN_INCLUDE_DIRS} 
          "${EIGEN_ROOT}" "$ENV{EIGEN_ROOT}"
    PATHS "$ENV{PROGRAMFILES}/Eigen 3.0.0" "$ENV{PROGRAMW6432}/Eigen 3.0.0"
          "$ENV{PROGRAMFILES}/Eigen" "$ENV{PROGRAMW6432}/Eigen"   
    PATH_SUFFIXES eigen3 include/eigen3 include)
  find_package_handle_standard_args(eigen DEFAULT_MSG EIGEN_INCLUDE_DIRS)
  set(EIGEN_DEFINITIONS ${EIGEN_DEFINITIONS} -DEIGEN_USE_NEW_STDVECTOR 
-DEIGEN_YES_I_KNOW_SPARSE_MODULE_IS_NOT_STABLE_YET)

# include_directories(${Pangolin_INCLUDE_DIRS})
include_directories(${CUDA_INCLUDE_DIRS})
include_directories(${EIGEN_INCLUDE_DIRS})
# include_directories(${SUITESPARSE_INCLUDE_DIRS})
include_directories(include)

file(GLOB srcs src/*.cpp)
file(GLOB cuda_utils src/cuda_utils/*.cu)
file(GLOB cuda_containers src/cuda_utils/containers/*.cpp)

set(CUDA_ARCH_BIN "30 35 50 52 61" CACHE STRING "Specify 'real' GPU arch to build binaries for, BIN(PTX) format is supported. Example: 1.3 2.1(1.3) or 13 21(13)")
set(CUDA_ARCH_PTX "" CACHE STRING "Specify 'virtual' PTX arch to build PTX intermediate code for. Example: 1.0 1.2 or 10 12")              

include(CudaComputeTargetFlags.cmake)                  
APPEND_TARGET_ARCH_FLAGS()

set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}  "-Xcompiler;-fPIC;")           
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "--ftz=true;--prec-div=false;--prec-sqrt=false") 

CUDA_COMPILE(cuda_objs ${cuda_utils})

set(CMAKE_CXX_FLAGS ${ADDITIONAL_CMAKE_CXX_FLAGS} "-O3 -msse2 -msse3 -Wall -std=c++11")

add_library(rgbd_odometry SHARED 
            ${srcs}
            ${cuda_utils}
            ${cuda_objs} 
            ${cuda_containers}
)
# target_link_libraries(rgbd_odometry
#                       ${Eigen_LIBRARIES}
#                       ${Pangolin_LIBRARIES}
#                       ${CUDA_LIBRARIES}
#                       ${SUITESPARSE_LIBRARIES}
# )
target_link_libraries(rgbd_odometry
                      ${Eigen_LIBRARIES}
                      ${CUDA_LIBRARIES}
)

install(TARGETS rgbd_odometry
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
)
