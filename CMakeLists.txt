cmake_minimum_required(VERSION 3.13.0) # 3.13 minimum for CMP0079
project(allovisor VERSION 0.2.0)
cmake_policy(SET CMP0079 NEW)

SET(LOVR_BUILD_BUNDLE ON CACHE BOOL "Always build lovr bundle on mac")
SET(LOVR_ENABLE_ENET OFF CACHE BOOL "Disable Lovr enet since allonet embeds its own")
set(ALLONET_USE_LUA ON CACHE BOOL "Request Lua bridge")
if(WIN32)
	set(ALLONET_BUILD_STATIC OFF CACHE BOOL "bundle a DLL")
elseif(APPLE OR ANDROID)
    set(ALLONET_BUILD_STATIC ON CACHE BOOL "Always build allonet static")
endif()

add_subdirectory (deps/lovr)
add_subdirectory (deps/allonet)

if(WIN32)
    target_link_libraries(lovr allonet)
	
	file(GLOB ZIP_FILES "${CMAKE_CURRENT_SOURCE_DIR}/lua/*")
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Alloverse-win64.zip
        COMMAND ${CMAKE_COMMAND} -E make_directory  ${CMAKE_CURRENT_BINARY_DIR}/Alloverse
		COMMAND copy deps\\lovr\\*.dll Alloverse
		COMMAND copy deps\\allonet\\allonet.dll Alloverse\\liballonet.dll
		COMMAND ${CMAKE_COMMAND} -E chdir ${CMAKE_CURRENT_SOURCE_DIR}/lua ${CMAKE_COMMAND} -E tar c ${CMAKE_CURRENT_BINARY_DIR}/lua.zip --format=zip -- ${ZIP_FILES}
		COMMAND copy /b deps\\lovr\\lovr.exe+lua.zip Alloverse\\Alloverse.exe
		COMMAND ${CMAKE_COMMAND} -E tar c ${CMAKE_CURRENT_BINARY_DIR}/Alloverse-win64.zip --format=zip Alloverse
        DEPENDS lovr allonet
    )
    add_custom_target(
        alloverse-dist ALL
        DEPENDS Alloverse-win64.zip
    )
elseif(APPLE)
    # Force-load allonet into lovr exe
    target_link_libraries(lovr "-Wl,-all_load" allonet )

    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Alloverse.app
        COMMAND cp -r ${CMAKE_CURRENT_BINARY_DIR}/deps/lovr/lovr.app ${CMAKE_CURRENT_BINARY_DIR}/Alloverse.app
        COMMAND cp -r ${CMAKE_CURRENT_SOURCE_DIR}/lua Alloverse.app/Contents/Resources/.lovr
        #COMMAND cp -r ${CMAKE_CURRENT_BINARY_DIR}/deps/allonet/liballonet.dylib Alloverse.app/Contents/Resources/.lovr/liballonet.so
        COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/src/mac/Info.plist Alloverse.app/Contents/Info.plist
        COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/src/mac/alloverse.icns Alloverse.app/Contents/Resources/alloverse.icns
        COMMAND zip ${CMAKE_CURRENT_BINARY_DIR}/Alloverse-mac.zip -r Alloverse.app
        DEPENDS lovr allonet
    )
    add_custom_target(
        alloverse-dist ALL
        DEPENDS Alloverse.app
    )
elseif(ANDROID)
    # Force-load allonet into lovr exe
    target_link_libraries(lovr "-Wl,--whole-archive" allonet "-Wl,--no-whole-archive")

    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/alloverse-release.apk
        COMMAND bash ${CMAKE_CURRENT_SOURCE_DIR}/build-android.sh build
        COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/deps/lovr-android/LovrApp/Projects/Android/build/outputs/apk/debug/alloverse-debug.apk .
        COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/deps/lovr-android/LovrApp/Projects/Android/build/outputs/apk/release/alloverse-release.apk .
    )
    add_custom_target(
        alloverse-dist ALL
        DEPENDS alloverse-release.apk
    )

elseif(UNIX) # linux unsupported atm

endif()