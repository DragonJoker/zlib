cmake_minimum_required( VERSION 3.15 )

project( ZLIB )

set( ZLIB_MANGLE_PREFIX "" CACHE STRING "Mangle prefix. For example 'cm_zlib_', 'z_', ..." )

set( ZLIB_VERSION_MAJOR "1" )
set( ZLIB_VERSION_MINOR "2" )
set( ZLIB_VERSION_PATCH "3" )

include_directories(
	"${ZLIB_SOURCE_DIR}"
	"${ZLIB_SOURCE_DIR}"
	"${ZLIB_BINARY_DIR}"
)
# source files for zlib
set( ZLIB_SRCS
	adler32.c  compress.c  crc32.c  deflate.c  gzio.c   inffast.c
	inflate.c  inftrees.c  trees.c  uncompr.c  zutil.c
)
# for windows add the .def and .rc files to the source list
# if building shared libs
if ( WIN32 )
	if ( BUILD_SHARED_LIBS )
		set( ZLIB_DLL 1 )
		if ( NOT UNIX )
			if ( NOT BORLAND )
				if ( NOT MINGW )
					configure_file( zlib.def.in ${ZLIB_BINARY_DIR}/zlib.def )
					set( ZLIB_SRCS ${ZLIB_SRCS} ${ZLIB_BINARY_DIR}/zlib.def zlib.rc )
				endif ( NOT MINGW )
			endif ( NOT BORLAND )
		endif ( NOT UNIX )
	endif ( BUILD_SHARED_LIBS )
endif ( WIN32 )

configure_file(${ZLIB_SOURCE_DIR}/.NoDartCoverage
	${ZLIB_BINARY_DIR}/.NoDartCoverage )
configure_file(${ZLIB_SOURCE_DIR}/zlibDllConfig.h.in
	${ZLIB_BINARY_DIR}/zlibDllConfig.h )
configure_file( ${ZLIB_SOURCE_DIR}/zlib_mangle.h.in
	${ZLIB_BINARY_DIR}/zlib_mangle.h )

foreach( name zlib zconf )
	CONFIGURE_FILE( ${ZLIB_SOURCE_DIR}/${name}.h
		${ZLIB_BINARY_DIR}/${name}.h COPYONLY )
endforeach( name )

add_library( ${PROJECT_NAME} ${ZLIB_SRCS} )
add_library( ZLIB::ZLIB ALIAS ${PROJECT_NAME} )
# To fix compilation problem: relocation R_X86_64_32 against `a local symbol' can not be
# used when making a shared object; recompile with -fPIC
# See http://www.cmake.org/pipermail/cmake/2007-May/014350.html
#
if ( CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" )
	set_target_properties( ${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-fPIC" )
endif ()
target_include_directories( ${PROJECT_NAME}
	INTERFACE
		${CMAKE_CURRENT_SOURCE_DIR}/include
)
if ( NOT ZLIB_SKIP_INSTALL )
	# Configure install locations.  This allows parent projects to modify
	# the install location.
	if ( NOT ZLIB_INSTALL_BIN_DIR)
		set(ZLIB_INSTALL_BIN_DIR bin)
	endif ()
	if ( NOT ZLIB_INSTALL_INCLUDE_DIR)
		set( ZLIB_INSTALL_INCLUDE_DIR include )
	endif ()
	if ( NOT ZLIB_INSTALL_LIB_DIR)
		set( ZLIB_INSTALL_LIB_DIR lib )
	endif ()
	if ( NOT ZLIB_INSTALL_DOC_DIR )
		set( ZLIB_INSTALL_DOC_DIR
			doc/zlib-${ZLIB_VERSION_MAJOR}.${ZLIB_VERSION_MINOR}.${ZLIB_VERSION_PATCH}
		)
	endif ()
	# Install library
	install( TARGETS ${PROJECT_NAME}
		RUNTIME DESTINATION ${ZLIB_INSTALL_BIN_DIR} COMPONENT RuntimeLibraries
		LIBRARY DESTINATION ${ZLIB_INSTALL_LIB_DIR} COMPONENT RuntimeLibraries
		ARCHIVE DESTINATION ${ZLIB_INSTALL_LIB_DIR} COMPONENT Development
	)
	# Install public headers
	install( FILES
		zlib.h
		zconf.h
		${ZLIB_BINARY_DIR}/zlib_mangle.h
		${ZLIB_BINARY_DIR}/zlibDllConfig.h
		DESTINATION ${ZLIB_INSTALL_INCLUDE_DIR}
		COMPONENT Development
	)
	install( FILES Copyright.txt
		DESTINATION ${ZLIB_INSTALL_DOC_DIR}
		COMPONENT RuntimeLibraries
	)
endif ()
