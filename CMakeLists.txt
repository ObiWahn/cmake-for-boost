project(boost)
cmake_minimum_required(VERSION 3.2)

option(CMAKE_BOOST_USE_SYSTEM_BOOST OFF)
option(CMAKE_BOOST_COMPONENTS "")

if(CMAKE_BOOST_USE_SYSTEM_BOOST)

    if(CMAKE_BOOST_COMPONENTS)
    find_package(Boost REQUIRED COMPONENTS ${CMAKE_BOOST_COMPONENTS})
    #TODO really required - test!! maybe we get rid of the 3 lines 
    else()
        find_package(Boost Boost_FIND_REQUIRED)
    endif()

    if(Boost_FOUND)
        #boost found we are good
        message(STATUS "using system's boost")
    else()
        # TODO - automatic fallback?
        message(FATAL "your system does not provide boost")
        set(CMAKE_BOOST_USE_SYSTEM_BOOST OFF)
    endif()

endif()


if(not CMAKE_BOOST_USE_SYSTEM_BOOST)
# boost not found we need to povide the vars that would otherwise be
# provided by the FindBoost module.
    message(STATUS "using external boost")
    set(Boost_FOUND TRUE)

    add_library(Boost::boost INTERFACE IMPORTED)
    set(Boost_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/include")
    set(Boost_LIBRARY_DIR "${CMAKE_CURRENT_BINARY_DIR}")
    set_target_properties(Boost::boost PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES "${Boost_INCLUDE_DIRS}"
    )

    if(NOT TARGET Boost::diagnostic_definitions)
        add_library(Boost::diagnostic_definitions INTERFACE IMPORTED)
        add_library(Boost::disable_autolinking INTERFACE IMPORTED)
        add_library(Boost::dynamic_linking INTERFACE IMPORTED)
    endif()

    if(WIN32)
        set_target_properties(Boost::disable_autolinking PROPERTIES
            INTERFACE_COMPILE_DEFINITIONS "BOOST_ALL_NO_LIB=1")
    endif()


   
    ## create list form components and iterate
    ## boost system 
    ##target_compile_definitions(Boost::system PRIVATE "BOOST_SYSTEM_STATIC_LINK=1")
    add_library(Boost::system static
        libs/system/src/error_code.cpp
    )
    target_link_libraries(Boost::system PUBLIC Boost::boost Boost::disable_autolinking)

endif()
