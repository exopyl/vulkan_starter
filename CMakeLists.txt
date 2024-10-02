cmake_minimum_required(VERSION 3.16)
project(VulkanStarter)

# C++ compiler configuration.
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)


# Sources
set(SOURCES
    src/application.cpp
    src/main.cpp
    src/window.cpp
    src/tiny_obj_loader.cpp
)
add_executable(${PROJECT_NAME} ${SOURCES})


# Make sure output goes to build/{Debug,Release}
if (MSVC)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_SOURCE_DIR}/build/Release)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_CURRENT_SOURCE_DIR}/build/Debug)

    set_target_properties(${PROJECT_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "$(ProjectDir)/$(Configuration)")

    message("CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG : ${CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG}")
    message("CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE : ${CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE}")
else ()
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/build/${CMAKE_BUILD_TYPE})
endif ()

# Vulkan
find_package(Vulkan REQUIRED)

target_link_libraries(${PROJECT_NAME} Vulkan::Vulkan)

# Shaders
set(SHADER_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src/shaders")
file(GLOB SHADERS "${SHADER_SOURCE_DIR}/*.vert" "${SHADER_SOURCE_DIR}/*.frag")

foreach(OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES})
    set(SHADER_BINARY_DIR "${CMAKE_BINARY_DIR}/${OUTPUTCONFIG}/shaders")
    file(MAKE_DIRECTORY ${SHADER_BINARY_DIR})
    foreach(SHADER ${SHADERS})
        get_filename_component(SHADER_NAME ${SHADER} NAME)
        set(SHADER_BINARY_NAME "${SHADER_BINARY_DIR}/${SHADER_NAME}.spv")
        list(APPEND SHADER_BINARY_NAMES ${SHADER_BINARY_NAME})
        message("Vulkan_GLSLC_EXECUTABLE : ${Vulkan_GLSLC_EXECUTABLE}")
        add_custom_command(
            MAIN_DEPENDENCY ${SHADER}
            OUTPUT ${SHADER_BINARY_NAME}
            COMMAND ${Vulkan_GLSLC_EXECUTABLE} ${SHADER} "-o" ${SHADER_BINARY_NAME}
            VERBATIM)
    endForeach()
endForeach()

add_custom_target(build_shaders DEPENDS ${SHADER_BINARY_NAMES})

add_dependencies(${PROJECT_NAME} build_shaders)


# glfw
set(GLFW_ROOT "C:/home/devthirdparties/glfw-3.3.8")
if (GLFW_ROOT)
    set(GLFW_INCLUDE ${GLFW_ROOT}/include)

    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

    add_subdirectory(${GLFW_ROOT} glfw)

    include_directories(${GLFW_INCLUDE})
    target_link_libraries(${PROJECT_NAME} glfw ${GLFW_LIBRARIES})
else ()
    find_package(GLFW3 REQUIRED)
    if (GLFW_FOUND)
        message("GLFW ${GLFW_VERSION} library found !")
        message(" - Headers : ${GLFW_INCLUDE_DIRS}")
        message(" - Binary : ${GLFW_LIBRARIES}")
        include_directories(${GLFW_INCLUDE_DIRS})
        link_libraries(${GLFW_LIBRARY_DIRS})
    endif ()
endif ()

