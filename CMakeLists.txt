cmake_minimum_required(VERSION 2.6.0)
project(RGBDOdometry)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}")

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} "-O3 -march=native")
set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} "-O3 -march=native")

find_package(CUDA REQUIRED)
find_package(Eigen3 REQUIRED)

set(OTHER_INCLUDES ${CUDA_INCLUDE_DIRS} ${EIGEN3_INCLUDE_DIRS})

include_directories(include)
include_directories(${OTHER_INCLUDES})

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

add_library(${PROJECT_NAME} SHARED ${srcs} ${cuda_utils} ${cuda_objs} ${cuda_containers})
target_link_libraries(${PROJECT_NAME} ${CUDA_LIBRARIES})

# Package config files

file(REMOVE "${CMAKE_BINARY_DIR}/${PROJECT_NAME}Targets.cmake")
export(TARGETS ${PROJECT_NAME} APPEND FILE "${CMAKE_BINARY_DIR}/${PROJECT_NAME}Targets.cmake")

set(CMAKECONFIG_INSTALL_DIR lib/cmake/${PROJECT_NAME})
file(RELATIVE_PATH REL_INCLUDE_DIR "${CMAKE_INSTALL_PREFIX}/${CMAKECONFIG_INSTALL_DIR}" "${CMAKE_INSTALL_PREFIX}/include")

# Build tree config
set(EXPORT_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/include)
configure_file(${CMAKE_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake @ONLY IMMEDIATE)

# Install tree config
set(EXPORT_INCLUDE_DIR "\${PROJECT_CMAKE_DIR}/${REL_INCLUDE_DIR}")
configure_file(${CMAKE_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}Config.cmake @ONLY)

# Add package to CMake package registery for use from the build tree
option(EXPORT_${PROJECT_NAME} "Export ${PROJECT_NAME} package for use by other software" ON)
if(EXPORT_${PROJECT_NAME})
    export(PACKAGE ${PROJECT_NAME})
endif()

# Install target

install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}Targets LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

install(FILES "${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}Config.cmake" DESTINATION ${CMAKECONFIG_INSTALL_DIR})
install(EXPORT ${PROJECT_NAME}Targets DESTINATION ${CMAKECONFIG_INSTALL_DIR})

# Uninstall target

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in" "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake" IMMEDIATE @ONLY)
add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
