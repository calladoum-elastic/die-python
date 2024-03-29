project(
    die-python
    LANGUAGES CXX
    VERSION 0.1.0
)

find_package(Python 3
    REQUIRED COMPONENTS Interpreter Development.Module
    OPTIONAL_COMPONENTS Development.SABIModule
)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "RelWithDebInfo"  "Release" )
endif()

execute_process(
    COMMAND "${Python_EXECUTABLE}" -m nanobind --cmake_dir
    OUTPUT_STRIP_TRAILING_WHITESPACE OUTPUT_VARIABLE NB_DIR)
list(APPEND CMAKE_PREFIX_PATH "${NB_DIR}")

find_package(nanobind CONFIG REQUIRED)
find_package(DieLibrary REQUIRED)

set(DIE_PYTHON_ROOT_DIR ${DIE_ROOT_DIR}/python)
set(DIE_BASE_ROOT ${dielibrary_SOURCE_DIR})

nanobind_add_module(
    _die NB_STATIC
    ./src/die.cpp
)


target_compile_definitions(_die PRIVATE)
# add_dependencies(_die Qt5::Core)
target_include_directories(
    _die
    PRIVATE
        ${DIE_BASE_ROOT}
        ${DIE_PYTHON_ROOT_DIR}/inc
        # ${qt5_SOURCE_DIR}
        # $<TARGET_PROPERTY:${qt5_SOURCE_DIR},INCLUDE_DIRECTORIES>
        # ${qt6_SOURCE_DIR}
        # $<TARGET_PROPERTY:${qt6_SOURCE_DIR},INCLUDE_DIRECTORIES>
)
target_link_directories(_die PRIVATE ${DIE_BASE_ROOT})

if(MSVC)
    target_link_libraries(_die PRIVATE die.lib)
endif()

if(APPLE)
    target_link_libraries(_die PRIVATE die "-framework CoreFoundation" -Wl -lm)
endif()

if(UNIX)
    target_link_libraries(_die PRIVATE die)
endif()

install(DIRECTORY die DESTINATION .)
install(TARGETS _die LIBRARY DESTINATION die)
install(DIRECTORY ${DIE_BASE_ROOT}/db DESTINATION die)
install(FILES
    ${DIE_BASE_ROOT}/die.h
    ${DIE_BASE_ROOT}/die.dll
    ${DIE_BASE_ROOT}/Qt5Core.dll
    ${DIE_BASE_ROOT}/Qt5Script.dll
    DESTINATION die
)
