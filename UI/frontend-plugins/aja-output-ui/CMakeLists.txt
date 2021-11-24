project(aja-output-ui)

if(DISABLE_AJA)
	message(STATUS "aja-output-ui plugin disabled")
	return()
endif()

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
	find_package(LibAJANTV2)
	if (NOT LIBAJANTV2_FOUND)
		message(STATUS "aja-output-ui plugin disabled (deps not found)")
		return()
	else()
		message("aja-output-ui includes: ${LIBAJANTV2_INCLUDE_DIRS}")
		message("aja-output-ui libs: ${LIBAJANTV2_LIBRARIES}")
	endif()
else()
	message(STATUS "aja-output-ui disabled (32-bit not supported)")
	return()
endif()

if(APPLE)
	find_library(COCOA Cocoa)
	include_directories(${COCOA})
endif()

if(UNIX AND NOT APPLE)
	find_package(X11 REQUIRED)
	link_libraries(${X11_LIBRARIES})
	include_directories(${X11_INCLUDE_DIR})

	find_package(Qt5X11Extras REQUIRED)
endif()

set(aja-output-ui_HEADERS
	${aja-output-ui_HEADERS}
	../../qt-wrappers.hpp
	../../properties-view.hpp
	../../properties-view.moc.hpp
	../../vertical-scroll-area.hpp
	../../double-slider.hpp
	../../slider-ignorewheel.hpp
	../../combobox-ignorewheel.hpp
	../../spinbox-ignorewheel.hpp
	AJAOutputUI.h
	aja-ui-main.h
	../../../plugins/aja/aja-ui-props.hpp
	../../../plugins/aja/aja-enums.hpp)

set(aja-output-ui_SOURCES
	${aja-output-ui_SOURCES}
	../../qt-wrappers.cpp
	../../properties-view.cpp
	../../vertical-scroll-area.cpp
	../../double-slider.cpp
	../../slider-ignorewheel.cpp
	../../combobox-ignorewheel.cpp
	../../spinbox-ignorewheel.cpp
	AJAOutputUI.cpp
	aja-ui-main.cpp)

set(aja-output-ui_UI
	${aja-output-ui_UI}
	forms/output.ui)

if(WIN32)
	set(MODULE_DESCRIPTION "OBS AJA Output UI")
	configure_file(${CMAKE_SOURCE_DIR}/cmake/winrc/obs-module.rc.in aja-output-ui.rc)
	list(APPEND aja-output-ui_SOURCES
		aja-output-ui.rc)
endif()

if(APPLE)
	set(aja-output-ui_PLATFORM_LIBS
		${COCOA})
endif()

qt5_wrap_ui(aja-output-ui_UI_HEADERS
	${aja-output-ui_UI})

add_library(aja-output-ui MODULE
	${aja-output-ui_HEADERS}
	${aja-output-ui_SOURCES}
	${aja-output-ui_UI_HEADERS})

if (APPLE)
	target_compile_definitions(aja-output-ui PUBLIC
		AJAMac
		AJA_MAC)
elseif(WIN32)
	target_compile_definitions(aja-output-ui PUBLIC
		AJA_WINDOWS
		_WINDOWS
		WIN32
		MSWindows)
elseif(UNIX AND NOT APPLE)
	target_compile_definitions(aja-output-ui PUBLIC
		AJA_LINUX
		AJALinux)
endif()

target_include_directories(aja-output-ui PUBLIC
	${LIBAJANTV2_INCLUDE_DIRS})

target_link_libraries(aja-output-ui
	${frontend-tools_PLATFORM_LIBS}
	obs-frontend-api
	Qt5::Widgets
	libobs)

set_target_properties(aja-output-ui PROPERTIES FOLDER "frontend")

install_obs_plugin_with_data(aja-output-ui data)