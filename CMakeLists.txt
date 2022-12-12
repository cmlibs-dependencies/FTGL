
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

if( BUILD_SHARED_LIBS )
	list(APPEND LIBRARY_PUBLIC_DEFINITIONS FTGL_LIBRARY)
ELSE()
	list(APPEND LIBRARY_PUBLIC_DEFINITIONS FTGL_LIBRARY_STATIC)
endif()

find_package(Freetype ${FREETYPE_VERSION} REQUIRED)

# Set required definitions for building FTGL.  This should really
# be in a config.h header file but that's more changes.
if( WIN32 )
    list(APPEND LIBRARY_PRIVATE_DEFINITIONS _CRT_SECURE_NO_WARNINGS)
endif( WIN32 )

include(ConfigureChecks.cmake )

# Configure header file
if( WIN32 )
configure_file( ftgl_config-win32.h.cmake
	${CMAKE_CURRENT_BINARY_DIR}/config/ftgl_config.h )
else()
configure_file( ftgl_config.h.cmake
	${CMAKE_CURRENT_BINARY_DIR}/config/ftgl_config.h )
endif()

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

# Create library target
add_library(${LIBRARY_TARGET_NAME} ${FTGL_SRCS} ${PUBLIC_HDRS})
target_link_libraries(${LIBRARY_TARGET_NAME} PUBLIC freetype)
target_compile_definitions(${LIBRARY_TARGET_NAME} PUBLIC ${LIBRARY_PUBLIC_DEFINITIONS})
if (LIBRARY_PRIVATE_DEFINITIONS)
    target_compile_definitions(${LIBRARY_TARGET_NAME} PRIVATE ${LIBRARY_PRIVATE_DEFINITIONS})
endif()
target_include_directories(${LIBRARY_TARGET_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/config ${CMAKE_CURRENT_SOURCE_DIR}/src)

# Set the install targets
install(TARGETS ${LIBRARY_TARGET_NAME}
    EXPORT ftgl-config
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
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
    "include(CMakeFindDependencyMacro)\n"
	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\n"
	"set(FREETYPE_FIND_SYSTEM ${FREETYPE_FIND_SYSTEM})\nfind_dependency(Freetype)\n"
	"set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})"
)
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/ftgl-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})	
