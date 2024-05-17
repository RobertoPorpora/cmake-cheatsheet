# CMake in Visual Studio Code
> Reference tutorial: https://www.youtube.com/watch?v=4gl2szb6d4Q  
> CMake Tools official documentation: https://github.com/microsoft/vscode-cmake-tools

## Setup
1. Install **MSYS2** and **mingw-w64 GCC**
    - Instructions here: https://www.msys2.org/
2. Install **CMake**
    - Execute this in cmd: ```winget install cmake```
3. Install Microsoft **Visual Studio Code** and add these extensions:
    - C/C++ (Microsoft)
    - CMake Tools (Microsoft)
    - CMake (twxs) [optional]

## Usage
1. Create an empty folder for your project.
2. Open VSCode in that folder.
3. View -> Command Palette.
4. Type "cmake", all cmake commands will appear:
    - "CMake: Quick Start" will set up your project, create a CMakeLists.txt file, a build folder, ecc...
    - "CMake: Configure" will execute cmake, it will create/update the build folder. This is auto-executed also when you save CMakeLists.txt
    - "CMake: Build" will build your project.
    - "CMake: Debug" will start the debugger.

## Suggested CmakeLists.txt preset
```CMake
cmake_minimum_required (VERSION 3.27)
project (YourProjectNameHere)
file (
    GLOB sourcefiles
    "src/*.h"
    "src/*.c"
)
add_executable(${PROJECT_NAME} ${sourcefiles})
```