cmake_minimum_required(VERSION 3.5)
project(ksnip LANGUAGES CXX VERSION 1.8.0)

if (DEFINED VERSION_SUFIX AND NOT "${VERSION_SUFIX}" STREQUAL "")
    set(KSNIP_VERSION_SUFIX "-${VERSION_SUFIX}")
endif()

set(KSNIP_VERSION "${PROJECT_VERSION}${KSNIP_VERSION_SUFIX}")

include(GNUInstallDirs)

if (WIN32)
	set(KSNIP_LANG_INSTALL_DIR "translations")
	set(KIMAGEANNOTATOR_LANG_INSTALL_DIR "translations")
elseif (APPLE)
	set(KSNIP_LANG_INSTALL_DIR "../Resources")
	set(KIMAGEANNOTATOR_LANG_INSTALL_DIR "../Resources")
elseif (UNIX)
	set(KSNIP_LANG_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATAROOTDIR}/ksnip/translations")
	set(KIMAGEANNOTATOR_LANG_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATAROOTDIR}/kImageAnnotator/translations")
endif ()

configure_file(src/BuildConfig.h.in ${CMAKE_CURRENT_BINARY_DIR}/src/BuildConfig.h)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(BUILD_TESTS "Build Unit Tests" OFF)

set(QT_MIN_VERSION "5.9.4")

find_package(Qt5 ${QT_MIN_VERSION} REQUIRED Widgets Network Xml PrintSupport DBus Svg)

if (WIN32)
    find_package(Qt5 ${QT_MIN_VERSION} REQUIRED WinExtras)
elseif (UNIX AND NOT APPLE)
    find_package(Qt5 ${QT_MIN_VERSION} REQUIRED X11Extras)

    # Without ECM we're unable to load XCB
    find_package(ECM REQUIRED NO_MODULE)
    set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

    find_package(X11 REQUIRED)
    find_package(XCB COMPONENTS XFIXES)
endif ()

set(KIMAGEANNOTATOR_MIN_VERSION "0.4.0")
find_package(kImageAnnotator ${KIMAGEANNOTATOR_MIN_VERSION} REQUIRED)

set(BASEPATH "${CMAKE_SOURCE_DIR}")
include_directories("${BASEPATH}")

add_subdirectory(src)
add_subdirectory(translations)
add_subdirectory(desktop)

if (BUILD_TESTS)
	configure_file(src/BuildConfig.h.in ${CMAKE_CURRENT_BINARY_DIR}/tests/BuildConfig.h)

	find_package(Qt5 ${QT_MIN_VERSION} REQUIRED Test)
	enable_testing()
	add_subdirectory(tests)

endif (BUILD_TESTS)
