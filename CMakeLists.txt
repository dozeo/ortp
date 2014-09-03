cmake_minimum_required(VERSION 2.8)

project(ortp)

# simplify output directories on platforms with multiple configurations
foreach(OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES})
	string(TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIG)
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_BINARY_DIR}/bin)
	set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_BINARY_DIR}/lib)
	set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_BINARY_DIR}/lib)
endforeach(OUTPUTCONFIG CMAKE_CONFIGURATION_TYPES)

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

list(APPEND srcs
	"src/avprofile.c"
	"src/b64.c"
	"src/dll_entry.c"
	"src/event.c"
	"src/jitterctl.c"
	"src/logging.c"
#	"src/master"
	"src/netsim.c"
	"src/ortp.c"
	"src/ortp_srtp.c"
	"src/payloadtype.c"
	"src/port.c"
	"src/posixtimer.c"
	"src/rtcp.c"
	"src/rtcpparse.c"
	"src/rtpparse.c"
	"src/rtpprofile.c"
	"src/rtpsession.c"
	"src/rtpsession_inet.c"
	"src/rtpsignaltable.c"
	"src/rtptimer.c"
	"src/scheduler.c"
	"src/sessionset.c"
	"src/str_utils.c"
	"src/stun.c"
	"src/stun_udp.c"
#	"src/system"
	"src/telephonyevents.c"
	"src/utils.c"
#	"src/winrttimer.cpp"
	"src/zrtp.c"
	)

list(APPEND hdrs
	"src/utils.h"
	"src/winrttimer.h"
	"src/scheduler.h"
	"src/rtptimer.h"
	"src/rtpsession_priv.h"
	"src/jitterctl.h"
	"src/ortp-config-win32.h"
	)

list(APPEND public_hdrs
	"include/ortp/b64.h"
	"include/ortp/event.h"
	"include/ortp/logging.h"
	"include/ortp/ortp.h"
	"include/ortp/ortp_srtp.h"
	"include/ortp/payloadtype.h"
	"include/ortp/port.h"
	"include/ortp/rtcp.h"
	"include/ortp/rtp.h"
	"include/ortp/rtpprofile.h"
	"include/ortp/rtpsession.h"
	"include/ortp/rtpsignaltable.h"
	"include/ortp/sessionset.h"
	"include/ortp/str_utils.h"
	"include/ortp/stun.h"
	"include/ortp/stun_udp.h"
	"include/ortp/telephonyevents.h"
	"include/ortp/zrtp.h"
)

option(BUILD_ORTP_SHARED "Build shared version of ortp" OFF)

if(MSVC)
	list(APPEND definitions "_CRT_SECURE_NO_WARNINGS")
endif()

if(BUILD_ORTP_SHARED)
	set(LIBTYPE SHARED)
	list(APPEND definitions "ORTP_EXPORTS")
else()
	set(LIBTYPE STATIC)
	list(APPEND definitions "ORTP_STATIC")
	list(APPEND interface_definitions "ORTP_STATIC")
endif()

if(WIN32)
	list(APPEND system_libs
		"wsock32"
		"ws2_32"
		"winmm"
		"delayimp"
		"qwave")
endif()

add_library(ortp ${LIBTYPE} ${srcs} ${hdrs} ${public_hdrs})
set_target_properties(ortp PROPERTIES
	PUBLIC_HEADERS "${public_hdrs}"
	INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
	COMPILE_DEFINITIONS "${definitions}"
	INTERFACE_COMPILE_DEFINITIONS "${interface_definitions}"
	INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
	)
target_link_libraries(ortp ${system_libs})