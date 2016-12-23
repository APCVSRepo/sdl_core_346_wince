
include_directories(
  .
)

set (SOURCES
  pthread.c
)

add_definitions(-DPTW32_STATIC_LIB)
add_definitions(-DHAVE_PTW32_CONFIG_H)
add_definitions(-DWINCE)
add_definitions(-D_UWIN_)

add_library(pthreadVC2 ${HEADERS} ${SOURCES})