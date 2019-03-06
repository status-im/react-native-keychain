set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_TYPE_NAMES ${REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_TYPE_NAMES}
                                                     \"RNKeychainManager\" PARENT_SCOPE)

set(REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_SRC ${REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_SRC}
                                              ${CMAKE_CURRENT_SOURCE_DIR}/rnkeychainmanager.cpp PARENT_SCOPE)

include(${CMAKE_ROOT}/Modules/ExternalProject.cmake)

set(qtkeychain_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/qtkeychain_ep-prefix/src/qtkeychain_ep)

set(qtkeychain_INCLUDE_DIR ${qtkeychain_PREFIX}/include/qt5keychain)
set(qtkeychain_STATIC_LIB ${qtkeychain_PREFIX}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}qt5keychain${CMAKE_STATIC_LIBRARY_SUFFIX})
set(qtkeychain_CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${qtkeychain_PREFIX}
                          -DCMAKE_INSTALL_LIBDIR=lib
                          -DQTKEYCHAIN_STATIC=ON
                          -DBUILD_TRANSLATIONS=OFF
                          ${qtkeychain_CMAKE_ARGS})
set(qtkeychain_CMAKE_ARGS ${qtkeychain_CMAKE_ARGS}
                          "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}"
                          "-DCMAKE_C_COMPILER_AR=${CMAKE_C_COMPILER_AR}"
                          "-DCMAKE_C_COMPILER_RANLIB=${CMAKE_C_COMPILER_RANLIB}"
                          "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}"
                          "-DCMAKE_CXX_COMPILER_AR=${CMAKE_CXX_COMPILER_AR}"
                          "-DCMAKE_CXX_COMPILER_RANLIB=${CMAKE_CXX_COMPILER_RANLIB}"
                          "-DCMAKE_LINKER=${CMAKE_LINKER}")
if (CMAKE_CROSSCOMPILING)
  set(qtkeychain_CMAKE_ARGS ${qtkeychain_CMAKE_ARGS}
                            "-DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}"
                            "-DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}"
                            "-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}"
                            "-DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS}"
                            # These are only useful if you're cross-compiling.
                            # They, however, will not hurt regardless.
                            "-DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}"
                            "-DCMAKE_SYSTEM_PROCESSOR=${CMAKE_SYSTEM_PROCESSOR}"
                            "-DCMAKE_AR=${CMAKE_AR}"
                            "-DCMAKE_RC_COMPILER=${CMAKE_RC_COMPILER}"
                            "-DCMAKE_EXE_LINKER_FLAGS=${CMAKE_EXE_LINKER_FLAGS}"
                            "-DCMAKE_COMPILER_PREFIX=${CMAKE_COMPILER_PREFIX}"
                            "-DCMAKE_FIND_ROOT_PATH=${CMAKE_FIND_ROOT_PATH}")
endif()

ExternalProject_Add(qtkeychain_ep
  GIT_REPOSITORY https://github.com/status-im/qtkeychain.git
  GIT_TAG d3c606c55adf8c2c2747556055652b3469f6c4c2
  GIT_SHALLOW TRUE
  CMAKE_ARGS ${qtkeychain_CMAKE_ARGS}
  CMAKE_CACHE_ARGS "-DBUILD_TRANSLATIONS:BOOL=OFF"
  BUILD_BYPRODUCTS ${qtkeychain_STATIC_LIB}
  LOG_BUILD 1
  LOG_DOWNLOAD 1
  LOG_INSTALL 1
)

if (WIN32)
  find_package(Qt5 COMPONENTS Core REQUIRED)
  set(qtkeychain_DEPS Qt5::Core)
  set(qtkeychain_USED_QT5_MODULES ${qtkeychain_USED_QT5_MODULES} Core)
endif()

if (UNIX AND NOT APPLE)
  find_package(Qt5 COMPONENTS DBus REQUIRED)
  set(qtkeychain_DEPS Qt5::DBus)
  set(qtkeychain_USED_QT5_MODULES ${qtkeychain_USED_QT5_MODULES} DBus)
endif()

if(DEFINED qtkeychain_USED_QT5_MODULES)
  foreach(COMP ${qtkeychain_USED_QT5_MODULES})
    set(USED_QT_MODULES ${USED_QT_MODULES} ${COMP} PARENT_SCOPE)
    set(REACT_NATIVE_DESKTOP_EXTERNAL_QT_COMPONENTS ${REACT_NATIVE_DESKTOP_EXTERNAL_PROJECT_DEPS} ${COMP} PARENT_SCOPE)
  endforeach(COMP ${qtkeychain_USED_QT5_MODULES})
endif()

set(REACT_NATIVE_DESKTOP_EXTERNAL_PROJECT_DEPS ${REACT_NATIVE_DESKTOP_EXTERNAL_PROJECT_DEPS} qtkeychain_ep PARENT_SCOPE)

set(REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_LIBS ${REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_LIBS}
  ${qtkeychain_STATIC_LIB} ${qtkeychain_DEPS} PARENT_SCOPE)

set(REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_INCLUDE_DIRS ${REACT_NATIVE_DESKTOP_EXTERNAL_MODULES_INCLUDE_DIRS}
  ${qtkeychain_INCLUDE_DIR} PARENT_SCOPE)
