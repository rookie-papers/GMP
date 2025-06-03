cmake_minimum_required(VERSION 3.14)

include(ExternalProject)

project(gmp_external LANGUAGES C CXX)

# install path
set(GMP_INSTALL_DIR ${CMAKE_BINARY_DIR}/gmp-install)

# construct GMP
ExternalProject_Add(gmp_external_proj
        SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
        CONFIGURE_COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure
        --prefix=${GMP_INSTALL_DIR}
        --enable-cxx
        ac_cv_func_asprintf=no

        BUILD_COMMAND ${CMAKE_MAKE_PROGRAM} -j8 MAKEINFO=true
        INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install MAKEINFO=true
        BUILD_IN_SOURCE 1
)

# link libgmp.a as CMake target
add_library(gmp STATIC IMPORTED GLOBAL)
set_target_properties(gmp PROPERTIES
        IMPORTED_LOCATION ${GMP_INSTALL_DIR}/lib/libgmp.a
        INTERFACE_INCLUDE_DIRECTORIES ${GMP_INSTALL_DIR}/include
)

# link libgmpxx.a as CMake target
add_library(gmpxx STATIC IMPORTED GLOBAL)
set_target_properties(gmpxx PROPERTIES
        IMPORTED_LOCATION ${GMP_INSTALL_DIR}/lib/libgmpxx.a
        INTERFACE_INCLUDE_DIRECTORIES ${GMP_INSTALL_DIR}/include
)

# create dependencies
add_dependencies(gmp gmp_external_proj)
add_dependencies(gmpxx gmp_external_proj)

# Alias (for the main project)
add_library(gmp::gmp ALIAS gmp)
add_library(gmp::gmpxx ALIAS gmpxx)
