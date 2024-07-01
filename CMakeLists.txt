cmake_minimum_required(VERSION 3.14)

project(gt-app VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/")

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core Quick)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core Quick Widgets SerialPort Network REQUIRED)
find_package(AHPGT REQUIRED)
find_package(DFU REQUIRED)
find_package(Threads REQUIRED)
find_package(M REQUIRED)

include_directories(${CMAKE_INCLUDE_PATH})
include_directories(${AHP_GT_INCLUDE_DIR})
include_directories(${DFU_INCLUDE_DIR})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/icon.ico ${CMAKE_CURRENT_BINARY_DIR}/icon.ico COPYONLY )
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h )
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/gt-configurator.iss.cmake ${CMAKE_CURRENT_BINARY_DIR}/gt-configurator.iss )

set(PROJECT_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/mainwindow.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/mainwindow.h
        ${CMAKE_CURRENT_SOURCE_DIR}/threads.h
        ${CMAKE_CURRENT_SOURCE_DIR}/resource.qrc
        ${CMAKE_CURRENT_SOURCE_DIR}/app.rc
        ${CMAKE_CURRENT_SOURCE_DIR}/qml.qrc
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(gt-app
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET gt-app APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(gt-app SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(gt-app
          ${PROJECT_SOURCES}
        )
        if(WIN32)
            target_link_libraries(gt-app PRIVATE comctl32)
            set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wl,--subsystem,windows")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wl,--subsystem,windows")
            set(CMAKE_LD_FLAGS "${CMAKE_LD_FLAGS} -Wl,--subsystem,windows -static-libstdc++ -static-libgcc")
            set_target_properties(gt-configurator PROPERTIES WIN32_EXECUTABLE TRUE)
        endif(WIN32)
    endif()
endif()

target_link_libraries(gt-app
  PRIVATE Qt${QT_VERSION_MAJOR}::Core Qt${QT_VERSION_MAJOR}::Quick PRIVATE ${DFU_LIBRARIES} ${AHP_GT_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${M_LIB} Qt5::Widgets Qt5::SerialPort Qt5::Network)

# Qt for iOS sets MACOSX_BUNDLE_GUI_IDENTIFIER automatically since Qt 6.1.
# If you are developing for iOS or macOS you should consider setting an
# explicit, fixed bundle identifier manually though.
if(${QT_VERSION} VERSION_LESS 6.1.0)
  set(BUNDLE_ID_OPTION MACOSX_BUNDLE_GUI_IDENTIFIER com.example.gt-app)
endif()
set_target_properties(gt-app PROPERTIES
    ${BUNDLE_ID_OPTION}
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

include(GNUInstallDirs)
install(TARGETS gt-app
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

if(QT_VERSION_MAJOR EQUAL 6)
    qt_import_qml_plugins(gt-app)
    qt_finalize_executable(gt-app)
endif()
