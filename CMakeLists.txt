
# Set the minimum version of cmake required to 2.6
cmake_minimum_required( VERSION 3.0 )

project(FTGL VERSION 2.1.3 LANGUAGES C CXX )
set(CMAKE_POSITION_INDEPENDENT_CODE YES)

SET( PROJECT_NAMESPACE_PREFIX FTGL )
SET( LIBRARY_TARGET_NAME ftgl )
# STRING( TOUPPER ${LIBRARY_TARGET_NAME}  )
SET( FTGL_MAJOR_VERSION "2" )
SET( FTGL_MINOR_VERSION "1" )
SET( FTGL_PATCH_VERSION "3" )

SET( PACKAGE_VERSION \"${FTGL_MAJOR_VERSION}.${FTGL_MINOR_VERSION}.${FTGL_PATCH_VERSION}\" ) 
# Set up library options
# Generic library options
OPTION( ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB "Build static ftgl" TRUE )
OPTION( ${PROJECT_NAMESPACE_PREFIX}_INSTALL_CONFIG "Install a config file for the library" FALSE )

IF( ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )
	SET( LIBRARY_BUILD_TYPE STATIC )
	SET( LIBRARY_INSTALL_TYPE ARCHIVE )
	ADD_DEFINITIONS( -DFTGL_LIBRARY_STATIC )
ELSE( ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )
	SET( LIBRARY_BUILD_TYPE SHARED )
	SET( LIBRARY_INSTALL_TYPE LIBRARY )
	IF( WIN32 )
		SET( LIBRARY_INSTALL_TYPE RUNTIME )
	ENDIF( WIN32 )
	ADD_DEFINITIONS( -DFTGL_LIBRARY )
ENDIF( ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )

find_package(Freetype REQUIRED)

SET( FREETYPE_PATH ${FREETYPE_INCLUDE_DIRS} )

# Set required definitions for building FTGL.  This should really
# be in a config.h header file but that's more changes.
IF( WIN32 )
	SET( FTGL_DEFINITIONS -D_CRT_SECURE_NO_WARNINGS )
ENDIF( WIN32 )
IF( NOT ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )
	SET( FTGL_DEFINITIONS ${FTGL_DEFINITIONS} -DFTGL_LIBRARY_SHARED )

ENDIF( NOT ${PROJECT_NAMESPACE_PREFIX}_BUILD_STATIC_LIB )

INCLUDE( ConfigureChecks.cmake )

# Configure header file
IF( WIN32 )
CONFIGURE_FILE( ftgl_config-win32.h.cmake
	${CMAKE_CURRENT_BINARY_DIR}/config/ftgl_config.h )
ELSE()
CONFIGURE_FILE( ftgl_config.h.cmake
	${CMAKE_CURRENT_BINARY_DIR}/config/ftgl_config.h )
ENDIF()

# Add configured header file to header file search path
include_directories( ${CMAKE_CURRENT_BINARY_DIR}/config)

add_definitions(${FTGL_DEFINITIONS})

INCLUDE_DIRECTORIES( ${FREETYPE_PATH} )
INCLUDE_DIRECTORIES( src )
INCLUDE_DIRECTORIES( src/FTFont )
INCLUDE_DIRECTORIES( src/FTGL )
INCLUDE_DIRECTORIES( src/FTGlyph )
INCLUDE_DIRECTORIES( src/FTLayout )

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

SET( CMAKE_DEBUG_POSTFIX "d" )
SET( CMAKE_MFC_FLAG 0 )

# Defines NGLIB_SRCS all the sources required for a stripped down library
# suitable for cmgui
INCLUDE( ftgl-cmgui.cmake )

IF( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64" )
	INCLUDE( ftgl-cmgui.cmake )
ENDIF()

# Create library target
ADD_LIBRARY(${LIBRARY_TARGET_NAME} ${LIBRARY_BUILD_TYPE} ${FTGL_SRCS} )

# Set the install targets
INSTALL(TARGETS ${LIBRARY_TARGET_NAME}
    EXPORT ftgl-config ${LIBRARY_INSTALL_TYPE}
	DESTINATION lib
	INCLUDES DESTINATION include/FTGL)
INSTALL(FILES ${PUBLIC_HDRS}
	DESTINATION include/FTGL
)
INSTALL(EXPORT ftgl-config 
	DESTINATION ${PACKAGE_CONFIG_DIR} )
	
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-version.cmake
    COMPATIBILITY AnyNewerVersion)
#file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake 
#    "include(CMakeFindDependencyMacro)\r\n"
#	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_I#MPORT_PREFIX})\r\n"
#)
#file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "find_dependency(BLAS ${BLAS_VERSION})\r\n")
#file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "find_dependency(LAPACK ${LAPACK_VERSION})\r\n")
#if (WITH_OPENMP)
#    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "find_dependency(OpenMP)\r\n")
#endif()
#if (WITH_SUPERLU AND NOT SUPERLU_INTERNAL)
#    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "find_dependency(SUPERLU)\r\n")
#endif()
#file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})")
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-version.cmake
    #${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})	


