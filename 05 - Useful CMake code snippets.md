# Useful CMake code snippets

```CMake

cmake_minimum_required(VERSION 3.14)



############
# sets the variable PROJECT_NAME = "AnyNameYouWant"
############

project(AnyNameYouWant)



############
# Finds all .cpp and .hpp filenames from 'src' folder
# and stores all these names into an array called 'srcArray'
############

file(GLOB
srcArray
    "src/*.hpp"
    "src/*.cpp"
)



############
# A safer way to add all source files from a folder.
# example
#   if ${PROJECT_NAME} is "johndoe"
#   then a varibale called "johndoe_sources" is created by ${PROJECT_NAME}_sources
#   so that ${${PROJECT_NAME}_sources} returns ${johndoe_sources}
#   and ${johndoe_sources} returns an array with all the .c and .h files from src folder
############

add_executable(${PROJECT_NAME})
file(GLOB ${PROJECT_NAME}_sources "src/*.c" "src/*.h")
target_sources(${PROJECT_NAME} PRIVATE "${${PROJECT_NAME}_sources}")



############
# copies directory 'path/to/source/folder'
# and its contents (files)
# into 'path/to/destination/folder'
############

file(
    COPY path/to/source/folder
    DESTINATION path/to/destination/folder
)


############
## Add raylib dependencies
############

find_package(raylib QUIET)
if (NOT raylib_FOUND)
    include(FetchContent)
    FetchContent_Declare(
        raylib
        GIT_REPOSITORY https://github.com/raysan5/raylib.git
        GIT_TAG 4.5.0
    )
    FetchContent_MakeAvailable(raylib)
endif()



############
## Add raylib-cpp dependencies
############

find_package(raylib_cpp QUIET)
if (NOT raylib_cpp_FOUND)
    include(FetchContent)
    FetchContent_Declare(
        raylib_cpp
        GIT_REPOSITORY https://github.com/RobLoach/raylib-cpp.git
        GIT_TAG v4.5.1
    )
    FetchContent_MakeAvailable(raylib_cpp)
endif()



## Create an executable called as 'PROJECT_NAME' created by compiling 'SOURCES'
add_executable(${PROJECT_NAME} ${SOURCES})

## Tell gcc that this is a GUI program and not a console program
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mwindows")

## Put the compiled binary into another subfolder called 'dist'
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/dist)

## Copy 'resources' folder into the compiled binary directory
add_custom_target(
    copy_resources ALL
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    ${PROJECT_SOURCE_DIR}/resources
    ${EXECUTABLE_OUTPUT_PATH}/resources
    COMMENT "Copying resources into binary directory"
)
add_dependencies(${PROJECT_NAME} copy_resources)

## I don't know...
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 11)
target_link_libraries(${PROJECT_NAME} PUBLIC raylib raylib_cpp)
    
## Web Configurations
if (${PLATFORM} STREQUAL "Web")

    ## Tell Emscripten to build an example.html file.
    set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".html")

    ## Required linker flags for using Raylib with Emscripten
    target_link_options(${PROJECT_NAME} PRIVATE -sEXPORTED_FUNCTIONS=['_main','_malloc'] -sEXPORTED_RUNTIME_METHODS=ccall -sUSE_GLFW=3)

endif()
```