cmake_minimum_required(VERSION 3.11)
project(dsa)

include(ExternalProject)

set(CMAKE_CXX_STANDARD 11)

set( CMAKE_BUILD_TYPE "Debug")

include_directories(include)

set(DSA_SRC_FILES
#        "src/kmp.cpp"
        "src/lcs.cpp")


# Add gtest dependency
set(GTEST_VERSION "1.8.0")

if(APPLE)
    set(GTEST_CMAKE_CXX_FLAGS "-fPIC -DGTEST_USE_OWN_TR1_TUPLE=1 -Wno-unused-value -Wno-ignored-attributes")
elseif(NOT MSVC)
    set(GTEST_CMAKE_CXX_FLAGS "-fPIC")
endif()

if(CMAKE_BUILD_TYPE)
    string(TOUPPER ${CMAKE_BUILD_TYPE} UPPERCASE_BUILD_TYPE)
endif()

set(GTEST_CMAKE_CXX_FLAGS "${EP_CXX_FLAGS} ${CMAKE_CXX_FLAGS_${UPPERCASE_BUILD_TYPE}} ${GTEST_CMAKE_CXX_FLAGS}")

set(GTEST_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/external/googletest-install")
set(GTEST_INCLUDE_DIR "${GTEST_PREFIX}/include")
set(GTEST_STATIC_LIB
        "${GTEST_PREFIX}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}gtest${CMAKE_STATIC_LIBRARY_SUFFIX}")
set(GTEST_MAIN_STATIC_LIB
        "${GTEST_PREFIX}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}gtest_main${CMAKE_STATIC_LIBRARY_SUFFIX}")
set(GMOCK_MAIN_STATIC_LIB
        "${GTEST_PREFIX}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}gmock_main${CMAKE_STATIC_LIBRARY_SUFFIX}")

set(GTEST_CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_INSTALL_PREFIX=${GTEST_PREFIX}
        -DCMAKE_CXX_FLAGS=${GTEST_CMAKE_CXX_FLAGS})

 set(GTEST_URL_MD5 "16877098823401d1bf2ed7891d7dce36")

ExternalProject_Add(googletest_ep
        PREFIX external/googletest
        URL "https://github.com/google/googletest/archive/release-${GTEST_VERSION}.tar.gz"
        URL_MD5 ${GTEST_URL_MD5}
        BUILD_BYPRODUCTS ${GTEST_STATIC_LIB} ${GTEST_MAIN_STATIC_LIB} ${GMOCK_MAIN_STATIC_LIB}
        CMAKE_ARGS ${GTEST_CMAKE_ARGS}
        ${EP_LOG_OPTIONS})
# end of add gtest dependency


add_library(dsa SHARED ${DSA_SRC_FILES})

add_executable(demo_test test/demo_test.cpp)
add_dependencies(demo_test dsa googletest_ep)
include_directories(SYSTEM ${GTEST_INCLUDE_DIR})
target_link_libraries(demo_test ${GTEST_STATIC_LIB} ${GTEST_MAIN_STATIC_LIB} ${GMOCK_MAIN_STATIC_LIB} pthread)