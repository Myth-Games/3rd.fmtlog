cmake_minimum_required(VERSION 3.15)

project(fmtlog CXX)

option(FMTLOG_USE_EXTERNAL_FMT "Use external fmt library" OFF)
option(FMTLOG_BUILD_TESTS "Build fmtlog tests" OFF)

if(MSVC)
    #add_compile_options(/std:c++latest)
else()
    add_compile_options(-Wall -O3)
    #add_compile_options(-Wall -Ofast -std=c++2a -DNDEBUG -march=skylake -flto -fno-exceptions -fno-rtti -fno-unwind-tables -fno-asynchronous-unwind-tables -DFMTLOG_NO_CHECK_LEVEL=1)
    #SET(CMAKE_AR  "gcc-ar")
    #SET(CMAKE_RANLIB  "gcc-ranlib")
    link_libraries(pthread)
endif()

if (FMTLOG_USE_EXTERNAL_FMT)
    find_package(fmt REQUIRED)
elseif (NOT TARGET fmt)
    add_subdirectory(fmt)
endif()

set(FMTLOG_INCLUDEDIRS $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}> $<INSTALL_INTERFACE:include>)

link_directories(.)
include_directories(fmt/include)

if (BUILD_SHARED_LIBS)
    add_library(fmtlog-shared SHARED fmtlog.cc)
    add_library(fmtlog::fmtlog ALIAS fmtlog-shared)
    target_compile_features(fmtlog-shared PUBLIC cxx_std_17)
    target_link_libraries(fmtlog-shared fmt)
    target_include_directories(fmtlog-shared
        PUBLIC
            ${FMTLOG_INCLUDEDIRS}
    )
    install(TARGETS fmtlog-shared)
else ()
    add_library(fmtlog-static fmtlog.cc)
    target_compile_features(fmtlog-static PUBLIC cxx_std_17)
    target_link_libraries(fmtlog-static fmt)
    target_include_directories(fmtlog-static
        PUBLIC
            ${FMTLOG_INCLUDEDIRS}
    )
    add_library(fmtlog::fmtlog ALIAS fmtlog-static)
    install(TARGETS fmtlog-static)
endif ()

if (FMTLOG_BUILD_TESTS)
    add_subdirectory(test) # Will break. TODO
endif ()
