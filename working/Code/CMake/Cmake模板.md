参考：[知乎 PeLi](https://zhuanlan.zhihu.com/p/635986515 )
```cmake
cmake_minimum_required(VERSION 3.24) project(ExampleProject) add_executable(ExampleProject) target_sources(ExampleProject PRIVATE ${PROJECT_SOURCE_DIR}/src/main.cpp) target_compile_definitions(ExampleProject PRIVATE -DFOO=1 -DBAR=2) target_include_directories(ExampleProject PRIVATE ${PROJECT_SOURCE_DIR}/external/include) target_link_libraries(ExampleProject PRIVATE ${PROJECT_SOURCE_DIR}/external/lib/libfoo.a)
```
