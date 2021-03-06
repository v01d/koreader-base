PROJECT(harfbuzz)
cmake_minimum_required(VERSION 2.8.3)

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/../cmake_modules")
include("koreader_thirdparty_common")
include("koreader_thirdparty_git")

enable_language(C CXX)

assert_var_defined(CC)
assert_var_defined(CFLAGS)
assert_var_defined(LDFLAGS)
assert_var_defined(FREETYPE_DIR)

ep_get_source_dir(SOURCE_DIR)
ep_get_binary_dir(BINARY_DIR)

set(CFG_ENV_VAR "CC=\"${CC}\" CXX=\"${CXX}\" CFLAGS=\"${CFLAGS}\" CXXFLAGS=\"${CXXFLAGS}\" LDFLAGS=\"${LDFLAGS}\"")
set(CFG_ENV_VAR "${CFG_ENV_VAR} FREETYPE_CFLAGS=\"-I${FREETYPE_DIR}/include/freetype2\"")
set(CFG_ENV_VAR "${CFG_ENV_VAR} FREETYPE_LIBS=\"-L${FREETYPE_DIR} -lfreetype\"")
set(CFG_OPTS "--prefix=\"${BINARY_DIR}\" --enable-shared --disable-static --with-freetype --with-ucdn --without-glib --without-gobject --without-cairo --without-fontconfig --without-icu --without-graphite2 --without-uniscribe --without-directwrite --without-coretext --host=\"${CHOST}\"")
set(CFG_CMD sh -c "${CFG_ENV_VAR} ${SOURCE_DIR}/configure ${CFG_OPTS}")

ko_write_gitclone_script(
    GIT_CLONE_SCRIPT_FILENAME
    https://github.com/harfbuzz/harfbuzz.git
    1.8.8
    ${SOURCE_DIR}
)

include(ExternalProject)
ExternalProject_Add(
    ${PROJECT_NAME}
    DOWNLOAD_COMMAND ${CMAKE_COMMAND} -P ${GIT_CLONE_SCRIPT_FILENAME}
    BUILD_IN_SOURCE 1
    PATCH_COMMAND NOCONFIGURE=1 ./autogen.sh
    CONFIGURE_COMMAND ${CFG_CMD}
    BUILD_COMMAND $(MAKE) -j${PARALLEL_JOBS} --silent
    INSTALL_COMMAND $(MAKE) -j${PARALLEL_JOBS} install
)
