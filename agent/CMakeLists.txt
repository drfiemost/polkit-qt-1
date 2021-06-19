set(polkit_qt_agent_SRCS
    polkitqt1-agent-session.cpp
    polkitqt1-agent-listener.cpp
    listeneradapter.cpp
    polkitqtlistener.cpp
)
add_library(${POLKITQT-1_AGENT_PCNAME} SHARED ${polkit_qt_agent_SRCS})

add_library(${POLKITQT-1_CAMEL_NAME}::Agent ALIAS ${POLKITQT-1_AGENT_PCNAME})

target_link_libraries(${POLKITQT-1_AGENT_PCNAME}
    PUBLIC
    ${QT_QTCORE_LIBRARY}
    ${POLKITQT-1_CORE_PCNAME}
    PRIVATE
    ${QT_QTCORE_LIBRARY}
    PkgConfig::POLKIT_AGENT
    PkgConfig::GOBJECT
)

set_target_properties(${POLKITQT-1_AGENT_PCNAME} PROPERTIES VERSION ${POLKITQT-1_LIBRARY_VERSION}
                                SOVERSION ${POLKITQT-1_ABI_VERSION}
                                DEFINE_SYMBOL MAKE_POLKITQT1_LIB
                                EXPORT_NAME Agent)

target_include_directories(${POLKITQT-1_AGENT_PCNAME} INTERFACE $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${POLKITQT-1_INCLUDE_PATH}>)


install(TARGETS ${POLKITQT-1_AGENT_PCNAME} EXPORT ${POLKITQT-1_CAMEL_NAME}Export ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                               LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                               RUNTIME DESTINATION bin)
