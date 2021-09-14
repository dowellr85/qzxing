cmake_minimum_required(VERSION 3.2)
project(QZXing)

SET(BIGINT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/zxing/bigint)
SET(WIN32_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/zxing/win32/zxing)
SET(ZXING_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/zxing/zxing)

set(CMAKE_AUTOMOC ON)
set(CMAKE_WARN_ON)

set(SOURCE_FILES
    CameraImageWrapper.cpp
    CameraImageWrapper.h
    ImageHandler.cpp
    ImageHandler.h
    QZXing.cpp
    )

set(INSTALL_HEADER_FILES
    QZXing.h
    QZXing_global.h
    )

set(QT_COMPONENTS Core Gui)

add_library(${PROJECT_NAME} ${SOURCE_FILES} ${INSTALL_HEADER_FILES})

option(QZXING_MULTIMEDIA "Include Multimedia Support" OFF)
if(QZXING_MULTIMEDIA)
    target_sources(${PROJECT_NAME} PRIVATE QZXingFilter.cpp QZXingFilter.h)
	target_compile_definitions(${PROJECT_NAME} PRIVATE QZXING_MULTIMEDIA)

    LIST(APPEND INSTALL_HEADER_FILES QZXingFilter.h)
    LIST(APPEND QT_COMPONENTS Multimedia)

    SET(QZXING_USE_QML ON)
endif(QZXING_MULTIMEDIA)

option(QZXING_USE_QML "Include QML Support" OFF)
if(QZXING_USE_QML)
    target_sources(${PROJECT_NAME} PRIVATE QZXingImageProvider.cpp QZXingImageProvider.h)
	target_compile_definitions(${PROJECT_NAME} PRIVATE QZXING_QML)
	
    LIST(APPEND INSTALL_HEADER_FILES QZXingImageProvider.h)
    LIST(APPEND QT_COMPONENTS Svg Quick QuickControls2)
endif(QZXING_USE_QML)

option(QZXING_USE_ENCODER "Include Encoder Support" OFF)
if(QZXING_USE_ENCODER)
    target_compile_definitions(${PROJECT_NAME} PRIVATE ENABLE_ENCODER_GENERIC)
endif(QZXING_USE_ENCODER)

if(WIN32)
    add_subdirectory(zxing/win32)
    if(MSVC)
        add_definitions(-D__STDC_LIMIT_MACROS)
    endif(MSVC)

    add_definitions(-DNO_ICONV)
endif(WIN32)

if(ANDROID)
    add_definitions(-DNO_ICONV)
endif()

if(VS_WINRT_COMPONENT)
    add_definitions(-DNO_ICONV)
endif(VS_WINRT_COMPONENT)

add_subdirectory(zxing/bigint)
add_subdirectory(zxing/zxing)

find_package(Qt5 COMPONENTS ${QT_COMPONENTS} REQUIRED)
foreach(COMPONENT ${QT_COMPONENTS})
    LIST(APPEND QT_LIBRARIES Qt5::${COMPONENT})
endforeach()
target_link_libraries(${PROJECT_NAME} ${QT_LIBRARIES})

# Change Global Definitions depending on how you want to use the library
target_compile_definitions(${PROJECT_NAME} PUBLIC DISABLE_LIBRARY_FEATURES)

# Target includes
target_include_directories(${PROJECT_NAME}
  PRIVATE
        .
        zxing
        zxing/win32/zxing
        zxing/zxing
        zxing/bigint
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

# Install files
install(TARGETS ${PROJECT_NAME} EXPORT qzxing-config DESTINATION lib)
install(FILES ${INSTALL_HEADER_FILES} DESTINATION include/qzxing)
install(EXPORT qzxing-config DESTINATION lib/cmake/qzxing)
