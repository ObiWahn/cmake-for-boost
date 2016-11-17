project(boost)
cmake_minimum_required(VERSION 3.6)

# TODO
# - proivde shared libs
# - add more libs that require to be build .e.g filesystem
# - add component interface

if(NOT Boost_FOUND AND NOT _CMAKE_BOOST_INCLUDED)
    set(_CMAKE_BOOST_INCLUDED TRUE)
    # boost not found we need to povide the vars that would otherwise be
    # provided by the FindBoost module.

    message(STATUS "== Using Boost provided by cmake-for-boost")
    set(Boost_FOUND TRUE)

    if(NOT TARGET Boost::diagnostic_definitions)
        add_library(Boost::diagnostic_definitions INTERFACE IMPORTED)
        add_library(Boost::disable_autolinking INTERFACE IMPORTED)
        add_library(Boost::dynamic_linking INTERFACE IMPORTED)
    endif()

    if(WIN32)
        set_target_properties(Boost::disable_autolinking PROPERTIES
            INTERFACE_COMPILE_DEFINITIONS "BOOST_ALL_NO_LIB=1")
    endif()

    set(Boost_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/include")
    set(Boost_LIBRARY_DIR "${CMAKE_CURRENT_BINARY_DIR}")

    add_library(boost_boost INTERFACE)
    target_include_directories(boost_boost INTERFACE ${Boost_INCLUDE_DIRS})
    set_target_properties(boost_boost PROPERTIES
        INTERFACE_COMPILE_DEFINITIONS "BOOST_SYSTEM_STATIC_LINK=1" #FIXME - shared
    )

    add_library(boost_system
        libs/system/src/error_code.cpp
    )
    target_link_libraries(boost_system PUBLIC boost_boost Boost::disable_autolinking)

    add_library(Boost::boost ALIAS boost_boost)
    add_library(Boost::system ALIAS boost_system)
endif()
