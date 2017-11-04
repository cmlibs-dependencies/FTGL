
# Set the minimum version of cmake required to 2.6
cmake_minimum_required( VERSION 3.0 )

project(FTGL VERSION 2.1.3 LANGUAGES C CXX )
set(CMAKE_POSITION_INDEPENDENT_CODE YES)
SET(CMAKE_OSX_ARCHITECTURES i386;x86_64)

set(PROJECT_NAMESPACE_PREFIX FTGL )
set(LIBRARY_TARGET_NAME ftgl )
# STRING( TOUPPER ${LIBRARY_TARGET_NAME}  )
set(FTGL_MAJOR_VERSION "2" )
set(FTGL_MINOR_VERSION "1" )
set(FTGL_PATCH_VERSION "3" )

set(PACKAGE_VERSION \"${FTGL_MAJOR_VERSION}.${FTGL_MINOR_VERSION}.${FTGL_PATCH_VERSION}\" ) 
# Set up library options
# Generic library options
OPTION( ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB "Build static ftgl" TRUE )
OPTION( ${PROJECT_NAMESPACE_PREFIX}_INSTALL_CONFIG "Install a config file for the library" FALSE )

if( ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )
	set(LIBRARY_BUILD_TYPE STATIC )
	set(LIBRARY_INSTALL_TYPE ARCHIVE )
	add_definitions( -DFTGL_LIBRARY_STATIC )
ELSE()
	set(LIBRARY_BUILD_TYPE SHARED )
	set(LIBRARY_INSTALL_TYPE LIBRARY )
	if( WIN32 )
		set(LIBRARY_INSTALL_TYPE RUNTIME )
	endif( WIN32 )
	add_definitions( -DFTGL_LIBRARY )
endif()

find_package(Freetype ${FREETYPE_VERSION} REQUIRED)

# Set required definitions for building FTGL.  This should really
# be in a config.h header file but that's more changes.
if( WIN32 )
	set(FTGL_DEFINITIONS -D_CRT_SECURE_NO_WARNINGS )
endif( WIN32 )
if( NOT ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )
	set(FTGL_DEFINITIONS ${FTGL_DEFINITIONS} -DFTGL_LIBRARY_SHARED )

endif( NOT ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )

include(ConfigureChecks.cmake )

# Configure header file
if( WIN32 )
configure_file( ftgl_config-win32.h.cmake
	${CMAKE_CURRENT_BINARY_DIR}/config/ftgl_config.h )
else()
configure_file( ftgl_config.h.cmake
	${CMAKE_CURRENT_BINARY_DIR}/config/ftgl_config.h )
endif()

# Add configured header file to header file search path
include_directories( ${CMAKE_CURRENT_BINARY_DIR}/config)

add_definitions(${FTGL_DEFINITIONS})

include_directories( src )
include_directories( src/FTFont )
#/include_directories( src/FTGL )
include_directories( src/FTGlyph )
include_directories( src/FTLayout )

set(PUBLIC_HDRS
	src/FTGL/FTBBox.h 
	src/FTGL/FTBitmapGlyph.h
	src/FTGL/FTBuffer.h
	src/FTGL/FTGLExtrdFont.h
	src/FTGL/FTGLPolygonFont.h
	src/FTGL/FTOutlineGlyph.h
	src/FTGL/FTSimpleLayout.h
	src/FTGL/FTBitmapGlyph.h
	src/FTGL/FTExtrdGlyph.h
	src/FTGL/ftgl.h
	src/FTGL/FTGLTextureFont.h
	src/FTGL/FTPixmapGlyph.h
	src/FTGL/FTTextureGlyph.h
	src/FTGL/FTBufferFont.h
	src/FTGL/FTFont.h
	src/FTGL/FTGLOutlineFont.h
	src/FTGL/FTGlyph.h
	src/FTGL/FTPoint.h
	src/FTGL/FTBufferGlyph.h
	src/FTGL/FTGLBitmapFont.h
	src/FTGL/FTGLPixmapFont.h
	src/FTGL/FTLayout.h
	src/FTGL/FTPolyGlyph.h )

set(CMAKE_DEBUG_POSTFIX "d" )
set(CMAKE_MFC_FLAG 0 )

# Defines NGLIB_SRCS all the sources required for a stripped down library
# suitable for cmgui
include(ftgl-cmgui.cmake )

if( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64" )
	include(ftgl-cmgui.cmake )
endif()

# Create library target
add_library(${LIBRARY_TARGET_NAME} ${LIBRARY_BUILD_TYPE} ${FTGL_SRCS})
target_link_libraries(${LIBRARY_TARGET_NAME} PUBLIC freetype)

# Set the install targets
install(TARGETS ${LIBRARY_TARGET_NAME}
    EXPORT ftgl-config ${LIBRARY_INSTALL_TYPE}
	DESTINATION lib
	INCLUDES DESTINATION include/FTGL)
install(FILES ${PUBLIC_HDRS}
	DESTINATION include/FTGL
)
install(EXPORT ftgl-config 
	DESTINATION ${PACKAGE_CONFIG_DIR} )
	
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-version.cmake
    COMPATIBILITY AnyNewerVersion)
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-dependencies.cmake 
    "include(CMakeFindDependencyMacro)\r\n"
	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\r\n"
	"set(FREETYPE_FIND_SYSTEM ${FREETYPE_FIND_SYSTEM})\r\nfind_dependency(Freetype)\r\n"
	"set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})"
)
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})	


