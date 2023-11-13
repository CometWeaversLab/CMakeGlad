# CMake Glad
**CMake Glad** is a CMake script that provides a function to add [Glad](https://github.com/Dav1dde/Glad) as a dependency of one or more CMake projects.

**Glad** is an OpenGL/Vulkan loader library, which source files are generated through a python script depending on
a specified set of arguments such as the OpenGL/Vulkan API version, the GLX/EGL/WGL context creation API version,
and the list of OpenGL/Vulkan extensions.

The official Glad repository already provides the `glad_add_library()` CMake function. However, this function performs 
the source files generation during the CMake build step, which means that said files are not available until the first 
build of your project, causing your IDE to highlight errors for missing symbols, and making it more tedious to just 
experiment around with the source files generation parameters.

The function provided by **CMake Glad** solves this issue, moving the source files generation during the CMake configuration 
step, hence making them always available even when you open the project for the first time.

Keep in mind that both **Glad** and **CMake Glad** both requires a python 3.7+ interpreter to be installed on your system.

## Usage
Suppose your CMake project has a structure similar to the following one,
with `Glad/` as a git submodule:

```
MyProject/
├── .Git/
├── Source/
│   └── Main.c
├── Dependencies/
│   └── Glad/
└── CMakeLists.txt
```

From the root directory of your project, you can issue the following command:

> $ git submodule add https://github.com/CometWeaversLab/CMakeGlad Dependencies/CMakeGlad/

Then modify your `CMakeLists.txt/` for it to look like something like this:

```cmake
project(MyProject
	VERSION 1.0
	LANGUAGES C
)
add_executable(MyProjectExecutable "main.c")

# Include CMake Glad and customize its properties
include("Dependencies/CMakeGlad/CMakeLists.txt")
set(GLAD_GIT_SUBMODULE_PATH "${PROJECT_SOURCE_DIR}/Dependencies/Glad" CACHE INTERNAL "" FORCE) 
set(GLAD_LIBRARY_MODE    "STATIC" CACHE INTERNAL "" FORCE)
set(GLAD_API_GL_PROFILE  "CORE"   CACHE INTERNAL "" FORCE)
set(GLAD_API_GL_VERSION  "4.3"    CACHE INTERNAL "" FORCE)
set(GLAD_API_WGL_VERSION "1.0"    CACHE INTERNAL "" FORCE)
set(GLAD_API_EXTENSIONS  "WGL_ARB_create_context" CACHE INTERNAL "" FORCE)

# Create the "Glad" target (will generate source files)
target_create_glad()

# Add the "Glad" target as a dependency of your project
target_add_dependency_glad(MyProject)
```

## Configuration
**CMake Glad** provides different configurable properties, most of which are strictly related to the parameters that **Glad** uses for
the source file generation. 
Please refer to the documentation of the [official project](https://github.com/Dav1dde/Glad) for more informations.

| Name                    | Default   | Description                                                                                                   |
| :---------------------- | :-------- | :------------------------------------------------------------------------------------------------------------ |
| PYTHON_INTERPRETER_PATH | automatic | Absolute path of the Python 3.7+ interpreter                                                                  |
| GLAD_GIT_SUBMODULE_PATH | ""        | Absolute path of the Git submodule for Glad                                                                   |
| GLAD_TARGTET_NAME       | "Glad"    | Display name for the Glad target. Can be used to customize the name (example: all lowercase)                  |
| GLAD_API_GL_PROFILE     | "CORE"    | Profile of the OpenGL rendering API to be provided by Glad                                                    |
| GLAD_API_GL_VERSION     | "4.6"     | Version of the OpenGL rendering API to be provided by Glad                                                    |
| GLAD_API_GLES1_VERSION  | "NONE"    | Version of the OpenGL ES (Embedded Systems) rendering API to be provided by Glad                              |
| GLAD_API_GLES2_VERSION  | "NONE"    | Version of the OpenGL ES2 (Embedded Systems) rendering API to be provided by Glad                             |
| GLAD_API_GLSC2_VERSION  | "NONE"    | Version of the OpenGL ES2 SC (Embedded Systems Safety-Critical) rendering API to be provided by Glad          |
| GLAD_API_VULKAN_VERSION | "NONE"    | Version of the Vulkan API to be provided by Glad                                                              |
| GLAD_API_EGL_VERSION    | "NONE"    | Version of the EGL context creation API to be provided by Glad                                                | 
| GLAD_API_GLX_VERSION    | "NONE"    | Version of the GLX (OpenGL extension for X11) context creation API to be provided by Glad                     |
| GLAD_API_WGL_VERSION    | "NONE"    | Version of the WGL (OpenGL extension for Windows) context creation API to be provided by Glad                 |
| GLAD_API_EXTENSIONS     | ""        | List of GL/EGL/WGL/GLX/VK extensions API to be provided by Glad (comma-separated)                             |
| GLAD_OPT_ALIAS          | OFF       | Enables function pointer aliasing                                                                             |
| GLAD_OPT_DEBUG          | OFF       | Enable generation of a debug build                                                                            |
| GLAD_OPT_HEADER_ONLY*   | ON        | Generate a header only version of Glad                                                                        |
| GLAD_OPT_LOADER         | OFF       | Include internal loaders for APIs                                                                             |
| GLAD_OPT_MERGE          | OFF       | Merge multiple APIs of the same specification into one file                                                   |
| GLAD_OPT_MX             | OFF       | Enables support for multiple GL contexts                                                                      |
| GLAD_OPT_MX_GLOBAL      | OFF       | Mimic global GL functions with context switching                                                              |
| GLAD_OPT_ON_DEMAND      | OFF       | On-demand function pointer loading, initialize on use (experimental)                                          |
| GLAD_OPT_REPRODUCIBLE   | ON        | Makes the generation of Glad source files reproducible by not fetching the latest specification from Khronos  |
| GLAD_OPT_QUIET          | ON        | Disable logging during generation of Glad source files                                                        |