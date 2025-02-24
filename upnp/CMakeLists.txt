if (WIN32)
	set (WIN_PREFIX "lib")
endif()

if (samples)
	add_subdirectory (sample)
endif()

set (UPNP_SOURCES
        src/UpnpLib.c
	src/api/UpnpActionComplete.c
	src/api/UpnpActionRequest.c
	src/api/UpnpDiscovery.c
	src/api/UpnpEvent.c
	src/api/UpnpEventSubscribe.c
	src/api/UpnpHttpHeaders.c
	src/api/UpnpFileInfo.c
	src/api/UpnpLog.cpp
	src/api/UpnpStateVarComplete.c
	src/api/UpnpStateVarRequest.c
	src/api/UpnpSubscriptionRequest.c
	src/api/UpnpString.cpp
	src/api/upnpapi.cpp
	src/genlib/client_table/GenlibClientSubscription.c
	src/genlib/client_table/client_table.c
	src/genlib/miniserver/miniserver.c
	src/genlib/net/sock.c
	src/genlib/net/http/httpparser.c
	src/genlib/net/http/httpreadwrite.c
	src/genlib/net/http/parsetools.c
	src/genlib/net/http/statcodes.c
	src/genlib/net/http/webserver.c
	src/genlib/net/uri/uri.c
	src/genlib/service_table/service_table.c
	src/genlib/util/list.c
	src/genlib/util/membuffer.c
	src/genlib/util/strintmap.c
	src/genlib/util/upnp_timeout.c
	src/threadutil/FreeList.c
	src/threadutil/LinkedList.c
	src/threadutil/ThreadPool.c
	src/threadutil/TimerThread.c
	src/urlconfig/urlconfig.c
)

if (gena)
	list (APPEND UPNP_SOURCES
		src/gena/gena_device.c
		src/gena/gena_ctrlpt.c
		src/gena/gena_callback2.c
	)
endif()

if (soap)
	list (APPEND UPNP_SOURCES
		src/soap/soap_device.c
		src/soap/soap_ctrlpt.c
		src/soap/soap_common.c
	)
endif()

if (ssdp)
	list (APPEND UPNP_SOURCES
		src/ssdp/SSDPResultData.c
		src/ssdp/SSDPResultDataCallback.c
		src/ssdp/ssdp_device.c
		src/ssdp/ssdp_ctrlpt.c
		src/ssdp/ssdp_server.c
	)
endif()

if (tools)
	list (APPEND UPNP_SOURCES
		src/api/upnptools.c
	)
endif()

if (uuid)
	list (APPEND UPNP_SOURCES
		src/uuid/md5.c
		src/uuid/sysdep.c
		src/uuid/uuid.c
	)
endif()

list (APPEND UPNP_HEADERS
	inc/Callback.h
	inc/ithread.h
	inc/list.h
	inc/upnp.h
	inc/UpnpActionComplete.h
	inc/UpnpActionRequest.h
	inc/UpnpDiscovery.h
	inc/UpnpEvent.h
	inc/UpnpEventSubscribe.h
	inc/UpnpHttpHeaders.h
	inc/UpnpFileInfo.h
	inc/UpnpGlobal.h
	inc/UpnpInet.h
	inc/UpnpIntTypes.h
	inc/UpnpStateVarComplete.h
	inc/UpnpStateVarRequest.h
	inc/UpnpStdInt.h
	inc/UpnpString.h
	inc/UpnpSubscriptionRequest.h
	inc/UpnpUniStd.h
	${CMAKE_CURRENT_SOURCE_DIR}/inc/UpnpLog.h
	${PUPNP_BINARY_DIR}/upnp/inc/upnpconfig.h
)

if (tools)
	list (APPEND UPNP_HEADERS
		inc/upnptools.h
	)
endif()

if (UPNP_BUILD_SHARED)
	add_library (upnp_shared SHARED
		${UPNP_SOURCES}
	)

	add_library (UPNP::Shared ALIAS upnp_shared)

	set_target_properties (upnp_shared PROPERTIES
		WINDOWS_EXPORT_ALL_SYMBOLS TRUE
	)

	target_compile_definitions (upnp_shared
		PRIVATE $<$<CONFIG:Debug>:STATS>
		PUBLIC $<IF:$<CONFIG:Debug>,DEBUG,NDEBUG>
	)

	target_include_directories (upnp_shared
		PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/threadutil/
		PUBLIC $<BUILD_INTERFACE:${PUPNP_BINARY_DIR}/>
		PUBLIC $<BUILD_INTERFACE:${PUPNP_BINARY_DIR}/upnp/inc/>
		PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc/>
		PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/inc/>
	)

	set_target_properties (upnp_shared PROPERTIES
		OUTPUT_NAME ${WIN_PREFIX}upnp
		EXPORT_NAME UPNP::Shared
		VERSION ${UPNP_VERSION_STRING}
		SOVERSION ${UPNP_VERSION_MAJOR}
		PUBLIC_HEADER "${UPNP_HEADERS}"
	)

	if (HAVE_MACRO_PREFIX_MAP)
		target_compile_options(upnp_shared PRIVATE -fmacro-prefix-map=${CMAKE_SOURCE_DIR}/=)
	endif()

	if (WIN32)
		target_link_libraries (upnp_shared
			PRIVATE ws2_32
			PRIVATE iphlpapi
		)
	endif()

	target_link_libraries (upnp_shared
		PUBLIC ixml_shared
		PUBLIC Threads::Shared
	)

	if (UPNP_ENABLE_OPEN_SSL)
		target_link_libraries (upnp_shared
			PRIVATE OpenSSL::SSL
		)
	endif()

	install (TARGETS upnp_shared
		EXPORT UPNP
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
			COMPONENT UPNP_Development
		INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/upnp
			COMPONENT UPNP_Development
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
			COMPONENT UPNP_RunTime
			NAMELINK_COMPONENT UPNP_Development
		PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/upnp
			COMPONENT UPNP_Development
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
			COMPONENT UPNP_RunTime
	)
endif()

if (UPNP_BUILD_STATIC)
	add_library (upnp_static STATIC
		${UPNP_SOURCES}
	)

	add_library (UPNP::Static ALIAS upnp_static)

	target_compile_definitions (upnp_static
		PRIVATE UPNP_STATIC_LIB
		PRIVATE $<$<CONFIG:Debug>:STATS>
		PUBLIC $<IF:$<CONFIG:Debug>,DEBUG,NDEBUG>
	)

	if (HAVE_MACRO_PREFIX_MAP)
		target_compile_options(upnp_static PRIVATE -fmacro-prefix-map=${CMAKE_SOURCE_DIR}/=)
	endif()

	target_include_directories (upnp_static
		PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/threadutil/
		PUBLIC $<BUILD_INTERFACE:${PUPNP_BINARY_DIR}/>
		PUBLIC $<BUILD_INTERFACE:${PUPNP_BINARY_DIR}/upnp/inc/>
		PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc/>
		PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/inc/>
	)

	if (WIN32)
		target_link_libraries (upnp_static
			INTERFACE ws2_32
			INTERFACE iphlpapi
		)
	endif()

	set_target_properties (upnp_static PROPERTIES
		OUTPUT_NAME ${WIN_PREFIX}upnp${STATIC_POSTFIX}
		EXPORT_NAME UPNP::Static
		VERSION ${UPNP_VERSION_MAJOR}
		SOVERSION ${UPNP_VERSION_STRING}
		PUBLIC_HEADER "${UPNP_HEADERS}"
	)

	target_link_libraries (upnp_static
		PUBLIC ixml_static
		PUBLIC Threads::Static
	)

	if (UPNP_ENABLE_OPEN_SSL)
		target_link_libraries (upnp_static
			PRIVATE OpenSSL::SSL
		)
	endif()

	install (TARGETS upnp_static
		EXPORT UPNP
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
			COMPONENT UPNP_Development
		INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/upnp
			COMPONENT UPNP_Development
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
			COMPONENT UPNP_RunTime
			NAMELINK_COMPONENT UPNP_Development
		PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/upnp
			COMPONENT UPNP_Development
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
			COMPONENT UPNP_RunTime
	)
endif()

install (EXPORT UPNP
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/UPNP
)

if (BUILD_TESTING)
	enable_testing()
	add_subdirectory (test)
endif()
