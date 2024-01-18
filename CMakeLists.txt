cmake_minimum_required(VERSION 3.22)
if(APPLE) # MoltenVK
    include_directories(/usr/local/include)
endif()
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake")
project(ProjectCxxPy)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# set(CMAKE_CXX_CLANG_TIDY clang-tidy)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(SWIG REQUIRED)
include(${SWIG_USE_FILE})
find_package(PythonLibs REQUIRED)
include_directories(${PYTHON_INCLUDE_PATH})

find_package(glfw3 CONFIG REQUIRED)
find_package(unofficial-shaderc CONFIG REQUIRED)
find_package(Vulkan REQUIRED)
find_package(VulkanHeaders CONFIG)

add_subdirectory(vk-bootstrap)

set(SOURCE_FILES lib.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(CMAKE_SWIG_FLAGS)
set_property(SOURCE lib.i PROPERTY CPLUSPLUS ON)
swig_add_library(LibPy TYPE SHARED LANGUAGE python SOURCES lib.i ${SOURCE_FILES})
target_include_directories(LibPy PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
swig_link_libraries(LibPy ${PYTHON_LIBRARIES})

add_library(LibCxx ${SOURCE_FILES})

add_executable(MainCxx main.cpp)
target_link_libraries(MainCxx PRIVATE
    LibCxx
    glfw
    unofficial::shaderc::shaderc
    vk-bootstrap::vk-bootstrap
    Vulkan::Vulkan
    Vulkan::Headers)
