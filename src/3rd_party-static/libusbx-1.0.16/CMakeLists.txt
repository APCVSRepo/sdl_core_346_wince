set(LIBUSB_INCLUDE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
set(LIBUSB_INCLUDE_DIRECTORY ${LIBUSB_INCLUDE_DIRECTORY} PARENT_SCOPE)

set(LIBUSB_LIBS_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
set(LIBUSB_LIBS_DIRECTORY ${LIBUSB_LIBS_DIRECTORY} PARENT_SCOPE)

include_directories (
  ./libusb/os/
  ./libusb/
  ./
)

set (LIBUSB_SOURCES
	./libusb/missing.c
	./libusb/core.c
	./libusb/descriptor.c
	./libusb/hotplug.c
	./libusb/io.c
	./libusb/strerror.c
	./libusb/sync.c
	./libusb/os/wince_usb.c
	./libusb/os/poll_windows.c
	./libusb/os/threads_windows.c
)

add_library("Libusb-1.0.16" ${LIBUSB_SOURCES})