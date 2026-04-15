# Author: Kang Lin <kl222@126.com>

cmake_minimum_required(VERSION 3.28)
project(qzxing)

# TODO: Modify the version to the correct version
SET(QZXING_VERSION "3.3.0")
# Find Git Version Patch
IF(EXISTS "${CMAKE_SOURCE_DIR}/.git")
    if(NOT GIT)
        SET(GIT $ENV{GIT})
    endif()
    if(NOT GIT)
        FIND_PROGRAM(GIT NAMES git git.exe git.cmd)
    endif()
    IF(GIT)
        EXECUTE_PROCESS(
            WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
            COMMAND ${GIT} describe --tags
            OUTPUT_VARIABLE GIT_VERSION  OUTPUT_STRIP_TRAILING_WHITESPACE
        )
        if(NOT GIT_VERSION)
            EXECUTE_PROCESS(
                WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                COMMAND ${GIT} rev-parse --short HEAD
                OUTPUT_VARIABLE GIT_VERSION OUTPUT_STRIP_TRAILING_WHITESPACE
            )
        endif()
        SET(QZXING_VERSION ${GIT_VERSION})
    ENDIF()
ENDIF()
message("QZXING_VERSION:${QZXING_VERSION}")

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(GenerateExportHeader)
include(CMakeDependentOption)

SET(BIGINT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/zxing/bigint)
SET(WIN32_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/zxing/win32/zxing)
SET(ZXING_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/zxing/zxing)

set(INSTALL_HEADER_FILES
    QZXing.h
    QZXing_global.h
    )

set(HEADER_FILES
    CameraImageWrapper.h
    ImageHandler.h
    ${INSTALL_HEADER_FILES}
    )

set(SOURCE_FILES
    CameraImageWrapper.cpp
    ImageHandler.cpp
    QZXing.cpp
    )

option(QZXING_MULTIMEDIA "Use multimedia" ON)
if(QZXING_MULTIMEDIA)
    list(APPEND QZXING_QT_COMPONENTS Multimedia)
    LIST(APPEND SOURCE_FILES QZXingFilter.cpp QZXingFilter.h)
    LIST(APPEND QZXING_PRIVATE_DEFINES QZXING_MULTIMEDIA)
endif(QZXING_MULTIMEDIA)

cmake_dependent_option(QZXING_USE_QML "Use QML" ON "QZXING_MULTIMEDIA" OFF)
if(QZXING_USE_QML)
    list(APPEND QZXING_QT_COMPONENTS Svg Quick QuickControls2)
    LIST(APPEND SOURCE_FILES QZXingImageProvider.cpp QZXingImageProvider.h)
    list(APPEND QZXING_PUBLIC_DEFINES QZXING_QML)
endif(QZXING_USE_QML)

cmake_dependent_option(QZXING_USE_ENCODER "Enable encoder" ON
                       "QZXING_MULTIMEDIA" OFF)
if(QZXING_USE_ENCODER)
    list(APPEND QZXING_PUBLIC_DEFINES ENABLE_ENCODER_GENERIC)
    list(APPEND QZXING_PRIVATE_DEFINES ENABLE_ENCODER_QR_CODE)
endif(QZXING_USE_ENCODER)

option(QZXING_USE_DECODER_QR_CODE "Use decoder QR code" ON)
if(QZXING_USE_DECODER_QR_CODE)
    list(APPEND QZXING_PRIVATE_DEFINES ENABLE_DECODER_QR_CODE)
endif()

option(QZXING_USE_DECODER_1D_BARCODES "Use decoder 1D barcodes" ON)
if(QZXING_USE_DECODER_1D_BARCODES)
    list(APPEND QZXING_PRIVATE_DEFINES ENABLE_DECODER_1D_BARCODES)
endif()

option(QZXING_USE_DECODER_DATA_MATRIX "Use decoder data matrix" ON)
if(QZXING_USE_DECODER_DATA_MATRIX)
    list(APPEND QZXING_PRIVATE_DEFINES ENABLE_DECODER_DATA_MATRIX)
endif()

option(QZXING_USE_DECODER_AZTEC "Use decoder AZTEC" ON)
if(QZXING_USE_DECODER_AZTEC)
    list(APPEND QZXING_PRIVATE_DEFINES ENABLE_DECODER_AZTEC)
endif()

option(QZXING_USE_DECODER_PDF17 "Use decoder PDF17" ON)
if(QZXING_USE_DECODER_PDF17)
    list(APPEND QZXING_PRIVATE_DEFINES ENABLE_DECODER_PDF17)
endif()

if(QZXING_USE_DECODER_1D_BARCODES)
    list(APPEND QZXING_PRIVATE_DEFINES ENABLE_DECODER_1D_BARCODES)
endif()

# Change Global Definitions depending on how you want to use the library
LIST(APPEND QZXING_PUBLIC_DEFINES DISABLE_LIBRARY_FEATURES)

# Need qt components
list(APPEND QZXING_QT_COMPONENTS Core Gui)
find_package(QT NAMES Qt6 Qt5 COMPONENTS Core)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS ${QZXING_QT_COMPONENTS})
message("QT_VERSION: ${Qt${QT_VERSION_MAJOR}_VERSION}")
FOREACH(_COMPONENT ${QZXING_QT_COMPONENTS})
    list(APPEND QZXING_QT_LIBRARIES Qt${QT_VERSION_MAJOR}::${_COMPONENT})
ENDFOREACH()
get_filename_component(QT_INSTALL_DIR "${Qt${QT_VERSION_MAJOR}_DIR}/../../.." ABSOLUTE)
message("QT_ROOT: ${QT_ROOT}")
message("QT_DIR: ${QT_DIR}")
message("Qt${QT_VERSION_MAJOR}_ROOT: ${Qt${QT_VERSION_MAJOR}_ROOT}")
message("Qt${QT_VERSION_MAJOR}_DIR: ${Qt${QT_VERSION_MAJOR}_DIR}")
message("QT_INSTALL_DIR: ${QT_INSTALL_DIR}")
message("qzxing QZXING_QT_LIBRARIES: ${QZXING_QT_LIBRARIES}")

# Enable qt build tool
SET(CMAKE_AUTOUIC ON)
SET(CMAKE_AUTOMOC ON)
SET(CMAKE_AUTORCC ON)
SET(CMAKE_VERBOSE_MAKEFILE ON)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(GenerateExportHeader)
include(CMakeDependentOption)

add_library(qzxing ${SOURCE_FILES} ${HEADER_FILES})

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

target_link_libraries(qzxing PRIVATE ${QZXING_QT_LIBRARIES})

# Target includes
target_include_directories(qzxing
  PRIVATE
        .
        zxing
        zxing/win32/zxing
        zxing/zxing
        zxing/bigint
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

target_compile_definitions(qzxing PUBLIC ${QZXING_PUBLIC_DEFINES})
target_compile_definitions(qzxing PRIVATE ${QZXING_PRIVATE_DEFINES})

set_target_properties(qzxing PROPERTIES
    PUBLIC_HEADER "${INSTALL_HEADER_FILES}" # Install head files
    )
set_target_properties(qzxing PROPERTIES VERSION ${QZXING_VERSION})

# Install

if(ANDROID)
    # Install target
    INSTALL(TARGETS qzxing
            EXPORT qzxingTarget
            RUNTIME DESTINATION "libs/${ANDROID_ABI}"
                COMPONENT Runtime
            LIBRARY DESTINATION "libs/${ANDROID_ABI}"
                COMPONENT Runtime
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/qzxing
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/qzxing
            )
else()
    # Install target
    if(WIN32)
        INSTALL(TARGETS qzxing
            EXPORT qzxingTarget
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
                COMPONENT Runtime
            LIBRARY DESTINATION "${CMAKE_INSTALL_BINDIR}"
                COMPONENT Runtime
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
            )
    else()
        INSTALL(TARGETS qzxing
            EXPORT qzxingTarget
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
                COMPONENT Runtime
            LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
                COMPONENT Runtime
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
            )
    endif()
endif()

export(TARGETS qzxing
       APPEND FILE ${CMAKE_BINARY_DIR}/qzxingTarget.cmake
)

install(EXPORT qzxingTarget
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/qzxing"
        )
# Install cmake configure files
configure_package_config_file(
    ${CMAKE_SOURCE_DIR}/cmake/qzxingConfig.cmake.in
    ${CMAKE_BINARY_DIR}/${PARA_NAME}Config.cmake.in
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/qzxing"
    )
install(FILES ${CMAKE_BINARY_DIR}/${PARA_NAME}Config.cmake.in
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/qzxing"
    RENAME qzxingConfig.cmake)
write_basic_package_version_file(
    "${CMAKE_BINARY_DIR}/qzxingConfigVersion.cmake"
    VERSION ${QZXING_VERSION}
    COMPATIBILITY AnyNewerVersion)
install(FILES "${CMAKE_BINARY_DIR}/qzxingConfigVersion.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/qzxing")

# Install pkg-config configure file
foreach(ed ${QZXING_PUBLIC_DEFINES})
    set(PC_QZXING_PUBLIC_DEFINES "${PC_QZXING_PUBLIC_DEFINES} -D${ed}")
endforeach()
configure_file(${CMAKE_SOURCE_DIR}/cmake/QZXing.pc.in ${CMAKE_BINARY_DIR}/qzxing.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/qzxing.pc
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
