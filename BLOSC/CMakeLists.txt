cmake_minimum_required (VERSION 3.1.0)
project (H5BLOSC C CXX)

#-----------------------------------------------------------------------------
# Basic H5BLOSC stuff here
#-----------------------------------------------------------------------------

#-----------------------------------------------------------------------------
# Define some CMake variables for use later in the project
#-----------------------------------------------------------------------------
set (H5BLOSC_RESOURCES_DIR           ${H5BLOSC_SOURCE_DIR}/config/cmake)
set (HDF_RESOURCES_DIR               ${H5BLOSC_SOURCE_DIR}/config/cmake)
set (H5BLOSC_SRC_DIR                 ${H5BLOSC_SOURCE_DIR}/src)

#-----------------------------------------------------------------------------
# Set the core names of all the libraries
#-----------------------------------------------------------------------------
set (H5BLOSC_LIB_CORENAME              "h5blosc")

#-----------------------------------------------------------------------------
# Set the true names of all the libraries if customized by external project
#-----------------------------------------------------------------------------
set (H5BLOSC_LIB_NAME              "${H5BLOSC_EXTERNAL_LIB_PREFIX}${H5BLOSC_LIB_CORENAME}")

#-----------------------------------------------------------------------------
# Set the target names of all the libraries
#-----------------------------------------------------------------------------
set (H5BLOSC_LIB_TARGET              ${H5BLOSC_LIB_CORENAME})

if (NOT H5BLOSC_INSTALL_BIN_DIR)
  set (H5BLOSC_INSTALL_BIN_DIR bin)
endif (NOT H5BLOSC_INSTALL_BIN_DIR)
if (NOT H5BLOSC_INSTALL_LIB_DIR)
  set (H5BLOSC_INSTALL_LIB_DIR lib/plugin)
endif (NOT H5BLOSC_INSTALL_LIB_DIR)
if (NOT H5BLOSC_INSTALL_INCLUDE_DIR)
  set (H5BLOSC_INSTALL_INCLUDE_DIR include)
endif (NOT H5BLOSC_INSTALL_INCLUDE_DIR)
if (NOT H5BLOSC_INSTALL_DATA_DIR)
  if (NOT WIN32)
    set (H5BLOSC_INSTALL_DATA_DIR share)
    set (H5BLOSC_INSTALL_CMAKE_DIR share/cmake)
  else (NOT WIN32)
    set (H5BLOSC_INSTALL_DATA_DIR ".")
    set (H5BLOSC_INSTALL_CMAKE_DIR cmake)
  endif (NOT WIN32)
endif (NOT H5BLOSC_INSTALL_DATA_DIR)

set (H5BLOSC_VERS_MAJOR 3)
set (H5BLOSC_VERS_MINOR 1)
set (H5BLOSC_VERS_RELEASE 3)

set (H5BLOSC_SOVERS_MAJOR 3)
set (H5BLOSC_SOVERS_MINOR 1)
set (H5BLOSC_SOVERS_INTERFACE 3)
set (H5BLOSC_SOVERS_RELEASE 0)
math (EXPR H5BLOSC_SOVERS_MAJOR ${H5BLOSC_SOVERS_INTERFACE}-${H5BLOSC_SOVERS_RELEASE})

#-----------------------------------------------------------------------------
set (H5BLOSC_PACKAGE "h5blosc")
set (H5BLOSC_PACKAGE_NAME "h5blosc")
set (H5BLOSC_PACKAGE_VERSION "${H5BLOSC_VERS_MAJOR}.${H5BLOSC_VERS_MINOR}.${H5BLOSC_VERS_RELEASE}")
set (H5BLOSC_PACKAGE_VERSION_STRING "${H5BLOSC_PACKAGE_VERSION}")
set (H5BLOSC_PACKAGE_VERSION_MAJOR "${H5BLOSC_VERS_MAJOR}")
set (H5BLOSC_PACKAGE_VERSION_MINOR "${H5BLOSC_VERS_MINOR}")
set (H5BLOSC_PACKAGE_STRING "${H5BLOSC_PACKAGE_NAME} ${H5BLOSC_PACKAGE_VERSION}")
set (H5BLOSC_PACKAGE_TARNAME "${H5BLOSC_PACKAGE_NAME}{HDF_PACKAGE_EXT}")
set (H5BLOSC_PACKAGE_URL "http://www.hdfgroup.org")
set (H5BLOSC_PACKAGE_BUGREPORT "help@hdfgroup.org")
set (H5BLOSC_PACKAGE_SOVERSION "${H5BLOSC_SOVERS_MAJOR}.${H5BLOSC_SOVERS_RELEASE}.${H5BLOSC_SOVERS_MINOR}")

#-----------------------------------------------------------------------------
# Include some macros for reusable code
#-----------------------------------------------------------------------------
include (${H5BLOSC_RESOURCES_DIR}/H5BLOSCMacros.cmake)

#-----------------------------------------------------------------------------
# Setup output Directories
#-----------------------------------------------------------------------------
if (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
  set (CMAKE_RUNTIME_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Executables."
  )
  set (CMAKE_LIBRARY_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Libraries"
  )
  set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all static libraries."
  )
else (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
  # if we are externally configured, but the project uses old cmake scripts
  # this may not be set
  if (NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
    set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${EXECUTABLE_OUTPUT_PATH})
  endif (NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
endif (NOT H5BLOSC_EXTERNALLY_CONFIGURED)

#-----------------------------------------------------------------------------
# Targets built within this project are exported at Install time for use
# by other projects using FindH5BLOSC.
#-----------------------------------------------------------------------------
if (NOT H5BLOSC_EXPORTED_TARGETS)
  set (H5BLOSC_EXPORTED_TARGETS "H5blosc-targets")
endif (NOT H5BLOSC_EXPORTED_TARGETS)

#-----------------------------------------------------------------------------
# To include a library in the list exported by the project AT BUILD TIME,
# add it to this variable. This is NOT used by Make Install, but for projects
# which include h5blosc as a sub-project within their build tree
#-----------------------------------------------------------------------------
set_global_variable (H5BLOSC_LIBRARIES_TO_EXPORT "")

#-----------------------------------------------------------------------------
# Run all the CMake configuration tests for our build environment
#-----------------------------------------------------------------------------
include (${H5BLOSC_RESOURCES_DIR}/ConfigureChecks.cmake)

#-----------------------------------------------------------------------------
# Option to Build Shared/Static libs, only shared is allowed
#-----------------------------------------------------------------------------
option (BUILD_SHARED_LIBS "Build Shared Libraries" ON)
set (LIB_TYPE SHARED)
if (BUILD_SHARED_LIBS)
  set (LIB_TYPE SHARED)
  add_definitions (-DH5BLOSC_BUILT_AS_DYNAMIC_LIB)
else (BUILD_SHARED_LIBS)
  message (FATAL_ERROR " ${H5BLOSC_PACKAGE_NAME} library must be built as a shared library")
endif (BUILD_SHARED_LIBS)

#-----------------------------------------------------------------------------
# HDF5 support
#-----------------------------------------------------------------------------
set (CMAKE_MODULE_PATH ${HDF_RESOURCES_DIR} ${CMAKE_MODULE_PATH})

add_definitions (-DH5_USE_18_API)

if (NOT HDF5_HDF5_HEADER)
  if (USE_SHARED_LIBS)
    set (FIND_HDF_COMPONENTS C shared static)
  else (USE_SHARED_LIBS)
    set (FIND_HDF_COMPONENTS C static)
  endif (USE_SHARED_LIBS)
  message (STATUS "HDF5 find comps: ${FIND_HDF_COMPONENTS}")
  set (SEARCH_PACKAGE_NAME ${HDF5_PACKAGE_NAME})

  find_package (HDF5 NAMES ${SEARCH_PACKAGE_NAME} COMPONENTS ${FIND_HDF_COMPONENTS})
  message (STATUS "HDF5 C libs:${HDF5_FOUND} static:${HDF5_static_C_FOUND} and shared:${HDF5_shared_C_FOUND}")
  if (HDF5_FOUND)
    add_executable (h5repack IMPORTED)
    add_executable (h5dump IMPORTED)
    if (NOT HDF5_static_C_FOUND AND NOT HDF5_shared_C_FOUND)
      #find library from non-dual-binary package
      set (FIND_HDF_COMPONENTS C)
      message (STATUS "HDF5 find comps: ${FIND_HDF_COMPONENTS}")

      find_package (HDF5 NAMES ${SEARCH_PACKAGE_NAME} COMPONENTS ${FIND_HDF_COMPONENTS})
      message (STATUS "HDF5 libs:${HDF5_FOUND} C:${HDF5_C_FOUND}")
      set (HDF5_LINK_LIBS ${HDF5_LINK_LIBS} ${HDF5_LIBRARIES})
      #Blosc source needs to be linked with HDF5
      set (LINK_LIBS ${LINK_LIBS} ${HDF5_LIBRARIES})
      if (USE_SHARED_LIBS AND WIN32)
        set_property (TARGET h5repack PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5repackdll")
        set_property (TARGET h5dump PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5dumpdll")
      else (USE_SHARED_LIBS AND WIN32)
        set_property (TARGET h5repack PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5repack")
        set_property (TARGET h5dump PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5dump")
      endif (USE_SHARED_LIBS AND WIN32)
    else (NOT HDF5_static_C_FOUND AND NOT HDF5_shared_C_FOUND)
      if (USE_SHARED_LIBS AND HDF5_shared_C_FOUND)
        set (HDF5_LINK_LIBS ${HDF5_LINK_LIBS} ${HDF5_C_SHARED_LIBRARY})
      else (HDF5_static_C_FOUND)
        set (HDF5_LINK_LIBS ${HDF5_LINK_LIBS} ${HDF5_C_STATIC_LIBRARY})
      endif (USE_SHARED_LIBS AND HDF5_shared_C_FOUND)
      #Blosc source needs to be linked with HDF5
      set (LINK_LIBS ${LINK_LIBS} ${HDF5_C_SHARED_LIBRARY})
      set_property (TARGET h5repack PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5repack")
      set_property (TARGET h5dump PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5dump")
    endif (NOT HDF5_static_C_FOUND AND NOT HDF5_shared_C_FOUND)
    set (HDF5_DUMP_EXECUTABLE $<TARGET_FILE:h5dump>)
    set (HDF5_REPACK_EXECUTABLE $<TARGET_FILE:h5repack>)
  else (HDF5_FOUND)
    find_package (HDF5) # Legacy find
#Legacy find_package does not set HDF5_TOOLS_DIR, so we set it here
    set(HDF5_TOOLS_DIR ${HDF5_LIBRARY_DIRS}/../bin)
#Legacy find_package does not set HDF5_BUILD_SHARED_LIBS, so we set it here
    if (USE_SHARED_LIBS AND EXISTS "${HDF5_LIBRARY_DIRS}/libhdf5.so")
      set (HDF5_BUILD_SHARED_LIBS 1)
    else ()
      set (HDF5_BUILD_SHARED_LIBS 0)
    endif ()
    set (HDF5_LINK_LIBS ${HDF5_LINK_LIBS} ${HDF5_LIBRARIES})
    #Blosc source needs to be linked with HDF5
    set (LINK_LIBS ${LINK_LIBS} ${HDF5_LIBRARIES})

    add_executable (h5repack IMPORTED)
    add_executable (h5dump IMPORTED)
    set_property (TARGET h5dump PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5repack")
    set_property (TARGET h5dump PROPERTY IMPORTED_LOCATION "${HDF5_TOOLS_DIR}/h5dump")
    set (HDF5_DUMP_EXECUTABLE $<TARGET_FILE:h5repack>)
    set (HDF5_DUMP_EXECUTABLE $<TARGET_FILE:h5dump>)
  endif (HDF5_FOUND)
  set (HDF5_PACKAGE_NAME ${SEARCH_PACKAGE_NAME})

  if (HDF5_FOUND)
    set (HDF5_HAVE_H5PUBCONF_H 1)
    set (HDF5_HAVE_HDF5 1)
    set (HDF5_HDF5_HEADER "h5pubconf.h")
    set (HDF5_INCLUDE_DIR_GEN ${HDF5_INCLUDE_DIR})
    message (STATUS "HDF5 found: INC=${HDF5_INCLUDE_DIR} TOOLS=${HDF5_TOOLS_DIR}")
  else (HDF5_FOUND)
    message (FATAL_ERROR " HDF5 is Required for ${H5BLOSC_PACKAGE_NAME} library")
  endif (HDF5_FOUND)
else (NOT HDF5_HDF5_HEADER)
  # This project is being called from within another and HDF5 is already configured
  set (HDF5_HAVE_H5PUBCONF_H 1)
  set (HDF5_HAVE_HDF5 1)
endif (NOT HDF5_HDF5_HEADER)
message (STATUS "HDF5 link libs: ${HDF5_LINK_LIBS}")

if (USE_SHARED_LIBS)
  set (H5_LIB_TYPE SHARED)
else (USE_SHARED_LIBS)
  set (H5_LIB_TYPE STATIC)
endif (USE_SHARED_LIBS)

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
if(WIN32)
    # try to use the system library
    find_package(Threads)
    if(Threads_FOUND)
        set(LIBS ${LIBS} ${CMAKE_THREAD_LIBS_INIT})
    endif(Threads_FOUND)
else(WIN32)
    find_package(Threads REQUIRED)
    set(LINK_LIBS ${LINK_LIBS} ${CMAKE_THREAD_LIBS_INIT})
endif(WIN32)

option(PREFER_EXTERNAL_COMPLIBS
    "When found, use the installed compression libs instead of included sources." ON)
if(NOT PREFER_EXTERNAL_COMPLIBS)
    message(STATUS "Finding external libraries disabled.  Using internal sources.")
endif(NOT PREFER_EXTERNAL_COMPLIBS)

option(DEACTIVATE_LZ4
    "Do not include support for the LZ4 library." OFF)
if(NOT DEACTIVATE_LZ4)
    if(PREFER_EXTERNAL_COMPLIBS)
        find_package(LZ4)
    endif(PREFER_EXTERNAL_COMPLIBS)
    # HAVE_LZ4 will be set to true because even if the library is
    # not found, we will use the included sources for it
    set(HAVE_LZ4 TRUE)
endif(NOT DEACTIVATE_LZ4)

option(DEACTIVATE_SNAPPY
    "Do not include support for the SNAPPY library." OFF)
if(NOT DEACTIVATE_SNAPPY)
    if(PREFER_EXTERNAL_COMPLIBS)
        find_package(Snappy)
    endif(PREFER_EXTERNAL_COMPLIBS)
    # HAVE_SNAPPY will be set to true because even if the library is not found,
    # we will use the included sources for it
    set(HAVE_SNAPPY TRUE)
endif(NOT DEACTIVATE_SNAPPY)

option(DEACTIVATE_ZLIB
    "Do not include support for the ZLIB library." OFF)
if(NOT DEACTIVATE_ZLIB)
    # import the ZLIB_ROOT environment variable to help finding the zlib library
    if(PREFER_EXTERNAL_COMPLIBS)
        set(ZLIB_ROOT $ENV{ZLIB_ROOT})
        find_package( ZLIB )
        if (NOT ZLIB_FOUND )
            message(STATUS "No zlib found.  Using internal sources.")
        endif (NOT ZLIB_FOUND )
    endif(PREFER_EXTERNAL_COMPLIBS)
    # HAVE_ZLIB will be set to true because even if the library is not found,
    # we will use the included sources for it
    set(HAVE_ZLIB TRUE)
endif(NOT DEACTIVATE_ZLIB)

if(PREFER_EXTERNAL_COMPLIBS)
  find_package (BLOSC NAMES ${BLOSC_PACKAGE_NAME})
  if (NOT BLOSC_FOUND)
      message(STATUS "No blosc found.  Using internal sources.")
  endif (NOT BLOSC_FOUND)
endif(PREFER_EXTERNAL_COMPLIBS)

if (MSVC)
  set (CMAKE_MFC_FLAG 0)
endif (MSVC)

set (MAKE_SYSTEM)
if (CMAKE_BUILD_TOOL MATCHES "make")
  set (MAKE_SYSTEM 1)
endif (CMAKE_BUILD_TOOL MATCHES "make")

set (CFG_INIT "/${CMAKE_CFG_INTDIR}")
if (MAKE_SYSTEM)
  set (CFG_INIT "")
endif (MAKE_SYSTEM)

# Set the "-msse2" build flag only if the CMAKE_C_FLAGS is not already set.
# Probably "-msse2" should be appended to CMAKE_C_FLAGS_RELEASE.
find_package(SSE)
if(CMAKE_C_COMPILER_ID STREQUAL GNU OR CMAKE_C_COMPILER_ID STREQUAL Clang)
     if(NOT CMAKE_C_FLAGS AND SSE2_TRUE)
         message(STATUS "SSE2 is here.  Adding support for it.")
         set(CMAKE_C_FLAGS -msse2 CACHE STRING "C flags." FORCE)
     endif(NOT CMAKE_C_FLAGS AND SSE2_TRUE)
endif(CMAKE_C_COMPILER_ID STREQUAL GNU OR CMAKE_C_COMPILER_ID STREQUAL Clang)

if(MSVC)
    if(NOT CMAKE_C_FLAGS)
        set(CMAKE_C_FLAGS "/Ox" CACHE STRING "C flags." FORCE)
    endif(NOT CMAKE_C_FLAGS)
endif(MSVC)

#-----------------------------------------------------------------------------
# Compiler specific flags : Shouldn't there be compiler tests for these
#-----------------------------------------------------------------------------
if (CMAKE_COMPILER_IS_GNUCC)
  set (CMAKE_C_FLAGS "${CMAKE_ANSI_CFLAGS} ${CMAKE_C_FLAGS} -std=c99 -fomit-frame-pointer -finline-functions -fno-common")
endif (CMAKE_COMPILER_IS_GNUCC)
if (CMAKE_COMPILER_IS_GNUCXX)
  set (CMAKE_CXX_FLAGS "${CMAKE_ANSI_CFLAGS} ${CMAKE_CXX_FLAGS} -fomit-frame-pointer -finline-functions -fno-common")
endif (CMAKE_COMPILER_IS_GNUCXX)

#-----------------------------------------------------------------------------
# This is in here to help some of the GCC based IDES like Eclipse
# and code blocks parse the compiler errors and warnings better.
#-----------------------------------------------------------------------------
if (CMAKE_COMPILER_IS_GNUCC)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fmessage-length=0")
endif (CMAKE_COMPILER_IS_GNUCC)
if (CMAKE_COMPILER_IS_GNUCXX)
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fmessage-length=0")
endif (CMAKE_COMPILER_IS_GNUCXX)

#-----------------------------------------------------------------------------
# Generate the config.h file containing user settings needed by compilation
#-----------------------------------------------------------------------------
configure_file (${H5BLOSC_RESOURCES_DIR}/config.h.in ${H5BLOSC_BINARY_DIR}/config.h @ONLY)

#-----------------------------------------------------------------------------
# Dashboard and Testing Settings
#-----------------------------------------------------------------------------
option (BUILD_TESTING "Build h5blosc Unit Testing" OFF)
if (BUILD_TESTING)
  set (DART_TESTING_TIMEOUT 1200 CACHE INTEGER
       "Timeout in seconds for each test (default 1200=20minutes)")
  enable_testing ()
  include (CTest)
endif (BUILD_TESTING)

add_subdirectory (${H5BLOSC_SOURCE_DIR}/src ${PROJECT_BINARY_DIR}/src)

option (BUILD_EXAMPLES "Build h5blosc Examples" OFF)
if (BUILD_EXAMPLES)
  add_subdirectory (${H5BLOSC_SOURCE_DIR}/example ${PROJECT_BINARY_DIR}/example)
endif (BUILD_EXAMPLES)

#-----------------------------------------------------------------------------
# Add file(s) to CMake Install
#-----------------------------------------------------------------------------

include (CMakePackageConfigHelpers)

#-----------------------------------------------------------------------------
# Add Target(s) to CMake Install for import into other projects
#-----------------------------------------------------------------------------
if (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
  install (
      EXPORT ${H5BLOSC_EXPORTED_TARGETS}
      DESTINATION ${H5BLOSC_INSTALL_CMAKE_DIR}
      FILE ${H5BLOSC_PACKAGE}${H5BLOSC_PACKAGE_EXT}-targets.cmake
      COMPONENT configinstall
  )
endif (NOT H5BLOSC_EXTERNALLY_CONFIGURED)

#-----------------------------------------------------------------------------
# Export all exported targets to the build tree for use by parent project
#-----------------------------------------------------------------------------
if (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
  export (
      TARGETS ${H5BLOSC_LIBRARIES_TO_EXPORT} ${H5BLOSC_LIB_DEPENDENCIES}
      FILE ${H5BLOSC_PACKAGE}${H5BLOSC_PACKAGE_EXT}-targets.cmake
  )
endif (NOT H5BLOSC_EXTERNALLY_CONFIGURED)

#-----------------------------------------------------------------------------
# Set includes needed for build
#-----------------------------------------------------------------------------
set (H5BLOSC_INCLUDES_BUILD_TIME
    ${H5BLOSC_SRC_DIR} ${H5BLOSC_BINARY_DIR}
)
#-----------------------------------------------------------------------------
# Set variables needed for installation
#-----------------------------------------------------------------------------
set (H5BLOSC_VERSION_STRING ${H5BLOSC_PACKAGE_VERSION})
set (H5BLOSC_VERSION_MAJOR  ${H5BLOSC_PACKAGE_VERSION_MAJOR})
set (H5BLOSC_VERSION_MINOR  ${H5BLOSC_PACKAGE_VERSION_MINOR})

#-----------------------------------------------------------------------------
# Configure the h5bzip2-config.cmake file for the build directory
#-----------------------------------------------------------------------------
set(INCLUDE_INSTALL_DIR ${H5BLOSC_INSTALL_INCLUDE_DIR} )
set(SHARE_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/${H5BLOSC_INSTALL_CMAKE_DIR}" )
set(CURRENT_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}" )
configure_package_config_file (
    ${H5BLOSC_RESOURCES_DIR}/h5blosc-config.cmake.in
    "${H5BLOSC_BINARY_DIR}/${H5BLOSC_PACKAGE}${H5BLOSC_PACKAGE_EXT}-config.cmake"
    INSTALL_DESTINATION "${H5BLOSC_INSTALL_CMAKE_DIR}"
    PATH_VARS INCLUDE_INSTALL_DIR SHARE_INSTALL_DIR CURRENT_BUILD_DIR
    INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}"
)

#-----------------------------------------------------------------------------
# Configure the blosc-config.cmake file for the install directory
#-----------------------------------------------------------------------------
set(INCLUDE_INSTALL_DIR ${H5BLOSC_INSTALL_INCLUDE_DIR} )
set(SHARE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/${H5BLOSC_INSTALL_CMAKE_DIR}" )
set(CURRENT_BUILD_DIR "${CMAKE_INSTALL_PREFIX}" )
configure_package_config_file (
    ${H5BLOSC_RESOURCES_DIR}/h5blosc-config.cmake.in
    "${H5BLOSC_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${H5BLOSC_PACKAGE}${H5BLOSC_PACKAGE_EXT}-config.cmake"
    INSTALL_DESTINATION "${H5BLOSC_INSTALL_CMAKE_DIR}"
    PATH_VARS INCLUDE_INSTALL_DIR SHARE_INSTALL_DIR CURRENT_BUILD_DIR
)
if (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
  install (
      FILES ${H5BLOSC_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${H5BLOSC_PACKAGE}${H5BLOSC_PACKAGE_EXT}-config.cmake
      DESTINATION ${H5BLOSC_INSTALL_CMAKE_DIR}
      COMPONENT configinstall
  )
endif (NOT H5BLOSC_EXTERNALLY_CONFIGURED)

#-----------------------------------------------------------------------------
# Configure the blosc-config-version.cmake file for the install directory
#-----------------------------------------------------------------------------
if (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
  configure_file (
      ${H5BLOSC_RESOURCES_DIR}/h5blosc-config-version.cmake.in
      ${H5BLOSC_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${H5BLOSC_PACKAGE}${H5BLOSC_PACKAGE_EXT}-config-version.cmake @ONLY
  )
  install (
      FILES ${H5BLOSC_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${H5BLOSC_PACKAGE}${H5BLOSC_PACKAGE_EXT}-config-version.cmake
      DESTINATION ${H5BLOSC_INSTALL_CMAKE_DIR}
      COMPONENT configinstall
  )
endif (NOT H5BLOSC_EXTERNALLY_CONFIGURED)

#-----------------------------------------------------------------------------
# Configure the README.txt file for the binary package
#-----------------------------------------------------------------------------
H5BLOSC_README_PROPERTIES()

#-----------------------------------------------------------------------------
# Check for Installation Utilities
#-----------------------------------------------------------------------------
if (WIN32)
  set (PF_ENV_EXT "(x86)")
  find_program (NSIS_EXECUTABLE NSIS.exe PATHS "$ENV{ProgramFiles}\\NSIS" "$ENV{ProgramFiles${PF_ENV_EXT}}\\NSIS")
  if(NOT CPACK_WIX_ROOT)
    file(TO_CMAKE_PATH "$ENV{WIX}" CPACK_WIX_ROOT)
  endif()
  find_program (WIX_EXECUTABLE candle  PATHS "${CPACK_WIX_ROOT}/bin")
endif (WIN32)

#-----------------------------------------------------------------------------
# Set the cpack variables
#-----------------------------------------------------------------------------
if (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
  set (CPACK_PACKAGE_VENDOR "HDF_Group")
  set (CPACK_PACKAGE_NAME "${H5BLOSC_PACKAGE_NAME}")
  if (CDASH_LOCAL)
    set (CPACK_PACKAGE_VERSION "${H5BLOSC_PACKAGE_VERSION}")
  else (CDASH_LOCAL)
    set (CPACK_PACKAGE_VERSION "${H5BLOSC_PACKAGE_VERSION_STRING}")
  endif (CDASH_LOCAL)
  set (CPACK_PACKAGE_VERSION_MAJOR "${H5BLOSC_PACKAGE_VERSION_MAJOR}")
  set (CPACK_PACKAGE_VERSION_MINOR "${H5BLOSC_PACKAGE_VERSION_MINOR}")
  set (CPACK_PACKAGE_VERSION_PATCH "")
  #set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
  set (CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.txt")
  set (CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.txt")
  set (CPACK_PACKAGE_RELOCATABLE TRUE)
  set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "h5blosc Installation")
  set (CPACK_PACKAGE_INSTALL_DIRECTORY "${CPACK_PACKAGE_VENDOR}/${CPACK_PACKAGE_NAME}/${CPACK_PACKAGE_VERSION}")

  set (CPACK_GENERATOR "TGZ")
  if (WIN32)
    set (CPACK_GENERATOR "ZIP")

    if (NSIS_EXECUTABLE)
      list (APPEND CPACK_GENERATOR "NSIS")
    endif (NSIS_EXECUTABLE)
    # Installers for 32- vs. 64-bit CMake:
    #  - Root install directory (displayed to end user at installer-run time)
    #  - "NSIS package/display name" (text used in the installer GUI)
    #  - Registry key used to store info about the installation
    set (CPACK_NSIS_PACKAGE_NAME "${H5BLOSC_PACKAGE_STRING}")
    if (CMAKE_CL_64)
      set (CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES64")
      set (CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION} (Win64)")
    else (CMAKE_CL_64)
      set (CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES")
      set (CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}")
    endif (CMAKE_CL_64)
    # set the install/unistall icon used for the installer itself
    # There is a bug in NSI that does not handle full unix paths properly.
    #set (CPACK_NSIS_MUI_ICON "${H5BLOSC_RESOURCES_DIR}\\\\H5BLOSC.ico")
    #set (CPACK_NSIS_MUI_UNIICON "${H5BLOSC_RESOURCES_DIR}\\\\H5BLOSC.ico")
    # set the package header icon for MUI
    #set (CPACK_PACKAGE_ICON "${H5BLOSC_RESOURCES_DIR}\\\\H5BLOSC.bmp")
    set (CPACK_NSIS_DISPLAY_NAME "${CPACK_NSIS_PACKAGE_NAME}, is a dynamic filter for HDF5")
    #set (CPACK_PACKAGE_INSTALL_DIRECTORY "plugins")
    set (CPACK_NSIS_CONTACT "${H5BLOSC_PACKAGE_BUGREPORT}")
    set (CPACK_NSIS_MODIFY_PATH ON)

    if (WIX_EXECUTABLE)
      list (APPEND CPACK_GENERATOR "WIX")
    endif (WIX_EXECUTABLE)
#WiX variables
    set (CPACK_WIX_UNINSTALL "1")
# .. variable:: CPACK_WIX_LICENSE_RTF
#  RTF License File
#
#  If CPACK_RESOURCE_FILE_LICENSE has an .rtf extension it is used as-is.
#
#  If CPACK_RESOURCE_FILE_LICENSE has an .txt extension it is implicitly
#  converted to RTF by the WiX Generator.
#  The expected encoding of the .txt file is UTF-8.
#
#  With CPACK_WIX_LICENSE_RTF you can override the license file used by the
#  WiX Generator in case CPACK_RESOURCE_FILE_LICENSE is in an unsupported
#  format or the .txt -> .rtf conversion does not work as expected.
    set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/README.txt")
# .. variable:: CPACK_WIX_PRODUCT_ICON
#  The Icon shown next to the program name in Add/Remove programs.
#    set(CPACK_WIX_PRODUCT_ICON "${H5BLOSC_RESOURCES_DIR}\\\\H5BLOSC.ico")
#
# .. variable:: CPACK_WIX_UI_BANNER
#
#  The bitmap will appear at the top of all installer pages other than the
#  welcome and completion dialogs.
#
#  If set, this image will replace the default banner image.
#
#  This image must be 493 by 58 pixels.
#
# .. variable:: CPACK_WIX_UI_DIALOG
#
#  Background bitmap used on the welcome and completion dialogs.
#
#  If this variable is set, the installer will replace the default dialog
#  image.
#
#  This image must be 493 by 312 pixels.
#
  elseif (APPLE)
    list (APPEND CPACK_GENERATOR "DragNDrop")
    set (CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)
    set (CPACK_PACKAGING_INSTALL_PREFIX "/${CPACK_PACKAGE_INSTALL_DIRECTORY}")
#    set (CPACK_PACKAGE_ICON "${H5BLOSC_RESOURCES_DIR}/h5blosc.gif")

    option (H5BLOSC_PACK_MACOSX_BUNDLE  "Package the H5BLOSC Library in a Bundle" OFF)
    if (H5BLOSC_PACK_MACOSX_BUNDLE)
      list (APPEND CPACK_GENERATOR "Bundle")
      set (CPACK_BUNDLE_NAME "${H5BLOSC_PACKAGE_STRING}")
      set (CPACK_BUNDLE_LOCATION "/")    # make sure CMAKE_INSTALL_PREFIX ends in /
      set (CMAKE_INSTALL_PREFIX "/${CPACK_BUNDLE_NAME}.framework/Versions/${CPACK_PACKAGE_VERSION}/${CPACK_PACKAGE_NAME}/")
#      set (CPACK_BUNDLE_ICON "${H5BLOSC_RESOURCES_DIR}/H5BZ2.icns")
#      set (CPACK_BUNDLE_PLIST "${H5BLOSC_BINARY_DIR}/CMakeFiles/Info.plist")
      set (CPACK_APPLE_GUI_INFO_STRING "H5BLOSC Plugin Library")
      set (CPACK_APPLE_GUI_COPYRIGHT "Copyright © ???. All rights reserved.")
      set (CPACK_SHORT_VERSION_STRING "${CPACK_PACKAGE_VERSION}")
      set (CPACK_APPLE_GUI_BUNDLE_NAME "${H5BLOSC_PACKAGE_STRING}")
      set (CPACK_APPLE_GUI_VERSION_STRING "${CPACK_PACKAGE_VERSION_STRING}")
      set (CPACK_APPLE_GUI_SHORT_VERSION_STRING "${CPACK_PACKAGE_VERSION}")
      #-----------------------------------------------------------------------------
      # Configure the Info.plist file for the install bundle
      #-----------------------------------------------------------------------------
#      configure_file (
#          ${H5BLOSC_RESOURCES_DIR}/CPack.Info.plist.in
#          ${H5BLOSC_BINARY_DIR}/CMakeFiles/Info.plist @ONLY
#      )
#      configure_file (
#          ${H5BLOSC_RESOURCES_DIR}/PkgInfo.in
#          ${H5BLOSC_BINARY_DIR}/CMakeFiles/PkgInfo @ONLY
#      )
#      configure_file (
#          ${H5BLOSC_RESOURCES_DIR}/version.plist.in
#          ${H5BLOSC_BINARY_DIR}/CMakeFiles/version.plist @ONLY
#      )
#      install (
#          FILES ${H5BLOSC_BINARY_DIR}/CMakeFiles/PkgInfo
#                ${H5BLOSC_BINARY_DIR}/CMakeFiles/version.plist
#          DESTINATION ..
#      )
    endif (H5BLOSC_PACK_MACOSX_BUNDLE)
  else (WIN32)
    list (APPEND CPACK_GENERATOR "STGZ")
    set (CPACK_PACKAGING_INSTALL_PREFIX "/${CPACK_PACKAGE_INSTALL_DIRECTORY}")
    set (CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)

    set (CPACK_DEBIAN_PACKAGE_SECTION "Libraries")
    set (CPACK_DEBIAN_PACKAGE_MAINTAINER "${H5BLOSC_PACKAGE_BUGREPORT}")

#      list (APPEND CPACK_GENERATOR "RPM")
    set (CPACK_RPM_PACKAGE_RELEASE "1")
    set (CPACK_RPM_COMPONENT_INSTALL ON)
    set (CPACK_RPM_PACKAGE_RELOCATABLE ON)
  endif (WIN32)

  set (CPACK_INSTALL_CMAKE_PROJECTS "${H5BLOSC_BINARY_DIR};h5blosc;libraries;/")

  set (CPACK_ALL_INSTALL_TYPES Full User)
  set (CPACK_INSTALL_TYPE_FULL_DISPLAY_NAME "Everything")

  set(CPACK_COMPONENTS_ALL libraries)

  include (CPack)

  cpack_add_component_group(Runtime)

  cpack_add_component (libraries
      DISPLAY_NAME "h5blosc Libraries"
      GROUP Runtime
  )

endif (NOT H5BLOSC_EXTERNALLY_CONFIGURED)
