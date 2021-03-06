# ################################################################
# * Copyright (c) 2015-present, Yann Collet, Facebook, Inc.
# * All rights reserved.
# *
# * This source code is licensed under the BSD-style license found in the
# * LICENSE file in the root directory of this source tree. An additional grant
# * of patent rights can be found in the PATENTS file in the same directory.
#
# You can contact the author at :
#  - zstd homepage : http://www.zstd.net/
# ################################################################

PROJECT(programs)

SET(CMAKE_INCLUDE_CURRENT_DIR TRUE)

# Define programs directory, where sources and header files are located
SET(LIBRARY_DIR ${ZSTD_SOURCE_DIR}/lib)
SET(PROGRAMS_DIR ${ZSTD_SOURCE_DIR}/programs)
INCLUDE_DIRECTORIES(${PROGRAMS_DIR} ${LIBRARY_DIR} ${LIBRARY_DIR}/common ${LIBRARY_DIR}/compress ${LIBRARY_DIR}/dictBuilder)

IF (ZSTD_LEGACY_SUPPORT)
    SET(PROGRAMS_LEGACY_DIR ${PROGRAMS_DIR}/legacy)
    INCLUDE_DIRECTORIES(${PROGRAMS_LEGACY_DIR} ${LIBRARY_DIR}/legacy)
ENDIF (ZSTD_LEGACY_SUPPORT)

IF (MSVC)
    SET(MSVC_RESOURCE_DIR ${ZSTD_SOURCE_DIR}/build/VS2010/zstd)
    SET(PlatformDependResources ${MSVC_RESOURCE_DIR}/zstd.rc)
ENDIF (MSVC)

ADD_EXECUTABLE(zstd ${PROGRAMS_DIR}/zstdcli.c ${PROGRAMS_DIR}/fileio.c ${PROGRAMS_DIR}/bench.c ${PROGRAMS_DIR}/datagen.c ${PROGRAMS_DIR}/dibio.c ${PlatformDependResources})
TARGET_LINK_LIBRARIES(zstd libzstd_shared)
ADD_CUSTOM_COMMAND(TARGET zstd POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink zstd zstdcat COMMENT "Creating zstdcat symlink")
ADD_CUSTOM_COMMAND(TARGET zstd POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink zstd unzstd COMMENT "Creating unzstd symlink")
INSTALL(TARGETS zstd RUNTIME DESTINATION "bin")
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/zstdcat DESTINATION "bin")
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/unzstd DESTINATION "bin")

IF (UNIX)
    FILE(COPY ${PROGRAMS_DIR}/zstd.1 DESTINATION .)
    ADD_CUSTOM_COMMAND(TARGET zstd POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink zstd.1 zstdcat.1 COMMENT "Creating zstdcat.1 symlink")
    ADD_CUSTOM_COMMAND(TARGET zstd POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink zstd.1 unzstd.1 COMMENT "Creating unzstd.1 symlink")
    INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/zstd.1 DESTINATION "share/man/man1")
    INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/zstdcat.1 DESTINATION "share/man/man1")
    INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/unzstd.1 DESTINATION "share/man/man1")

    ADD_EXECUTABLE(zstd-frugal ${PROGRAMS_DIR}/zstdcli.c ${PROGRAMS_DIR}/fileio.c)
    TARGET_LINK_LIBRARIES(zstd-frugal libzstd_shared)
    SET_TARGET_PROPERTIES(zstd-frugal PROPERTIES COMPILE_DEFINITIONS "ZSTD_NOBENCH;ZSTD_NODICT")
ENDIF (UNIX)

IF (ZSTD_MULTITHREAD_SUPPORT)
    SET_TARGET_PROPERTIES(zstd PROPERTIES COMPILE_DEFINITIONS "ZSTD_MULTITHREAD")

    SET(THREADS_PREFER_PTHREAD_FLAG ON)
    FIND_PACKAGE(Threads REQUIRED)
    IF (CMAKE_USE_PTHREADS_INIT)
        TARGET_LINK_LIBRARIES(zstd ${CMAKE_THREAD_LIBS_INIT})
    ELSE()
        MESSAGE(SEND_ERROR "ZSTD currently does not support thread libraries other than pthreads")
    ENDIF()

    ADD_CUSTOM_COMMAND(TARGET zstd POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink zstd zstdmt COMMENT "Creating zstdmt symlink")
    INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/zstdmt DESTINATION "bin")
ENDIF (ZSTD_MULTITHREAD_SUPPORT)
