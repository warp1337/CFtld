# Open The CMake GUI
# specify the source directory and the binary directory
# press configure.
#
# UNIX Makefile:
# 1) go to the binary folder and type "make" to build the project
#
# Microsoft Visual C++:
# 1) open the .sln file
# 2) build the project "ALL_BUILD" to build the opentld project

CMAKE_MINIMUM_REQUIRED(VERSION 2.8.3)

project(cf_tld_ros)


# SET(DEFAULT_CXX_FLAGS "" CACHE STRING "-O2 -march=native -Wno-deprecated-declarations")
# SET(CMAKE_CXX_FLAGS "${DEFAULT_CXX_FLAGS} ${CMAKE_CXX_FLAGS} -std=gnu++11 -Wno-deprecated-declarations")
SET(OpenCV_MIN_VERSION "3.3.1")
SET(ROS_VERSION "kinetic")
SET(ROS_BUILD_TYPE Release)
SET(CMAKE_BUILD_TYPE Release)

FIND_PACKAGE(catkin REQUIRED COMPONENTS tf
                                        roscpp
                                        std_msgs
                                        cv_bridge
                                        sensor_msgs
                                        people_msgs
                                        geometry_msgs
                                        message_filters
                                        message_runtime
                                        image_transport
                                        visualization_msgs
                                        bayes_people_tracker_msgs
)

# add c++11 support
if(CMAKE_COMPILER_IS_GNUCC)
    ADD_DEFINITIONS ( -std=c++11 )
endif(CMAKE_COMPILER_IS_GNUCC)

# find_package(OpenCV) does not account for proper path setup
# on windows; proper path setup requires path "OPENCV_DIR"
# pointing to ie. \build_opencv_3\install\x64\vc12
set(OPENCV_DIR_HINT "")

set(OpenCV_STATIC OFF)
find_package(OpenCV REQUIRED HINTS ${OPENCV_DIR_HINT})
# OpenCV 3.0 does currently not include its' includes correctly
include_directories(${OpenCV_DIR}/include ${catkin_INCLUDE_DIRS})

CATKIN_PACKAGE(INCLUDE_DIRS
               CATKIN_DEPENDS tf
                              roscpp
                              std_msgs
                              cv_bridge
                              sensor_msgs
                              people_msgs
                              geometry_msgs
                              message_filters
                              message_runtime
                              image_transport
                              bayes_people_tracker_msgs
)


#-------------------------------------------------------------------------------
#version
set(TLD_VERSION_MAJOR 1)
set(TLD_VERSION_MINOR 4)
set(TLD_VERSION_PATCH 0)
set(TLD_VERSION ${TLD_VERSION_MAJOR}.${TLD_VERSION_MINOR}.${TLD_VERSION_PATCH})

#------------------------------------------------------------------------------
#build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
endif(NOT CMAKE_BUILD_TYPE)

option(USE_SYSTEM_LIBS "Use the installed version of libconfig++." OFF)
option(WITH_OPENMP "Use OpenMP." OFF)

if(WITH_OPENMP)
    find_package(OpenMP REQUIRED)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif(WITH_OPENMP)

if(WIN32)
    add_definitions(-DLIBCONFIGXX_STATIC -DLIBCONFIG_STATIC) #Needed when linking libconfig statically
endif(WIN32)

if(APPLE)
    add_definitions(-DHAVE_XLOCALE_H)
endif(APPLE)

if(NOT MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif(NOT MSVC)

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

#-------------------------------------------------------------------------------
# add subdirectories
add_subdirectory(src/3rdparty/cf_tracking)

if(NOT USE_SYSTEM_LIBS)
    add_subdirectory(src/3rdparty/libconfig)
endif(NOT USE_SYSTEM_LIBS)

add_subdirectory(src/libopentld)
add_subdirectory(src/opentld)

configure_file("${PROJECT_SOURCE_DIR}/OpenTLDConfig.cmake.in" "${PROJECT_BINARY_DIR}/OpenTLDConfig.cmake" @ONLY)

# INSTALL(TARGETS clf_perception_surb RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
# INSTALL(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/data" DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
