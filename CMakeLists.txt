cmake_minimum_required(VERSION 3.20)

set(NAME libgfortran)
set(VERSION 13.2.0)

project(cmeel-${NAME} VERSION ${VERSION})

if(DEFINED ENV{CMEEL_JOBS})
  set(CMEEL_JOBS
      $ENV{CMEEL_JOBS}
      CACHE STRING "number of build jobs" FORCE)
else()
  set(CMEEL_JOBS
      4
      CACHE STRING "number of build jobs")
endif()

include(FindPkgConfig)
pkg_check_modules(GMP REQUIRED gmp)
pkg_check_modules(MPFR REQUIRED mpfr)

if(APPLE)
  set(LIB_PATH "DYLD_LIBRARY_PATH=${GMP_PREFIX}/lib")
else()
  set(LIB_PATH "LD_LIBRARY_PATH=${GMP_PREFIX}/lib")
endif()

include(ExternalProject)
ExternalProject_Add(
  ${NAME}
  URL "https://ftpmirror.gnu.org/gcc/gcc-${VERSION}/gcc-${VERSION}.tar.xz"
  URL_HASH "SHA1=5f95b6d042fb37d45c6cbebfc91decfbc4fb493c"
  DOWNLOAD_EXTRACT_TIMESTAMP OFF
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} "-E" "env" ${LIB_PATH} "--" "../${NAME}/configure"
    "--prefix=${CMAKE_INSTALL_PREFIX}/tmp" "--with-gmp=${GMP_PREFIX}"
    "--with-mpfr=${MPFR_PREFIX}" "--with-mpc=${GMP_PREFIX}" # this is not super
                                                            # nice.
    "--disable-multilib" "--enable-lto" "--disable-bootstrap" "VERBOSE=1"
  BUILD_COMMAND ${LIB_PATH} "make" "VERBOSE=1" "-j${CMEEL_JOBS}"
  INSTALL_COMMAND ${LIB_PATH} "make" "install" "VERBOSE=1"
  COMMAND ${CMAKE_COMMAND} "-E" "make_directory" "${CMAKE_INSTALL_PREFIX}/lib"
  COMMAND
    ${CMAKE_COMMAND} "-E" "copy" "-t" "${CMAKE_INSTALL_PREFIX}/lib"
    "${CMAKE_INSTALL_PREFIX}/tmp/lib64/libgfortran.so"
    "${CMAKE_INSTALL_PREFIX}/tmp/lib64/libgfortran.so.5"
    "${CMAKE_INSTALL_PREFIX}/tmp/lib64/libgfortran.so.5.0.0"
  COMMAND ${CMAKE_COMMAND} "-E" "rm" "-r" "${CMAKE_INSTALL_PREFIX}/tmp")

# dummy file for install target
install(FILES README.md DESTINATION share/cmeel-${NAME}/)
