如果你在VMware中使用的Ubuntu虚拟机忘记了登录密码，可以按照以下步骤来重置密码：

1. **重启Ubuntu虚拟机**：首先，关闭正在运行的虚拟机，然后从VMware中重新启动它。
2. **进入GRUB菜单**：启动过程中，长按`Shift`键，以便进入GRUB引导加载器菜单。对于某些版本的Ubuntu，可能需要快速连续按`Esc`键来显示GRUB菜单。
3. **选择恢复模式**：在GRUB菜单中，使用键盘的上下箭头选择带有`(... recovery mode)`或`Advanced options for Ubuntu`的选项，然后按`Enter`键。
4. **进入根shell**：系统会显示一个恢复菜单，选择"root Drop to root shell prompt"或者类似的选项（可能标记为"root"或者带有"(drop to root shell)"的描述），然后按`Enter`键。
5. **重置密码**：在出现的命令行界面中，你可以使用`passwd 用户名`命令来重置密码。例如，如果你的用户名是`david`，则输入`passwd david`，然后按`Enter`键。接下来，按照提示设置新的密码，并再次确认新密码。
6. **更新密码**：成功更改密码后，你会看到类似于"password updated successfully"的提示。
7. **退出并重启**：完成密码重置后，输入`exit`命令退出root shell，然后在恢复菜单中选择"Resume normal boot"或者直接输入`reboot`命令来重启虚拟机。
8. **使用新密码登录**：系统重启后，你应该能够使用新设置的密码登录到Ubuntu了。

请注意，这些步骤适用于大多数Ubuntu版本，但根据你所用的具体Ubuntu版本，某些细节可能会有所不同。如果遇到任何问题，建议查阅针对你Ubuntu版本的官方文档或在线帮助。