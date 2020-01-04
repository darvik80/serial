cmake_minimum_required(VERSION 3.2)
project(serial)

message(${CMAKE_BINARY_DIR} "test")

if(EXISTS ${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
    include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
    conan_basic_setup(NO_OUTPUT_DIRS)
    set (MAIN_LIBS ${CONAN_LIBS})
else()
    message(WARNING "The file conanbuildinfo.cmake doesn't exist, you have to run conan install first")
endif()

if (APPLE)
    message("Apple build")
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
else()
    message("Unix build")
endif()

if(UNIX AND NOT APPLE)
    # If Linux, add rt and pthread
    set(rt_LIBRARIES rt)
    set(pthread_LIBRARIES pthread)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
)

if(APPLE)
    # If OSX
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})
if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

## Include headers
include_directories(include)

## Uncomment for example
add_executable(${PROJECT_NAME}-example examples/serial_example.cc)
add_dependencies(${PROJECT_NAME}-example ${PROJECT_NAME})
target_link_libraries(${PROJECT_NAME}-example ${PROJECT_NAME})

## Tests
enable_testing()
add_subdirectory(tests)

## Install executable
install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

## Install headers
install(FILES include/serial/serial.h include/serial/v8stdint.h DESTINATION include/serial)

