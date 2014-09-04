cmake_minimum_required(VERSION 2.8)

project(ortp)

set(ORTP_MAJOR_VERSION 0)
set(ORTP_MINOR_VERSION 21)
set(ORTP_MICRO_VERSION 1)
set(ORTP_VERSION "${ORTP_MAJOR_VERSION}.${ORTP_MINOR_VERSION}.${ORTP_MICRO_VERSION}")
SET(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

set(PACKAGE "ortp")
set(PACKAGE_NAME "ortp")
set(PACKAGE_TARNAME "ortp")
set(PACKAGE_STRING "ortp ${ORTP_VERSION}")
set(PACKAGE_URL "https://github.com/dozeo/ortp")
set(PACKAGE_VERSION "${ORTP_VERSION}")
set(VERSION "${ORTP_VERSION}")

include(CheckIncludeFiles)
include(CheckFunctionExists)
include(CheckLibraryExists)
include(CheckPrototypeExists)
include(TestBigEndian)
include(CheckCCompiler)

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

# configure
check_include_files(dlfcn.h HAVE_DLFCN_H)
check_include_files(fcntl.h HAVE_FCNTL_H)
check_include_files(inttypes.h HAVE_INTTYPES_H)
check_include_files(linux/soundcard.h HAVE_LINUX_SOUNDCARD_H)
check_include_files(memory.h HAVE_MEMORY_H)
check_include_files(openssl/hmac.h HAVE_OPENSSL_HMAC_H)
check_include_files(openssl/md5.h HAVE_OPENSSL_MD5_H)
check_include_files(poll.h HAVE_POLL_H)
check_include_files(stdint.h HAVE_STDINT_H)
check_include_files(stdlib.h HAVE_STDLIB_H)
check_include_files(strings.h HAVE_STRINGS_H)
check_include_files(string.h HAVE_STRING_H)
check_include_files(sys/audio.h HAVE_SYS_AUDIO_H)
check_include_files(sys/poll.h HAVE_SYS_POLL_H)
check_include_files(sys/shm.h HAVE_SYS_SHM_H)
check_include_files(sys/stat.h HAVE_SYS_STAT_H)
check_include_files(sys/time.h HAVE_SYS_TIME_H)
check_include_files(sys/types.h HAVE_SYS_TYPES_H)
check_include_files(sys/uio.h HAVE_SYS_UIO_H)
check_include_files(unistd.h HAVE_UNISTD_H)

check_function_exists(select HAVE_SELECT)
check_function_exists(seteuid HAVE_SETEUID)
check_function_exists(socket HAVE_SOCKET)
check_function_exists(select HAVE_SELECT)
check_function_exists(strerror HAVE_STRERROR)

if (NOT DEFINED STDC_HEADERS)

	message(STATUS "Checking whether system has ANSI C header files")
	check_include_files("stdlib.h;stdarg.h;string.h;float.h" StandardHeadersExist)
	if (StandardHeadersExist)
		check_prototype_exists (memchr string.h memchrExists)
		check_prototype_exists (free stdlib.h freeExists)

		if (memchrExists AND freeExists)
			message (STATUS "ANSI C header files - found")
			set(STDC_HEADERS 1 CACHE INTERNAL "System has ANSI C header files")
		endif()
	endif()

	if (NOT STDC_HEADERS)
		message (STATUS "ANSI C header files - not found")
		set (STDC_HEADERS 0 CACHE INTERNAL "System has ANSI C header files")
	endif()

endif()

# Define to 1 if you can safely include both <sys/time.h> and <time.h>
if (HAVE_SYS_TIME_H)
	check_include_files("sys/time.h;time.h" TIME_WITH_SYS_TIME)
endif (HAVE_SYS_TIME_H)

TEST_BIG_ENDIAN(IS_BIG_ENDIAN)
if(IS_BIG_ENDIAN)
	set(WORDS_BIGENDIAN 1)
else()
	set(WORDS_BIGENDIAN 0)
endif()

CHECK_C_COMPILER_SUPPORTS_INLINE(COMPILER_INLINE COMPILER_INLINE_SUPPORTED)
message(STATUS "Compiler inline: ${COMPILER_INLINE}")
message(STATUS "Compiler inline supported: ${COMPILER_INLINE_SUPPORTED}")
if(${COMPILER_INLINE} MATCHES "inline")
	unset(COMPILER_INLINE)
endif()

CHECK_C_COMPILER_SUPPORTS_CONST(CONST_ANSI_C)
message(STATUS "Compiler has ANSI C Const: ${CONST_ANSI_C}")
if(NOT ${CONST_ANSI_C})
	set(const 1)
endif()

# TODO: change this to 20000 on hpux
set(POSIXTIMER_INTERVAL 10000)

if(APPLE)
	set(__APPLE_USE_RFC_2292 1)
endif()

option(BUILD_ENABLE_MEMORY_LEAK_CHECK "Enable memory leak checks" OFF)
option(BUILD_ORTP_SHARED "Build shared version of ortp" OFF)
option(BUILD_ORTP_DEBUG_MODE "Enable debug mode" OFF)
option(BUILD_WITH_SRTP "Build with libsrtp enabled (untested)" OFF)
option(BUILD_WITH_ZRTP "Build with zrtp enabled (untested)" OFF)
option(BUILD_ORTP_NO_CONNECT "Disable connect() on udp sockets" OFF)

if(BUILD_ENABLE_MEMORY_LEAK_CHECK AND NOT DEBUG)
	set(ENABLE_MEMCHECK 1)
endif()

if(BUILD_ORTP_DEBUG_MODE)
	list(APPEND definitions "ORTP_DEBUG_MODE")
endif()

if(BUILD_WITH_SRTP)
	find_package(SRTP OPTIONAL)
	if (SRTP_FOUND)
		set(HAVE_SRTP 1)
	else()
		message(STATUS "srtp package not found... disabling BUILD_WITH_SRTP")
	endif()
endif()

if(HAVE_SRTP)
	check_library_exists(libsrtp srtp_shutdown "" HAVE_SRTP_SHUTDOWN)
endif()

if(BUILD_WITH_ZRTP)
	find_package(zrtp OPTIONAL)
	if(zrtp_FOUND)
		set(HAVE_zrtp 1)
	else()
		message(STATUS "zrtp package not found... disabling BUILD_WITH_ZRTP")
	endif()
endif()

if(BUILD_ORTP_NO_CONNECT)
	set(NOCONNECT 1)
endif()

set(ORTP_DEFAULT_THREAD_STACK_SIZE 0)

configure_file(ortp-config.cmake.h.in
	${CMAKE_CURRENT_BINARY_DIR}/ortp-config.h
	@ONLY)

if(MSVC)
	list(APPEND definitions "_CRT_SECURE_NO_WARNINGS")
else()
	list(APPEND definitions "HAVE_CONFIG_H")
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
	list(APPEND srcs
		"src/dll_entry.c")

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
	INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include;${CMAKE_CURRENT_BINARY_DIR}"
	COMPILE_DEFINITIONS "${definitions}"
	INTERFACE_COMPILE_DEFINITIONS "${interface_definitions}"
	INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include;${CMAKE_CURRENT_BINARY_DIR}"
	)
target_link_libraries(ortp ${system_libs})

