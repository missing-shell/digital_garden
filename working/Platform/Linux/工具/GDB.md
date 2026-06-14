## vscode + cpp
- C配置同，仅`task.json`中的` "command": "E:\\Toolchinas\\MinGW\\bin\\g++.exe",`改为`gcc`
- 选择编辑器时，一定要明确选择的是gcc还是g++
#### 运行
`task.json`
- `"${file}",` 代表需要编译的所有.c文件
- `c++`和`cpp`不能相互识别
- `"${workspaceFolder}\\*.c++",` 可以单独指定文件，在同一行中使用逗号隔开
- 修改`"${fileDirname}\\${fileBasenameNoExtension}.exe"` 以便只生成一个test.exe文件
```json
  "args": [
                "-fdiagnostics-color=always",
                "-g",
                //"${file}",
                "${workspaceFolder}\\*.c++", //编译当前文件夹下的所有.cpp文件
                "-o",
                //"${fileDirname}\\${fileBasenameNoExtension}.exe"
                "${fileDirname}\\test.exe"//生成的可执行文件名称
            ],
```

#### 调试
`lanuch.json`
- ` "program": "${fileDirname}\\test.exe"` 同`task.json`文件中指定的生成的可执行文件
- `"miDebuggerPath": "/path/to/gdb",` 指定GDB位置，位置同`task.json`文件中的` "command": "E:\\Toolchinas\\MinGW\\bin\\g++.exe"`路径
```json
"configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            //"program": "输入程序名称，例如 ${workspaceFolder}/a.exe",
            "program": "${fileDirname}\\test.exe",//需要调试的可执行文件exe
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            //"miDebuggerPath": "/path/to/gdb",
            "miDebuggerPath":"E:\\Toolchinas\\MinGW\\bin\\gdb.exe",//gdb位置
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        }

    ]
```
## linux
- **(gdb) 管道启动**：这个配置包含了`pipeTransport`字段，指定了使用SSH通过管道启动GDB的方式。这意味着这个配置可能是为了在远程服务器上调试程序，通过SSH管道将调试命令发送到远程服务器上的GDB。
## cmake
