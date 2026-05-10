# The Flutter tooling requires that developers have CMake 3.10 or later
# installed. You should not increase this version, as doing so will cause
# the plugin to fail to compile for some customers of the plugin.
cmake_minimum_required(VERSION 3.10)

project(native_zip VERSION 0.0.1 LANGUAGES C)

add_library(native_zip SHARED
        "native_zip.c"
        "my_zlib.c"
        "my_compress.c"
        "my_zip.c"
        "my_zip_async.c"
        "my_zip_utils.c"
        "my_task_notify.c"
        "my_utils.c"
        "my_file.c"
        "my_file_windows.c"
        "my_file_posix.c"
        "my_thread.c"
        "my_threadpool.c"
        "my_queue.c"
        "my_message_queue.c"
        "my_atomic_int_max.c"
        # "my_hashmap.c"
)

set_target_properties(native_zip PROPERTIES
        PUBLIC_HEADER native_zip.h
        OUTPUT_NAME "native_zip"
)

target_compile_definitions(native_zip PUBLIC DART_SHARED_LIB)

if (ANDROID)
  # Support Android 15 16k page size
  target_link_options(native_zip PRIVATE "-Wl,-z,max-page-size=16384")
endif()

# prebuilt static libraries (built with /MD to match Flutter's CRT)
if(CMAKE_GENERATOR_PLATFORM STREQUAL "ARM64")
  set(PREBUILT_LIBS "${CMAKE_CURRENT_SOURCE_DIR}/../windows/prebuilt/arm64/lib")
  message(NOTICE "[native_zip] using ARM64 prebuilt libraries")
else()
  set(PREBUILT_LIBS "${CMAKE_CURRENT_SOURCE_DIR}/../windows/prebuilt/x64/lib")
  message(NOTICE "[native_zip] using x64 prebuilt libraries")
endif()

set(PREBUILT_INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/../windows/prebuilt/include")

target_include_directories(native_zip PUBLIC
  ${PREBUILT_INCLUDE}
)

# Link zlib and libzip statically
# dependency that could fail to load alongside other plugins (e.g. flutter_angle).
target_link_libraries(native_zip PRIVATE
  ${PREBUILT_LIBS}/zlibstatic.lib
  ${PREBUILT_LIBS}/zip.lib
)

# Suppress LNK4098 warning caused by zip.lib (built with /MD) pulling in MSVCRT
if(MSVC)
  target_link_options(native_zip PRIVATE /NODEFAULTLIB:MSVCRT)
endif()