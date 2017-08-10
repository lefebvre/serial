################################################################################
#                                                                              #
#             serial  TriBITS Project and Package CMakeLists.txt File           #
#                                                                              #
################################################################################
  #
  # Default MPI to OFF
  #
  SET(TPL_ENABLE_MPI OFF CACHE BOOL "" FORCE)
  #
  # Default Fortran OFF
  #
  SET(serial_ENABLE_Fortran OFF CACHE BOOL "" FORCE)

IF (NOT TRIBITS_PROCESSING_PACKAGE)
  # This CMakeLists.txt file is being processed as the serial TriBITS projects's base
  # CMakeLists.txt file!  (See comments at bottom of this file.)
  CMAKE_MINIMUM_REQUIRED(VERSION 2.8.12 FATAL_ERROR)
  INCLUDE("${CMAKE_CURRENT_SOURCE_DIR}/ProjectName.cmake")
  PROJECT(${PROJECT_NAME} NONE)
  # enable C++11 by default
  SET(serial_ENABLE_CXX11 ON CACHE BOOL "Compile using the C++11 standard" FORCE)

  # disable generating HTML dependencies webpage and xml files
  SET(${PROJECT_NAME}_DEPS_XML_OUTPUT_FILE OFF CACHE BOOL "")
  # disable TriBITS export system to save time configuring
  SET(${PROJECT_NAME}_ENABLE_INSTALL_CMAKE_CONFIG_FILES OFF CACHE BOOL "")
  SET(${PROJECT_NAME}_ENABLE_EXPORT_MAKEFILES OFF CACHE BOOL "")
  SET(${PROJECT_NAME}_TRIBITS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/TriBITS/tribits" CACHE PATH "")
  INCLUDE("${${PROJECT_NAME}_TRIBITS_DIR}/TriBITS.cmake")

  TRIBITS_PROJECT_ENABLE_ALL()
ELSE()
  # This CMakeLists.txt file is being processed as the TriBITS package file.
  INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/include)
  TRIBITS_PACKAGE(serial)
  IF(APPLE)
    FIND_LIBRARY(IOKIT_LIBRARY IOKit)
    FIND_LIBRARY(FOUNDATION_LIBRARY Foundation)
  ENDIF()
  SET(serial_SOURCE
    src/serial.cc
  )
  SET(serial_HEADERS
    include/serial/serial.h
    include/serial/v8stdint.h
  )
  IF(APPLE)
     LIST(APPEND serial_SOURCE
         src/impl/unix.cc
         src/impl/list_ports/list_ports_osx.cc
     )
  ELSEIF(UNIX)
     LIST(APPEND serial_SOURCE
         src/impl/unix.cc
         src/impl/list_ports/list_ports_linux.cc
     )
  ELSE()
     LIST(APPEND serial_SOURCE
         src/impl/win.cc
         src/impl/list_ports/list_ports_win.cc
     )
  ENDIF()

  TRIBITS_ADD_LIBRARY(serial
    SOURCE ${serial_SOURCE}
    NOINSTALLHEADERS ${serial_SOURCE}
  )
  IF(APPLE)
    TARGET_LINK_LIBRARIES(serial ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
  ENDIF()
  #
  # add example
  #
  TRIBITS_ADD_EXECUTABLE(serial_example
      SOURCES examples/serial_example.cc
      NOEXEPREFIX
  )
  TRIBITS_PACKAGE_POSTPROCESS()
ENDIF()

# NOTE: In order to allow the `serial/` directory to be both a TriBITS
# package (for inclusion in other TriBITS projects) and to be a TriBITS
# project itself, you only have to put in a simple if statement in this
# top-level CMakeLists.txt file.  That is all!
