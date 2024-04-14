cmake_minimum_required(VERSION 3.10)
project(glfw LANGUAGES C)

# Static runtime
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Release>:Release>")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Distribution>:Distribution>")

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/${PROJECT_NAME}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/${PROJECT_NAME}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/${PROJECT_NAME}")

# Files
set(SOURCE_FILES
    "src/context.c"
    "src/init.c"
    "src/input.c"
    "src/monitor.c"
    "src/platform.c"
    "src/vulkan.c"
    "src/window.c"
    "src/egl_context.c"
    "src/osmesa_context.c"
)

set(HEADER_FILES
    "include/glfw/glfw3.h"
    "include/glfw/glfw3native.h"
    "src/internal.h"
    "src/platform.h"
    "src/mappings.h"
)

# Linux-specific files and settings
if(UNIX)
    list(APPEND SOURCE_FILES
        "src/x11_init.c"
        "src/x11_monitor.c"
        "src/x11_window.c"
        "src/xkb_unicode.c"
        "src/posix_time.c"
        "src/posix_thread.c"
        "src/glx_context.c"
        "src/linux_joystick.c"
    )
    set(LINUX_DEFINES "_GLFW_X11")
endif()

# macOS-specific files and settings
if(APPLE)
    list(APPEND SOURCE_FILES
        "src/cocoa_init.m"
        "src/cocoa_monitor.m"
        "src/cocoa_window.m"
        "src/cocoa_joystick.m"
        "src/cocoa_time.c"
        "src/nsgl_context.m"
        "src/posix_thread.c"
        "src/posix_module.c"
    )
    set(MACOS_DEFINES "_GLFW_COCOA")
endif()

# Windows-specific files and settings
if(WIN32)
    list(APPEND SOURCE_FILES
        "src/win32_init.c"
        "src/win32_module.c"
        "src/win32_joystick.c"
        "src/win32_monitor.c"
        "src/win32_time.c"
        "src/win32_thread.c"
        "src/win32_window.c"
        "src/wgl_context.c"
    )
    set(WINDOWS_DEFINES "_GLFW_WIN32" "_CRT_SECURE_NO_WARNINGS")
endif()

# Define the target
add_library(${PROJECT_NAME} STATIC ${SOURCE_FILES} ${HEADER_FILES})

# Include directories
target_include_directories(${PROJECT_NAME} PRIVATE
    "include"
)

# Define system-specific defines
target_compile_definitions(${PROJECT_NAME} PRIVATE
    ${LINUX_DEFINES}
    ${MACOS_DEFINES}
    ${WINDOWS_DEFINES}
)

# Configuration-specific settings
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:/MTd>
    )
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:/Zi>
    )
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Release>:/MT>
    )
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Release>:/O2>
    )
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Distribution")
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Distribution>:/MT>
    )
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Distribution>:/O2>
    )
endif()