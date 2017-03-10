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

    if(NOT TARGET boost_boost)
        add_library(boost_boost INTERFACE)
        target_include_directories(boost_boost INTERFACE ${Boost_INCLUDE_DIRS})
        set_target_properties(boost_boost PROPERTIES
            INTERFACE_COMPILE_DEFINITIONS "BOOST_SYSTEM_STATIC_LINK=1" #FIXME - shared
        )
    endif()

    if(NOT TARGET boost_system)
        add_library(boost_system
            libs/system/src/error_code.cpp
        )
        target_link_libraries(boost_system PUBLIC boost_boost Boost::disable_autolinking)
    endif()

    if(NOT TARGET boost_thread)
        set(boost_thread_src_general
            libs/thread/src/future.cpp
            libs/thread/src/tss_null.cpp
        )

        if(MSVC)
            set(boost_thread_src_os_specific
                libs/thread/src/win32/tss_dll.cpp
                libs/thread/src/win32/tss_pe.cpp
                libs/thread/src/win32/thread.cpp
            )
        else() # we assume pthread of all other cases
            set(boost_thread_src_os_specific
                libs/thread/src/pthread/once_atomic.cpp
                libs/thread/src/pthread/once.cpp
                libs/thread/src/pthread/thread.cpp
            )
        endif()

        add_library(boost_thread
            ${boost_thread_src_general}
            ${boost_thread_src_os_specific}
        )
        target_link_libraries(boost_thread PUBLIC boost_boost Boost::disable_autolinking)
    endif()

    add_library(Boost::boost ALIAS boost_boost)
    add_library(Boost::system ALIAS boost_system)
    add_library(Boost::thread ALIAS boost_thread)
endif()
