# 构建WinPE启动U盘

1. 安装Windows ADK，包括adkwinpesetup以及adksetup
2. 在搜索栏搜索`部署和映像工具环境`，右键，以管理员权限运行
3. 在上述创建的cmd命令行中通过命令创建一个WinPE工作集，使用`copype amd64 C:\WinPE_amd64`命令
4. 再打开一个拥有管理员权限的powershell命令行
5. 挂载WinPE启动镜像，通过命令`Mount-WindowsImage -Path "C:\WinPE_amd64\mount" -ImagePath "C:\WinPE_amd64\media\sources\boot.wim" -Index 1 -Verbose`实现
6. ~~将应用程序添加至镜像中，通过命令`Add-WindowsDriver -Path "<Mount_folder_path>" -Driver "<Source_path>" -Recurse`实现~~
7. ~~将可选组件添加至镜像中，通过命令`Add-WindowsPackage -PackagePath "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\amd64\WinPE_OCs\<Component>.cab" -Path "<Mount_folder_path>" -Verbose`实现~~
8. 在路径`C:\WinPE_amd64\mount\Program Files (x86)`下放置需要在WinPE中运行的程序，可以创建一个App文件夹
9. **关闭所有与路径`C:\WinPE_amd64\mount`有关的窗口、命令行！！！**
10. 卸载镜像并保存更改，通过命令`Dismount-WindowsImage -Path "C:\WinPE_amd64\mount" -Save -Verbose`实现
11. 回到`部署和映像工具环境`所创建的cmd命令行中
12. 初始化启动U盘(应使用容量不大于32G的U盘)：

```cmd
Diskpart
list disk
select disk x (Where "x" its the letter of the pendrive)
clean
convert MBR
create partition primary
format fs=fat32 quick label="WINPE"
assign letter "x"
quit
```

U盘初始化完成。

12. 将WinPE工作集拷贝到启动U盘中，通过启动U盘的盘符进行操作，使用命令`MakeWinPEMedia /UFD C:\WinPE_amd64 P:（其中P是U盘的驱动号）`实现
13. 之后就可以通过更改BIOS中的启动选项，通过该U盘启动并进入WinPE中。

## 注意事项

若在步骤9，`卸载镜像并保存更改`中报错，可以通过以下命令查看已挂载的镜像

```powershell
Get-WindowsImage -Mounted
```

卸载已挂载镜像的两种方式：

1. 卸载并保存

```powershell
Dismount-WindowsImage -Path "<Path>" -Save
```

2. 卸载但不保存

```powershell
Dismount-WindowsImage -Path "<Path>" -Discard
```

**若Path路径不存在，则需要通过注册表删除，在注册表的`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WIMMount\Mounted Images`路径下记录了当前的挂载镜像情况，将它们删除即可。**