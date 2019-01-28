
px4_add_git_submodule(TARGET git_nuttx PATH "nuttx")
px4_add_git_submodule(TARGET git_nuttx_apps PATH "apps")

if(NOT PX4_BOARD)
	message(FATAL_ERROR "PX4_BOARD must be set (eg px4_fmu-v2)")
endif()

if ("x${config_expanded}" STREQUAL "x")
	# copy PX4 board Compressed config into nuttx and inflate it
	add_custom_command(
		OUTPUT
			${NUTTX_DIR}/.config
			${NUTTX_DIR}/arch/arm/include/math.h
		COMMAND ${CMAKE_COMMAND} -E copy ${NUTTX_CONFIG_DIR}/${NUTTX_CONFIG}/defconfig ${NUTTX_DIR}/.config
		COMMAND kconfig-tweak --set-str CONFIG_APPS_DIR "../apps"
		COMMAND ${CMAKE_COMMAND} -E copy_directory ${NUTTX_CONFIG_DIR}/ ${CMAKE_CURRENT_BINARY_DIR}/nuttx-config
		COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/nuttx-config/src
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/math.h ${NUTTX_DIR}/arch/arm/include/ # copy arm math.h into NuttX source
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/nsh_romfsimg.h ${CMAKE_CURRENT_BINARY_DIR}/nuttx-config/include/
		COMMAND make --no-print-directory --silent -C ${NUTTX_DIR} CONFIG_ARCH_BOARD_CUSTOM=y olddefconfig
		DEPENDS
			${NUTTX_CONFIG_DIR}/${NUTTX_CONFIG}/defconfig
			${NUTTX_CONFIG_DIR}/include/board.h
			${NUTTX_CONFIG_DIR}/scripts/ld.script
			${NUTTX_DIR}/configs/dummy/Kconfig
			${CMAKE_CURRENT_SOURCE_DIR}/math.h
			${CMAKE_CURRENT_SOURCE_DIR}/nsh_romfsimg.h
		WORKING_DIRECTORY ${NUTTX_DIR}
		COMMENT "Copying NuttX config ${NUTTX_CONFIG} and configuring"
		)
else()
	# copy PX4 board config into nuttx
	add_custom_command(
		OUTPUT
			${NUTTX_DIR}/.config
			${NUTTX_DIR}/arch/arm/include/math.h
		COMMAND ${CMAKE_COMMAND} -E copy ${NUTTX_CONFIG_DIR}/${NUTTX_CONFIG}/defconfig ${NUTTX_DIR}/.config
		COMMAND ${CMAKE_COMMAND} -E copy_directory ${NUTTX_CONFIG_DIR}/ ${CMAKE_CURRENT_BINARY_DIR}/nuttx-config
		COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/nuttx-config/src
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/math.h ${NUTTX_DIR}/arch/arm/include/ # copy arm math.h into NuttX source
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/nsh_romfsimg.h ${CMAKE_CURRENT_BINARY_DIR}/nuttx-config/include/
		DEPENDS
			${NUTTX_CONFIG_DIR}/${NUTTX_CONFIG}/defconfig
			${NUTTX_CONFIG_DIR}/include/board.h
			${NUTTX_CONFIG_DIR}/scripts/ld.script
			${CMAKE_CURRENT_BINARY_DIR}/nuttx_copy.stamp
			${CMAKE_CURRENT_BINARY_DIR}/apps_copy.stamp
			${NUTTX_DIR}/configs/dummy/Kconfig
			${CMAKE_CURRENT_SOURCE_DIR}/math.h
			${CMAKE_CURRENT_SOURCE_DIR}/nsh_romfsimg.h
		WORKING_DIRECTORY ${NUTTX_DIR}
		COMMENT "Copying NuttX uncompressed config ${NUTTX_CONFIG} and configuring"
		)
endif()

add_subdirectory(nuttx)
