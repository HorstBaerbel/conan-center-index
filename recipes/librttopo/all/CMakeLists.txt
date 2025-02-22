cmake_minimum_required(VERSION 3.4)
project(rttopo C)

include(conanbuildinfo.cmake)
conan_basic_setup()

set(RTTOPO_HDR_DIR ${CMAKE_CURRENT_SOURCE_DIR}/source_subfolder/headers)
set(RTTOPO_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/source_subfolder/src)

set(SRID_MAX 999999)
set(SRID_USR_MAX 998999)
configure_file(${RTTOPO_HDR_DIR}/librttopo_geom.h.in librttopo_geom.h @ONLY)
file(WRITE rttopo_config.h "")

file(GLOB RTTOPO_SRC_FILES ${RTTOPO_SRC_DIR}/*.c)
add_library(${PROJECT_NAME} ${RTTOPO_SRC_FILES})
target_compile_definitions(${PROJECT_NAME} PRIVATE
    $<$<BOOL:${MSVC}>:_CRT_SECURE_NO_WARNINGS>
    RTGEOM_DEBUG_LEVEL=0
    LIBRTGEOM_VERSION="${LIBRTGEOM_VERSION_MAJOR}.${LIBRTGEOM_VERSION_MINOR}.${LIBRTGEOM_VERSION_PATCH}"
    RTGEOM_GEOS_VERSION=${RTGEOM_GEOS_VERSION}
)
set_target_properties(${PROJECT_NAME} PROPERTIES
    WINDOWS_EXPORT_ALL_SYMBOLS ON
    SOVERSION ${LIBRTGEOM_VERSION_MAJOR}
)
target_include_directories(${PROJECT_NAME} PUBLIC ${RTTOPO_HDR_DIR} ${CMAKE_CURRENT_BINARY_DIR})

find_package(geos REQUIRED)
target_link_libraries(${PROJECT_NAME} PUBLIC GEOS::geos_c)

find_library(M_LIB m)
if(M_LIB)
    target_link_libraries(${PROJECT_NAME} PRIVATE m)
endif()

include(GNUInstallDirs)
install(
  TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(FILES ${RTTOPO_HDR_DIR}/librttopo.h ${CMAKE_CURRENT_BINARY_DIR}/librttopo_geom.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
