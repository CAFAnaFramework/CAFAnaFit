############   top level config
cmake_minimum_required(VERSION 3.14)
project(CAFAnaFit)

set(CMAKE_CXX_STANDARD 17)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules")

# # Bake version number into the library (CAFAna/Core/Version.cxx)
# execute_process(COMMAND git describe --tags OUTPUT_VARIABLE CAFANACORE_VERSION OUTPUT_STRIP_TRAILING_WHITESPACE)
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCAFANACORE_VERSION=\\\"${CAFANACORE_VERSION}\\\"")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -DNDEBUG -g")  # needed to ensure assert() causes an abort() even in optimized builds
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--no-undefined")

# non-negotiable dependencies
find_package(CAFAnaCore REQUIRED)
find_package(OscLib REQUIRED)
find_package(ROOT REQUIRED)
find_package(StanMath REQUIRED)
find_package(Eigen REQUIRED)
find_package(Boost REQUIRED)
find_package(Ifdhc REQUIRED)

# pthread / std::thread
find_package(Threads REQUIRED)
link_libraries(Threads::Threads)

# ROOT does things differently depending on whether it's built for C++17 or not :-\
string(REGEX REPLACE ".*c\\+\\+([0-9][0-9]).*" "\\1" ROOT_CXX_STANDARD ${ROOT_CXX_FLAGS})
if(ROOT_CXX_STANDARD LESS_EQUAL 14)
    message(WARNING "ROOT was built with C++${ROOT_CXX_STANDARD} standard.  Overriding default C++ standard version (C++${CMAKE_CXX_STANDARD}) to use C++14 (lowest supported by this package).  Cross your fingers it builds...")
    set(CMAKE_CXX_STANDARD 14)
endif()

# this is quite lazy -- it links ALL the ROOT libraries into the designated one.
# in principle we should figure out which ones we actually need and link only those.
# but.... some other time
function(link_root target)
    unset(libs CACHE)
    list(APPEND libs ${ROOT_LIBRARIES})
    foreach(lib ${libs})
        target_link_libraries(${target} PUBLIC ${lib})
    endforeach()
endfunction()

# we want to keep the subdirectory structure for the headers
function(install_headers headers)
    foreach(header ${headers})
        string(REGEX REPLACE "${CMAKE_SOURCE_DIR}/(.*)/[^/]*" "\\1" dir ${header})
        install(FILES ${header} DESTINATION inc/${dir})
    endforeach(header headers)
endfunction()


set(TARGET_LIBDIR lib)
set(TARGET_BINDIR bin)

###########   directory configuration

include_directories(
        ${CMAKE_SOURCE_DIR}
        ${CAFANACORE_INC}
        ${ROOT_INCLUDE_DIRS}
        ${OSCLIB_INC}
        ${STAN_MATH_INC}
        ${EIGEN_INC}
        ${BOOST_INC}
        ${IFDHC_INC}
)

# by default make the bin/ and lib/ dirs in the top of the source dir.
# can by overridden on command line if desired
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}" CACHE PATH "..." FORCE)
endif()
message(STATUS "Installing into: ${CMAKE_INSTALL_PREFIX}")

###########   now go into the subdirs and do the actual work
add_subdirectory(CAFAna)
