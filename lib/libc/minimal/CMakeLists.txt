# SPDX-License-Identifier: Apache-2.0

# As minimal libc will be built as a zephyr_library, clear c_library to
# prevent accidentally requiring or linking against another libc implementation.
set_property(TARGET linker PROPERTY c_library "")

zephyr_system_include_directories(include)

zephyr_library()

set(GEN_DIR ${ZEPHYR_BINARY_DIR}/include/generated)
set(STRERROR_TABLE_H ${GEN_DIR}/libc/minimal/strerror_table.h)

zephyr_library_compile_options($<TARGET_PROPERTY:compiler,no_builtin>)
zephyr_library_compile_options($<TARGET_PROPERTY:compiler,prohibit_lto>)

zephyr_library_sources(
  source/stdlib/atoi.c
  source/stdlib/strtol.c
  source/stdlib/strtoul.c
  source/stdlib/strtoll.c
  source/stdlib/strtoull.c
  source/stdlib/bsearch.c
  source/stdlib/exit.c
  source/stdlib/qsort.c
  source/string/strerror.c
  source/string/strncasecmp.c
  source/string/strstr.c
  source/string/string.c
  source/string/strspn.c
  source/stdout/stdout_console.c
  source/stdout/sprintf.c
  source/stdout/fprintf.c
  source/math/sqrtf.c
  source/math/sqrt.c
  ${STRERROR_TABLE_H}
)

if(CONFIG_MINIMAL_LIBC_TIME)
  zephyr_library_sources(source/time/gmtime.c)
endif()

zephyr_library_sources_ifdef(CONFIG_MINIMAL_LIBC_RAND source/stdlib/rand.c)

add_custom_command(
  OUTPUT ${STRERROR_TABLE_H}
  COMMAND
  ${PYTHON_EXECUTABLE}
  ${ZEPHYR_BASE}/scripts/build/gen_strerror_table.py
  -i include/errno.h
  -o ${STRERROR_TABLE_H}
  DEPENDS include/errno.h
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)
