if(CONFIG_MBEDTLS)
zephyr_interface_library_named(mbedTLS)

if(CONFIG_MBEDTLS_BUILTIN)
  target_compile_definitions(${IMAGE}mbedTLS INTERFACE
	MBEDTLS_CONFIG_FILE="${CONFIG_MBEDTLS_CFG_FILE}"
	)

  target_include_directories(${IMAGE}mbedTLS INTERFACE
	include
	configs
	)

  zephyr_library()

  file(GLOB
    mbedtls_sources # This is an output parameter
    library/*.c
    )

  zephyr_library_sources(
    zephyr_init.c
    ${mbedtls_sources}
  )

if(CONFIG_ARCH_POSIX AND CONFIG_ASAN AND NOT CONFIG_64BIT)
  # i386 assembly code used in MBEDTLS does not compile with size optimization
  # if address sanitizer is enabled, as such switch default optimization level
  # to speed
  set_property(SOURCE library/bignum.c APPEND PROPERTY COMPILE_OPTIONS
      "${OPTIMIZE_FOR_SPEED_FLAG}")
endif ()

  zephyr_library_link_libraries(${IMAGE}mbedTLS)
else()
  assert(CONFIG_MBEDTLS_LIBRARY "MBEDTLS was enabled, but neither BUILTIN or LIBRARY was selected.")

  # NB: CONFIG_MBEDTLS_LIBRARY is not regression tested and is
  # therefore susceptible to bit rot

  target_include_directories(${IMAGE}mbedTLS INTERFACE
	${CONFIG_MBEDTLS_INSTALL_PATH}
	)

  zephyr_link_libraries(
    ${IMAGE}mbedtls_external
    -L${CONFIG_MBEDTLS_INSTALL_PATH}
    gcc
    )
  # Lib mbedtls_external depends on libgcc (I assume?) so to allow
  # mbedtls_external to link with gcc we need to ensure it is placed
  # after mbedtls_external on the linkers command line.
endif()

target_link_libraries(${IMAGE}mbedTLS INTERFACE ${IMAGE}zephyr_interface)
endif()
