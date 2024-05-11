cmake_minimum_required(VERSION 3.23)
project(angles)

add_library(angles INTERFACE)

target_include_directories(angles INTERFACE include)

if (EMSCRIPTEN)
    # Set Emscripten linker flags necessary to run full ES3 emulation in WebGL2
    target_link_options(angles INTERFACE
        -sFULL_ES3 -sMIN_WEBGL_VERSION=2 -sMAX_WEBGL_VERSION=2 -sWASM=1
    )
    set (ANGLES_USE_ANGLE OFF CACHE BOOL "" FORCE)
else()
    set (ANGLES_USE_ANGLE ON CACHE BOOL "" FORCE)
    string(TOUPPER ${CMAKE_SYSTEM_PROCESSOR} SYS_PROCESSOR)
    # Cache target lib folder and extension for each platform
    # This is later used in the angles_copy_libs function

    # TODO: add more binaries. For now, only MacOS Silicon and Windows x64 are supported
    if (APPLE)
        if ("${SYS_PROCESSOR}" STREQUAL "ARM64")
            set(angles_LIBS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/lib/macos/arm64 CACHE STRING "" FORCE)
        else()
            message(FATAL_ERROR "No Angle libs are available for MacOS x86_64")
        endif()

        set(angles_LIBS_EXT "dylib" CACHE STRING "" FORCE)

        # Use metal backend on apple devices
        set(ENV{ANGLE_DEFAULT_PLATFORM} "metal")
    elseif(MSVC)
        if ("${SYS_PROCESSOR}" STREQUAL "ARM64")
            message(FATAL_ERROR "No Angle libs are available for Windows ARM64")
            # set(angles_LIBS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/lib/windows/arm64 CACHE STRING "" FORCE)
        else()
            set(angles_LIBS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/lib/windows/x64 CACHE STRING "" FORCE)
        endif()

        set(angles_LIBS_EXT "lib" CACHE STRING "" FORCE)
    else()
        message(FATAL_ERROR "Angles platform is not supported")
    endif()

    target_link_libraries(angles INTERFACE
        ${angles_LIBS_DIR}/libEGL.${angles_LIBS_EXT}
        ${angles_LIBS_DIR}/libGLESv2.${angles_LIBS_EXT}
    )
endif()

# Copy libraries to output directory for the current target
# Call this in the same CMakeLists.txt as the add_executable command for the
# corresponding binary exe. If TARGET_NAME param is not set, it is automatically
# set to the current PROJECT_NAME
function(angles_copy_libs)
    if (ANGLES_USE_ANGLE)
        if (${ARGV0})
            set(TARGET_NAME ${ARGV0})
        else()
            set(TARGET_NAME ${PROJECT_NAME})
        endif()

        if (MSVC)
            execute_process(
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${angles_LIBS_DIR}/libEGL.dll ${CMAKE_CURRENT_BINARY_DIR}/libEGL.dll
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${angles_LIBS_DIR}/libGLESv2.dll ${CMAKE_CURRENT_BINARY_DIR}/libGLESv2.dll
            )
        else()
            execute_process(
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${angles_LIBS_DIR}/libEGL.${angles_LIBS_EXT} ${CMAKE_CURRENT_BINARY_DIR}/libEGL.${angles_LIBS_EXT}
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${angles_LIBS_DIR}/libGLESv2.${angles_LIBS_EXT} ${CMAKE_CURRENT_BINARY_DIR}/libGLESv2.${angles_LIBS_EXT}
            )
        endif()
    endif()
endfunction()
