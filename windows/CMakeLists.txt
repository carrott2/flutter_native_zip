# The Flutter tooling requires that developers have a version of Visual Studio
# installed that includes CMake 3.14 or later. You should not increase this
# version, as doing so will cause the plugin to fail to compile for some
# customers of the plugin.
cmake_minimum_required(VERSION 3.14)

# Project-level configuration.
set(PROJECT_NAME "native_zip")
project(${PROJECT_NAME} LANGUAGES CXX)

# Invoke the build for native code shared with the other target platforms.
# This can be changed to accommodate different builds.
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/../src" "${CMAKE_CURRENT_BINARY_DIR}/shared")

# Static CRT linkage on Windows so native_zip.dll has no MSVC runtime
# dependency that could conflict with other plugins (e.g. flutter_angle).
if(MSVC)
  foreach(flag_var
    CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
    CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
    CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
    CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
    string(REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
  endforeach()
endif()

include(FetchContent)

# zlib (static)
FetchContent_Declare(
  zlib
  URL https://zlib.net/fossils/zlib-1.3.1.tar.gz
  URL_HASH SHA256=9a93b2b7dfdac77ceba5a558a580e74667dd6fede4585b91eefb60f03b72df23
)
set(ZLIB_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(zlib)

# libzip (static, no shared lib, no tools)
FetchContent_Declare(
  libzip
  URL https://libzip.org/download/libzip-1.10.1.tar.gz
  URL_HASH SHA256=dc3c8d5b4c8bec293ef12624a64a610b30c4b54b4c658fcf3e9e30dda96a6e68
)
set(BUILD_SHARED_LIBS  OFF CACHE BOOL "" FORCE)
set(BUILD_TOOLS        OFF CACHE BOOL "" FORCE)
set(BUILD_REGRESS      OFF CACHE BOOL "" FORCE)
set(BUILD_EXAMPLES     OFF CACHE BOOL "" FORCE)
set(BUILD_DOC          OFF CACHE BOOL "" FORCE)
set(ENABLE_LZMA        OFF CACHE BOOL "" FORCE)
set(ENABLE_ZSTD        OFF CACHE BOOL "" FORCE)
set(ENABLE_BZIP2       OFF CACHE BOOL "" FORCE)
set(LIBZIP_DO_INSTALL  OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(libzip)

# add 'include' folders for zlib and libzip headers.
# Both source and binary dirs are needed: source for the public headers,
# binary for generated headers (zconf.h for zlib, zipconf.h for libzip).
target_include_directories(${PROJECT_NAME} PRIVATE
  ${zlib_SOURCE_DIR}
  ${zlib_BINARY_DIR}
  ${libzip_SOURCE_DIR}/lib
  ${libzip_BINARY_DIR}/lib
)

target_link_libraries(${PROJECT_NAME} PRIVATE zlibstatic zip)

# List of absolute paths to libraries that should be bundled with the plugin.
# This list could contain prebuilt libraries, or libraries created by an
# external build triggered from this build file.
set(native_zip_bundled_libraries
  # Defined in ../src/CMakeLists.txt.
  # This can be changed to accommodate different builds.
  $<TARGET_FILE:native_zip>
  PARENT_SCOPE
)
