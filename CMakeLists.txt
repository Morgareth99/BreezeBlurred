
project(breezeblurred)
set(PROJECT_VERSION "0.1")
set(PROJECT_VERSION_MAJOR 0)

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

include(WriteBasicConfigVersionFile)
include(FeatureSummary)

find_package(ECM 0.0.9 REQUIRED NO_MODULE)

set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR} ${CMAKE_SOURCE_DIR}/cmake)
include(ECMInstallIcons)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(GenerateExportHeader)
# include(GtkUpdateIconCache)

find_package(KDecoration2 REQUIRED)

#find_package(FFTW REQUIRED)


# old stuff
add_definitions(-DTRANSLATION_DOMAIN="breeze_kwin_deco")

find_package(KF5 REQUIRED COMPONENTS CoreAddons GuiAddons ConfigWidgets WindowSystem I18n)
find_package(Qt5 CONFIG REQUIRED COMPONENTS DBus)

### XCB
find_package(XCB COMPONENTS XCB)
set_package_properties(XCB PROPERTIES
  DESCRIPTION "X protocol C-language Binding"
  URL "http://xcb.freedesktop.org"
  TYPE OPTIONAL
  PURPOSE "Required to pass style properties to native Windows on X11 Platform"
)

if(UNIX AND NOT APPLE)

  set(BREEZE_HAVE_X11 ${XCB_XCB_FOUND})
  if (XCB_XCB_FOUND)
    find_package(Qt5 REQUIRED CONFIG COMPONENTS X11Extras)
  endif()

else()

  set(BREEZE_HAVE_X11 FALSE)

endif()

################# configuration #################
configure_file(config-breeze.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-breeze.h )

################# includes #################
include_directories(${CMAKE_SOURCE_DIR}/libbreezecommon)
include_directories(${CMAKE_BINARY_DIR}/libbreezecommon)

################# newt target #################
### plugin classes
# set(breezedecoration_SRCS
set(breezeblurred_SRCS
    breezebutton.cpp
    breezedecoration.cpp
    breezeexceptionlist.cpp
    breezesettingsprovider.cpp
    breezesizegrip.cpp)

# kconfig_add_kcfg_files(breezedecoration_SRCS breezesettings.kcfgc)
kconfig_add_kcfg_files(breezeblurred_SRCS breezesettings.kcfgc)

### config classes
### they are kept separately because they might move in a separate library in the future
# set(breezedecoration_config_SRCS
set(breezeblurred_config_SRCS
    config/breezeconfigwidget.cpp
    config/breezedetectwidget.cpp
    config/breezeexceptiondialog.cpp
    config/breezeexceptionlistwidget.cpp
    config/breezeexceptionmodel.cpp
    config/breezeitemmodel.cpp
)

# set(breezedecoration_config_PART_FORMS
set(breezeblurred_config_PART_FORMS
   config/ui/breezeconfigurationui.ui
   config/ui/breezedetectwidget.ui
   config/ui/breezeexceptiondialog.ui
   config/ui/breezeexceptionlistwidget.ui
)

set(breezecommon_LIB_SRCS
    breezeboxshadowrenderer.cpp
)

find_path(FFTW_INCLUDES fftw3.h)

find_library(FFTW_LIBRARIES NAMES fftw3)

include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(FFTW DEFAULT_MSG
                                  FFTW_INCLUDES FFTW_LIBRARIES)

mark_as_advanced(FFTW_INCLUDES FFTW_LIBRARIES)

# ki18n_wrap_ui(breezedecoration_config_PART_FORMS_HEADERS ${breezedecoration_config_PART_FORMS})
ki18n_wrap_ui(breezeblurred_config_PART_FORMS_HEADERS ${breezeblurred_config_PART_FORMS})

### build library
# add_library(breezedecoration MODULE
    # ${breezedecoration_SRCS}
    # ${breezedecoration_config_SRCS}
    # ${breezedecoration_config_PART_FORMS_HEADERS})
add_library(breezeblurred MODULE
    ${breezeblurred_SRCS}
    ${breezeblurred_config_SRCS}
    ${breezeblurred_config_PART_FORMS_HEADERS}
    ${breezecommon_LIB_SRCS})
    
generate_export_header(breezeblurred
        BASE_NAME breezecommon
        EXPORT_FILE_NAME breezecommon_export.h)

# target_link_libraries(breezedecoration
target_link_libraries(breezeblurred
    PUBLIC
        Qt5::Core
        Qt5::Gui
        Qt5::DBus
    PRIVATE
        ${FFTW_LIBRARIES}
        KDecoration2::KDecoration
        KF5::ConfigCore
        KF5::CoreAddons
        KF5::ConfigWidgets
        KF5::GuiAddons
        KF5::I18n
        KF5::WindowSystem)

if(BREEZE_HAVE_X11)
  # target_link_libraries(breezedecoration
  target_link_libraries(breezeblurred
    PUBLIC
      Qt5::X11Extras
      XCB::XCB)
endif()


install(TARGETS breezeblurred DESTINATION ${PLUGIN_INSTALL_DIR}/org.kde.kdecoration2)
install(FILES config/breezeblurredconfig.desktop DESTINATION  ${SERVICES_INSTALL_DIR})
# install(TARGETS breezedecoration DESTINATION ${PLUGIN_INSTALL_DIR}/org.kde.kdecoration2)
# install(FILES config/breezedecorationconfig.desktop DESTINATION  ${SERVICES_INSTALL_DIR})
