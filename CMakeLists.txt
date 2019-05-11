# Author: Shinjiro Sueda
# Edited: Nick Weidner
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

# Name of the project
PROJECT(eol-cloth)

set(CMAKE_CXX_STANDARD 11)

#-----------------------------------------------
# USER SETTINGS
#-----------------------------------------------

# COMPILER OPTIONS

OPTION(ONLINE "Build with online rendering supported" ON)

# Include support for one or multiple solvers
OPTION(MOSEK "Build with Mosek support" OFF)
OPTION(GUROBI "Build with Gurobi support" OFF)
OPTION(IGL "Build with libIGL support" OFF) # Not used in the current version


# LIBRARY PATHS
# These paths are machine specific
# These paths are examples

SET(DEF_DIR_EIGEN "D:\\c++\\eigen-3.3.4")

IF(${ONLINE})
    SET(DEF_DIR_GLM "D:\\c++\\glm-0.9.8.4")
    SET(DEF_DIR_GLFW "D:\\c++\\glfw-3.2.1")
    SET(DEF_DIR_GLEW "D:\\c++\\glew-2.1.0")
    ADD_DEFINITIONS(-DEOLC_ONLINE)
ENDIF()

IF(${MOSEK}) 
    SET(DEF_DIR_MOSEK "C:\\Program Files\\Mosek\\8\\tools\\platform\\win64x86") # Location of /h and /bin
    SET(DEF_BINARY_MOSEK "mosek64_8_1.lib") # Platform specific
    ADD_DEFINITIONS(-DkMosekLicensePath="C:/Users/weid691/mosek/mosek.lic") # Path to license file
    ADD_DEFINITIONS(-DEOLC_MOSEK)
ENDIF()

IF(${GUROBI})
    SET(DEF_DIR_GUROBI "C:\\gurobi752\\win64")
    SET(DEF_BINARY_GUROBI "gurobi75.lib") # Platform specific
    SET(DEF_BINARY_GUROBI_RELEASE "gurobi_c++md2017.lib") # Platform specific
    SET(DEF_BINARY_GUROBI_DEBUG "gurobi_c++mdd2017.lib") # Platform specific
    ADD_DEFINITIONS(-DEOLC_GUROBI)
ENDIF()

IF(${IGL})
    SET(DEF_DIR_IGL "C:\\c++\\libigl-1.2.1\\include") # Does not build static lib
    ADD_DEFINITIONS(-DEOLC_IGL)
ENDIF()

# Additional defines
ADD_DEFINITIONS(-D_USE_MATH_DEFINES)
ADD_DEFINITIONS(-D_CRT_NO_VA_START_VALIDATION)

#-----------------------------------------------
# END USER SETTINGS, DON'T TOUCH BELOW
#-----------------------------------------------

# Use glob to get the list of all source files.
# We don't really need to include header and resource files to build, but it's
# nice to have them also show up in IDEs.
FILE(GLOB SOURCES "${PROJECT_SOURCE_DIR}/src/*.cpp" "${PROJECT_SOURCE_DIR}/src/*.c")
FILE(GLOB HEADERS "${PROJECT_SOURCE_DIR}/src/*.hpp" "${PROJECT_SOURCE_DIR}/src/*.h")
FILE(GLOB_RECURSE GLSL "resources/*.glsl")
FILE(GLOB ARCSIMC "${PROJECT_SOURCE_DIR}/src/external/ArcSim/*.cpp" "${PROJECT_SOURCE_DIR}/src/external/ArcSim/*.c")
FILE(GLOB ARCSIMH "${PROJECT_SOURCE_DIR}/src/external/ArcSim/*.hpp" "${PROJECT_SOURCE_DIR}/src/external/ArcSim/*.h")
FILE(GLOB JSONC "${PROJECT_SOURCE_DIR}/src/external/Json/*.cpp" "${PROJECT_SOURCE_DIR}/src/external/Json/*.c")
FILE(GLOB JSONH "${PROJECT_SOURCE_DIR}/src/external/Json/*.hpp" "${PROJECT_SOURCE_DIR}/src/external/Json/*.h")

IF(${ONLINE})
    FILE(GLOB ONLINEC "${PROJECT_SOURCE_DIR}/src/online/*.cpp" "${PROJECT_SOURCE_DIR}/src/online/*.c")
    FILE(GLOB ONLINEH "${PROJECT_SOURCE_DIR}/src/online/*.hpp" "${PROJECT_SOURCE_DIR}/src/online/*.h")
ENDIF()
IF(${MOSEK})
    FILE(GLOB MOSEKC "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Mosek/*.cpp" "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Mosek/*.c")
    FILE(GLOB MOSEKH "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Mosek/*.hpp" "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Mosek/*.h")
ENDIF()
IF(${GUROBI})
    FILE(GLOB GUROBIC "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Gurobi/*.cpp" "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Gurobi/*.c")
    FILE(GLOB GUROBIH "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Gurobi/*.hpp" "${PROJECT_SOURCE_DIR}/src/external/SolverWrappers/Gurobi/*.h")
ENDIF()
# FILE(GLOB_RECURSE SOURCES 
# RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
# ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
# ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h)
# FILE(GLOB_RECURSE EXTERNAL 
# RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
# ${CMAKE_CURRENT_SOURCE_DIR}/external/ArcSim/*.cpp
# ${CMAKE_CURRENT_SOURCE_DIR}/external/ArcSim/*.hpp)

# SOURCE_GROUP(Source FILES ${SOURCES})
# SOURCE_GROUP(External FILES ${EXTERNAL})

# Set the executable.
ADD_EXECUTABLE(${CMAKE_PROJECT_NAME} ${SOURCES} ${HEADERS} ${GLSL} ${ARCSIMC} ${ARCSIMH} ${JSONC} ${JSONH} ${ONLINEC} ${ONLINEH} ${MOSEKC} ${MOSEKH} ${GUROBIC} ${GUROBIH})

# Get the EIGEN environment variable. Since EIGEN is a header-only library, we
find_package(Eigen3 REQUIRED)
INCLUDE_DIRECTORIES(${EIGEN3_INCLUDE_DIR})

find_package(Threads REQUIRED)
target_link_libraries(${CMAKE_PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT})

IF(${ONLINE})
    # Get the GLM environment variable. Since GLM is a header-only library, we
    # just need to add it to the include directory.
    find_package(glm REQUIRED)
    INCLUDE_DIRECTORIES(${GLM_INCLUDE_DIR})

    # Get the GLFW environment variable. There should be a CMakeLists.txt in the 
    find_package(glfw3 REQUIRED)
    INCLUDE_DIRECTORIES(${GLFW_INCLUDE_DIR})
    TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} glfw)

    # Get the GLEW environment variable.
    find_package(GLEW)
    INCLUDE_DIRECTORIES(${GLEW_INCLUDE_DIR})
    TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} ${GLEW_LIBRARIES})
ENDIF()

# Get the MOSEK environment variable.
IF(${MOSEK})
    IF(NOT MOSEK_DIR)
        SET(MOSEK_DIR "$ENV{MOSEK_DIR}")
    ENDIF()
    IF(NOT MOSEK_DIR)
        # The environment variable was not set
        SET(ERR_MSG "Please point the environment variable MOSEK_DIR to the root directory of your MOSEK installation.")
        MESSAGE(STATUS "Looking for MOSEK in ${DEF_DIR_MOSEK}")
        IF(IS_DIRECTORY ${DEF_DIR_MOSEK})
            MESSAGE(STATUS "Found!")
            SET(MOSEK_DIR ${DEF_DIR_MOSEK})
        ELSE()
            MESSAGE(FATAL_ERROR ${ERR_MSG})
        ENDIF()
    ENDIF()
    INCLUDE_DIRECTORIES(${MOSEK_DIR}/h)
    # With prebuilt binaries
    TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} ${MOSEK_DIR}/bin/${DEF_BINARY_MOSEK})
ENDIF()

# Get the MOSEK environment variable.
IF(${GUROBI})
    message("CMake Paths " ${CMAKE_PREFIX_PATH})
    find_path(GUROBI_INCLUDE_DIR NAMES gurobi_c++.h HINTS $ENV{GUROBI_HOME})
    find_library(GUROBI_LIBRARY NAMES gurobi gurobi81 gurobi810 HINTS $ENV{GUROBI_HOME})
    find_library(GUROBI_CXX_LIBRARY NAMES gurobi_c++ HINTS $ENV{GUROBI_HOME})
    message("gurobi: " ${GUROBI_INCLUDE_DIR} " " ${GUROBI_LIBRARY} " " ${GUROBI_CXX_LIBRARY})
    set(GUROBI_INCLUDE_DIRS ${GUROBI_INCLUDE_DIR})
    set(GUROBI_LIBRARIES ${GUROBI_LIBRARY} ${GUROBI_CXX_LIBRARY})
    # if (NOT GUROBI_INCLUDE_DIR OR NOT GUROBI_LIBRARIES)
    #     message(FATAL_ERROR "GUROBI_INCLUDE_DIR or GUROBI_LIBRARIES not set")
    # endif()
    INCLUDE_DIRECTORIES(${GUROBI_INCLUDE_DIR})
    # With prebuilt binaries
    TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} ${GUROBI_LIBRARIES})
    #TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} debug ${GUROBI_DIR}/lib/${DEF_BINARY_GUROBI_DEBUG})
    #TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} optimized ${GUROBI_DIR}/lib/${DEF_BINARY_GUROBI_RELEASE})
ENDIF(${GUROBI})

# Get the IGL environment variable.
IF(${IGL})
    IF(NOT IGL_INCLUDE_DIR)
        SET(IGL_INCLUDE_DIR "$ENV{IGL_INCLUDE_DIR}")
    ENDIF()
    IF(NOT IGL_INCLUDE_DIR)
        # The environment variable was not set
        SET(ERR_MSG "Please point the environment variable IGL_INCLUDE_DIR to the root directory of your IGL installation.")
        MESSAGE(STATUS "Looking for IGL in ${DEF_DIR_IGL}")
        IF(IS_DIRECTORY ${DEF_DIR_IGL})
            MESSAGE(STATUS "Found!")
            SET(GUROBI_DIR ${DEF_DIR_IGL})
        ELSE()
            MESSAGE(FATAL_ERROR ${ERR_MSG})
        ENDIF()
    ENDIF()
    INCLUDE_DIRECTORIES(${IGL_INCLUDE_DIR})
ENDIF()

# OS specific options and libraries
IF(WIN32)
    # c++11 is enabled by default.
    # -Wall produces way too many warnings.
    # -pedantic is not supported.
    # Disable warning 4996.
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4996")
    IF(${ONLINE})
        TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} opengl32.lib)
    ELSE()
        TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME})
    ENDIF()
ELSE()
    # Enable all pedantic warnings.
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -pedantic")
    IF(${ONLINE})
        IF(APPLE)
            # Add required frameworks for GLFW.
            TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} "-framework OpenGL -framework Cocoa -framework IOKit -framework CoreVideo")
        ELSE()
            #Link the Linux OpenGL library
            TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME} "GL")
        ENDIF()
    ENDIF()
ENDIF()
