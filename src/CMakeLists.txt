CMAKE_MINIMUM_REQUIRED (VERSION 3.19)

INCLUDE ("${CMAKE_SOURCE_DIR}/cmake-common/sac2c-variables.cmake")
INCLUDE ("${CMAKE_SOURCE_DIR}/cmake-common/generate-sac2c-dependency-targets.cmake")

# C files relatively to thes CMakeLists.txt.
#SET (C_DEPS_SRC
#    src/../../.c
#    ...
#)

# SaC Mod files relatively to this CMakeLists.txt.
SET (SAC_SRC_MOD
    Example.sac
)

# SaC Prog files relatively to this CMakeLists.txt.
SET (SAC_SRC_PROG
    use-example.sac
)

# This functions tries to match a module name with its stored
# path information in a list. It returns on the first result, or
# returns empty.
FUNCTION (MATCH_NAME_IN_LIST list name ret)
    MESSAGE (DEBUG "Searching for ${name} in list...")
    FOREACH (item ${list})
        IF ("${item}" MATCHES "${name}\.(sac|xsac)")
            MESSAGE (DEBUG "   we have a match: ${item}!")
            SET (${ret} ${item} PARENT_SCOPE)
            BREAK ()
        ENDIF ()
    ENDFOREACH ()
ENDFUNCTION ()

# For every C source, compile an object file maintaining the right location
# in the binary dir so that sac files can pick it up.
# XXX do we ever pass some extra flags, etc to C files?
#FOREACH(name ${C_DEPS_SRC})
#  SET (src "${CMAKE_CURRENT_SOURCE_DIR}/${name}")
#
#  GET_FILENAME_COMPONENT (dir ${name} DIRECTORY)
#
#  GET_FILENAME_COMPONENT (dst ${name} NAME_WE)
#  SET (dst "${CMAKE_CURRENT_BINARY_DIR}/${dir}/${dst}${OBJEXT}")
#
#  # Make sure that we put the object file in the same location where
#  # the source file was.
#  FILE (MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/${dir}")
#
#  ADD_CUSTOM_COMMAND (
#    OUTPUT "${dst}"
#    MAIN_DEPENDENCY "${src}"
#    IMPLICIT_DEPENDS C "${src}"
#    COMMAND ${SAC2C} -v0 -noprelude -cc ccmod -o "${dst}" "${src}"
#    COMMENT "Generating ${dst}"
#  )
#ENDFOREACH(name)

# Make a directory for sac2c output
FILE (MAKE_DIRECTORY "${DLL_BUILD_DIR}/${TARGET_ENV}/${SBI}")

# For every sac mod file, compile Tree and Mod files.
FOREACH (name ${SAC_SRC_MOD})
    SET (src "${CMAKE_CURRENT_SOURCE_DIR}/${name}")
    # sac2c requires computes objectfiles relatively to the working directory
    # of the call to sac2c.
    GET_FILENAME_COMPONENT (dir "${CMAKE_CURRENT_BINARY_DIR}/${name}" DIRECTORY)
    GET_FILENAME_COMPONENT (dst ${name} NAME_WE)
    SET (mod
        "${DLL_BUILD_DIR}/${TARGET_ENV}/${SBI}/lib${dst}Mod${MODEXT}")
    SET (tree
        "${DLL_BUILD_DIR}/tree/${TARGET_ENV}/lib${dst}Tree${TREE_DLLEXT}")

    RESOLVE_SAC_DEPS_AS_TARGETS ("${name}" "<TARGET>-module-<NAME>" target_list objs_list source_list)
    MESSAGE (STATUS "Computing dependencies for `${name}'")
    SET (src_deps)
    FOREACH (src_name ${source_list})
        MESSAGE (DEBUG "Searching for ${src_name} in source path list...")
        MATCH_NAME_IN_LIST ("${SAC_SRC_MOD}" "${src_name}" src_path)
        IF (src_path)
            MESSAGE (DEBUG "   found ${src_path}")
            LIST (APPEND src_deps "${CMAKE_CURRENT_SOURCE_DIR}/${src_path}")
        ELSE ()
            MESSAGE (FATAL_ERROR "Unable to find path to ${src_name} in sources!")
        ENDIF ()
    ENDFOREACH ()
    LIST (APPEND deps_list ${src_deps} ${objs_list} ${target_list})
    UNSET (src_deps)
    UNSET (objs_list)
    UNSET (source_list)
    UNSET (target_list)
    MESSAGE (DEBUG "For ${name} the computed dependences are: ${deps_list}")

    # Make sure that we have a directory we are changing to.
    FILE (MAKE_DIRECTORY "${dir}")

    ADD_CUSTOM_COMMAND (
        OUTPUT ${mod} ${tree}
        COMMAND
        ${SAC2C} -v0 -linksetsize ${LINKSETSIZE} -o ${DLL_BUILD_DIR} ${src}
        WORKING_DIRECTORY
            ${dir}
        MAIN_DEPENDENCY "${src}"
        DEPENDS ${deps_list}
        COMMENT "Building ${name} module for target `${TARGET}'")
    UNSET (deps_list)

    # Install compiled Tree/Mod parts of the compiled module
    # to the corresponding locations.
    INSTALL (
        FILES ${mod}
        DESTINATION ${INSTALL_MOD_DIR}/${TARGET_ENV}/${SBI})
    INSTALL (
        FILES ${tree}
        DESTINATION ${INSTALL_TREE_DIR}/tree/${TARGET_ENV})

    # Make a call to the command that compiles the module a part
    # of the default build process.
    ADD_CUSTOM_TARGET (${TARGET}-module-${dst} ALL DEPENDS "${mod}" "${tree}")
ENDFOREACH (name)

# For every sac prog file, compile the binary file.
FOREACH (name ${SAC_SRC_PROG})
    SET (src "${CMAKE_CURRENT_SOURCE_DIR}/${name}")
    # sac2c requires computes objectfiles relatively to the working directory
    # of the call to sac2c.
    GET_FILENAME_COMPONENT (dir "${CMAKE_CURRENT_BINARY_DIR}/${name}" DIRECTORY)
    GET_FILENAME_COMPONENT (dst ${name} NAME_WE)
    SET (prog "${CMAKE_BINARY_DIR}/${dst}-${TARGET}")

    RESOLVE_SAC_DEPS_AS_TARGETS ("${name}" "<TARGET>-module-<NAME>" target_list objs_list source_list)
    MESSAGE (STATUS "Computing dependencies for `${name}'")
    SET (src_deps)
    FOREACH (src_name ${source_list})
        MESSAGE (DEBUG "Searching for ${src_name} in source path list...")
        MATCH_NAME_IN_LIST ("${SAC_SRC_MOD}" "${src_name}" src_path)
        IF (src_path)
            MESSAGE (DEBUG "   found ${src_path}")
            LIST (APPEND src_deps "${CMAKE_CURRENT_SOURCE_DIR}/${src_path}")
        ELSE ()
            MESSAGE (FATAL_ERROR "Unable to find path to ${src_name} in sources!")
        ENDIF ()
    ENDFOREACH ()
    LIST (APPEND deps_list ${src_deps} ${objs_list} ${target_list})
    UNSET (src_deps)
    UNSET (objs_list)
    UNSET (source_list)
    UNSET (target_list)
    MESSAGE (DEBUG "For ${name} the computed dependences are: ${deps_list}")

    ADD_CUSTOM_COMMAND (
        OUTPUT ${prog}
        COMMAND
        ${SAC2C} -v0 -linksetsize ${LINKSETSIZE} -o ${prog} ${src}
        WORKING_DIRECTORY
            ${dir}
        MAIN_DEPENDENCY "${src}"
        DEPENDS ${deps_list}
        COMMENT "Building ${name} program for target `${TARGET}'")
    UNSET (deps_list)

    # Install compiled Tree/Mod parts of the compiled module
    # to the corresponding locations.
    INSTALL (
        FILES ${prog}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

    # Make a call to the command that compiles the module a part
    # of the default build process.
    ADD_CUSTOM_TARGET (${TARGET}-prog-${dst} ALL DEPENDS ${prog})
ENDFOREACH (name)
