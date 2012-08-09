cmake_minimum_required(VERSION 2.8)

################ ADD CMAKE MODULES ##################
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules")

################### SETUP PROJECT ###################
project(openshot-audio)
set(PROJECT_VERSION 0.1.2)
set(SO_VERSION 1)
MESSAGE("--------------------------------------------------------------")
MESSAGE("---- Generating build files for ${PROJECT_NAME} (${PROJECT_VERSION})")
MESSAGE("--------------------------------------------------------------")


# Enable stack-unwinding support in c objects on gcc-based platforms.
# Failing to do so will cause your program to be terminated when a png
# or a jpeg exception is thrown on linux or macosx.
IF (CMAKE_COMPILER_IS_GNUCC)
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fexceptions")
ENDIF(CMAKE_COMPILER_IS_GNUCC)


IF (WIN32)
	SET_PROPERTY(GLOBAL PROPERTY JUCE_WINDOWS "JUCE_WINDOWS")
	SET_PROPERTY(GLOBAL PROPERTY JUCE_MINGW "JUCE_MINGW")
	ADD_DEFINITIONS(-DDONT_AUTOLINK_TO_JUCE_LIBRARY)

	SET(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -march=i686")

	SET(JUCE_PLATFORM_SPECIFIC_LIBRARIES
		advapi32.lib
		comdlg32.lib
		gdi32.lib
		GlU32.lib
		Imm32.dll
		kernel32.lib
		ole32.lib
		OpenGL32.lib
		rpcrt4.lib
		shell32.lib
		Shlwapi.dll
		user32.lib
		vfw32.lib
		version.lib
		winmm.lib
		wininet.lib
		ws2_32.lib
	)
ELSE (WIN32)
	IF   (UNIX)
		IF   (APPLE)
			SET_PROPERTY(GLOBAL PROPERTY JUCE_MAC "JUCE_MAC")
			SET(JUCE_PLATFORM_SPECIFIC_LIBRARIES "-framework Carbon -framework Cocoa -framework CoreFoundation -framework CoreAudio -framework CoreMidi -framework IOKit -framework AGL -lobjc")
			SET(JUCE_PLATFORM_SPECIFIC_LIBRARIES "-framework Carbon -framework Cocoa -framework CoreFoundation -framework CoreAudio -framework CoreMidi -framework IOKit -framework AGL -lobjc")
		ELSE (APPLE)
			SET_PROPERTY(GLOBAL PROPERTY JUCE_LINUX "JUCE_LINUX")
			FIND_PACKAGE(X11 REQUIRED)
			INCLUDE_DIRECTORIES(${X11_INCLUDE_DIR})
			INCLUDE_DIRECTORIES(/usr/include/freetype2)
			ADD_DEFINITIONS(-DLINUX)
			SET(JUCE_PLATFORM_SPECIFIC_LIBRARIES ${X11_LIBRARIES} asound freetype Xinerama)
		ENDIF(APPLE)
	ENDIF(UNIX)
ENDIF(WIN32)

# Include header directories
include_directories("JuceLibraryCode")

# List of modules to build
FILE(GLOB JUCE_SOURCES
	JuceLibraryCode/modules/juce_core/juce_core.cpp
	JuceLibraryCode/modules/juce_data_structures/juce_data_structures.cpp
	JuceLibraryCode/modules/juce_events/juce_events.cpp
	JuceLibraryCode/modules/juce_graphics/juce_graphics.cpp
	JuceLibraryCode/modules/juce_gui_basics/juce_gui_basics.cpp
	JuceLibraryCode/modules/juce_gui_extra/juce_gui_extra.cpp
	JuceLibraryCode/modules/juce_audio_basics/juce_audio_basics.cpp
	JuceLibraryCode/modules/juce_audio_devices/juce_audio_devices.cpp
	JuceLibraryCode/modules/juce_audio_formats/juce_audio_formats.cpp
	JuceLibraryCode/modules/juce_audio_processors/juce_audio_processors.cpp
)

ADD_LIBRARY(openshot-audio SHARED
	${JUCE_SOURCES}
)

# Set SONAME and other library properties
set_target_properties(openshot-audio
			PROPERTIES
			VERSION ${PROJECT_VERSION}
			SOVERSION ${SO_VERSION})

TARGET_LINK_LIBRARIES(openshot-audio
	${JUCE_PLATFORM_SPECIFIC_LIBRARIES}
)

# PROCESS SUB-DIRECTORIES
add_subdirectory(src)

# Install Files
INSTALL(FILES ${CMAKE_CURRENT_SOURCE_DIR}/JuceLibraryCode/JuceHeader.h DESTINATION include/libopenshot-audio)
INSTALL(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/JuceLibraryCode DESTINATION include/libopenshot-audio FILES_MATCHING PATTERN "*.h")
INSTALL(TARGETS openshot-audio DESTINATION lib)

################### DOCUMENTATION ###################
# Find Doxygen (used for documentation)
include(cmake/Modules/UseDoxygen.cmake)

# Install Doxygen html documentation
file(GLOB_RECURSE doc_files ${CMAKE_CURRENT_BINARY_DIR}/doc/html/*.*)
INSTALL(FILES ${doc_files} DESTINATION share/doc/libopenshot-audio)

# Install manpage
INSTALL(FILES ${CMAKE_CURRENT_SOURCE_DIR}/doc/openshot-audio-test-sound.1 DESTINATION share/doc/libopenshot-audio)