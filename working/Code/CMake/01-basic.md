# Concepts

## CMakeLists.txt

CMakeLists.txt is the file which should store all your CMake commands. When
cmake is run in a folder it will look for this file and if it does not exist cmake
will exit with an error.

```bash
mkdir build
cd build
cmake ..
make
```

## A-hello-cmake$ tree

```$ tree
.
├── CMakeLists.txt
├── main.cpp
```

```c
cmake_minimum_required(VERSION 3.5)
project (hello_world)
add_executable(hello_world main.cpp)
```

## B-hello-headers$ tree

```$ tree
.
├── CMakeLists.txt
├── include
│   └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```

```C
set(SOURCES
  src/Hello.cpp
  src/main.cpp
)
add_executable(${PROJECT_NAME} ${SOURCES})

// # Including Directories (设置静态库的头文件目录)

target_include_directories(target
    PRIVATE
    ${PROJECT_SOURCE_DIR}/include
)
```

- +PRIVATE+ - 该目录被添加到该目标的包含目录中
- +INTERFACE+ - 该目录被添加到链接此库的任何目标的包含目录中。
- +PUBLIC+ - 如上所述，它包含在该库以及链接该库的任何目标中。

## Static

```$ tree
.
├── CMakeLists.txt
├── include
│   └── static
│       └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp```
```

```C
# 添加静态库
add_library(hello_library STATIC src/Hello.cpp)

# 设置静态库的头文件目录
target_include_directories(hello_library PUBLIC include)

# 添加可执行文件
add_executable(hello_binary src/main.cpp)

# 链接静态库到可执行文件
target_link_libraries(hello_binary PRIVATE hello_library)
```

## Share

```$ tree

.
├── CMakeLists.txt
├── include
│   └── shared
│       └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```

```c

# 添加共享库
cmake_minimum_required(VERSION 3.5)

project(hello_library)

############################################################
# Create a library
############################################################

#Generate the shared library from the library sources
add_library(hello_library SHARED
    src/Hello.cpp
)
#使用了ALIAS选项为共享库创建别名，用于提高代码的可读性和可维护性。这个别名允许您在其他地方引用共享库，而不是直接使用其原始名称。
add_library(hello::library ALIAS hello_library)

# 设置共享库的头文件目录
target_include_directories(hello_library
    PUBLIC
        ${PROJECT_SOURCE_DIR}/include
)

############################################################
# Create an executable
############################################################

# Add an executable with the above sources
add_executable(hello_binary
    src/main.cpp
)

# link the new hello_library target with the hello_binary target
target_link_libraries( hello_binary
    PRIVATE
        hello::library
)
```

## Install

```$ tree
.
├── cmake-examples.conf
├── CMakeLists.txt
├── include
│   └── installing
│       └── Hello.h
├── README.adoc
└── src
    ├── Hello.cpp
    └── main.cpp
```

```C

############################################################
# Create a library
############################################################

#Generate the shared library from the library sources
add_library(cmake_examples_inst SHARED
    src/Hello.cpp
)

target_include_directories(cmake_examples_inst
    PUBLIC
        ${PROJECT_SOURCE_DIR}/include
)

############################################################
# Create an executable
############################################################

# Add an executable with the above sources
add_executable(cmake_examples_inst_bin
    src/main.cpp
)

# link the new hello_library target with the hello_binary target
target_link_libraries( cmake_examples_inst_bin
    PRIVATE
        cmake_examples_inst
)

############################################################
# Install
############################################################

# Binaries
install (TARGETS cmake_examples_inst_bin
    DESTINATION bin)

# Library
# Note: may not work on windows
install (TARGETS cmake_examples_inst
    LIBRARY DESTINATION lib)

# Header files
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
    DESTINATION include)

# Config
install (FILES cmake-examples.conf
    DESTINATION etc)

```

## Set Default Build Type

The default build type provided by CMake is to include no compiler flags for
optimization. For some projects you may want to
set a default build type so that you do not have to remember to set it.

To do this you can add the following to your top level CMakeLists.txt

----

```C
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message("Setting build type to 'RelWithDebInfo' as none was specified.")
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)

//Set the possible values of build type for cmake-gui

set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()
```

----

## Set Default C++ Flags (complie flags)

The default `CMAKE_CXX_FLAGS` is either empty or contains the appropriate flags
for the build type.

To set additional default compile flags you can add the following to your top level CMakeLists.txt

----

```C
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)
```

----

Similarly to +CMAKE_CXX_FLAGS+ other options include:

- Setting C compiler flags using +CMAKE_C_FLAGS+
- Setting linker flags using +CMAKE_LINKER_FLAGS+.

### [NOTE]

====

The values `CACHE STRING "Set C++ Compiler Flags" FORCE` from the above command
are used to force this variable to be set in the CMakeCache.txt file.

Once set the +CMAKE_C_FLAGS+ and +CMAKE_CXX_FLAGS+ will set a compiler flag / definition globally for all targets in this directory or any included sub-directories. This method is not recommended for general usage now and the +target_compile_definitions+ function is preferred.

### Set CMake Flags

Similar to the build type a global C++ compiler flag can be set using the following methods.

- Using a gui tool such as ccmake / cmake-gui

image::cmake-gui-set-cxx-flag.png[cmake-gui set cxx flag]

- Passing into cmake

----

```C
cmake .. -DCMAKE_CXX_FLAGS="-DEX3"
```

----

## thild-party-libray

## compilling-with-clang

## building-with-ninja

## importanted-targets

## cpp-standard

### common-method

### cxx-standard

### complile-features
