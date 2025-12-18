cmake_minimum_required(VERSION 3.23)
project(cudasift
    VERSION 2.0.0
    LANGUAGES CXX CUDA
)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(LIBRARY_TYPE STATIC)
if (BUILD_SHARED_LIBS)
    set(LIBRARY_TYPE SHARED)
endif ()

if (NOT DEFINED CMAKE_CUDA_ARCHITECTURES)
    if (DEFINED CUDA_ARCHS)
        set(CMAKE_CUDA_ARCHITECTURES "${CUDA_ARCHS}")
    else ()
        set(CMAKE_CUDA_ARCHITECTURES "61;75;86")
    endif ()
endif ()

add_library(cudasift ${LIBRARY_TYPE}
    src/cuda_image.cu
    src/cuda_sift_host.cu
    src/cuda_sift_device.cu
    src/matching.cu
    src/geometry.cpp
)

add_library(cudasift::cudasift ALIAS cudasift)

target_sources(cudasift
    PUBLIC
    FILE_SET public_headers
    TYPE HEADERS
    BASE_DIRS include/
    FILES
        include/cudasift/cuda_image.h
        include/cudasift/cuda_sift.h
        include/cudasift/cuda_sift_device.h
        include/cudasift/cuda_sift_host.h
        include/cudasift/cuda_utils.h
)


target_include_directories(cudasift
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/cudasift>
        $<INSTALL_INTERFACE:include/cudasift>
    PRIVATE
        ${CUDAToolkit_INCLUDE_DIRS}
)


set_property(TARGET cudasift
    PROPERTY CUDA_ARCHITECTURES "${CMAKE_CUDA_ARCHITECTURES}")

find_package(CUDAToolkit REQUIRED)
find_package(OpenCV CONFIG REQUIRED)

target_link_libraries(cudasift
    PUBLIC
    CUDA::cudart
    ${OpenCV_LIBS}
)
target_compile_definitions(cudasift
    PRIVATE
    VERBOSE
    _CRT_SECURE_NO_WARNINGS
)

target_compile_options(cudasift
    PRIVATE
    -O3
    -msse2
    $<$<COMPILE_LANGUAGE:CUDA>:-lineinfo -Wno-deprecated-gpu-targets>
)

set_target_properties(cudasift
    PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    EXPORT_NAME cudasift
)

include(GNUInstallDirs)

install(
    TARGETS cudasift
    EXPORT cudasift-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    FILE_SET public_headers DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)


install(
        EXPORT cudasift-targets
        NAMESPACE cudasift::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cudasift
)

include(CMakePackageConfigHelpers)

configure_package_config_file(
    ${PROJECT_SOURCE_DIR}/cmake/cudasift-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/cudasift-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cudasift
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/cudasift-config-version.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)


install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/cudasift-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/cudasift-config-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cudasift
)
export(PACKAGE cudasift)
