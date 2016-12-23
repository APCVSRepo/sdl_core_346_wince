# Copyright (c) 2014, Ford Motor Company
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following
# disclaimer in the documentation and/or other materials provided with the
# distribution.
#
# Neither the name of the Ford Motor Company nor the names of its contributors
# may be used to endorse or promote products derived from this software
# without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required(VERSION 2.8.0)

set (PROJECT smartDeviceLinkCore)

#Jenkins integration section
#dont modify this section if you don't know details about integration with Jenkins!!!
set (HMI "web" CACHE STRING "HMI type")
option(HMI2 "Use Qt HMI" OFF)
option(EXTENDED_MEDIA_MODE "Turn on and off extended Madia Manager features relates to PulseAudio A2DP and GStreamer" OFF)
option(BUILD_SHARED_LIBS "Build all libraries as shared (if ON) or static (if OFF)" OFF)
option(BUILD_BT_SUPPORT "Bluetooth support" ON)
option(BUILD_USB_SUPPORT "libusb support" ON)
option(BUILD_BACKTRACE_SUPPORT "backtrace support" ON)
option(BUILD_TESTS "Possibility to build and run tests" OFF)
option(TELEMETRY_MONITOR "Enable profiling time test util" OFF)
option(ENABLE_LOG "Logging feature" ON)
option(ENABLE_GCOV "gcov code coverage feature" OFF)
option(ENABLE_SANITIZE "Sanitize tool" OFF)
option(ENABLE_SECURITY "Security Ford protocol protection" ON)
option(ENABLE_HMI_PTU_DECRYPTION "Policy table update parsed by hmi" ON)

option(BUILD_TARGET_LIBRARY    "build target as library" ON)

set(OS_TYPE_OPTION 	"$ENV{OS_TYPE}")
set(DEBUG_OPTION 	"$ENV{DEBUG}")
set(HMI_TYPE_OPTION 	"$ENV{HMI_TYPE}")
set(TARGET_OPTION 	"$ENV{TARGET}")
set(MEDIA_MODE_OPTION 	"$ENV{MEDIA_MODE}")
set(HMI_ADAPTER_OPTION 	"$ENV{HMI_ADAPTER}")
set(ENABLE_TESTS_OPTION "$ENV{ENABLE_TESTS}")
set(ENABLE_LOG_OPTION 	"$ENV{ENABLE_LOG}")
set(ARCH_TYPE_OPTION	"$ENV{ARCH_TYPE}")
set(POLICY_OPTION 	"$ENV{POLICY_TYPE}")
set(SECURITY_OPTION 	"$ENV{SECURITY_MODE}")
set(COMPONENTS_DIR ${CMAKE_SOURCE_DIR}/src/components)
set(SNAPSHOT_TAG 	"$ENV{SNAPSHOT_TAG}")

set(CMAKE_CXX_COMPILER   $ENV{CMAKE_CXX_COMPILER})
set(CMAKE_C_COMPILER     $ENV{CMAKE_C_COMPILER})

if (ARCH_TYPE_OPTION)
  if (NOT (${ARCH_TYPE_OPTION} STREQUAL "x86") AND NOT (${ARCH_TYPE_OPTION} STREQUAL "armv7"))
    message(AUTHOR_WARNING "HW architecture is not defined, using x86. Allowed values are x86/armv7 (case sensitive)")
    set(ARCH_TYPE_OPTION	"x86")
  endif ()
else ()
    set(ARCH_TYPE_OPTION	"x86")
endif()

if (HMI_TYPE_OPTION)
  if (${HMI_TYPE_OPTION} STREQUAL "HTML5")
    message(STATUS "Jenkins integration: select HTML5 HMI")
    set (HMI "web")
  elseif (${HMI_TYPE_OPTION} STREQUAL "NONE")
    message(STATUS "Jenkins integration: select HMI none")
    set (HMI "no")
  else ()
    message(STATUS "Jenkins integration: select QML HMI none")
    set (HMI "qt")
  endif()
endif()

if (MEDIA_MODE_OPTION)
  if (${MEDIA_MODE_OPTION} STREQUAL "EXTENDED_MEDIA")
    message(STATUS "Jenkins integration: select extended media mode")
    set (EXTENDED_MEDIA_MODE ON)
  endif()
endif()

if (DEBUG_OPTION)
  if (${DEBUG_OPTION} STREQUAL "DBG_OFF")
    message(STATUS "Jenkins integration: build release version")
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
  endif()
endif()

if (HMI_ADAPTER_OPTION)
  if (${HMI_ADAPTER_OPTION} STREQUAL "MESSAGEBROKER")
    message(STATUS "Jenkins integration: selected HMI adapter MESSAGEBROKER")
    set (HMIADAPTER "messagebroker")
  elseif (${HMI_ADAPTER_OPTION} STREQUAL "DBUS")
    message(STATUS "Jenkins integration: selected HMI adapter DBUS")
    set (HMIADAPTER "dbus")
  elseif (${HMI_ADAPTER_OPTION} STREQUAL "MQUEUE")
    message(STATUS "Jenkins integration: selected HMI adapter MQUEUE")
    set (HMIADAPTER "mqueue")
  endif()
endif()

if (ENABLE_LOG_OPTION)
  if (${ENABLE_LOG_OPTION} STREQUAL "LOG_OFF")
    message(STATUS "Jenkins integration: Log is turned off")
    set (ENABLE_LOG OFF)
  endif()
endif()

if (ENABLE_TESTS_OPTION)
  if (${ENABLE_TESTS_OPTION} STREQUAL "TESTS_OFF")
    message(STATUS "Jenkins integration: Unit tests is turned off")
    set (BUILD_TESTS OFF)
  elseif(${ENABLE_TESTS_OPTION} STREQUAL "TESTS_ON")
    message(STATUS "Jenkins integration: Unit tests is turned on")
    set (BUILD_TESTS ON)
  endif()
endif()

if (SECURITY_OPTION)
  if (${SECURITY_OPTION} STREQUAL "SEC_OFF")
    message(STATUS "Jenkins integration: Security is turned OFF")
    set (ENABLE_SECURITY OFF)
  endif()
endif()

#Jenkins integration section end

add_custom_target(pasa-tarball
  COMMAND ${CMAKE_SOURCE_DIR}/tools/Utils/export-customer-specific.sh ${CMAKE_SOURCE_DIR} ${CMAKE_BINARY_DIR} pasa
  COMMAND tar -cz -C /tmp/PASA -f ${CMAKE_BINARY_DIR}/pasa.tar.gz .
  DEPENDS HMI_API MOBILE_API v4_protocol_v1_2_no_extra
)

add_custom_target(ford-tarball
  COMMAND ${CMAKE_SOURCE_DIR}/tools/Utils/export-customer-specific.sh ${CMAKE_SOURCE_DIR} ${CMAKE_BINARY_DIR} ford
  COMMAND tar -cz -C /tmp/FORD -f ${CMAKE_BINARY_DIR}/ford.tar.gz .
  DEPENDS HMI_API MOBILE_API v4_protocol_v1_2_no_extra
)

add_custom_target(genivi-tarball
  COMMAND ${CMAKE_SOURCE_DIR}/tools/Utils/export-customer-specific.sh ${CMAKE_SOURCE_DIR} ${CMAKE_BINARY_DIR} genivi
  COMMAND tar -cz -C /tmp/GENIVI -f ${CMAKE_BINARY_DIR}/genivi.tar.gz .
)


project (${PROJECT})

# Only WinCE platform be supported.
set(CMAKE_SYSTEM_NAME "WindowsCE")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

# Please do not change compiler/linker flags if You do not know how particular
# flag is handled by CMake
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR})
set(ARCHIVE_OUTPUT_DIRECTORY ./bin)

set (CMAKE_CXX_FLAGS "/wd\"4819\" /wd\"4244\" /wd\"4390\" /wd\"4996\"  /EHsc /bigobj /Z7 /GF")
set (CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG") #It will be appended to CMAKE_CXX_FLAGS in release
set (CMAKE_C_FLAGS ${CMAKE_CXX_FLAGS})  
set (CMAKE_EXE_LINKER_FLAGS "/SUBSYSTEM:CONSOLE /ENTRY:")

add_definitions(-D_WIN32_WCE=$(CEVER))
add_definitions(-D_CONSOLE)
add_definitions(-DUNDER_CE)
add_definitions(-D$(ARCHFAM))
add_definitions(-D$(_ARCHFAM_))
add_definitions(-DOS_WINCE)
add_definitions(-DWIN32_LEAN_AND_MEAN)
add_definitions(-DHAVE_PTW32_CONFIG_H)
add_definitions(-D_UNICODE)
add_definitions(-DUNICODE)
add_definitions(-DWINCE)
add_definitions(-D_WIN32)
add_definitions(-DLOG4CXX_STATIC)
add_definitions(-DAPR_DECLARE_STATIC)
set (BUILD_BT_SUPPORT OFF)
set (BUILD_USB_SUPPORT ON)
set (BUILD_AVAHI_SUPPORT OFF)
set (BUILD_BACKTRACE_SUPPORT OFF)
set (EXTENDED_MEDIA_MODE OFF)

if(BUILD_TARGET_LIBRARY)
add_definitions(-DBUILD_TARGET_LIB)
endif()

if (BUILD_USB_SUPPORT)
  add_definitions(-DUSB_SUPPORT)
  message(STATUS "USB support is enabled")
endif()


if (BUILD_BT_SUPPORT)
  add_definitions(-DBLUETOOTH_SUPPORT)
  message(STATUS "Bluetooth support is enabled")
endif()

if (BUILD_BACKTRACE_SUPPORT)
  add_definitions(-DBACKTRACE_SUPPORT)
endif()

if(ENABLE_LOG)
  add_definitions(-DENABLE_LOG)
  set(install-3rd_party_logger "install-3rd_party_logger")
endif()

if (TELEMETRY_MONITOR)
  add_definitions(-DTELEMETRY_MONITOR)
endif()

#option(CMAKE_BUILD_TYPE "build type" "Debug")
#set(CMAKE_BUILD_TYPE "Release")
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
set ( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DDEBUG" )
add_definitions(-DDEBUG)
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
set ( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DRELEASE" )
add_definitions(-DRELEASE)
else()
set (CMAKE_BUILD_TYPE "Debug")
set ( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DDEBUG" )
add_definitions(-DDEBUG)
endif()

# --- Interface generator

find_package(PythonInterp)
if(NOT PYTHONINTERP_FOUND)
  message(STATUS "Python interpreter is not found")
  message(STATUS "To install it type in the command line:")
  message(STATUS "sudo apt-get install python")
  message(FATAL_ERROR "Exiting!")
endif(NOT PYTHONINTERP_FOUND)

set(INTEFRACE_GENERATOR "${PROJECT_SOURCE_DIR}/tools/InterfaceGenerator/Generator.py")
set(INTEFRACE_GENERATOR_CMD ${PYTHON_EXECUTABLE} -B ${INTEFRACE_GENERATOR})
file(GLOB_RECURSE INTERFACE_GENERATOR_DEPENDENCIES "${PROJECT_SOURCE_DIR}/tools/InterfaceGenerator/*.*")

macro (GenerateInterface arg_xml_name arg_namespace parser_type)
    string(REGEX MATCH "^[a-zA-Z_0-9]*[^.]" file_name ${arg_xml_name})     # TODO: make expression more robust

    set(hpp_file
      "${CMAKE_CURRENT_BINARY_DIR}/${file_name}.h"
      "${CMAKE_CURRENT_BINARY_DIR}/${file_name}_schema.h"
    )

    set(cpp_file "${CMAKE_CURRENT_BINARY_DIR}/${file_name}_schema.cc")
    set(full_xml_name "${CMAKE_CURRENT_SOURCE_DIR}/${arg_xml_name}")

    add_custom_command( OUTPUT ${hpp_file} ${cpp_file}
                        COMMAND ${INTEFRACE_GENERATOR_CMD} ${full_xml_name} ${arg_namespace} ${CMAKE_CURRENT_BINARY_DIR} "--parser-type" "${parser_type}"
                        DEPENDS ${INTERFACE_GENERATOR_DEPENDENCIES} ${full_xml_name}
                        COMMENT "Generating files:\n   ${hpp_file}\n   ${cpp_file}\nfrom:\n   ${arg_xml_name} ..."
                        VERBATIM
                       )

    include_directories (
        ${COMPONENTS_DIR}/smart_objects/include
        ${COMPONENTS_DIR}/formatters/include/
        ${CMAKE_BINARY_DIR}
    )

    add_library("${file_name}" ${cpp_file})
endmacro(GenerateInterface)

# --- Useful macro
macro(create_test NAME SOURCES LIBS)
add_executable("${NAME}" ${CMAKE_SOURCE_DIR}/src/components/test_main.cc ${SOURCES})
target_link_libraries("${NAME}" ${LIBS})
target_link_libraries("${NAME}" Utils)
add_test(${NAME} ${NAME})
endmacro(create_test)


# Include direction and link library
include_directories (
  ${CMAKE_SOURCE_DIR}/include/all
  ${CMAKE_SOURCE_DIR}/include/wince
  ${CMAKE_SOURCE_DIR}/src/3rd_party/apache-log4cxx-win32-0.10.0/apache-log4cxx-0.10.0/src/main/include
)

link_directories(
  ${CMAKE_SOURCE_DIR}/lib/wince
)

# --replace in list macro
macro(LIST_REPLACE LIST INDEX NEWVALUE)
    list(INSERT ${LIST} ${INDEX} ${NEWVALUE})
    MATH(EXPR __INDEX "${INDEX} + 1")
    list (REMOVE_AT ${LIST} ${__INDEX})
endmacro(LIST_REPLACE)


# Building application

# --- Type HMI
set (WEB_HMI ON)
add_definitions(-DWEB_HMI)

if (HMI STREQUAL "qt" AND NOT DEFINED HMIADAPTER)
    set(HMIADAPTER "dbus")
endif()
if (HMI STREQUAL "web" AND NOT DEFINED HMIADAPTER)
    set(HMIADAPTER "messagebroker")
endif()
if (HMI STREQUAL "no" AND NOT DEFINED HMIADAPTER)
    set(HMIADAPTER "mqueue")
endif()

if (HMIADAPTER STREQUAL "dbus")
    set(HMI_DBUS_API ON)
    add_definitions(-DDBUS_HMIADAPTER)
    add_definitions(-DHMI_DBUS_API)
    set(install-3rd_party_dbus "install-3rd_party_dbus")
endif()
if (HMIADAPTER STREQUAL "messagebroker")
    set(HMI_JSON_API ON)
    add_definitions(-DMESSAGEBROKER_HMIADAPTER)
    add_definitions(-DHMI_JSON_API)
endif()
if (HMIADAPTER STREQUAL "mqueue")
    set(HMI_JSON_API ON)
    add_definitions(-DMQUEUE_HMIADAPTER)
    add_definitions(-DHMI_JSON_API)
endif()

# --- Directory with SDL interfaces, global types and ProtocolLib component
include_directories(
  ${COMPONENTS_DIR}/include
  ${COMPONENTS_DIR}/protocol/include
)

if (BUILD_TESTS)
include_directories(
  ${COMPONENTS_DIR}/include/test
)
endif ()

# --- 3rd party libs
INCLUDE(${CMAKE_CURRENT_SOURCE_DIR}/src/3rd_party/set_3rd_party_paths.cmake)

set(3RD_PARTY_SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/3rd_party)
set(3RD_PARTY_BINARY_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/src/3rd_party)

set (install-3rd_party_logger_var "")
set (install-3rd_party_dbus_var "")

if(NO_REBUILD_3RD_PARTY)
  set(NO_REBUILD_3RD_PARTY_LOGGER ON)
  set(NO_REBUILD_3RD_PARTY_DBUS ON)
endif()

if(FORCE_3RD_PARTY)
  if(NO_REBUILD_3RD_PARTY)
  	message(FATAL_ERROR "Please don't turn on both FORCE_3RD_PARTY and NO_REBUILD_3RD_PARTY at the same time.")
  else()
	set(FORCE_3RD_PARTY_LOGGER ON)
	set(FORCE_3RD_PARTY_DBUS ON)
  endif()
endif()

add_subdirectory(${3RD_PARTY_SOURCE_DIRECTORY} ${3RD_PARTY_BINARY_DIRECTORY} EXCLUDE_FROM_ALL)
add_custom_target(install-3rd_party
  DEPENDS ${install-3rd_party_logger_var}
  DEPENDS ${install-3rd_party_dbus_var}
  WORKING_DIRECTORY ${3RD_PARTY_BINARY_DIRECTORY}
)

if(ENABLE_LOG)
  include_directories ( ${LOG4CXX_INCLUDE_DIRECTORY} )
endif()

if(ENABLE_SECURITY)
  add_definitions(-DENABLE_SECURITY)
  set(SecurityManagerLibrary SecurityManager)
  set(SecurityManagerIncludeDir ${COMPONENTS_DIR}/security_manager/include)
  set(SecurityManagerTestIncludeDir ${CMAKE_SOURCE_DIR}/test/components/security_manager/include)
endif()

if(ENABLE_HMI_PTU_DECRYPTION)
  message("USE DHMI_PTU_PARSER")
  add_definitions(-DUSE_HMI_PTU_DECRYPTION)
endif()

#set (RTLIB rt)
set (RTLIT )

# Building tests
if(BUILD_TESTS)
  enable_testing()
  add_definitions(-DBUILD_TESTS)
  # Framework GoogleTest is also integrated together gmock
  # and must not be added separately
  add_subdirectory(./src/3rd_party-static/gmock-1.7.0)
endif()

# --- 3rd party libs (static)
add_subdirectory(./src/3rd_party-static)

# --- Tools
#add_subdirectory(./tools)


# --- Components
add_subdirectory(./src/components)

# --- Main application
add_subdirectory(./src/appMain)

# --- Plugins
add_subdirectory(./src/plugins)


# Building tests
if(BUILD_TESTS)
    # Directory test is deprecated. Use src/components/<name>/test
    include(Dart)
    #add_subdirectory(./test)
endif()
                                                                                                                                                                                         
# Building documentation                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    

# At first creating directory for generated documentation. Unfortunately doxygen
# cannot generate it byself
find_package(Doxygen)
  if(DOXYGEN_FOUND)
    option(DOXYGEN_ENABLE_DIAGRAMS "Enable graphical diagram generation" ON)

    if(DOXYGEN_ENABLE_DIAGRAMS)
      set(DOXYGEN_ENABLE_DIAGRAMS_PARAM "YES")
    else(DOXYGEN_ENABLE_DIAGRAMS)
      set(DOXYGEN_ENABLE_DIAGRAMS_PARAM "NO")
    endif()
    configure_file("${PROJECT_SOURCE_DIR}/Doxyfile" "${PROJECT_BINARY_DIR}/Doxyfile")
    file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/doc/doxygen")
    add_custom_target(doxygen COMMAND ${DOXYGEN_EXECUTABLE} "${PROJECT_BINARY_DIR}/Doxyfile")
  else(DOXYGEN_FOUND)
    message(STATUS "Doxygen not found. Documentation will not be generated")
    message(STATUS "To enable documentation generation please install doxygen and graphviz packages")
    message(STATUS "sudo apt-get install doxygen graphviz")
    message(STATUS "To enable processing of MscGen comments please install mscgen")
    message(STATUS "sudo apt-get install mscgen")
endif(DOXYGEN_FOUND)

