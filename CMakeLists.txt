cmake_minimum_required(VERSION 3.26)
project(angles)

add_library(angles INTERFACE)

target_include_directories(angles INTERFACE include)

if (EMSCRIPTEN)
    # Set Emscripten linker flags necessary to run full ES3 emulation in WebGL2
    target_link_options(angles INTERFACE 
        -sFULL_ES3 -sMIN_WEBGL_VERSION=2 -sMAX_WEBGL_VERSION=2 -sWASM=1
    )
    set (USE_ANGLE OFF)
else()
    set (USE_ANGLE ON)

    # Cache target lib folder and extension for each platform
    # This is later used in the angles_copy_libs function

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
        ${angles_LIBS_DIR}/libEGL.${angles_LIBS_EXT}
        ${angles_LIBS_DIR}/libGLESv2.${angles_LIBS_EXT}
    )
endif()

# Copy libraries to output directory for the current target
# Call this in the same CMakeLists.txt as the add_executable command for the
# corresponding binary exe. If TARGET_NAME param is not set, it is automatically
# set to the current PROJECT_NAME
function(angles_copy_libs TARGET_NAME)
    if (USE_ANGLE)
        if (NOT TARGET_NAME)
            set(TARGET_NAME ${PROJECT_NAME})
        endif()

        add_custom_target(angles_copy_libs-${TARGET_NAME}
            BYPRODUCTS
            ${CMAKE_CURRENT_BINARY_DIR}/libEGL.${angles_LIBS_EXT}
            ${CMAKE_CURRENT_BINARY_DIR}/libGLESv2.${angles_LIBS_EXT}
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${angle_LIBS_DIR}/libEGL.${angles_LIBS_EXT} ${CMAKE_CURRENT_BINARY_DIR}/libEGL.${angles_LIBS_EXT}
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${angle_LIBS_DIR}/libGLESv2.${angles_LIBS_EXT} ${CMAKE_CURRENT_BINARY_DIR}/libGLESv2.${angles_LIBS_EXT}
        )
        add_dependencies(${TARGET_NAME} angle_copy_libs-${TARGET_NAME})
    endif()
endfunction()
