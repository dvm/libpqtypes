cmake_minimum_required(VERSION 3.20)

project(pqtypes VERSION 1.5.3 LANGUAGES C DESCRIPTION "libpqtypes is a libpq extension")

include(CheckFunctionExists)
include(CheckStructHasMember)
include(CheckSymbolExists)
include(CheckTypeSize)
include(GNUInstallDirs)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Set build type to Release or Debug" FORCE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
endif()

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

set(SOURCES
    ${PROJECT_SOURCE_DIR}/source/src/array.c
    ${PROJECT_SOURCE_DIR}/source/src/datetime.c
    ${PROJECT_SOURCE_DIR}/source/src/error.c
    ${PROJECT_SOURCE_DIR}/source/src/events.c
    ${PROJECT_SOURCE_DIR}/source/src/exec.c
    ${PROJECT_SOURCE_DIR}/source/src/geo.c
    ${PROJECT_SOURCE_DIR}/source/src/handler.c
    ${PROJECT_SOURCE_DIR}/source/src/misc.c
    ${PROJECT_SOURCE_DIR}/source/src/network.c
    ${PROJECT_SOURCE_DIR}/source/src/numerics.c
    ${PROJECT_SOURCE_DIR}/source/src/param.c
    ${PROJECT_SOURCE_DIR}/source/src/port.c
    ${PROJECT_SOURCE_DIR}/source/src/record.c
    ${PROJECT_SOURCE_DIR}/source/src/spec.c
    ${PROJECT_SOURCE_DIR}/source/src/utils.c
    ${PROJECT_SOURCE_DIR}/source/src/varlena.c
)

set(HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/source/src/libpqtypes.h
)

set(PRIVATE_HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/source/src/libpqtypes-int.h
    ${CMAKE_CURRENT_BINARY_DIR}/pqt_config.h
)

add_library(${PROJECT_NAME} ${SOURCES} ${HEADERS} ${PRIVATE_HEADERS})

set_target_properties(${PROJECT_NAME} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    SOVERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    PUBLIC_HEADER "${HEADERS}"
)

set_target_properties(${PROJECT_NAME} PROPERTIES
    POSITION_INDEPENDENT_CODE 1
)

find_package(Threads REQUIRED)
find_package(PostgreSQL REQUIRED)

target_include_directories(${PROJECT_NAME} PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/source/src>"
)

target_include_directories(${PROJECT_NAME} PRIVATE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/source/src>"
    ${PostgreSQL_INCLUDE_DIRS}
)

target_link_libraries(${PROJECT_NAME} PRIVATE
    ${PostgreSQL_LIBRARIES}
    Threads::Threads
)

if(CMAKE_COMPILER_IS_GNUCC)
    target_compile_options(${PROJECT_NAME} PRIVATE # -ggdb -rdynamic
        -Wall
        -Wextra
        -Wclobbered
        -Wempty-body
        -Wignored-qualifiers
        -Wmissing-parameter-type
        -Wold-style-declaration
        -Woverride-init
        -Wpointer-arith
        -Wsign-compare
        -Wtype-limits
        -Wuninitialized
        -fno-strict-aliasing
        -fomit-frame-pointer
        -funit-at-a-time
        -funroll-all-loops
        -fsigned-char
    )
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        _GNU_SOURCE
        _REENTRANT
    )
endif(CMAKE_COMPILER_IS_GNUCC)

if(Threads_FOUND)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        PQT_THREAD_SAFE
        _THREAD_SAFE
    )
endif(Threads_FOUND)

# Configure

check_include_file(arpa/inet.h HAVE_ARPA_INET_H)
check_include_file(limits.h HAVE_LIMITS_H)
check_include_file(linux/limits.h HAVE_LINUX_LIMITS_H)
check_include_file(math.h HAVE_MATH_H)
check_include_file(netdb.h HAVE_NETDB_H)
check_include_file(netinet/in.h HAVE_NETINET_IN_H)
check_include_file(stddef.h HAVE_STDDEF_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(sys/socket.h HAVE_SYS_SOCKET_H)
check_include_file(sys/time.h HAVE_SYS_TIME_H)
check_include_file(time.h HAVE_TIME_H)
check_include_file(dlfcn.h HAVE_DLFCN_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(memory.h HAVE_MEMORY_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(sys/stat.h HAVE_SYS_STAT_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(unistd.h HAVE_UNISTD_H)

check_function_exists(getaddrinfo HAVE_GETADDRINFO)
check_function_exists(hstrerror HAVE_HSTRERROR)
check_function_exists(localtime_r HAVE_LOCALTIME_R)
check_function_exists(memcmp HAVE_MEMCMP)
check_function_exists(strtol HAVE_STRTOL)
check_function_exists(strtoll HAVE_STRTOLL)
check_function_exists(strtoul HAVE_STRTOUL)
check_function_exists(vsnprintf HAVE_VSNPRINTF)

list(APPEND CMAKE_REQUIRED_LIBRARIES m)
check_function_exists(ceil HAVE_CEIL)
check_function_exists(floor HAVE_FLOOR)
check_function_exists(rint HAVE_RINT)
list(REMOVE_ITEM CMAKE_REQUIRED_LIBRARIES m)

check_struct_has_member("struct tm" "tm_zone"   "time.h" HAVE_STRUCT_TM_TM_ZONE)
check_struct_has_member("struct tm" "tm_gmtoff" "time.h" HAVE_STRUCT_TM_TM_GMTOFF)

if(HAVE_SYS_SOCKET_H)
    check_struct_has_member("struct sockaddr_storage" "ss_len" "sys/socket.h" HAVE_STRUCT_SOCKADDR_STORAGE_SS_LEN)
endif(HAVE_SYS_SOCKET_H)
if(NOT HAVE_STRUCT_SOCKADDR_STORAGE_SS_LEN AND HAVE_NETINET_IN_H)
    check_struct_has_member("struct sockaddr_storage" "ss_len" "netinet/in.h" HAVE_STRUCT_SOCKADDR_STORAGE_SS_LEN)
endif(NOT HAVE_STRUCT_SOCKADDR_STORAGE_SS_LEN AND HAVE_NETINET_IN_H)

if(HAVE_ARPA_INET_H)
    list(APPEND CMAKE_EXTRA_INCLUDE_FILES "arpa/inet.h")
    check_type_size(socklen_t HAVE_SOCKLEN_T)
    list(REMOVE_ITEM CMAKE_EXTRA_INCLUDE_FILES "arpa/inet.h")
endif(HAVE_ARPA_INET_H)
if(NOT HAVE_SOCKLEN_T AND HAVE_SYS_SOCKET_H)
    list(APPEND CMAKE_EXTRA_INCLUDE_FILES "sys/socket.h")
    check_type_size(socklen_t HAVE_SOCKLEN_T)
    list(REMOVE_ITEM CMAKE_EXTRA_INCLUDE_FILES "sys/socket.h")
endif(NOT HAVE_SOCKLEN_T AND HAVE_SYS_SOCKET_H)

target_compile_definitions(${PROJECT_NAME} PRIVATE
    HAVE_CONFIG_H
)

configure_file(pqt_config.h.in pqt_config.h @ONLY)
configure_file(libpqtypes.pc.in libpqtypes.pc @ONLY)

# Install library related files

install(TARGETS ${PROJECT_NAME}
    EXPORT pqtypes-export
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Development
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT RuntimeLibraries
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT RuntimeLibraries
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} COMPONENT Development
)

# Install pkg-config

install(FILES ${CMAKE_BINARY_DIR}/libpqtypes.pc
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig
)

# Install man pages

file(GLOB_RECURSE MANPAGES CONFIGURE_DEPENDS
    ${PROJECT_SOURCE_DIR}/source/docs/man3/*.3
)

install(FILES ${MANPAGES}
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/man/man3
)

# Export the target under the build-tree (no need to install)
export(EXPORT pqtypes-export
    FILE "${CMAKE_BINARY_DIR}/pqtypes-targets.cmake"
    NAMESPACE pqtypes::
)
add_library(pqtypes::pqtypes ALIAS pqtypes)

# CMake config
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/pqtypesConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_file(pqtypesConfig.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/pqtypesConfig.cmake"
    COPYONLY
)

set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/pqtypes)

install(EXPORT pqtypes-export
    FILE pqtypes-targets.cmake
    NAMESPACE pqtypes::
    DESTINATION ${INSTALL_CONFIGDIR}
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/pqtypesConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/pqtypesConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
    COMPONENT Development
)

# Packing

# these are cache variables, so they could be overwritten with -D,
if(BUILD_SHARED_LIBS)
    set(CPACK_PACKAGE_NAME ${PROJECT_NAME} CACHE STRING "The resulting package name")
else(BUILD_SHARED_LIBS)
    set(CPACK_PACKAGE_NAME ${PROJECT_NAME}-static CACHE STRING "The resulting package name")
endif(BUILD_SHARED_LIBS)

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${PROJECT_DESCRIPTION} CACHE STRING "Package description for the package metadata")
set(CPACK_PACKAGE_VENDOR "eSilo")
set(CPACK_PACKAGE_CONTACT "libpqtypes@esilo.com")

set(CPACK_DEBIAN_PACKAGE_MAINTAINER "")
set(CPACK_VERBATIM_VARIABLES YES)

set(CPACK_PACKAGE_INSTALL_DIRECTORY ${CPACK_PACKAGE_NAME})
set(CPACK_OUTPUT_FILE_PREFIX "${CMAKE_SOURCE_DIR}/_packages")

set(CPACK_PACKAGING_INSTALL_PREFIX "/usr/local")

set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/source/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/source/README")

set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT)
set(CPACK_COMPONENTS_GROUPING ALL_COMPONENTS_IN_ONE)
set(CPACK_DEB_COMPONENT_INSTALL YES)

include(CPack)

# set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
# include(Packing)
