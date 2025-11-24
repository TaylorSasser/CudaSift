cmake_minimum_required(VERSION 3.23)

project(cudaSift
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
    src/cudaImage.cu
    src/cudaSiftH.cu
    src/cudaSiftD.cu
    src/matching.cu
    src/geomFuncs.cpp
)

target_sources(cudasift
    PUBLIC
    FILE_SET public_headers
    TYPE HEADERS
    BASE_DIRS include/
    FILES
        include/cudasift/cudaImage.h
        include/cudasift/cudaSift.h
        include/cudasift/cudaSiftD.h
        include/cudasift/cudaSiftH.h
        include/cudasift/cudautils.h
)


target_include_directories(cudasift
    PUBLIC
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
    PRIVATE
    ${CUDAToolkit_INCLUDE_DIRS}
)

target_include_directories(cudasift
    PUBLIC
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
    PRIVATE
        ${CUDAToolkit_INCLUDE_DIRS}
)

target_compile_features(cudasift PRIVATE cxx_std_17)

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
    EXPORT_NAME cudaSift
)

include(GNUInstallDirs)

install(TARGETS cudasift
        EXPORT cudasift_targets
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        FILE_SET public_headers DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)


install(
        EXPORT cudasift_targets
        NAMESPACE cudasift::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cudasift
)

include(CMakePackageConfigHelpers)

configure_package_config_file(
        ${PROJECT_SOURCE_DIR}/cmake/cudasift_config.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/cudasift_config.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cudasift
)

write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/cudasift_config_version.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
)


install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/cudasift_config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/cudasift_config_version.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cudasift
)
