set(ENCRYPTION_INCLUDE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(ENCRYPTION_INCLUDE_DIRECTORY ${ENCRYPTION_INCLUDE_DIRECTORY} PARENT_SCOPE)

include_directories (
  ./include
)

if(CMAKE_SYSTEM_NAME STREQUAL "WindowsCE")
  include_directories(${CMAKE_SOURCE_DIR}/include/wince/)
  include_directories(${CMAKE_SOURCE_DIR}/lib/wince/)
endif()

set (SOURCES
  ./src/Base64.cpp
  ./src/hashing.cc
)

set (LIBRARIES ssleay32 libeay32)

add_library(encryption ${SOURCES})
target_link_libraries(encryption ${LIBRARIES})
