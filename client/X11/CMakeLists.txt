# FreeRDP: A Remote Desktop Protocol Implementation
# FreeRDP X11 Client
#
# Copyright 2012 Marc-Andre Moreau <marcandre.moreau@gmail.com>
# Copyright 2013 Corey Clayton <can.of.tuna@gmail.com>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set(MODULE_NAME "xfreerdp-client")
set(MODULE_PREFIX "FREERDP_CLIENT_X11_CONTROL")

include(FindXmlto)
include_directories(${X11_INCLUDE_DIRS})

set(${MODULE_PREFIX}_SRCS
	xf_gdi.c
	xf_gdi.h
	xf_rail.c
	xf_rail.h
	xf_tsmf.c
	xf_tsmf.h
	xf_input.c
	xf_input.h
	xf_event.c
	xf_event.h
	xf_input.c
	xf_input.h
	xf_channels.c
	xf_channels.h
	xf_cliprdr.c
	xf_cliprdr.h
	xf_monitor.c
	xf_monitor.h
	xf_graphics.c
	xf_graphics.h
	xf_keyboard.c
	xf_keyboard.h
	xf_window.c
	xf_window.h
	xf_client.c
	xf_client.h)

if(WITH_CLIENT_INTERFACE)
	if(CLIENT_INTERFACE_SHARED)
		add_library(${MODULE_NAME} SHARED ${${MODULE_PREFIX}_SRCS})
	else()
		add_library(${MODULE_NAME} ${${MODULE_PREFIX}_SRCS})
	endif()
	set_target_properties(${MODULE_NAME} PROPERTIES VERSION ${FREERDP_VERSION} SOVERSION ${FREERDP_API_VERSION} PREFIX "lib")
else()
	set(${MODULE_PREFIX}_SRCS ${${MODULE_PREFIX}_SRCS} cli/xfreerdp.c cli/xfreerdp.h)
	add_executable(${MODULE_NAME} ${${MODULE_PREFIX}_SRCS})
	set_target_properties(${MODULE_NAME} PROPERTIES OUTPUT_NAME "xfreerdp")
endif()

set(${MODULE_PREFIX}_LIBS
	${X11_LIBRARIES}
	${CMAKE_DL_LIBS})

if(WITH_MANPAGES)
	if(XMLTO_FOUND)
		add_custom_command(OUTPUT xfreerdp.1
			COMMAND ${XMLTO_EXECUTABLE} man ${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp.1.xml
			DEPENDS xfreerdp.1.xml)

		add_custom_target(xfreerdp.manpage ALL
			DEPENDS xfreerdp.1)

		install(FILES ${CMAKE_CURRENT_BINARY_DIR}/xfreerdp.1 DESTINATION share/man/man1)
	else(XMLTO_FOUND)
		message(WARNING "WITH_MANPAGES was set, but xmlto was not found. man-pages will not be installed")
	endif(XMLTO_FOUND)
endif(WITH_MANPAGES)

set(XSHM_FEATURE_TYPE "REQUIRED")
set(XSHM_FEATURE_PURPOSE "X11 shared memory")
set(XSHM_FEATURE_DESCRIPTION "X11 shared memory extension")

set(XINERAMA_FEATURE_TYPE "RECOMMENDED")
set(XINERAMA_FEATURE_PURPOSE "multi-monitor")
set(XINERAMA_FEATURE_DESCRIPTION "X11 multi-monitor extension")

set(XEXT_FEATURE_TYPE "RECOMMENDED")
set(XEXT_FEATURE_PURPOSE "X11 extension")
set(XEXT_FEATURE_DESCRIPTION "X11 core extensions")

set(XCURSOR_FEATURE_TYPE "RECOMMENDED")
set(XCURSOR_FEATURE_PURPOSE "cursor")
set(XCURSOR_FEATURE_DESCRIPTION "X11 cursor extension")

set(XV_FEATURE_TYPE "RECOMMENDED")
set(XV_FEATURE_PURPOSE "video")
set(XV_FEATURE_DESCRIPTION "X11 video extension")

set(XI_FEATURE_TYPE "RECOMMENDED")
set(XI_FEATURE_PURPOSE "input")
set(XI_FEATURE_DESCRIPTION "X11 input extension") 

set(XRENDER_FEATURE_TYPE "RECOMMENDED")
set(XRENDER_FEATURE_PURPOSE "rendering")
set(XRENDER_FEATURE_DESCRIPTION "X11 render extension") 

find_feature(XShm ${XSHM_FEATURE_TYPE} ${XSHM_FEATURE_PURPOSE} ${XSHM_FEATURE_DESCRIPTION})
find_feature(Xinerama ${XINERAMA_FEATURE_TYPE} ${XINERAMA_FEATURE_PURPOSE} ${XINERAMA_FEATURE_DESCRIPTION})
find_feature(Xext ${XEXT_FEATURE_TYPE} ${XEXT_FEATURE_PURPOSE} ${XEXT_FEATURE_DESCRIPTION})
find_feature(Xcursor ${XCURSOR_FEATURE_TYPE} ${XCURSOR_FEATURE_PURPOSE} ${XCURSOR_FEATURE_DESCRIPTION})
find_feature(Xv ${XV_FEATURE_TYPE} ${XV_FEATURE_PURPOSE} ${XV_FEATURE_DESCRIPTION})
find_feature(Xi ${XI_FEATURE_TYPE} ${XI_FEATURE_PURPOSE} ${XI_FEATURE_DESCRIPTION})
find_feature(Xrender ${XRENDER_FEATURE_TYPE} ${XRENDER_FEATURE_PURPOSE} ${XRENDER_FEATURE_DESCRIPTION})

if(WITH_XINERAMA)
	add_definitions(-DWITH_XINERAMA)
	include_directories(${XINERAMA_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XINERAMA_LIBRARIES})
endif()

if(WITH_XEXT)
	add_definitions(-DWITH_XEXT)
	include_directories(${XEXT_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XEXT_LIBRARIES})
endif()

if(WITH_XCURSOR)
	add_definitions(-DWITH_XCURSOR)
	include_directories(${XCURSOR_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XCURSOR_LIBRARIES})
endif()

if(WITH_XV)
	add_definitions(-DWITH_XV)
	include_directories(${XV_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XV_LIBRARIES})
endif()

if(WITH_XI)
	add_definitions(-DWITH_XI)
	include_directories(${XI_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XI_LIBRARIES})
endif()

if(WITH_XRENDER)
	add_definitions(-DWITH_XRENDER)
	include_directories(${XRENDER_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XRENDER_LIBRARIES})
endif()

include_directories(${CMAKE_SOURCE_DIR}/resources)

set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} freerdp-client)

set_complex_link_libraries(VARIABLE ${MODULE_PREFIX}_LIBS MONOLITHIC ${MONOLITHIC_BUILD}
	MODULE freerdp
	MODULES freerdp-core freerdp-gdi freerdp-locale freerdp-primitives freerdp-rail freerdp-utils)

target_link_libraries(${MODULE_NAME} ${${MODULE_PREFIX}_LIBS})

if(WITH_IPP)
    target_link_libraries(${MODULE_NAME} ${IPP_LIBRARY_LIST})
endif()

if(WITH_CLIENT_INTERFACE)
	install(TARGETS ${MODULE_NAME} DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT libraries)
	add_subdirectory(cli)
else()
	install(TARGETS ${MODULE_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT client)
endif()

set_property(TARGET ${MODULE_NAME} PROPERTY FOLDER "Client/X11")

