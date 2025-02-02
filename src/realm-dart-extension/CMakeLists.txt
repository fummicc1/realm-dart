cmake_minimum_required(VERSION 3.15)

project(realm_dart_extension)

message ("Project: realm_dart_extension")
message ("CMAKE_BINARY_DIR is ${CMAKE_BINARY_DIR}")
message ("PROJECT_SOURCE_DIR is ${PROJECT_SOURCE_DIR}")
message ("DART_INCLUDE_DIR is ${DART_INCLUDE_DIR}")
message ("Configuration is ${CMAKE_CONFIGURATION_TYPES}")

set(SOURCES
    realm_dart_extension.cpp
    realm-js-common/js_realm.cpp
    dart/dart_types.cpp
    dart/dart_init.cpp
    dart/platform.cpp
)

set(HEADERS
)

if(CMAKE_SYSTEM_NAME MATCHES "^Windows")
    list(APPEND SOURCES win/realm_dart_extension_dllmain_win.cpp)
endif()

set (INCLUDE_DIRS
    realm-js-common
    dart
    ../object-store/src
    ../object-store/external/json
    #includes are common for Win and Mac
    ../vendor-include/realm-win-x64/include
    ../dart-include
)

add_library(realm_dart_extension SHARED ${SOURCES} ${HEADERS})

set_target_properties(realm_dart_extension PROPERTIES POSITION_INDEPENDENT_CODE 1)

if(APPLE)
    set_target_properties(realm_dart_extension PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
   
    set(CMAKE_CXX_VISIBILITY_PRESET hidden)
    set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)

    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden")
endif()

target_include_directories(realm_dart_extension PUBLIC ${DART_INCLUDE_DIR} ${HEADERS}, ${INCLUDE_DIRS})

target_compile_definitions(realm_dart_extension PUBLIC -DDART_SHARED_LIB)

if(CMAKE_SYSTEM_NAME MATCHES "^Windows")    
    if (CMAKE_CONFIGURATION_TYPES MATCHES "Debug") 
        set (BUILD_SUFFIX "-dbg") 
    endif()

    set (LINK_LIBS
        "${PROJECT_SOURCE_DIR}/../dart-include/lib/windows-x64/dart.lib"
        "${PROJECT_SOURCE_DIR}/../vendor-include/realm-win-x64/lib/object-store${BUILD_SUFFIX}.lib"
        "${PROJECT_SOURCE_DIR}/../vendor-include/realm-win-x64/lib/realm${BUILD_SUFFIX}.lib"
        "${PROJECT_SOURCE_DIR}/../vendor-include/realm-win-x64/lib/realm-parser${BUILD_SUFFIX}.lib"
    )
endif()

if(APPLE)    
    set (LINK_LIBS
        "${PROJECT_SOURCE_DIR}/../vendor-include/macos/object-store.a"
        "${PROJECT_SOURCE_DIR}/../vendor-include/macos/librealm-parser.a"
        "${PROJECT_SOURCE_DIR}/../vendor-include/macos/librealm.a"
    )
endif()

message("LINK_LIBS is ${LINK_LIBS}")

target_link_libraries(realm_dart_extension ${LINK_LIBS})

set_property(TARGET realm_dart_extension PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

if(CMAKE_SYSTEM_NAME MATCHES "^Windows")
    #copy the dll to the lib directory
    add_custom_command(TARGET realm_dart_extension POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_BINARY_DIR}/src/realm-dart-extension/$<CONFIGURATION>/realm_dart_extension.dll"  ${PROJECT_SOURCE_DIR}../../../lib/realm_dart_extension.dll)
    if (CMAKE_CONFIGURATION_TYPES MATCHES "Debug") 
        add_custom_command(TARGET realm_dart_extension POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_BINARY_DIR}/src/realm-dart-extension/$<CONFIGURATION>/realm_dart_extension.pdb"  ${PROJECT_SOURCE_DIR}../../../lib/realm_dart_extension.pdb)
    endif()

    #copy the dll to the root directory of realm-dart so VS Code dart test runner can run and debug the tests.
    add_custom_command(TARGET realm_dart_extension POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_BINARY_DIR}/src/realm-dart-extension/$<CONFIGURATION>/realm_dart_extension.dll"  ${PROJECT_SOURCE_DIR}../../../realm_dart_extension.dll)

    #copy the dll to the test directory of realm-dart so VS Code dart debug test runner can run and debug the tests. If the dll is not besides the tests then the library is not loaded correctly and the cached RealmLib types are not the same. Dart: investigate this
    add_custom_command(TARGET realm_dart_extension POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_BINARY_DIR}/src/realm-dart-extension/$<CONFIGURATION>/realm_dart_extension.dll"  ${PROJECT_SOURCE_DIR}../../../test/realm_dart_extension.dll)


    if (CMAKE_CONFIGURATION_TYPES MATCHES "Release") 
        #copy the Release dll to the bin directory of realm-dart so it gets into the realm_dart package
        add_custom_command(TARGET realm_dart_extension POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_BINARY_DIR}/src/realm-dart-extension/$<CONFIGURATION>/realm_dart_extension.dll"  ${PROJECT_SOURCE_DIR}../../../bin/realm_dart_extension.dll)
    endif()
endif()

if(APPLE)
    #copy the dylib to the bin directory of realm-dart so it gets into the realm_dart package
    add_custom_command(TARGET realm_dart_extension POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_BINARY_DIR}/src/realm-dart-extension/$<CONFIGURATION>/librealm_dart_extension.dylib"  ${PROJECT_SOURCE_DIR}../../../bin/librealm_dart_extension.dylib)
endif()