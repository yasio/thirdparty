set(lib_name openssl)

project(${lib_name})

include(../cmake/PlatformHelper.cmake)

if(NOT IS_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/prebuilt)
    set (package_version v49)
    set (package_name buildware_dist_${package_version})
    message(AUTHOR_WARNING "Downloading openssl prebuilt from https://github.com/axmolengine/buildware ...")
    if (NOT EXISTS ${CMAKE_CURRENT_LIST_DIR}/${package_name}.zip)
        file(DOWNLOAD https://github.com/axmolengine/buildware/releases/download/${package_version}/${package_name}.zip ${CMAKE_CURRENT_LIST_DIR}/${package_name}.zip)
    endif()
    file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_LIST_DIR}/${package_name}.zip DESTINATION ${CMAKE_CURRENT_LIST_DIR}/ PATTERNS ${package_name}/openssl/**)
    file(RENAME ${CMAKE_CURRENT_LIST_DIR}/${package_name}/openssl/include ${CMAKE_CURRENT_LIST_DIR}/include)
    file(RENAME ${CMAKE_CURRENT_LIST_DIR}/${package_name}/openssl/prebuilt ${CMAKE_CURRENT_LIST_DIR}/prebuilt)
    file(REMOVE_RECURSE ${CMAKE_CURRENT_LIST_DIR}/${package_name})
    file(REMOVE ${CMAKE_CURRENT_LIST_DIR}/${package_name}.zip)
endif()

# -----macro: set openssl sub target-----
macro(set_openssl_sub_target sub_target_name sub_lib_name)
  
  if(WINDOWS)
    add_library(${sub_target_name} SHARED IMPORTED GLOBAL)
  else()
    add_library(${sub_target_name} STATIC IMPORTED GLOBAL)
  endif()

  set_target_properties(${sub_target_name} PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
  )

  set(imp_lib_name "")
  if(WINDOWS)
    set(impl_lib_suffix "")
    if(WIN64)
        set(impl_lib_suffix "-${ARCH_ALIAS}")
    endif()
    set(imp_lib_name "${platform_spec_path}/lib${sub_lib_name}.lib")
    set_target_properties(${sub_target_name} PROPERTIES
      IMPORTED_LOCATION "${platform_spec_path}/lib${sub_lib_name}-3${impl_lib_suffix}.dll"
      IMPORTED_IMPLIB "${platform_spec_path}/lib${sub_lib_name}.lib"
    )
  else()
    set(imp_lib_name "${platform_spec_path}/lib${sub_lib_name}.a")
    set_target_properties(${sub_target_name} PROPERTIES
      IMPORTED_LOCATION "${platform_spec_path}/lib${sub_lib_name}.a"
    )
  endif()
  if(${sub_lib_name} STREQUAL "ssl")
    set(OPENSSL_ROOT_DIR "${platform_spec_path}" CACHE STRING "" FORCE)
    set(OPENSSL_SSL_LIBRARY "${imp_lib_name}" CACHE STRING "" FORCE)
  else()
    set(OPENSSL_CRYPTO_LIBRARY "${imp_lib_name}" CACHE STRING "" FORCE)
  endif()
endmacro()

# -----sub target 1: ssl-----
set(target_name OpenSSL::SSL ssl)
set_openssl_sub_target(${target_name})

# -----sub target 2: crypto-----
set(target_name OpenSSL::Crypto crypto)
set_openssl_sub_target(${target_name})

# -----store inc & lib dir
set(OPENSSL_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include" CACHE STRING "OpenSSL include dir" FORCE)
if (WINDOWS)
    set(OPENSSL_LIBRARY_DIR "${platform_spec_path}" CACHE INTERNAL "OpenSSL Library Directory" )
endif()
