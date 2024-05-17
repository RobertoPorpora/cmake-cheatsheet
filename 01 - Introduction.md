# Brief Introduction to CMake

## Why Might CMake Interest Me?

1. It is widely used and supported by any IDE (even VSCode via some extensions). If you want to use a C or C++ library, in 99% of cases, you will encounter CMake.
2. It is the closest thing today to a package manager (like npm, pip, and cargo) for C and C++.
3. It is practically the only way to achieve true cross-platform portability of your C/C++ project. If set up correctly, it can recompile your project for any platform without modifying any line of code.
4. There are countless other reasons, for example, it is quite easy to:
    - integrate unit testing
    - create installers
    - integrate Git
    - etc...

## How Does It Work?

### The Problem

Consider you have written the source code of a C program, so you have a source code folder like this:
```
project
    src
        main.c
        file1.h
        file1.c
        file2.h
        file2.c
```

To compile this, you need to perform a series of steps on the command line, typically:
- Compile main.c
- Compile file1.c
- Compile file2.c
- Link all these files, adding any external libraries as needed.

During development, you will likely do this operation a million times, once for every modification.  
The first thing that comes to mind in this situation is to automate with a build script, i.e., a **makefile**.

The script to compile and run would look like this:

```bat
:: change cwd (current working directory) to this script's directory
cd /D "%~dp0"

:: cwd = project folder
cd ..

:: create a directory named 'build' ('2>nul' = 'hide errors')
mkdir build 2>nul

:: compile all .c files from cwd and create an .exe into 'build'
:: 'compiling' means 'preprocessing, compiling, assembly, and linking'
gcc *.c -o .\build\hello.exe

:: run the .exe
.\build\hello.exe

:: stop to show the output before closing the window
pause
```

The script to compile in debug mode would be slightly different:
```bat
:: change cwd (current working directory) to this script's directory
cd /D "%~dp0"

:: cwd = project folder
cd ..

:: create a directory named 'build' if it doesn't already exist
mkdir build 2>nul

:: compile all .c files from cwd and create an .exe into 'build'
:: 'compiling' means 'preprocessing, compiling, assembly, and linking'
:: -g generates debug information needed by GDB debugger
gcc -g *.c -o .\build\hello.exe
```

This approach is already not bad, but it has a series of inefficiencies:
1. You need a different makefile for each configuration (debug mode, run mode, release mode, other compilers...)
2. You need a set of different makefiles for each operating system.
3. You need to write makefiles for each external dependency (library) that needs to be compiled for the specific mode and operating system.
4. There could be more than one compilation output: for example, 2 shared libraries compiled and 1 executable.

### The Solution

CMake provides a scripting language to abstractly describe how the project should be compiled.  
The purpose of CMake is not to "compile the project" but rather:

- Generate the makefiles:
```sh
cmake -S src -B build -G "MinGW Makefiles"
# meaning:
# cmake                --> call the CMake program installed on the PC
# -G "MinGW Makefiles" --> generate the makefiles for MinGW
# -B build             --> put the makefiles in the "build" folder
# -S src               --> the sources to be compiled are in the "src" folder.
```

- Provide easy access to the build command.
```sh
cmake --build
# meaning:
# CMake, please start the build
# (of the last project for which you generated the makefiles)
```

- Other super-cool features that are not fundamental to get started.

Once you have a *CMakeLists.txt* script for a project, CMake can generate and regenerate the makefiles for any configuration.

To regenerate the MakeFiles, it is sufficient to change the command line, for example:
```sh
cmake -S src -B buildunix -G "Unix Makefiles"
# meaning:
# generate the makefiles for Unix
# and put them in the "buildunix" folder.
# the sources to be compiled are in the "src" folder.
```

or:
```sh
cmake -S src -B cbproj -G "CodeBlocks - Ninja"
# meaning:
# generate a CodeBlocks project to be compiled with Ninja
# and put everything in the "cbproj" folder
# the sources to be compiled are in the "src" folder.
```

The user can even forget about the makefiles (which become just temporary products of the build).  
Moreover, if the compiler is installed on the system (hopefully...), it can be triggered directly by CMake.  
For example, I could ask CMake to generate files for Visual Studio Community (similarly to how we saw above) and then launch the build with `cmake --build` and the files will be compiled without even opening Visual Studio (which, however, must obviously be installed on the PC with the appropriate workload for what is being asked).  
The user only needs to:
1. maintain a single CMake script (which will travel with the sources)
2. run CMake specifying
    - compiler (gcc, visual studio, xcode, clang, ...)
    - mode (release, debug, optimized, ...)
    - destination folder for the generated makefiles
    - any options provided through the script itself

## Typical Basic Project Structure?

Below is a suggestion for structuring a project with CMake.
All paths are configurable, and not all projects follow this standard.
However, this is the structure I have seen most often (with these specific folder names), and I find it quite sensible.

```
project
    CMakeLists.txt
    src
        main.c
        file1.h
        file1.c
        file2.h
        file2.c
    vendor
        Lib001
            CMakeLists.txt
            src
        Lib002
            CMakeLists.txt
            src
            vendor
        Lib003
            CMakeLists.txt
            src
    build
    bin
```

*project* is the root directory containing the entire project.  
Inside it, there are 5 key elements:
- *CMakeLists.txt* is the CMake script containing the initial instructions to compile the content of the *project* folder.  
All CMake scripts MUST HAVE this specific name, case-sensitive, and with a .txt extension.
- *src* is the folder containing the source files written by the project author.
- *vendor* is the folder containing external libraries (dependencies) needed to compile *project*.
- *build* is a folder intended to receive the output of the CMake script, i.e., the makefiles and everything needed for the build.  
The user can mostly ignore the content of this folder as the files present in it are generally temporary.  
For those using Git, this folder is one of the first things to add to .gitignore.
- *bin* is the folder intended for the compilation output.

As mentioned, *vendor* is the folder containing external libraries (dependencies).  
Suppose *Lib001*, *Lib002*, and *Lib003* are 3 different libraries downloaded from GitHub.  
As you can see, all libraries have their own src folder of source files and a CMakeLists.txt script.  
Additionally, *Lib002* also has its own *vendor* folder with further sub-dependencies structured similarly.

This situation is ideal for CMake because the project author only needs to write in their own *project/CMakeLists.txt* file the calls to the 3 files:
- *project/vendor/Lib001/CMakeLists.txt*
- *project/vendor/Lib002/CMakeLists.txt*
- *project/vendor/Lib003/CMakeLists.txt*

The project author does not need to worry much about the content of these files.  
At most, they will need to verify the names of the produced outputs.  
And possibly select some simple high-level options provided by the scripts themselves (usually designed by the library authors with maximum portability and simplicity in mind).

In practice, the author only needs to worry about writing *project/CMakeLists.txt*.  
This file, excluding the necessary comments for explanation, could consist of only 8 instructions:

```CMake
#######################################
# lines preceded by '#' are comments ignored by the script executor.
# inline comments after instructions are also allowed.
#######################################

#######################################
# define a minimum cmake version required to execute this script
#######################################

cmake_minimum_required(VERSION 3.26)

#######################################
# name the project 'project'
#######################################

project(project)

#######################################
# declare that the 'vendor/Lib001' folder contains another 'CMakeLists.txt' file
# it MUST be executed before proceeding.
# probably it is a dependency needed to compile 'project'.
# same for Lib002 and Lib003
#######################################

add_subdirectory(vendor/Lib001)
add_subdirectory(vendor/Lib002)
add_subdirectory(vendor/Lib003)

#######################################
# define an executable target named 'project'.
# i.e., the build output will be an executable named 'project(.exe)'.
# this

 file should be obtained by compiling the files in src.
#######################################

add_executable(
project
    src/main.c
    src/file1.h
    src/file1.c
    src/file2.h
    src/file2.c
)

#######################################
# the 'project' target depends on other targets generated at this point.
# each add_subdirectory() has run a script.
# it is assumed that:
# - vendor/Lib001 generated the 'lib1' library target
# - vendor/Lib002 generated the 'lib2' library target
# - vendor/Lib003 generated the 'lib3' library target
# ignore PRIVATE, it will be explained later.
# essentially declaring that:
# - 'lib1' is a (private) dependency of 'project'
# - 'lib2' is a (private) dependency of 'project'
# - 'lib3' is a (private) dependency of 'project'

target_link_libraries(
project
    PRIVATE lib1
    PRIVATE lib2
    PRIVATE lib3
)

#######################################
# request to use the 'bin' folder
# as the output for the compilation of the 'project' target

set_target_properties(
project
    PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin"
)
```