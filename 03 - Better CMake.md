# Better CMake
*Content by Jefferson Amstutz, Codeotaku.*  
*Summary by Roberto Porpora.*

## Episode 1: Basic Project Setup and Usage
> VIDEO: https://www.youtube.com/watch?v=ffwB60oKr-w

With the installation of CMake, two graphical user interfaces are provided:
- ccmake
- cmake-gui

Both allow setting options and cache variables to customize the build script output.

### Command Line Examples:

```cmake -G ninja ..``` executes CMake, specifying to generate makefiles for 'ninja' and to look for the 'CMakeLists.txt' script in the '..' directory, which is the parent directory of the current working directory.

```cmake --build``` starts the build using the last system for which the makefiles were generated.

## Episode 2: Functions & Macros
> VIDEO: https://www.youtube.com/watch?v=4BvyJGfFKNg

> Knowing how to use functions in CMake is not essential.  
> If you're in a hurry, you can skip directly to episode 3, where more important fundamental concepts are covered, and come back here later.  
> This section is mainly useful for understanding other projects (especially public libraries you might want to use as dependencies).

```CMake
##############################
# Comments in CMake start with '#'
# They are ignored during script execution
##############################

##############################
# The project is named 'hello'...
##############################

project(hello)

##############################
# Declare a function named 'print' with 1 argument named 'var'
##############################

function(print var)

	##############################	
	# Function body
	##############################
	
	##############################
	# Display a message on the console during compilation
	# ${var} means 'the value of var'
	# ${${var}} means 'the value of the value of var'
	# The key takeaway is that ${var} returns a string.
	# This string is also the name of a variable within the script
	# It will be clearer with an example later on
	##############################

	message("${var} = ${${var}}")
	
	##############################
	# End of the function body
	##############################

endfunction()

##############################
# Example
# ${var} returns 'PROJECT_NAME'
# ${${var}} returns ${PROJECT_NAME}, which in turn returns 'hello' (as above)
# So, the console will display 'PROJECT_NAME = hello'
##############################

print(PROJECT_NAME)
```

### Notable Variables:
```${ARGN}``` is the list of all function arguments.  
```${ARGC}``` is the count of all function arguments.  
```${ARGV0}``` is the first function argument.  
```${ARGV1}``` is the second function argument.  
and so on...

```CMake
foreach(var ${ARGN})
	# This loop runs ${ARGC} times.
	# It is implicit that:
	#     the 1st time, var = ${ARGV0}
	#     the 2nd time, var = ${ARGV1}
	#     the 3rd time, var = ${ARGV2}
	#     and so on...
endforeach()
```

```$ENV{myvar}``` Looks up an environment variable named 'myvar'.  
The environment variable could have been declared in the terminal with ```export myvar = s```  
The tutorial is done on Linux, I'm not sure if it's the same for other operating systems.

## Episode 3: Targets
> VIDEO: https://www.youtube.com/watch?v=flTCHZ92Uak

There are 2 types of 'targets':
1. executable
2. library (which itself can be of 3 types)
	- static (library will be embedded in an executable)
	- shared (library will be an external .dll that the executable will point to)
	- module (almost never used)

Example of usage:
```CMake
add_executable(
tizio
	main.c
	file1.c
	file1.h
	file2.c
	file2.h
)
target_link_libraries(tizio PRIVATE caio)
```

In the example, an executable target named 'tizio' is created with add_executable().  
The target must be built by compiling the source files 'main.c', 'file1.c', 'file1.h', 'file2.c', and 'file2.h'.  
Additionally, the 'tizio' target must privately link to the 'caio' target.  
The meaning of this last sentence will be clearer later on.  
**IMPORTANT NOTE # 1:** 'caio' is a TARGET!  
'caio' must have been created somewhere with an instruction like 'add_library(caio)'.  
**IMPORTANT NOTE # 2:** 'tizio' points to 'caio' in *PRIVATE mode*.  
There are 3 *modes*:
1. ```PRIVATE``` = 'caio' will be used only by 'tizio'.
2. ```INTERFACE``` = 'caio' will be used only by targets that link to 'tizio', but 'tizio' won't use 'caio'.
3. ```PUBLIC``` = PRIVATE + INTERFACE

Summary example:
```CMake
target_link_libraries(t1 INTERFACE t2 t3)
target_link_libraries(t4 PRIVATE t1)
target_link_libraries(t4 PUBLIC t5)
target_link_libraries(t6 PRIVATE t4)
```
The result of this will be that:
- t1, t2, and t3 won't use any library.
- t4 will use the libraries t2, t3, and t5.
- t5 won't use any library.
- t6 will use the libraries t4 and t5.

The same syntax and rules apply to 'target_include_directories()'.

## add_subdirectory()

This is not part of the video playlist, but I think it's important at this point to add a couple of basic commands.

**add_subdirectory(path_to_folder)** adds the *path_to_folder* to the project.
What does this mean?
1. There MUST be a 'CMakeLists.txt' file inside path_to_folder.
2. The script from point 1 will be executed along with the script that called it with add_subdirectory().
3. This allows creating a tree of scripts where the executable or library you want to build calls other libraries as dependencies (subfolders).
4. Ideally, each library should have its own CMake project configured with one or more targets that can be pointed to by the main project that wants to use the library as a dependency.  
I continue with an example of integrating the SDL library, which should clarify things further.

## SDL2 + CMake
> VIDEO: https://www.youtube.com/watch?v=Mi47TQ4Tsr8

I have a project called 'Pippo', an app based on SDL, with the following folder structure:

```
Pippo
  CMakeLists.txt
  src
    CMakeLists.txt
    main.c
  vendor
    CMakeLists.txt
    SDL
      ...a bunch of other stuff,
	  but it's just what you download from GitHub 
	  without any modifications...
```

> The name 'vendor' is typical for the folder that collects the sources of all external dependencies, typically created by others, indeed, vendors.  
> 'src' stands for project-specific sources of Pippo.


### Pippo/CMakeLists.txt

```CMake
cmake_minimum_required(VERSION 3.26)
project(Pippo)

# Execute the file inside the 'vendor' folder
add_subdirectory(vendor)

# Execute the file inside the 'src' folder
add_subdirectory(src)

# The rest is unrelated to what I'm explaining, it just creates a 'bin' folder dedicated to the compiled executable
set_target_properties(${PROJECT_NAME}
    PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin"
)
```

### Pippo/vendor/CMakeLists.txt

```CMake
# Create a virtual target that only serves to group all dependencies into a single library
add_library(vendor INTERFACE)

# The next 2 commands are unrelated to what I'm explaining but serve to force, specifically for SDL, compilation as a static library
option (BUILD_SHARED_LIBS "Build shared libraries" ON)
set(BUILD_SHARED_LIBS OFF)

# Link to the 'vendor' target all targets to be linked to my sources later
target_link_libraries(vendor
	INTERFACE SDL3::SDL3  # here I'm hooking up to the targets generated by the SDL CMake scripts... you have to read the SDL documentation for specifics...
	# here I could go on listing all my other dependencies, one per line.
	# but in this case I have only one.
)

# I also need to add all the subdirectories that contain the dependencies, so they get compiled
add_subdirectory(SDL)
```

### Pippo/src/CMakeLists.txt

```CMake
# PROJECT_NAME obviously refers to the project 'Pippo' that called this file as a subdirectory
add_executable(${PROJECT_NAME})

# This is an alternative way to attach source files
# along with the line above, it is totally equivalent to 'add_executable(${PROJECT_NAME} main.c)'
target_sources(${PROJECT_NAME} PRIVATE	main.c)

# here I finally point to the 'vendor' aggregator
target_link_libraries(${PROJECT_NAME}
	PRIVATE vendor
)
```

### Pippo/src/main.c

```c
// it's not ideal to write sources like this.
// but this should compile and work.

#include <SDL.h>
int main() {
	SDL_Log("%s","hello world");
	return 0;
}
```

With this, you should have everything you need to start creating your CMake projects.  
Of course, there are a lot of other useful things, but if you're in a hurry, you can stop here.

### Specific Notes for the SDL Library

```SDL3::SDL3``` is an alias of ```SDL3::SDL3-static``` or ```SDL3::SDL3-shared``` .  
The differentiator is the ```BUILD_SHARED_LIBS``` variable which by default is ```ON``` .  
So, the default selection is ```SDL3::SDL3-shared``` .

It is generally suggested (without specifying the reasons) to:
- Compile SDL with Visual Studio instead of MinGW.
- Compile with ```BUILD_SHARED_LIBS = ON```.

in case of shared compilation, it is suggested to add below ```project(pippo)``` :
```CMake
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
```

## Episode 4: find_package()

```find_package()``` is used to find targets that are outside the project and import them so they can be used.

Where the search is performed depends on the operating system.

It is expected that the package is somehow installed in the operating system, but not available within the project (otherwise it would be ```add_subdirectory()``` and not ```find_package()``` ).

In any case, ```find_package()``` searches for a *CMake config* and not the package itself.

For example: ```find_package(pippo)``` just searches in the filesystem for a file named exactly *pippoConfig.cmake*.

A CMake project can also give hints about where a required package might be found with ```find_package()``` .

In the project, there must be a folder named *CMake* and it must contain *findpippo.cmake*.

One way to tell CMake to use a specific folder to search for packages is to set the environment variable ```CMAKE_PREFIX_PATH = path_to_search``` .