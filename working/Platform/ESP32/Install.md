# ESP-IDF 安装

## 参考

- [安装](https://www.bilibili.com/video/BV1ah4y177mR/?spm_id_from=333.788.top_right_bar_window_custom_collection.content.click&vd_source=0aba2f1927cca4db51a7dad74151f079)
- [入门](https://www.bilibili.com/video/BV1nR4y1o7VE/?spm_id_from=333.788.top_right_bar_window_custom_collection.content.click&vd_source=0aba2f1927cca4db51a7dad74151f079)

不建议参考[安装](https://www.bilibili.com/video/BV1ah4y177mR/?spm_id_from=333.788.top_right_bar_window_custom_collection.content.click&vd_source=0aba2f1927cca4db51a7dad74151f079)中设置项目`project`文件夹的方式，推荐放在非插件文件目录下，方便插件的卸载更新和项目代码的保留

网络参考步骤：

- 首先下载esp-idf(本人未采用)
- 设置用户环境变量(本人未采用)
- 安装vscode
- 安装esp-idf插件

本人步骤

- 安装vscode
- 安装插件
- 设置系统环境变量

### 非必要选项

#### 在Windows环境变量中设置PATH(安装插件后)

IDF_PATH

```C
e:\Espressif\v5.1.1\esp-idf
```

IDF_TOOLS_PATH

```C
e:\Espressif\v5.1.1\.espressif
```

#### vscode中出现不影响编译的报错

目前实践成功的解决方法

- 选择esp-idf框架中的编译器，而不是`mingw`之类
- 确保vscode中esp-idf的环境变量正确
- 手动设置系统环境变量
- 更新setting.json

## 调试

- 乐鑫官方文档：[JTAG 调试](https://docs.espressif.com/projects/esp-idf/zh_CN/v5.1.1/esp32c6/api-guides/jtag-debugging/index.html)

    首先参照文档设置。

    若在终端中无法运行相应，则设置环境变量。

- 设置环境变量

    使用`echo %OPENOCD_SCRIPTS%`查看路径，如果没有返回具体路径，需设置变量路径

    首先导航到相应路径

    设置系统变量 - **持久化环境变量**

    `OPENOCD_SCRIPTS`

    `E:\Espressif\v5.1.1\.espressif\tools\openocd-esp32\v0.12.0-esp32-20230419\openocd-esp32\bin`

    在`cmd`中输入`echo %OPENOCD_SCRIPTS%`,若返回上述路径，则说明设置成功

- 终端运行`OpenCDC`

  命令参照官方文档

  `openocd -f board/esp32c6-builtin.cfg`

### 使用`vscode+usb内部JTAG`调试步骤

- 以`windows`平台为例

    在`.vscode`目录下添加`launch.json`文件

    将`USB`线连接到开发板`USB`口

    选择对应串口

    在选择芯片之后选择`via USB-JTAG`

    选择`JTAG`或`UART`烧录方式

    编译、烧录（有时会提示是否开启`OpenCDC`，选择是则跳过下一步）

    通过底部导航开启`OpenCDC`

    选择断点

    `F5`调试

#### 在`.vscode`目录下添加`launch.json`文件

- 乐鑫文档 [Configuration for Visual Studio Code Debug](https://github.com/espressif/vscode-esp-idf-extension/blob/master/docs/DEBUGGING.md)

    复制`Use Microsoft C/C++ Extension to Debug`中的内容粘贴到`launch.json`。

    更改`"name": "GDB",`中的`GDB`为芯片型号

- 仅供参考

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "ESP32-C6",
      "type": "cppdbg",
      "request": "launch",
      "MIMode": "gdb",
      "miDebuggerPath": "${command:espIdf.getXtensaGdb}",
      "program": "${workspaceFolder}/build/${command:espIdf.getProjectName}.elf",
      "windows": {
        "program": "${workspaceFolder}\\build\\${command:espIdf.getProjectName}.elf"
      },
      "cwd": "${workspaceFolder}",
      "environment": [
        {
          "name": "PATH",
          "value": "${config:idf.customExtraPaths}"
        }
      ],
      "setupCommands": [
        {
          "text": "target remote :3333"
        },
        {
          "text": "set remote hardware-watchpoint-limit 2"
        },
        {
          "text": "mon reset halt"
        },
        {
          "text": "thb app_main"
        },
        {
          "text": "flushregs"
        }
      ],
      "externalConsole": false,
      "logging": {
        "engineLogging": true
      }
    }
  ]
}
```
