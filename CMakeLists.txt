cmake_minimum_required(VERSION 3.13)
project(discord-rpc)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(DISCORD_RPC_SOURCES
  discord_rpc.h
  discord_register.h
  discord_rpc.cpp
  discord_rpc_connection.h
  discord_rpc_connection.cpp
  discord_serialization.h
  discord_serialization.cpp
  discord_connection.h
  discord_backoff.h
  discord_msg_queue.h
)

if(UNIX)
  list(APPEND DISCORD_RPC_SOURCES discord_connection_unix.cpp)
  if(APPLE)
    list(APPEND DISCORD_RPC_SOURCES discord_register_osx.m)
    add_definitions(-DDISCORD_OSX)
  else()
    list(APPEND DISCORD_RPC_SOURCES discord_register_linux.cpp)
    add_definitions(-DDISCORD_LINUX)
  endif()
endif()

if(WIN32)
  list(APPEND DISCORD_RPC_SOURCES discord_connection_win.cpp discord_register_win.cpp)
  add_definitions(-DDISCORD_WINDOWS)
endif()

add_library(${PROJECT_NAME} ${DISCORD_RPC_SOURCES})

set_target_properties(${PROJECT_NAME} PROPERTIES
  CMAKE_CXX_STANDARD 17
  CMAKE_CXX_STANDARD_REQUIRED ON
  PUBLIC_HEADER discord_rpc.h
)

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(APPLE)
  target_link_libraries(${PROJECT_NAME} PRIVATE "-framework AppKit")
endif()

if(WIN32)
  target_link_libraries(${PROJECT_NAME} PRIVATE psapi advapi32)
endif()

install(EXPORT ${PROJECT_NAME}Targets
  NAMESPACE ${PROJECT_NAME}::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

configure_package_config_file(${PROJECT_NAME}Config.cmake.in ${PROJECT_NAME}Config.cmake INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}Targets
  FRAMEWORK DESTINATION ${FRAMEWORK_INSTALL_DIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
