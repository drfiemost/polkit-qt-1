enable_testing()

add_executable(polkit-qt-test
    test.cpp
)

target_link_libraries(polkit-qt-test
    Qt4::QtCore
    Qt4::QtDBus
    Qt4::QtTest
    Qt4::QtGui
    ${POLKITQT-1_CORE_PCNAME}
)

add_test(BaseTest ${CMAKE_CURRENT_BINARY_DIR}/polkit-qt-test)
