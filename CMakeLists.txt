cmake_minimum_required (VERSION 3.1.0)
project (SDLMIXER)

include(CheckLibraryExists)
include(CheckIncludeFiles)
include(CheckIncludeFile)

set(CMAKE_MODULE_PATH "${SDLMIXER_SOURCE_DIR}/cmake")

find_package(SDL2 REQUIRED)

set(SDL2_LIBRARIES "${SDL2_LIBRARY}")
set(SDL2_INCLUDE_DIRS "${SDL2_INCLUDE_DIR}")
set(SDLMIXER_LIBRARIES "")

include_directories(${SDL2_INCLUDE_DIRS})

set(SDLMIXER_SUPPORT_CMD_MUSIC ON CACHE BOOL "Support for external command for playing music")
set(SDLMIXER_SUPPORT_WAV_MUSIC ON CACHE BOOL "Support for WAV music")
set(SDLMIXER_SUPPORT_MODPLUG_MUSIC ON CACHE BOOL "Support for ModPlug music")
set(SDLMIXER_SUPPORT_MOD_MUSIC ON CACHE BOOL "Support for Mod music")
set(SDLMIXER_SUPPORT_MID_MUSIC ON CACHE BOOL "Support for MIDI music")
	set(SDLMIXER_SUPPORT_MID_MUSIC_TIMIDITY ON CACHE BOOL "Support for MIDI over TIMIDITI music")
	set(SDLMIXER_SUPPORT_MID_MUSIC_FLUIDSYNTH ON CACHE BOOL "Support for MIDI over FluidSynth music")
	set(SDLMIXER_SUPPORT_MID_MUSIC_NATIVE ON CACHE BOOL "Support for native MIDI music")

set(SDLMIXER_SUPPORT_OGG_MUSIC ON CACHE BOOL "Support for OggVorbis music")
set(SDLMIXER_SUPPORT_MP3_MUSIC ON CACHE BOOL "Support for MP3 music")
set(SDLMIXER_SUPPORT_MP3_MAD_MUSIC ON CACHE BOOL "Support for MP3 MAD music")
set(SDLMIXER_SUPPORT_FLAC_MUSIC ON CACHE BOOL "Support for FLAC music")

if(SDLMIXER_SUPPORT_CMD_MUSIC)
	add_definitions(-DCMD_MUSIC)
endif()
if(SDLMIXER_SUPPORT_WAV_MUSIC)
	add_definitions(-DWAV_MUSIC)
endif()
if(SDLMIXER_SUPPORT_MODPLUG_MUSIC)
	add_definitions(-DMODPLUG_MUSIC)
	add_subdirectory(external/libmodplug-0.8.8.4)
	include_directories(external/libmodplug-0.8.8.4/include)
	list(APPEND SDLMIXER_LIBRARIES MODPLUG)
endif()
if(SDLMIXER_SUPPORT_MOD_MUSIC)
	add_definitions(-DMOD_MUSIC)
	if (NOT TARGET MIKMOD)
		add_subdirectory(external/libmikmod-3.1.12)
	endif()
	include_directories(external/libmikmod-3.1.12/include)
	list(APPEND SDLMIXER_LIBRARIES MIKMOD)
endif()

if(SDLMIXER_SUPPORT_MID_MUSIC)
	add_definitions(-DMID_MUSIC)
	if(SDLMIXER_SUPPORT_MID_MUSIC_TIMIDITY)
		add_definitions(-DUSE_TIMIDITY_MIDI)
		include_directories("${SDLMIXER_SOURCE_DIR}/timidity")
	endif()
	if(SDLMIXER_SUPPORT_MID_MUSIC_FLUIDSYNTH)
		add_definitions(-DUSE_FLUIDSYNTH_MIDI)
	endif()
	if(SDLMIXER_SUPPORT_MID_MUSIC_NATIVE)
		add_definitions(-DUSE_NATIVE_MIDI)
		include_directories("${SDLMIXER_SOURCE_DIR}/native_midi")
	endif()
endif()

if(SDLMIXER_SUPPORT_OGG_MUSIC)
	add_definitions(-DOGG_MUSIC)
	if (NOT TARGET OGG)
		add_subdirectory(external/libogg-1.3.1)
	endif()
	include_directories(external/libogg-1.3.1/include)
	list(APPEND SDLMIXER_LIBRARIES OGG)

	add_subdirectory(external/libvorbis-1.3.3)
	include_directories(external/libvorbis-1.3.3/include)
	list(APPEND SDLMIXER_LIBRARIES VORBIS_FILE)

endif()

if(SDLMIXER_SUPPORT_MP3_MUSIC)
	add_definitions(-DMP3_MUSIC)
	add_subdirectory(external/smpeg2-2.0.0)
	include_directories(external/smpeg2-2.0.0)
	list(APPEND SDLMIXER_LIBRARIES SMPEG)
endif()

if(SDLMIXER_SUPPORT_MP3_MAD_MUSIC)
	add_definitions(-DMP3_MAD_MUSIC)
endif()

if(SDLMIXER_SUPPORT_FLAC_MUSIC)
	add_definitions(-DFLAC_MUSIC)
	add_subdirectory(external/flac-1.2.1)
	include_directories(external/flac-1.2.1/include)
	list(APPEND SDLMIXER_LIBRARIES FLAC)
endif()

set(SDLMIXER_SRC
	dynamic_flac.c
	dynamic_fluidsynth.c
	dynamic_mod.c
	dynamic_modplug.c
	dynamic_mp3.c
	dynamic_ogg.c
	effect_position.c
	effect_stereoreverse.c
	effects_internal.c
	fluidsynth.c
	load_aiff.c
	load_flac.c
	load_mp3.c
	load_ogg.c
	load_voc.c
	mixer.c
	music.c
	music_cmd.c
	music_flac.c
	music_mad.c
	music_mod.c
	music_modplug.c
	music_ogg.c
	wavestream.c
)

set(SDLMIXER_SOURCES
	${SDLMIXER_SRC}
)

include_directories(./)

add_library(SDLMIXER SHARED
	${SDLMIXER_SOURCES}
)

set_target_properties(SDLMIXER
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)

target_link_libraries (SDLMIXER
	${SDL2_LIBRARIES} ${SDLMIXER_LIBRARIES}
)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY build)
set_target_properties(SDLMIXER PROPERTIES
	PREFIX ""
	OUTPUT_NAME "SDL_mixer"
)
