cmake_minimum_required(VERSION 3.5)
project("PolkitQt-1")

set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

IF("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR
        "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    add_compile_options("$<$<CONFIG:DEBUG>:-Wall;-Wextra;-pedantic>")
ENDIF()

set(CMAKE_AUTOMOC TRUE)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules ${CMAKE_MODULE_PATH})

# Used to create CMake config files
include(CMakePackageConfigHelpers)

# Used to set installation paths
include(GNUInstallDirs)

include(FeatureSummary)

message(STATUS "Using Qt 4")

set(QT_MIN_VERSION "4.4.0")

find_package(Qt4 REQUIRED)

include (${QT_USE_FILE})
include (InstallSettings)

set(POLKITQT-1_PCNAME "polkit-qt-1")
set(POLKITQT-1_CORE_PCNAME "polkit-qt-core-1")
set(POLKITQT-1_GUI_PCNAME "polkit-qt-gui-1")
set(POLKITQT-1_AGENT_PCNAME "polkit-qt-agent-1")
set(POLKITQT-1_CAMEL_NAME "PolkitQt-1")
set(POLKITQT-1_EXAMPLE "polkit-example")
set(POLKITQT-1_EXAMPLE_HELPER "polkit-example-helper")
set(POLKITQT-1_AGENT_EXAMPLE "polkit-agent-example")
set(POLKITQT-1_INCLUDE_PATH "polkit-qt-1")


find_package(PkgConfig REQUIRED)

pkg_check_modules(POLKIT_GOBJECT polkit-gobject-1 REQUIRED IMPORTED_TARGET)
pkg_check_modules(POLKIT_AGENT polkit-agent-1 REQUIRED IMPORTED_TARGET)
pkg_check_modules(GLIB2 glib-2.0>=2.36 REQUIRED IMPORTED_TARGET)
pkg_check_modules(GOBJECT gobject-2.0 REQUIRED IMPORTED_TARGET)

add_definitions(-DQT_NO_KEYWORDS)

include (CheckFunctionExists)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/core
    ${CMAKE_CURRENT_BINARY_DIR}
)

include_directories(
    ${QT_QTXML_INCLUDE_DIR}
)

# Check for older polkit
set(CMAKE_REQUIRED_INCLUDES ${POLKIT_GOBJECT_INCLUDE_DIRS} ${POLKIT_AGENT_INCLUDE_DIRS})
set(CMAKE_REQUIRED_LIBRARIES ${POLKIT_GOBJECT_LIBRARIES} ${POLKIT_AGENT_LIBRARIES})
check_function_exists(polkit_agent_listener_register HAVE_POLKIT_AGENT_LISTENER_REGISTER)
check_function_exists(polkit_authority_get_sync HAVE_POLKIT_AUTHORITY_GET_SYNC)
check_function_exists(polkit_system_bus_name_get_user_sync HAVE_POLKIT_SYSTEM_BUS_NAME_GET_USER_SYNC)

if (NOT HAVE_POLKIT_AGENT_LISTENER_REGISTER OR NOT HAVE_POLKIT_AUTHORITY_GET_SYNC)
    message(STATUS "You have an older polkit-1 version: Polkit-Qt-1 will be built in compatibility mode")
    add_definitions(-DPOLKIT_QT_1_COMPATIBILITY_MODE)
endif (NOT HAVE_POLKIT_AGENT_LISTENER_REGISTER OR NOT HAVE_POLKIT_AUTHORITY_GET_SYNC)

option(USE_COMMON_CMAKE_PACKAGE_CONFIG_DIR "Prefer to install the <package>Config.cmake files to lib/cmake/<package> instead of lib/<package>/cmake" TRUE)

set(POLKITQT-1_VERSION_MAJOR "0")
set(POLKITQT-1_VERSION_MINOR "112")
set(POLKITQT-1_VERSION_PATCH "0")
set(POLKITQT-1_VERSION_STRING
    "${POLKITQT-1_VERSION_MAJOR}.${POLKITQT-1_VERSION_MINOR}.${POLKITQT-1_VERSION_PATCH}")

# The SONUMBER of the polkit-qt-1 libraries
set(POLKITQT-1_ABI_VERSION 1)
# The version number of the polkit-qt-1 libraries
set(POLKITQT-1_LIBRARY_VERSION "${POLKITQT-1_ABI_VERSION}.${POLKITQT-1_VERSION_MINOR}.${POLKITQT-1_VERSION_PATCH}")
set(POLKITQT-1_LIB_NAMESPACE ${POLKITQT-1_CAMEL_NAME})

configure_file(polkitqt1-version.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/polkitqt1-version.h)
configure_file(polkitqt1-config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/polkitqt1-config.h)

install(FILES
    gui/polkitqt1-gui-action.h
    gui/polkitqt1-gui-actionbutton.h
    gui/polkitqt1-gui-actionbuttons.h

    core/polkitqt1-authority.h
    core/polkitqt1-details.h
    core/polkitqt1-identity.h
    core/polkitqt1-subject.h
    core/polkitqt1-temporaryauthorization.h
    core/polkitqt1-actiondescription.h

    agent/polkitqt1-agent-listener.h
    agent/polkitqt1-agent-session.h

    ${CMAKE_CURRENT_BINARY_DIR}/polkitqt1-version.h
    polkitqt1-export.h

    DESTINATION
    ${CMAKE_INSTALL_INCLUDEDIR}/${POLKITQT-1_INCLUDE_PATH} COMPONENT Devel)

install(FILES
    includes/PolkitQt1/Authority
    includes/PolkitQt1/Details
    includes/PolkitQt1/Identity
    includes/PolkitQt1/Subject
    includes/PolkitQt1/TemporaryAuthorization
    includes/PolkitQt1/ActionDescription
    DESTINATION
    ${CMAKE_INSTALL_INCLUDEDIR}/${POLKITQT-1_INCLUDE_PATH}/PolkitQt1 COMPONENT Devel)

install(FILES
    includes/PolkitQt1/Gui/Action
    includes/PolkitQt1/Gui/ActionButton
    includes/PolkitQt1/Gui/ActionButtons
    DESTINATION
    ${CMAKE_INSTALL_INCLUDEDIR}/${POLKITQT-1_INCLUDE_PATH}/PolkitQt1/Gui COMPONENT Devel)

install(FILES
    includes/PolkitQt1/Agent/Listener
    includes/PolkitQt1/Agent/Session
    DESTINATION
    ${CMAKE_INSTALL_INCLUDEDIR}/${POLKITQT-1_INCLUDE_PATH}/PolkitQt1/Agent COMPONENT Devel)

if(NOT WIN32)
    # Pkgconfig
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${POLKITQT-1_PCNAME}.pc.cmake ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_PCNAME}.pc @ONLY)
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_PCNAME}.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig )
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${POLKITQT-1_CORE_PCNAME}.pc.cmake ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_CORE_PCNAME}.pc
                   @ONLY)
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_CORE_PCNAME}.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig )
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${POLKITQT-1_GUI_PCNAME}.pc.cmake ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_GUI_PCNAME}.pc @ONLY)
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_GUI_PCNAME}.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig )
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${POLKITQT-1_AGENT_PCNAME}.pc.cmake ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_AGENT_PCNAME}.pc
                   @ONLY)
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${POLKITQT-1_AGENT_PCNAME}.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig )
endif(NOT WIN32)


if(USE_COMMON_CMAKE_PACKAGE_CONFIG_DIR)
   set(_PolkitQt-1Config_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/${POLKITQT-1_CAMEL_NAME})
else(USE_COMMON_CMAKE_PACKAGE_CONFIG_DIR)
   set(_PolkitQt-1Config_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/${POLKITQT-1_CAMEL_NAME}/cmake)
endif(USE_COMMON_CMAKE_PACKAGE_CONFIG_DIR)


# Create the CMake Config files
configure_package_config_file(PolkitQt-1Config.cmake.in
                              ${CMAKE_BINARY_DIR}/${POLKITQT-1_CAMEL_NAME}Config.cmake
                              INSTALL_DESTINATION ${_PolkitQt-1Config_INSTALL_DIR}
                              PATH_VARS CMAKE_INSTALL_LIBDIR CMAKE_INSTALL_INCLUDEDIR
)

write_basic_package_version_file(${CMAKE_BINARY_DIR}/${POLKITQT-1_CAMEL_NAME}ConfigVersion.cmake
                                 VERSION ${POLKITQT-1_VERSION_STRING}
                                 COMPATIBILITY SameMajorVersion
                                )

install(FILES ${CMAKE_BINARY_DIR}/${POLKITQT-1_CAMEL_NAME}Config.cmake
              ${CMAKE_BINARY_DIR}/${POLKITQT-1_CAMEL_NAME}ConfigVersion.cmake
        DESTINATION ${_PolkitQt-1Config_INSTALL_DIR}
       )

install(EXPORT ${POLKITQT-1_CAMEL_NAME}Export FILE ${POLKITQT-1_CAMEL_NAME}Targets.cmake
        DESTINATION ${_PolkitQt-1Config_INSTALL_DIR}
        NAMESPACE ${POLKITQT-1_LIB_NAMESPACE}::)

option(BUILD_EXAMPLES "Builds a set of examples for polkit-qt-1" OFF)
if (BUILD_EXAMPLES)
    add_subdirectory(examples)
endif (BUILD_EXAMPLES)

add_subdirectory(cmake)

option(BUILD_TEST "Builds unit tests for polkit-qt-1" OFF)
if (BUILD_TEST)
    add_subdirectory(test)
endif (BUILD_TEST)

add_subdirectory(core)
add_subdirectory(gui)
add_subdirectory(agent)

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)

# Always last!
include (PolkitQt-1Dist)
