cmake_minimum_required(VERSION 3.20)

project(CS106BLIBRARY VERSION 23.1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

set(QT_LIBNAME Qt6)

find_package(${QT_LIBNAME} COMPONENTS Widgets Gui Network REQUIRED)

set(LIBRARYDIR ${CMAKE_CURRENT_LIST_DIR})

set(USER "Lsyx")
set(SPL_VERSION 23.1)
string(TIMESTAMP _DATE_ "%Y%m%d")

configure_file(${LIBRARYDIR}/private/build.h.cmake ${LIBRARYDIR}/private/build.h @ONLY)
file(GLOB LIBSOURCEFILES CONFIGURE_DEPENDS 
                  collections/*.cpp
                  console/*.cpp
                  graphics/*.cpp
                  io/*.cpp
                  private/*.cpp
                  system/*.cpp
                  testing/*.cpp
                  util/*.cpp)
file(GLOB LIBHEADERS CONFIGURE_DEPENDS  
                  collections/*.h
                  console/*.h
                  graphics/*.h
                  io/*.h
                  private/*.h
                  system/*.h
                  testing/*.h
                  util/*.h)

include(GNUInstallDirs)
include(GenerateExportHeader)

set(cs106b cs106b)
add_library(${cs106b} STATIC 
    ${LIBSOURCEFILES}
    ${LIBHEADERS}
    ${LIBRARYDIR}/images.qrc)

add_library(Stanford::${cs106b} ALIAS ${cs106b})

generate_export_header(${cs106b})
set_target_properties(${cs106b} PROPERTIES DEBUG_POSTFIX d)

set(INTERFACE_HEADERS 
    ${LIBRARYDIR}
    ${LIBRARYDIR}/collections
    ${LIBRARYDIR}/console
    ${LIBRARYDIR}/graphics 
    ${LIBRARYDIR}/io 
    ${LIBRARYDIR}/system
    ${LIBRARYDIR}/testing
    ${LIBRARYDIR}/util)

target_include_directories(${cs106b} PUBLIC
    "$<BUILD_INTERFACE:${INTERFACE_HEADERS}>"
    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")

target_link_libraries(${cs106b} PUBLIC
    ${QT_LIBNAME}::Core
    ${QT_LIBNAME}::Gui
    ${QT_LIBNAME}::Widgets
    ${QT_LIBNAME}::Network)

target_compile_definitions(${cs106b} PRIVATE QT_DEPRECATED_WARNINGS)


install(TARGETS ${cs106b} FILE_SET HEADERS)
install(FILES ${LIBHEADERS} 
              "${CMAKE_CURRENT_BINARY_DIR}/${cs106b}_export.h" 
              DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(DIRECTORY ${LIBRARYDIR}/resources DESTINATION .)

install(TARGETS ${cs106b} EXPORT ${cs106b}Targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION  ${CMAKE_INSTALL_LIBDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

include(CmakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${cs106b}/${cs106b}-config-version.cmake
    VERSION ${SPL_VERSION} 
    COMPATIBILITY AnyNewerVersion)

export(EXPORT ${cs106b}Targets 
    FILE ${CMAKE_CURRENT_BINARY_DIR}/${cs106b}/${cs106b}-targets.cmake
    NAMESPACE Stanford::)

set(ConfigPackageLocation lib/cmake/${cs106b})

configure_package_config_file(cmake/${cs106b}-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/${cs106b}/${cs106b}-config.cmake 
    INSTALL_DESTINATION ${ConfigPackageLocation})

install(EXPORT ${cs106b}Targets 
    FILE ${cs106b}-targets.cmake
    NAMESPACE Stanford::
    DESTINATION
    ${ConfigPackageLocation})

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${cs106b}/cs106b-config.cmake 
    ${CMAKE_CURRENT_BINARY_DIR}/${cs106b}/cs106b-config-version.cmake
    DESTINATION
    ${ConfigPackageLocation})