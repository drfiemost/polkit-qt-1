set(polkit_qt_gui_SRCS
    polkitqt1-gui-action.cpp
    polkitqt1-gui-actionbutton.cpp
    polkitqt1-gui-actionbuttons.cpp
)

add_library(${POLKITQT-1_GUI_PCNAME} SHARED ${polkit_qt_gui_SRCS})

add_library(${POLKITQT-1_CAMEL_NAME}::Gui ALIAS ${POLKITQT-1_GUI_PCNAME})

target_link_libraries(${POLKITQT-1_GUI_PCNAME}
    PUBLIC
    ${POLKITQT-1_CORE_PCNAME}
    ${QT_QTGUI_LIBRARY}
    PRIVATE
    ${QT_QTCORE_LIBRARY}
    ${QT_QTDBUS_LIBRARY}
)

set_target_properties(${POLKITQT-1_GUI_PCNAME} PROPERTIES VERSION ${POLKITQT-1_LIBRARY_VERSION}
                                SOVERSION ${POLKITQT-1_ABI_VERSION}
                                DEFINE_SYMBOL MAKE_POLKITQT1_LIB
                                EXPORT_NAME Gui)

target_include_directories(${POLKITQT-1_GUI_PCNAME} INTERFACE $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${POLKITQT-1_INCLUDE_PATH}>)

install(TARGETS ${POLKITQT-1_GUI_PCNAME} EXPORT ${POLKITQT-1_CAMEL_NAME}Export ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                               LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                               RUNTIME DESTINATION bin)
