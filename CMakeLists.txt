cmake_minimum_required(VERSION 3.5)

set(imgui_VERSION 1.91.5)

project(imgui_cmake
        VERSION 1.0
        DESCRIPTION "Dear ImGUI CMake Package"
        LANGUAGES CXX
)
set(CMAKE_CXX_STANDARD 20)

########################################################################################################################
# OPTIONS
########################################################################################################################

set(IMGUI_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/imgui)
set(BACKENDS_DIR ${IMGUI_SOURCE_DIR}/backends)

set(SRC_FILES
        ${IMGUI_SOURCE_DIR}/imgui.cpp
        ${IMGUI_SOURCE_DIR}/imgui_demo.cpp
        ${IMGUI_SOURCE_DIR}/imgui_draw.cpp
        ${IMGUI_SOURCE_DIR}/imgui_widgets.cpp
        ${IMGUI_SOURCE_DIR}/imgui_tables.cpp
        ${IMGUI_SOURCE_DIR}/misc/cpp/imgui_stdlib.cpp
)
set(H_FILES
        ${IMGUI_SOURCE_DIR}/imgui.h
        ${IMGUI_SOURCE_DIR}/imconfig.h
        ${IMGUI_SOURCE_DIR}/imgui_internal.h
        ${IMGUI_SOURCE_DIR}/imstb_rectpack.h
        ${IMGUI_SOURCE_DIR}/imstb_textedit.h
        ${IMGUI_SOURCE_DIR}/imstb_truetype.h
        ${IMGUI_SOURCE_DIR}/misc/cpp/imgui_stdlib.h
)

set(IMGUI_FEATURES "")

if (IMGUI_BACKEND_WIN32)
    list(APPEND SRC_FILES ${BACKENDS_DIR}/imgui_impl_win32.cpp)
    list(APPEND H_FILES ${BACKENDS_DIR}/imgui_impl_win32.h)
    list(APPEND IMGUI_FEATURES "WIN32")
endif ()
if (IMGUI_BACKEND_GLFW)
    list(APPEND SRC_FILES ${BACKENDS_DIR}/imgui_impl_glfw.cpp)
    list(APPEND H_FILES ${BACKENDS_DIR}/imgui_impl_glfw.h)
    list(APPEND IMGUI_FEATURES "GLFW")
endif ()
if (IMGUI_BACKEND_VULKAN)
    list(APPEND SRC_FILES ${BACKENDS_DIR}/imgui_impl_vulkan.cpp)
    list(APPEND H_FILES ${BACKENDS_DIR}/imgui_impl_vulkan.h)
    list(APPEND IMGUI_FEATURES "VULKAN")
endif ()
if (IMGUI_BACKEND_OPENGL3)
    list(APPEND SRC_FILES ${BACKENDS_DIR}/imgui_impl_opengl3.cpp)
    list(APPEND H_FILES ${BACKENDS_DIR}/imgui_impl_opengl3.h)
    list(APPEND IMGUI_FEATURES "OPENGL3")
endif ()

message(STATUS "Using ImGUI features: ${IMGUI_FEATURES}")

file(GLOB FONTS_FILES ${IMGUI_SOURCE_DIR}/misc/fonts/*.ttf)


########################################################################################################################
# TARGETS
########################################################################################################################

set(ALL_FILES
        ${H_FILES}
        ${SRC_FILES}
        ${FONTS_FILES}
)

add_library(imgui STATIC ${ALL_FILES})
add_library(imgui::imgui ALIAS imgui)

target_include_directories(imgui PUBLIC
        $<BUILD_INTERFACE:${IMGUI_SOURCE_DIR}>
        $<BUILD_INTERFACE:${BACKENDS_DIR}>
        $<INSTALL_INTERFACE:include/imgui>
)


########################################################################################################################
# INSTALLATION
########################################################################################################################

set(CMAKE_PACKAGE_NAME "imgui")
string(TOLOWER ${CMAKE_PACKAGE_NAME} CMAKE_PACKAGE_NAME_LOWER)
set(TARGETS_EXPORT_NAME "${CMAKE_PACKAGE_NAME}Targets")

install(TARGETS imgui
        EXPORT ${TARGETS_EXPORT_NAME}
)
install(FILES ${H_FILES}
        DESTINATION include/imgui
)
install(FILES ${FONTS_FILES}
        DESTINATION misc/fonts
)

########################################################################################################################
# EXPORT
########################################################################################################################
include(CMakePackageConfigHelpers)

set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(CONFIG_INSTALL_DIR "lib/cmake/${CMAKE_PACKAGE_NAME}")
set(CONFIG_FILE "${GENERATED_DIR}/${CMAKE_PACKAGE_NAME}Config.cmake")
set(VERSION_FILE "${GENERATED_DIR}/${CMAKE_PACKAGE_NAME}ConfigVersion.cmake")

file(WRITE "${CONFIG_FILE}.in"
        "@PACKAGE_INIT@
        include(\"\${CMAKE_CURRENT_LIST_DIR}/${TARGETS_EXPORT_NAME}.cmake\")
        ")

write_basic_package_version_file("${VERSION_FILE}" VERSION ${imgui_VERSION} COMPATIBILITY SameMajorVersion)
configure_package_config_file("${CONFIG_FILE}.in" "${CONFIG_FILE}"
        INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
        )

install(EXPORT ${TARGETS_EXPORT_NAME}
        NAMESPACE imgui::
        DESTINATION "${CONFIG_INSTALL_DIR}"
)
install(FILES "${CONFIG_FILE}" "${VERSION_FILE}"
        DESTINATION "${CONFIG_INSTALL_DIR}"
)