cmake_minimum_required(VERSION 3.12 FATAL_ERROR)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_INSTALL_BINDIR ".")
set(CMAKE_INSTALL_LIBDIR ".")
set(CMAKE_INSTALL_DATADIR ".")
include(GNUInstallDirs)

option(BITTBOY "Set to ON if targeting a Bittboy device" ${BITTBOY})
option(GAMESHELL "Set to ON if targeting a GameShell device" ${GAMESHELL})
option(PANDORA "Set to ON if targeting an OpenPandora device" ${PANDORA})
option(ODROID "Set to ON if targeting an ODroid device" ${ODROID})
option(PYRA "Set to ON if targeting a DragonBox Pyra device" ${PYRA})
option(CHIP "Set to ON if targeting a PocketCHIP device" ${CHIP})
option(USE_CCACHE "Set to ON to use ccache if present in the system" ${USE_CCACHE})
option(EMSCRIPTEN "Set to ON to build using EMSCRIPTEN" ${EMSCRIPTEN})

option(USE_SDL2 "Use SDL2 instead of SDL1.2" ${USE_SDL2})
option(USE_SDL3 "Use SDL3 instead of SDL1.2" ${USE_SDL3})

if(CMAKE_C_COMPILER EQUAL "emcc")
	set(EMSCRIPTEN ON)
endif()

if(EMSCRIPTEN)
	add_definitions(-D_SDL)
	add_definitions(-D__EMSCRIPTEN__)
	if(USE_SDL3)
		add_definitions(-D_SDL3)
		find_package(SDL3 REQUIRED)
		find_package(SDL3_mixer REQUIRED)
		include_directories(${SDL3_INCLUDE_DIRS} ${SDL3_mixer_INCLUDE_DIRS})
	else()
		set(USE_SDL2 ON CACHE BOOL "" FORCE)
		add_definitions(-D_SDL2)
		add_compile_options("-sUSE_SDL=2" "-sUSE_SDL_MIXER=2" "-sUSE_OGG=1" "-sUSE_VORBIS=1")
	endif()
endif()

if(BITTBOY)
	add_definitions(-DBITTBOY)
endif()
if(GAMESHELL)
	add_definitions(-DGAMESHELL)
endif()
if(PANDORA)
	add_definitions(-DPANDORA)
endif()
if(PYRA)
	add_definitions(-DPYRA)
endif()
if(CHIP)
	add_definitions(-DCHIP)
endif()
if(ODROID)
	add_definitions(-DODROID)
endif()

if(USE_SDL3)
	add_definitions(-D_SDL -D_SDL3)
	find_package(SDL3 REQUIRED)
	find_package(SDL3_mixer REQUIRED)

	INCLUDE_DIRECTORIES(${SDL3_INCLUDE_DIRS})
	INCLUDE_DIRECTORIES(${SDL3_mixer_INCLUDE_DIRS})
elseif(USE_SDL2)
	add_definitions(-D_SDL -D_SDL2)
	find_package(SDL2 REQUIRED)
	#find_package(SDL2_mixer REQUIRED)
	
	INCLUDE_DIRECTORIES(${SDL2_INCLUDE_DIRS})
	#INCLUDE_DIRECTORIES(${SDL2_MIXER_INCLUDE_DIRS})
else()
	add_definitions(-D_SDL)
	find_package(SDL REQUIRED)
	find_package(SDL_mixer REQUIRED)
	
	INCLUDE_DIRECTORIES(${SDL_INCLUDE_DIRS})
	INCLUDE_DIRECTORIES(${SDL_MIXER_INCLUDE_DIRS})
endif()
endif(EMSCRIPTEN)

if(USE_CCACHE)
    find_program(CCACHE_FOUND ccache)
    if(CCACHE_FOUND)
        set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
        set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
    endif()
endif()

#Need c++11 and c99
if (CMAKE_VERSION VERSION_LESS "3.1")
	if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
		set (CMAKE_C_FLAGS "-std=gnu99 ${CMAKE_C_FLAGS}")
		set (CMAKE_CXX_FLAGS "-std=c+11 ${CMAKE_CXX_FLAGS}")
	endif ()
else ()
	set (CMAKE_C_STANDARD 99)
	set (CMAKE_CXX_STANDARD 11)
endif ()

set(HCL_SOURCE_FILES
	src/collision.c
	src/effect.c
	src/enemy.c
	src/game.c
	src/hero.c
	src/ini.c
	src/inventory.c
	src/main.c
	src/object.c
	src/options.c
	src/PHL.c
	src/platform.c
	src/qda.c
	src/stagedata.c
	src/text.c
	src/titlescreen.c
	src/weapon.c
	src/enemies/batboss.c
	src/enemies/bat.c
	src/enemies/bee.c
	src/enemies/boar.c
	src/enemies/boomknight.c
	src/enemies/crab.c
	src/enemies/devil.c
	src/enemies/dodo.c
	src/enemies/dog.c
	src/enemies/firewheel.c
	src/enemies/fish.c
	src/enemies/garm.c
	src/enemies/gas.c
	src/enemies/ghoul.c
	src/enemies/golem.c
	src/enemies/gyra.c
	src/enemies/heads.c
	src/enemies/hydra.c
	src/enemies/jellyfish.c
	src/enemies/knight.c
	src/enemies/lolidra.c
	src/enemies/pendulum.c
	src/enemies/podoboo.c
	src/enemies/poisonknight.c
	src/enemies/pumpkin.c
	src/enemies/seal.c
	src/enemies/skeleton.c
	src/enemies/skull.c
	src/enemies/slime.c
	src/enemies/slug.c
	src/enemies/thwomp.c
	src/enemies/waterjumper.c
	src/enemies/wizard.c
)
if(USE_SDL3)
	set(HCL_SDL_FILES
		src/sdl3/audio.c
		src/sdl3/input.c
		src/sdl3/graphics.c
		src/sdl3/system.c
		src/xBRZ/xbrz.cpp
		src/sdl3/scale.cpp
	)
elseif(USE_SDL2)
	set(HCL_SDL_FILES
		src/sdl2/audio.c
		src/sdl2/input.c
		src/sdl2/graphics.c
		src/sdl2/system.c
		src/xBRZ/xbrz.cpp
		src/sdl2/scale.cpp
	)
else()
	set(HCL_SDL_FILES
		src/sdl/audio.c
		src/sdl/input.c
		src/sdl/graphics.c
		src/sdl/system.c
		src/xBRZ/xbrz.cpp
		src/sdl/scale.cpp
	)
endif()

if(PANDORA)
	add_definitions(-mcpu=cortex-a8)
	add_definitions(-mfpu=neon)
	add_definitions(-mfloat-abi=softfp)
	add_definitions(-ftree-vectorize)
endif(PANDORA)

if(NOT EMSCRIPTEN)
	if(USE_SDL3)
		set (EXTRA_LIBS ${EXTRA_LIBS} SDL3::SDL3)
		set (EXTRA_LIBS ${EXTRA_LIBS} SDL3_mixer::SDL3_mixer)
	elseif(USE_SDL2)
		set (EXTRA_LIBS ${EXTRA_LIBS} ${SDL2_LIBRARIES})
		set (EXTRA_LIBS ${EXTRA_LIBS} SDL2_mixer)
	else()
		set (EXTRA_LIBS ${EXTRA_LIBS} ${SDL_LIBRARY})
		set (EXTRA_LIBS ${EXTRA_LIBS} ${SDL_MIXER_LIBRARY})
	endif()
endif()

add_executable(hcl ${HCL_SOURCE_FILES} ${HCL_SDL_FILES})
set_target_properties(hcl PROPERTIES OUTPUT_NAME "hcl")
target_link_libraries (hcl ${EXTRA_LIBS} m)
if(EMSCRIPTEN)
	set(CMAKE_EXECUTABLE_SUFFIX ".html")
	target_compile_options(hcl PUBLIC "-O2")
	set_target_properties(hcl PROPERTIES
		RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/docs"
		OUTPUT_NAME "index"
	)
	if(USE_SDL3)
		target_link_libraries(hcl SDL3::SDL3 SDL3_mixer::SDL3_mixer)
	else()
		target_link_options(hcl PRIVATE "-sUSE_SDL=2" "-sUSE_SDL_MIXER=2" "-sUSE_OGG=1" "-sUSE_VORBIS=1")
	endif()
	target_link_options(hcl PRIVATE
		"-sWASM=1"
		"-sASSERTIONS=1"
		"-sTOTAL_MEMORY=128MB"
		"-sALLOW_MEMORY_GROWTH=1"
		"-sEXIT_RUNTIME=1"
		"--preload-file" "${CMAKE_SOURCE_DIR}/data@data"
		"--shell-file" "${CMAKE_SOURCE_DIR}/src/template.html"
	)
endif()

# Main binary
install(TARGETS hcl
	RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
	BUNDLE  DESTINATION "${CMAKE_INSTALL_BINDIR}"
)

# Support data
install(DIRECTORY
	"${CMAKE_CURRENT_SOURCE_DIR}/data"
 	DESTINATION "${CMAKE_INSTALL_DATADIR}"
        FILE_PERMISSIONS
          OWNER_READ OWNER_WRITE
          GROUP_READ
          WORLD_READ
        DIRECTORY_PERMISSIONS
          OWNER_READ OWNER_EXECUTE OWNER_WRITE
          GROUP_READ GROUP_EXECUTE GROUP_WRITE
          WORLD_READ WORLD_EXECUTE
	)

# Icon and shortcut
if(LINUX)
	install(
 		FILES "${CMAKE_CURRENT_SOURCE_DIR}/io.github.ptitSeb.hydracastlelabyrinth.desktop" DESTINATION "share/applications"
	)
	install(
		FILES "${CMAKE_CURRENT_SOURCE_DIR}/io.github.ptitSeb.hydracastlelabyrinth.png" DESTINATION "share/icons/hicolor/256x256/apps"
	)
	install(
 		FILES "${CMAKE_CURRENT_SOURCE_DIR}/io.github.ptitSeb.hydracastlelabyrinth.metainfo.xml" DESTINATION "share/appdata"
   	)
endif()
