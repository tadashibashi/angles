cmake_minimum_required(VERSION 3.26)
project(angles)

add_library(angles INTERFACE)

target_include_directories(angles INTERFACE include)

if (EMSCRIPTEN)
    # Set Emscripten linker flags necessary to run full ES3 emulation in WebGL2
    target_link_options(angles INTERFACE 
        -sFULL_ES3 -sMIN_WEBGL_VERSION=2 -sMAX_WEBGL_VERSION=2 -sWASM=1
    )
else()
    # Cache target lib folder and extension for each platform
    # This is later used in the angle_copy_libs function

    # TODO: add more binaries. For now, only MacOS Silicon is supported
    if (APPLE)
        if ("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "arm64")
            set(angles_LIBS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/lib/macos/arm64" CACHE STRING "" FORCE)
        else()
            message(FATAL_ERROR "No Angle libs are available for MacOS x86_64")
        endif()

        set(angles_LIBS_EXT "dylib" CACHE STRING "" FORCE)
    endif()

    target_link_libraries(angles INTERFACE
        ${angle_LIBS_DIR}/libEGL.${angle_LIBS_EXT}
        ${angle_LIBS_DIR}/libGLESv2.${angle_LIBS_EXT}
    )
endif()

# Copy libraries to the correct directory
function(copy_gl_libs)
    if (NOT EMSCRIPTEN)

    endif()
endfunction()

