cmake_minimum_required(VERSION 3.7.0 FATAL_ERROR) #CPACK_DEBIAN_<COMPONENT>_PACKAGE_NAME
include(${CMAKE_SOURCE_DIR}/cmake/RequireOutOfSourceBuild.cmake)

find_package(IRODS 4.2.8 EXACT REQUIRED CONFIG)

set(CMAKE_C_COMPILER ${IRODS_EXTERNALS_FULLPATH_CLANG}/bin/clang)
set(CMAKE_CXX_COMPILER ${IRODS_EXTERNALS_FULLPATH_CLANG}/bin/clang++)

set(CMAKE_EXE_LINKER_FLAGS_INIT "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++")

project(authentication-command C CXX)

set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

include(${IRODS_TARGETS_PATH})

if (NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build {Debug, Release}." FORCE)
  message(STATUS "Setting unspecified CMAKE_BUILD_TYPE to '${CMAKE_BUILD_TYPE}'. This is the correct setting for normal builds.")
endif()

if (NOT CPACK_PACKAGING_INSTALL_PREFIX)
  set(CPACK_PACKAGING_INSTALL_PREFIX "/" CACHE STRING "Package root path. \"/\" is correct for normal package builds.." FORCE)
  message(STATUS "Setting unspecified CPACK_PACKAGING_INSTALL_PREFIX to '${CPACK_PACKAGING_INSTALL_PREFIX}'. This is the correct setting for normal builds.")
endif()

set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_INSTALL_RPATH ${IRODS_EXTERNALS_FULLPATH_CLANG_RUNTIME}/lib)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++")
set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -stdlib=libc++")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -stdlib=libc++")
add_compile_options(-nostdinc++ -Wall -Wextra -Werror -Wno-unused-parameter)
link_libraries(c++abi)
include_directories(${IRODS_EXTERNALS_FULLPATH_CLANG}/include/c++/v1)

set(
  IRODS_CLIENT_ICOMMANDS_EXECUTABLES
  iauth
  )

foreach(EXECUTABLE ${IRODS_CLIENT_ICOMMANDS_EXECUTABLES})
  add_executable(
    ${EXECUTABLE}
    ${CMAKE_SOURCE_DIR}/src/${EXECUTABLE}.cpp
    )
  target_link_libraries(
    ${EXECUTABLE}
    PRIVATE
    irods_client
    irods_plugin_dependencies
    irods_common
    ${IRODS_EXTERNALS_FULLPATH_AVRO}/lib/libavrocpp.so
    ${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_filesystem.so
    ${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_program_options.so
    ${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_system.so
    ${IRODS_EXTERNALS_FULLPATH_ZMQ}/lib/libzmq.so
    dl
    )
  target_include_directories(
    ${EXECUTABLE}
    PRIVATE
    ${IRODS_INCLUDE_DIRS}
    ${IRODS_EXTERNALS_FULLPATH_AVRO}/include
    ${IRODS_EXTERNALS_FULLPATH_BOOST}/include
    ${IRODS_EXTERNALS_FULLPATH_ARCHIVE}/include
    ${IRODS_EXTERNALS_FULLPATH_CPPZMQ}/include
    ${IRODS_EXTERNALS_FULLPATH_ZMQ}/include
    ${IRODS_EXTERNALS_FULLPATH_JSON}/include
    ${CMAKE_SOURCE_DIR}/../plugins
    )
  target_compile_definitions(${EXECUTABLE} PRIVATE ${IRODS_COMPILE_DEFINITIONS} BOOST_SYSTEM_NO_DEPRECATED)
  target_compile_options(${EXECUTABLE} PRIVATE -Wno-write-strings)
  set_property(TARGET ${EXECUTABLE} PROPERTY CXX_STANDARD ${IRODS_CXX_STANDARD})
  install(
    TARGETS
    ${EXECUTABLE}
    RUNTIME
    DESTINATION usr/bin
    )

  if (IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "centos")
    if (IRODS_LINUX_DISTRIBUTION_VERSION_MAJOR STREQUAL "6")
      continue()
    endif()
  endif()

  if (${EXECUTABLE} STREQUAL "iapitest")
    continue()
  endif()

endforeach()

if (NOT CPACK_DEBIAN_PACKAGE_VERSION)
  set(CPACK_DEBIAN_PACKAGE_VERSION ${IRODS_CPACK_DEBIAN_PACKAGE_VERSION})
endif()

set(CPACK_PACKAGE_FILE_NAME "irods-authentication_command${IRODS_PACKAGE_FILE_NAME_SUFFIX}")
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_COMPONENT_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_COMPONENTS_GROUPING IGNORE)
set(CPACK_PACKAGE_VERSION ${IRODS_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR ${IRODS_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${IRODS_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${IRODS_VERSION_PATCH})
set(CPACK_PACKAGE_CONTACT "Renaissance Computing Institute <info@irods.org>")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "The integrated Rule-Oriented Data System")
set(CPACK_PACKAGE_VENDOR "Renaissance Computing Institute <info@irods.org>")

set(CPACK_DEB_COMPONENT_INSTALL OFF)
set(CPACK_DEBIAN_PACKAGE_SECTION "contrib/science")
set(CPACK_DEBIAN_COMPRESSION_TYPE "gzip")
set(CPACK_DEBIAN_PACKAGE_PRIORITY "extra")
set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://irods.org")
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS OFF)
set(CPACK_DEBIAN_PACKAGE_CONTROL_STRICT_PERMISSION ON)

set(CPACK_RPM_COMPONENT_INSTALL OFF)
set(CPACK_RPM_PACKAGE_RELEASE "1")
set(CPACK_RPM_PACKAGE_LICENSE "BSD-3-Clause")
set(CPACK_RPM_PACKAGE_VENDOR "iRODS Consortium")
set(CPACK_RPM_PACKAGE_URL "https://irods.org")
set(CPACK_RPM_PACKAGE_AUTOREQ 0)
set(CPACK_RPM_PACKAGE_AUTOPROV 0)
set(CPACK_RPM_PACKAGE_RELOCATABLE ON)

set(CPACK_ARCHIVE_COMPONENT_INSTALL OFF)

set(CPACK_DEBIAN_PACKAGE_NAME "irods-authentication-command")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "${IRODS_PACKAGE_DEPENDENCIES_STRING}, irods-runtime (= ${CPACK_DEBIAN_PACKAGE_VERSION}), libc6, libssl1.0.0")
set(CPACK_DEBIAN_PACKAGE_REPLACES "irods-icat, irods-resource")

set(CPACK_RPM_PACKAGE_NAME "irods-authentication-command")
set(CPACK_RPM_PACKAGE_OBSOLETES "irods-icat, irods-resource")
if (IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "centos" OR IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "centos linux")
  set(CPACK_RPM_PACKAGE_REQUIRES "${IRODS_PACKAGE_DEPENDENCIES_STRING}, irods-runtime = ${IRODS_VERSION}, openssl")
elseif (IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "opensuse")
  set(CPACK_RPM_PACKAGE_REQUIRES "${IRODS_PACKAGE_DEPENDENCIES_STRING}, irods-runtime = ${IRODS_VERSION}, libopenssl1_0_0")
endif()

if (NOT CPACK_GENERATOR)
  set(CPACK_GENERATOR ${IRODS_CPACK_GENERATOR} CACHE STRING "CPack generator to use, e.g. {DEB, RPM, TGZ}." FORCE)
  message(STATUS "Setting unspecified CPACK_GENERATOR to ${CPACK_GENERATOR}. This is the correct setting for normal builds.")
endif()

include(CPack)
