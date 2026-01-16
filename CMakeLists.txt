cmake_minimum_required(VERSION 3.18)

project(TensorRT_Learn LANGUAGES CXX CUDA)

# === 配置 C++ 和 CUDA 标准 ===
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_STANDARD 17)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

# === 配置 GPU 架构 ===
set(CMAKE_CUDA_ARCHITECTURES 120)

# === 查找依赖库 ===
find_package(CUDAToolkit REQUIRED)
find_package(ZLIB REQUIRED)
find_package(Threads REQUIRED)

# === 自动查找 TensorRT ===
set(TENSORRT_ROOT "" CACHE PATH "Path to TensorRT installation")

find_path(TENSORRT_INCLUDE_DIR NvInfer.h
    HINTS ${TENSORRT_ROOT}
    PATHS /usr/local/tensorrt/include
          /usr/include/x86_64-linux-gnu
          /usr/include
          /usr/local/cuda/include
    PATH_SUFFIXES include
    DOC "Path to TensorRT headers"
)

find_library(TENSORRT_LIB nvinfer
    HINTS ${TENSORRT_ROOT}
    PATHS /usr/local/tensorrt/lib
          /usr/lib/x86_64-linux-gnu
          /usr/lib
          /usr/local/cuda/lib64
    PATH_SUFFIXES lib lib64
    DOC "Path to TensorRT library"
)

if(TENSORRT_INCLUDE_DIR AND TENSORRT_LIB)
    message(STATUS "Found TensorRT headers: ${TENSORRT_INCLUDE_DIR}")
    message(STATUS "Found TensorRT lib: ${TENSORRT_LIB}")
else()
    message(FATAL_ERROR "TensorRT not found. Please set TENSORRT_ROOT to your TensorRT install path.")
endif()

# === 包含头文件目录 ===
include_directories(${TENSORRT_INCLUDE_DIR})
include_directories(${CMAKE_SOURCE_DIR}/include)
include_directories(${CMAKE_SOURCE_DIR}/src)

# === 定义源文件 ===
set(SOURCES
    src/01-SimpleDemo/main.cpp
    include/cookbookHelper.cu
    include/calibrator.cpp
    include/cnpy.cpp
)

# === 创建可执行文件 ===
add_executable(main ${SOURCES})

# === 编译选项 ===
target_compile_options(main PRIVATE 
    $<$<COMPILE_LANGUAGE:CUDA>:-use_fast_math>
)

# === 链接库 ===
target_link_libraries(main PRIVATE
    CUDA::cudart
    ${TENSORRT_LIB} 
    ZLIB::ZLIB
    ${CMAKE_DL_LIBS}
)