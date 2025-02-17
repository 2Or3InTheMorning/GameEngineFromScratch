cmake_minimum_required (VERSION 3.20) 
project (GameEngineFromScratch)
source_group(TREE ${PROJECT_SOURCE_DIR})

set (CMAKE_C_STANDARD 11)
set (CMAKE_CXX_STANDARD 20)
set (CMAKE_CXX_EXTENSIONS OFF)
set (CMAKE_EXPORT_COMPILE_COMMANDS ON)
set (THREADS_PREFER_PTHREAD_FLAG ON)

option(USE_ISPC "Use ISPC for geometry math" ON)
option(D3D12_RHI_DEBUG "D3d12 Debug Layer" ON)

IF(UNIX)
    IF(APPLE)
        set(MYGE_TARGET_PLATFORM "Darwin")
        set(OS_MACOS 1)
    ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES Android)
        set(MYGE_TARGET_PLATFORM "Android")
        set(ANDROID 1)
        set(OS_ANDROID 1)
    ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
        set(MYGE_TARGET_PLATFORM "FreeBSD")
        include_directories("/usr/local/include")
        set(BSD 1)
        set(OS_BSD 1)
    ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES Emscripten)
        set(MYGE_TARGET_PLATFORM "Emscripten")
        set(WA 1)
        set(OS_WEBASSEMBLY 1)
        unset (USE_ISPC)
    ELSE(APPLE)
        set(MYGE_TARGET_PLATFORM "Linux")
        set(OS_LINUX 1)
    ENDIF(APPLE)
ELSEIF(WIN32)
    set(MYGE_TARGET_PLATFORM "Windows")
        set(OS_WINDOWS 1)
ENDIF(UNIX)

if(MSVC)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D _CRT_SECURE_NO_WARNINGS /MP /utf-8")
endif(MSVC)

if(ANDROID)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-command-line-argument")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-command-line-argument")
endif(ANDROID)

if(WA)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -s WASM=1 -s ALLOW_MEMORY_GROWTH=1 -s FORCE_FILESYSTEM=1 -s LZ4=1 --emrun")
    set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g4 --source-map-base http://localhost:8080/")
endif(WA)

set (CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DDEBUG -D_DEBUG")
set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG -D_DEBUG")

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
include(TargetArchDetect)

target_architecture(TARGET_ARCH)

if((TARGET_ARCH MATCHES "x86_64" OR TARGET_ARCH MATCHES "ia64") AND NOT OF_32BIT)
        set(ARCH_BIT 64)
else()
        set(ARCH_BIT 32)
endif()

include_directories("${PROJECT_SOURCE_DIR}/")
include_directories("${PROJECT_SOURCE_DIR}/Framework/Ability")
include_directories("${PROJECT_SOURCE_DIR}/Framework/Algorism")
include_directories("${PROJECT_SOURCE_DIR}/Framework/CodeGen")
include_directories("${PROJECT_SOURCE_DIR}/Framework/Common")
include_directories("${PROJECT_SOURCE_DIR}/Framework/DrawPass")
include_directories("${PROJECT_SOURCE_DIR}/Framework/DrawSubPass")
include_directories("${PROJECT_SOURCE_DIR}/Framework/DispatchPass")
include_directories("${PROJECT_SOURCE_DIR}/Framework/GeomMath")
include_directories("${PROJECT_SOURCE_DIR}/Framework/Geometries")
include_directories("${PROJECT_SOURCE_DIR}/Framework/Interface")
include_directories("${PROJECT_SOURCE_DIR}/Framework/Manager")
include_directories("${PROJECT_SOURCE_DIR}/Framework/Parser")
include_directories("${PROJECT_SOURCE_DIR}/Framework/RenderStack")
include_directories("${PROJECT_SOURCE_DIR}/Framework/RenderStack/DataStructure")
include_directories("${PROJECT_SOURCE_DIR}/Framework/SceneGraph")
include_directories("${PROJECT_SOURCE_DIR}/Physics")
include_directories("${PROJECT_SOURCE_DIR}/RHI")
include_directories("${PROJECT_SOURCE_DIR}/Platform/${MYGE_TARGET_PLATFORM}")
include_directories("${PROJECT_SOURCE_DIR}/Platform/Cef")

set(MYGE_EXTERNAL_ROOT ${PROJECT_SOURCE_DIR}/External/${MYGE_TARGET_PLATFORM}/)
list(APPEND CMAKE_MODULE_PATH "${MYGE_EXTERNAL_ROOT}cmake")
include_directories("${MYGE_EXTERNAL_ROOT}")
include_directories("${MYGE_EXTERNAL_ROOT}include")
include_directories("${MYGE_EXTERNAL_ROOT}include/bullet")
include_directories("${MYGE_EXTERNAL_ROOT}../src")
include_directories("${MYGE_EXTERNAL_ROOT}../src/imgui")
include_directories("${MYGE_EXTERNAL_ROOT}../src/glad/include")

set(MYGE_EXTERNAL_LIBRARY_PATH ${MYGE_EXTERNAL_ROOT}lib/)
set(MYGE_EXTERNAL_FRAMEWORK_PATH ${MYGE_EXTERNAL_ROOT}framework/)

find_library(XG_LIBRARY xg PATHS ${MYGE_EXTERNAL_LIBRARY_PATH} NO_CMAKE_FIND_ROOT_PATH NO_SYSTEM_ENVIRONMENT_PATH)
find_library(OPENDDL_LIBRARY OpenDDL PATHS ${MYGE_EXTERNAL_LIBRARY_PATH} NO_CMAKE_FIND_ROOT_PATH NO_SYSTEM_ENVIRONMENT_PATH)
find_library(OPENGEX_LIBRARY OpenGEX PATHS ${MYGE_EXTERNAL_LIBRARY_PATH} NO_CMAKE_FIND_ROOT_PATH NO_SYSTEM_ENVIRONMENT_PATH)
find_library(ZLIB_LIBRARY NAMES z zlib zlibd PATHS ${MYGE_EXTERNAL_LIBRARY_PATH} NO_CMAKE_FIND_ROOT_PATH NO_SYSTEM_ENVIRONMENT_PATH)
find_library(ISPCTEXCOMP_LIBRARY ispc_texcomp PATHS ${MYGE_EXTERNAL_LIBRARY_PATH} NO_CMAKE_FIND_ROOT_PATH NO_SYSTEM_ENVIRONMENT_PATH)

find_package(SDL2)
find_package(Vulkan)

if(Vulkan_FOUND)
set(HAS_VULKAN 1)
include_directories(${Vulkan_INCLUDE_DIRS})
endif(Vulkan_FOUND)

if(SDL2_FOUND)
set(HAS_SDL2 1)
endif(SDL2_FOUND)

include(CTest)

set(CPACK_PACKAGE_NAME "GEFS")
set(CPACK_PACKAGE_VENDOR "chenwenli.com")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "GEFS - Game Engine From Scratch")
set(CPACK_PACKAGE_VERSION "0.1.0")
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "1")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "GEFS")
SET(CPACK_NSIS_MODIFY_PATH ON)
include(CPack)

include(PlatformDependencies)
configure_file(${PROJECT_SOURCE_DIR}/config.h.in ${PROJECT_SOURCE_DIR}/config.h)
add_subdirectory(Framework)
add_subdirectory(Platform)
add_subdirectory(Physics)
add_subdirectory(RHI)
add_subdirectory(Test)
add_subdirectory(Game)
add_subdirectory(Viewer)
add_subdirectory(Asset)
add_subdirectory(Utility)

# ------------------------- Begin Generic CMake Variable Logging ------------------

# /*	C++ comment style not allowed	*/


# current generator in use 
MESSAGE( STATUS "CMAKE_GENERATOR:         " ${CMAKE_GENERATOR} )

# if you are building in-source, this is the same as CMAKE_SOURCE_DIR, otherwise 
# this is the top level directory of your build tree 
MESSAGE( STATUS "CMAKE_BINARY_DIR:         " ${CMAKE_BINARY_DIR} )

# if you are building in-source, this is the same as CMAKE_CURRENT_SOURCE_DIR, otherwise this 
# is the directory where the compiled or generated files from the current CMakeLists.txt will go to 
MESSAGE( STATUS "CMAKE_CURRENT_BINARY_DIR: " ${CMAKE_CURRENT_BINARY_DIR} )

# this is the directory, from which cmake was started, i.e. the top level source directory 
MESSAGE( STATUS "CMAKE_SOURCE_DIR:         " ${CMAKE_SOURCE_DIR} )

# this is the directory where the currently processed CMakeLists.txt is located in 
MESSAGE( STATUS "CMAKE_CURRENT_SOURCE_DIR: " ${CMAKE_CURRENT_SOURCE_DIR} )

# contains the full path to the top level directory of your build tree 
MESSAGE( STATUS "PROJECT_BINARY_DIR: " ${PROJECT_BINARY_DIR} )

# contains the full path to the root of your project source directory,
# i.e. to the nearest directory where CMakeLists.txt contains the PROJECT() command 
MESSAGE( STATUS "PROJECT_SOURCE_DIR: " ${PROJECT_SOURCE_DIR} )

# set this variable to specify a common place where CMake should put all executable files
# (instead of CMAKE_CURRENT_BINARY_DIR)
MESSAGE( STATUS "EXECUTABLE_OUTPUT_PATH: " ${EXECUTABLE_OUTPUT_PATH} )

# set this variable to specify a common place where CMake should put all libraries 
# (instead of CMAKE_CURRENT_BINARY_DIR)
MESSAGE( STATUS "LIBRARY_OUTPUT_PATH:     " ${LIBRARY_OUTPUT_PATH} )

# tell CMake to search first in directories listed in CMAKE_MODULE_PATH
# when you use FIND_PACKAGE() or INCLUDE()
MESSAGE( STATUS "CMAKE_MODULE_PATH: " ${CMAKE_MODULE_PATH} )

# this is the complete path of the cmake which runs currently (e.g. /usr/local/bin/cmake) 
MESSAGE( STATUS "CMAKE_COMMAND: " ${CMAKE_COMMAND} )

# this is the CMake installation directory 
MESSAGE( STATUS "CMAKE_ROOT: " ${CMAKE_ROOT} )

# this is the filename including the complete path of the file where this variable is used. 
MESSAGE( STATUS "CMAKE_CURRENT_LIST_FILE: " ${CMAKE_CURRENT_LIST_FILE} )

# this is linenumber where the variable is used
MESSAGE( STATUS "CMAKE_CURRENT_LIST_LINE: " ${CMAKE_CURRENT_LIST_LINE} )

# this is used when searching for include files e.g. using the FIND_PATH() command.
MESSAGE( STATUS "CMAKE_INCLUDE_PATH: " ${CMAKE_INCLUDE_PATH} )

# this is used when searching for libraries e.g. using the FIND_LIBRARY() command.
MESSAGE( STATUS "CMAKE_LIBRARY_PATH: " ${CMAKE_LIBRARY_PATH} )

## HOST
# the complete system name, e.g. "Linux-2.4.22", "FreeBSD-5.4-RELEASE" or "Windows 5.1" 
MESSAGE( STATUS "CMAKE_HOST_SYSTEM: " ${CMAKE_HOST_SYSTEM} )

# the short system name, e.g. "Linux", "FreeBSD" or "Windows"
MESSAGE( STATUS "CMAKE_HOST_SYSTEM_NAME: " ${CMAKE_HOST_SYSTEM_NAME} )

# only the version part of CMAKE_SYSTEM 
MESSAGE( STATUS "CMAKE_HOST_SYSTEM_VERSION: " ${CMAKE_HOST_SYSTEM_VERSION} )

# the processor name (e.g. "Intel(R) Pentium(R) M processor 2.00GHz") 
MESSAGE( STATUS "CMAKE_HOST_SYSTEM_PROCESSOR: " ${CMAKE_HOST_SYSTEM_PROCESSOR} )

## TARGET
# the complete system name, e.g. "Linux-2.4.22", "FreeBSD-5.4-RELEASE" or "Windows 5.1" 
MESSAGE( STATUS "CMAKE_SYSTEM: " ${CMAKE_SYSTEM} )

# the short system name, e.g. "Linux", "FreeBSD" or "Windows"
MESSAGE( STATUS "CMAKE_SYSTEM_NAME: " ${CMAKE_SYSTEM_NAME} )

# only the version part of CMAKE_SYSTEM 
MESSAGE( STATUS "CMAKE_SYSTEM_VERSION: " ${CMAKE_SYSTEM_VERSION} )

# the processor name (e.g. "Intel(R) Pentium(R) M processor 2.00GHz") 
MESSAGE( STATUS "CMAKE_SYSTEM_PROCESSOR: " ${CMAKE_SYSTEM_PROCESSOR} )

# is TRUE on all UNIX-like OS's, including Apple OS X and CygWin
MESSAGE( STATUS "UNIX: " ${UNIX} )

# is TRUE on Windows, including CygWin 
MESSAGE( STATUS "WIN32: " ${WIN32} )

# is TRUE on Apple OS X
MESSAGE( STATUS "APPLE: " ${APPLE} )

# is TRUE when using the MinGW compiler in Windows
MESSAGE( STATUS "MINGW: " ${MINGW} )

# is TRUE on Windows when using the CygWin version of cmake
MESSAGE( STATUS "CYGWIN: " ${CYGWIN} )

# is TRUE on Windows when using a Borland compiler 
MESSAGE( STATUS "BORLAND: " ${BORLAND} )

# is TRUE on PS4
MESSAGE( STATUS "ORBIS: " ${ORBIS} )

# Microsoft compiler 
MESSAGE( STATUS "MSVC: " ${MSVC} )
MESSAGE( STATUS "MSVC_IDE: " ${MSVC_IDE} )
MESSAGE( STATUS "MSVC60: " ${MSVC60} )
MESSAGE( STATUS "MSVC70: " ${MSVC70} )
MESSAGE( STATUS "MSVC71: " ${MSVC71} )
MESSAGE( STATUS "MSVC80: " ${MSVC80} )
MESSAGE( STATUS "CMAKE_COMPILER_2005: " ${CMAKE_COMPILER_2005} )


# set this to true if you don't want to rebuild the object files if the rules have changed, 
# but not the actual source files or headers (e.g. if you changed the some compiler switches) 
MESSAGE( STATUS "CMAKE_SKIP_RULE_DEPENDENCY: " ${CMAKE_SKIP_RULE_DEPENDENCY} )

# since CMake 2.1 the install rule depends on all, i.e. everything will be built before installing. 
# If you don't like this, set this one to true.
MESSAGE( STATUS "CMAKE_SKIP_INSTALL_ALL_DEPENDENCY: " ${CMAKE_SKIP_INSTALL_ALL_DEPENDENCY} )

# If set, runtime paths are not added when using shared libraries. Default it is set to OFF
MESSAGE( STATUS "CMAKE_SKIP_RPATH: " ${CMAKE_SKIP_RPATH} )

# set this to true if you are using makefiles and want to see the full compile and link 
# commands instead of only the shortened ones 
MESSAGE( STATUS "CMAKE_VERBOSE_MAKEFILE: " ${CMAKE_VERBOSE_MAKEFILE} )

# this will cause CMake to not put in the rules that re-run CMake. This might be useful if 
# you want to use the generated build files on another machine. 
MESSAGE( STATUS "CMAKE_SUPPRESS_REGENERATION: " ${CMAKE_SUPPRESS_REGENERATION} )


# A simple way to get switches to the compiler is to use ADD_DEFINITIONS(). 
# But there are also two variables exactly for this purpose: 

# the compiler flags for compiling C sources 
MESSAGE( STATUS "CMAKE_C_FLAGS: "                   ${CMAKE_C_FLAGS} )
MESSAGE( STATUS "CMAKE_C_FLAGS_DEBUG: "             ${CMAKE_C_FLAGS_DEBUG})
MESSAGE( STATUS "CMAKE_C_FLAGS_RELEASE: "           ${CMAKE_C_FLAGS_RELEASE})
MESSAGE( STATUS "CMAKE_C_FLAGS_RELWITHDEBINFO: "    ${CMAKE_C_FLAGS_RELWITHDEBINFO})
MESSAGE( STATUS "CMAKE_C_FLAGS_MINSIZEREL: "        ${CMAKE_C_FLAGS_MINSIZEREL})

# the compiler flags for compiling C++ sources 
MESSAGE( STATUS "CMAKE_CXX_FLAGS: "                 ${CMAKE_CXX_FLAGS} )
MESSAGE( STATUS "CMAKE_CXX_FLAGS_DEBUG: "           ${CMAKE_CXX_FLAGS_DEBUG} )
MESSAGE( STATUS "CMAKE_CXX_FLAGS_RELEASE: "         ${CMAKE_CXX_FLAGS_RELEASE} )
MESSAGE( STATUS "CMAKE_CXX_FLAGS_RELWITHDEBINFO: "  ${CMAKE_CXX_FLAGS_RELWITHDEBINFO} )
MESSAGE( STATUS "CMAKE_CXX_FLAGS_MINSIZEREL: "      ${CMAKE_CXX_FLAGS_MINSIZEREL} )

# Choose the type of build.  Example: SET(CMAKE_BUILD_TYPE Debug) 
MESSAGE( STATUS "CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE} )

# if this is set to ON, then all libraries are built as shared libraries by default.
MESSAGE( STATUS "BUILD_SHARED_LIBS: " ${BUILD_SHARED_LIBS} )

# the compiler used for C files 
MESSAGE( STATUS "CMAKE_C_COMPILER: " ${CMAKE_C_COMPILER} )

# the compiler used for C++ files 
MESSAGE( STATUS "CMAKE_CXX_COMPILER: " ${CMAKE_CXX_COMPILER} )

# if the compiler is a variant of gcc, this should be set to 1 
MESSAGE( STATUS "CMAKE_COMPILER_IS_GNUCC: " ${CMAKE_COMPILER_IS_GNUCC} )

# if the compiler is a variant of g++, this should be set to 1 
MESSAGE( STATUS "CMAKE_COMPILER_IS_GNUCXX : " ${CMAKE_COMPILER_IS_GNUCXX} )

# the tools for creating libraries 
MESSAGE( STATUS "CMAKE_AR: " ${CMAKE_AR} )
MESSAGE( STATUS "CMAKE_RANLIB: " ${CMAKE_RANLIB} )

MESSAGE( STATUS "CMAKE_CROSSCOMPILING: " ${CMAKE_CROSSCOMPILING} )

MESSAGE( STATUS "TARGET_ARCH: " ${TARGET_ARCH} )

MESSAGE( STATUS "XG_LIBRARY: " ${XG_LIBRARY} )
MESSAGE( STATUS "OPENDDL_LIBRARY: " ${OPENDDL_LIBRARY} )
MESSAGE( STATUS "OPENGEX_LIBRARY: " ${OPENGEX_LIBRARY} )
MESSAGE( STATUS "ZLIB_LIBRARY: " ${ZLIB_LIBRARY} )
MESSAGE( STATUS "ISPCTEXCOMP_LIBRARY: " ${ISPCTEXCOMP_LIBRARY} )

MESSAGE( STATUS "BULLET_COLLISION_LIBRARY: " ${BULLET_COLLISION_LIBRARY} )
MESSAGE( STATUS "BULLET_DYNAMICS_LIBRARY: " ${BULLET_DYNAMICS_LIBRARY} )
MESSAGE( STATUS "BULLET_LINEARMATH_LIBRARY: " ${BULLET_LINEARMATH_LIBRARY} )

MESSAGE( STATUS "SDL2_INCLUDE_DIRS: " ${SDL2_INCLUDE_DIR} )
MESSAGE( STATUS "SDL2_LIBRARIES: " ${SDL2_LIBRARY} )

MESSAGE( STATUS "VULKAN_INCLUDE_DIRS: " ${Vulkan_INCLUDE_DIRS} )
MESSAGE( STATUS "VULKAN_LIBRARIES: " ${Vulkan_LIBRARIES} )

#
#MESSAGE( STATUS ": " ${} )

# ------------------------- End of Generic CMake Variable Logging ------------------

