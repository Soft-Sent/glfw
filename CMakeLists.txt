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
=======
cmake_minimum_required(VERSION 3.16...3.28 FATAL_ERROR)

project(GLFW VERSION 3.5.0 LANGUAGES C HOMEPAGE_URL "https://www.glfw.org/")
>>>>>>> upstream/master

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
if (DEFINED GLFW_USE_WAYLAND AND UNIX AND NOT APPLE)
    message(FATAL_ERROR
        "GLFW_USE_WAYLAND has been removed; delete the CMake cache and set GLFW_BUILD_WAYLAND and GLFW_BUILD_X11 instead")
endif()

cmake_dependent_option(GLFW_BUILD_WIN32 "Build support for Win32" ON "WIN32" OFF)
cmake_dependent_option(GLFW_BUILD_COCOA "Build support for Cocoa" ON "APPLE" OFF)
cmake_dependent_option(GLFW_BUILD_X11 "Build support for X11" ON "UNIX;NOT APPLE" OFF)
cmake_dependent_option(GLFW_BUILD_WAYLAND "Build support for Wayland" ON "UNIX;NOT APPLE" OFF)

cmake_dependent_option(GLFW_USE_HYBRID_HPG "Force use of high-performance GPU on hybrid systems" OFF
                       "WIN32" OFF)
cmake_dependent_option(USE_MSVC_RUNTIME_LIBRARY_DLL "Use MSVC runtime library DLL" ON
                       "MSVC" OFF)

set(GLFW_LIBRARY_TYPE "${GLFW_LIBRARY_TYPE}" CACHE STRING
    "Library type override for GLFW (SHARED, STATIC, OBJECT, or empty to follow BUILD_SHARED_LIBS)")

if (GLFW_LIBRARY_TYPE)
    if (GLFW_LIBRARY_TYPE STREQUAL "SHARED")
        set(GLFW_BUILD_SHARED_LIBRARY TRUE)
    else()
        set(GLFW_BUILD_SHARED_LIBRARY FALSE)
    endif()
else()
    set(GLFW_BUILD_SHARED_LIBRARY ${BUILD_SHARED_LIBS})
endif()

list(APPEND CMAKE_MODULE_PATH "${GLFW_SOURCE_DIR}/CMake/modules")

find_package(Threads REQUIRED)

#--------------------------------------------------------------------
# Report backend selection
#--------------------------------------------------------------------
if (GLFW_BUILD_WIN32)
    message(STATUS "Including Win32 support")
endif()
if (GLFW_BUILD_COCOA)
    message(STATUS "Including Cocoa support")
endif()
if (GLFW_BUILD_WAYLAND)
    message(STATUS "Including Wayland support")
endif()
if (GLFW_BUILD_X11)
    message(STATUS "Including X11 support")
endif()

#--------------------------------------------------------------------
# Apply Microsoft C runtime library option
# This is here because it also applies to tests and examples
#--------------------------------------------------------------------
if (MSVC AND NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
endif()

#--------------------------------------------------------------------
# Create generated files
#--------------------------------------------------------------------
include(CMakePackageConfigHelpers)

set(GLFW_CONFIG_PATH "${CMAKE_INSTALL_LIBDIR}/cmake/glfw3")

configure_package_config_file(CMake/glfw3Config.cmake.in
                              src/glfw3Config.cmake
                              INSTALL_DESTINATION "${GLFW_CONFIG_PATH}"
                              NO_CHECK_REQUIRED_COMPONENTS_MACRO)

write_basic_package_version_file(src/glfw3ConfigVersion.cmake
                                 VERSION ${GLFW_VERSION}
                                 COMPATIBILITY SameMajorVersion)

#--------------------------------------------------------------------
# Add subdirectories
#--------------------------------------------------------------------
add_subdirectory(src)

if (GLFW_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

if (GLFW_BUILD_TESTS)
    add_subdirectory(tests)
endif()

if (GLFW_BUILD_DOCS)
    add_subdirectory(docs)
endif()

#--------------------------------------------------------------------
# Install files other than the library
# The library is installed by src/CMakeLists.txt
#--------------------------------------------------------------------
if (GLFW_INSTALL)
    install(DIRECTORY include/GLFW DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
            FILES_MATCHING PATTERN glfw3.h PATTERN glfw3native.h)

    install(FILES "${GLFW_BINARY_DIR}/src/glfw3Config.cmake"
                  "${GLFW_BINARY_DIR}/src/glfw3ConfigVersion.cmake"
            DESTINATION "${GLFW_CONFIG_PATH}")

    install(EXPORT glfwTargets FILE glfw3Targets.cmake
            EXPORT_LINK_INTERFACE_LIBRARIES
            DESTINATION "${GLFW_CONFIG_PATH}")
    install(FILES "${GLFW_BINARY_DIR}/src/glfw3.pc"
            DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

    # Only generate this target if no higher-level project already has
    if (NOT TARGET uninstall)
        configure_file(CMake/cmake_uninstall.cmake.in
                       cmake_uninstall.cmake IMMEDIATE @ONLY)

        add_custom_target(uninstall
                          "${CMAKE_COMMAND}" -P
                          "${GLFW_BINARY_DIR}/cmake_uninstall.cmake")
        set_target_properties(uninstall PROPERTIES FOLDER "GLFW3")
    endif()
>>>>>>> upstream/master
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Distribution")
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Distribution>:/MT>
    )
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Distribution>:/O2>
    )
endif()