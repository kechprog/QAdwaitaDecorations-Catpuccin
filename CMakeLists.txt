cmake_minimum_required(VERSION 3.16)

project(QAdwaitaDecorations
    VERSION 0.1.5
    DESCRIPTION "Qt Wayland decoration plugin using libadwaita style"
    LANGUAGES CXX C)

option(USE_QT6 "Use Qt6 instead of Qt5" OFF)

set(CMAKE_AUTOMOC ON)

if (USE_QT6)
    set(QT_MIN_VERSION "6.5.0")
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
else()
    set(QT_MIN_VERSION "5.15.2")
    set(CMAKE_CXX_STANDARD 14)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

include(GNUInstallDirs)
include(FeatureSummary)

if (USE_QT6)
    find_package(QT NAMES Qt6 COMPONENTS Core Gui Svg Wayland Widgets REQUIRED)
else()
    find_package(QT NAMES Qt5 COMPONENTS Core Gui Svg Wayland Widgets REQUIRED)
endif()

find_package(Qt${QT_VERSION_MAJOR} ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS
    Core
    Gui
    Svg
    WaylandClient
    Widgets
)

find_package(Qt${QT_VERSION_MAJOR}Gui ${QT_MIN_VERSION} CONFIG REQUIRED Private)
if (NOT USE_QT6)
    find_package(Qt${QT_VERSION_MAJOR}ThemeSupport REQUIRED)
endif()

# NOTE: I don't know how to do this only in case of qt_config(xkbcommon).
# We would miss an include in QWaylandDisplay header file.
if (NOT USE_QT6)
    find_package(Qt${QT_VERSION_MAJOR}XkbCommonSupport ${QT_MIN_VERSION})
endif()

if (HAS_QT6_SUPPORT OR USE_QT6)
    message(STATUS "Enabling Qt Wayland decoration shadows support")
    message(STATUS "NOTE: This support requires changes in Qt Wayland from Qt 6.2")
    add_definitions(-DHAS_QT6_SUPPORT)
endif()

# Update the plugin installation path
if (NOT QT_PLUGINS_DIR)
    if (NOT USE_QT6)
        get_target_property(REAL_QMAKE_EXECUTABLE ${Qt5Core_QMAKE_EXECUTABLE}
                            IMPORTED_LOCATION)
        execute_process(COMMAND "${REAL_QMAKE_EXECUTABLE}" -query QT_INSTALL_PLUGINS
                        OUTPUT_VARIABLE QT_PLUGINS_DIR
                        ERROR_QUIET OUTPUT_STRIP_TRAILING_WHITESPACE)
    else()
        set(QT_PLUGINS_DIR "/usr/lib64/qt6/plugins")
    endif()
endif()

# Set flags for different build types
if (CMAKE_BUILD_TYPE STREQUAL "Release")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -flto")
    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -flto")
    set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} -flto")
    set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} -flto")
elseif (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g")
endif()

add_subdirectory(src)

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
