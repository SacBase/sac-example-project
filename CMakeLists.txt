CMAKE_MINIMUM_REQUIRED (VERSION 3.19)

# This is an example project that builds both a SaC module and a SaC program.
# The program depends on the module; these are located under `src/`.
# It is important that create the sac2crc file for this project, otherwise
# its tricky to correctly link the SaC modules with the program.

PROJECT (sac-example-module VERSION 1.1.0 HOMEPAGE_URL "https://sac-home.org/")

# Where the compiled sac modules result
SET (DLL_BUILD_DIR  "${PROJECT_BINARY_DIR}/lib")

# For what targets we build modules
SET (TARGETS            seq mt_pth  CACHE STRING "Build stdlib-cuda for these targets")
SET (SAC2C_EXEC                     CACHE STRING "A path to sac2c compiler")
SET (LINKSETSIZE        "0"         CACHE STRING "Set a value for -linksetsize parameter of sac2c")

# Check whether sac2c is operational
INCLUDE ("cmake-common/check-sac2c.cmake")
INCLUDE ("cmake-common/misc-macros.cmake")

# add our custom Find*.cmake in local cmake directory
#LIST (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# For every target run CMakeLists.txt in src
FOREACH (TARGET IN ITEMS ${TARGETS})
    ADD_SUBDIRECTORY (src   src-${TARGET})
ENDFOREACH ()

# This build target is responsible for generating the package sac2crc file
CREATE_SAC2CRC_TARGET ("examplemodule" "${DLL_BUILD_DIR}" "${DLL_BUILD_DIR}" "")
