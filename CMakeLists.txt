project(mrmcplatform)

cmake_minimum_required(VERSION 2.6)
enable_language(CXX)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR})

find_package(mrmc REQUIRED)
find_package(TinyXML REQUIRED)
find_package(Threads REQUIRED)
find_package(platform REQUIRED)
include(UseMultiArch.cmake)

set(mrmcplatform_NAME mrmcplatform)
set(mrmcplatform_DESCRIPTION "mrmc add-on platform support library")
set(mrmcplatform_VERSION_MAJOR ${APP_VERSION_MAJOR})
set(mrmcplatform_VERSION_MINOR ${APP_VERSION_MINOR})
set(mrmcplatform_VERSION_PATCH 0)

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

set(mrmcplatform_INCLUDE_DIRS ${TINYXML_INCLUDE_DIR} "${CMAKE_INSTALL_PREFIX}/include/mrmc")
IF(WIN32)
  LIST(APPEND mrmcplatform_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include/mrmc/windows")
ENDIF(WIN32)
set(mrmcplatform_LIBRARIES ${CMAKE_THREAD_LIBS_INIT} ${TINYXML_LIBRARIES})

if(NOT ${CORE_SYSTEM_NAME} STREQUAL "")
  if(${CORE_SYSTEM_NAME} STREQUAL "osx" OR ${CORE_SYSTEM_NAME} STREQUAL "ios")
    list(APPEND mrmcplatform_LIBRARIES "-framework CoreVideo -framework IOKit")
  endif()
endif()

set(SOURCES src/util/XMLUtils.cpp)

include_directories(${TINYXML_INCLUDE_DIR} ${MRMC_INCLUDE_DIR} ${platform_INCLUDE_DIRS})

add_library(mrmcplatform ${SOURCES} ${PLAT_SOURCES})
target_link_libraries(mrmcplatform ${mrmcplatform_LIBRARIES} ${platform_LIBRARIES})
set_target_properties(mrmcplatform PROPERTIES VERSION ${mrmcplatform_VERSION_MAJOR}.${mrmcplatform_VERSION_MINOR}.${mrmcplatform_VERSION_PATCH}
                                              SOVERSION ${mrmcplatform_VERSION_MAJOR}.0)

install(TARGETS mrmcplatform DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES src/util/XMLUtils.h
        DESTINATION include/mrmc/util)

IF(NOT WIN32)
  # Pkgconfig
  include(PkgConfigHandler.cmake)
  configure_pc_file(mrmcplatform mrmcplatform.pc.in
                                 mrmcplatform.pc
                                 ${CMAKE_INSTALL_PREFIX}
                                 ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}
                                 ${CMAKE_INSTALL_PREFIX}/include)

  install(FILES ${CMAKE_BINARY_DIR}/mrmcplatform.pc
          DESTINATION ${CMAKE_INSTALL_LIBDIR_NOARCH}/pkgconfig)
ENDIF(NOT WIN32)

# config mode
configure_file (mrmcplatform-config.cmake.in
                mrmcplatform-config.cmake @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/mrmcplatform-config.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR_NOARCH}/mrmcplatform)

# Quell warnings with in-tree builds
set(MRMC_BUILD_DIR ${MRMC_BUILD_DIR})
