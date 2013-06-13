cmake_minimum_required(VERSION 2.8)
project (wpa_supplicant)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)


set(SELF_CFLAGS "")
set(SELF_CPPFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

if(NOT TARGET build_libssl)
	add_subdirectory(libssl-cmake)
endif(NOT TARGET build_libssl)
set(SELF_DEPENDS "${SELF_DEPENDS};build_libssl")

set(SELF_CFLAGS "-I${CMAKE_BINARY_DIR}/include")
set(SELF_CPPFLAGS "-I${CMAKE_BINARY_DIR}/include")
set(SELF_LDFLAGS "-L${CMAKE_BINARY_DIR}/lib")

set(TOOL_TARGET_NAME "build_wpa_supplicant")
set(TOOL_TARGET_BUILD_OUTPUT "BUILD_WPA_SUPPLICANT")
set(TOOL_TARGET_COMMENT "build wpa_supplicant")
set(TOOL_TARGET_SOURCE wpa_supplicant-2.0.tar.gz)
set(TOOL_TARGET_DIR wpa_supplicant-2.0)
set(CONFIG_ADD_OPTIONS "")
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget http://hostap.epitest.fi/releases/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/.config ${TOOL_TARGET_DIR}/wpa_supplicant
			COMMAND cd ${TOOL_TARGET_DIR}/wpa_supplicant && ${target_platform_preconfigure_options} LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" CFLAGS="${SELF_CFLAGS}" ${CONFIG_ADD_OPTIONS} make
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget http://hostap.epitest.fi/releases/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/.config ${TOOL_TARGET_DIR}/wpa_supplicant
			COMMAND cd ${TOOL_TARGET_DIR}/wpa_supplicant && ${target_platform_preconfigure_options} LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" CFLAGS="${SELF_CFLAGS}" ${CONFIG_ADD_OPTIONS} make
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(${TOOL_TARGET_NAME} ALL DEPENDS ${SELF_DEPENDS} ${TOOL_TARGET_BUILD_OUTPUT} )
