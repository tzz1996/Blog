# edk2R13995中添加UEFI SHELL 2.0
- edk2中有内置的shellbinpkg，里面有可执行的.efi二进制文件。
- edk2中关于如何添加SHELL 2.0的描述如下：
> 文件位置：ShellPkg\Readme.txt

```
1. Add this shell build to the NT32 build:
   Add the shell.inf to the [components] section as it is in the ShellPkg.dsc.
2. Update system PCDs to support this new module
   Update the PCD as follows using the Shell's PCD:
   gEfiIntelFrameworkModulePkgTokenSpaceGuid.PcdShellFile|{ 0x83, 0xA5, 0x04, 0x7C, 0x3E, 0x9E, 0x1C, 0x4F, 0xAD, 0x65, 0xE0, 0x52, 0x68, 0xD0, 0xB4, 0xD1 }
3. Remove the old shell from the NT32 Firmware list
   Remove the FILE APPLICATION section for the old shell.
4. Add this shell to the NT32 firmware list
   Add the Shell.INF to the end of the list of DXE modules.
5. Build NT32
```

## 安装步骤

### 1.配置Nt32Pkg.dsc文件

> 此文件用于通过vs和basetool编译出在win32环境下的模拟运行环境。

- 在`[LibraryClasses.common]`标签下添加`PathLib|ShellPkg/Library/BasePathLib/BasePathLib.inf`
> 这个库在一些版本中不需要单独添加，由于13995中会报错提示缺少PathLib这个库的引用，因此在这里添加。

- 在`[PcdsFixedAtBuild]`标签下添加SHELL 2.0的pcd：
`gEfiIntelFrameworkModulePkgTokenSpaceGuid.PcdShellFile|{ 0x83, 0xA5, 0x04, 0x7C, 0x3E, 0x9E, 0x1C, 0x4F, 0xAD, 0x65, 0xE0, 0x52, 0x68, 0xD0, 0xB4, 0xD1 }`

- 在`[Components.IA32]`标签下添加：

```
ShellPkg/Application/Shell/Shell.inf {
    <LibraryClasses>
      DebugLib|MdePkg/Library/UefiDebugLibConOut/UefiDebugLibConOut.inf
      ShellCommandLib|ShellPkg/Library/UefiShellCommandLib/UefiShellCommandLib.inf
      NULL|ShellPkg/Library/UefiShellLevel2CommandsLib/UefiShellLevel2CommandsLib.inf
      NULL|ShellPkg/Library/UefiShellLevel1CommandsLib/UefiShellLevel1CommandsLib.inf
      NULL|ShellPkg/Library/UefiShellLevel3CommandsLib/UefiShellLevel3CommandsLib.inf
      NULL|ShellPkg/Library/UefiShellDriver1CommandsLib/UefiShellDriver1CommandsLib.inf
      NULL|ShellPkg/Library/UefiShellInstall1CommandsLib/UefiShellInstall1CommandsLib.inf
      NULL|ShellPkg/Library/UefiShellDebug1CommandsLib/UefiShellDebug1CommandsLib.inf
      NULL|ShellPkg/Library/UefiShellNetwork1CommandsLib/UefiShellNetwork1CommandsLib.inf
      HandleParsingLib|ShellPkg/Library/UefiHandleParsingLib/UefiHandleParsingLib.inf
      FileHandleLib|ShellPkg/Library/UefiFileHandleLib/UefiFileHandleLib.inf
      ShellLib|ShellPkg/Library/UefiShellLib/UefiShellLib.inf
      SortLib|ShellPkg/Library/UefiSortLib/UefiSortLib.inf
      PrintLib|MdePkg/Library/BasePrintLib/BasePrintLib.inf
    <PcdsFixedAtBuild>
      gEfiMdePkgTokenSpaceGuid.PcdDebugPropertyMask|0xFF
      gEfiShellPkgTokenSpaceGuid.PcdShellLibAutoInitialize|FALSE
      gEfiMdePkgTokenSpaceGuid.PcdUefiLibMaxPrintBufferSize|16000
  }
```
> 标签`[Components.IA32]`表示在编译IA32架构的处理器时，都会编译这个标签下的.inf模块文件。

### 2.配置Nt32Pkg.fdf文件

> 此文件用于生成对应的.fd固件文件。

- 在`DXE Phase modules`模块下添加`INF  ShellPkg/Application/Shell/Shell.inf`
> 这个Shell.inf为UEFI SHELL 2.0的工程文件。

- 注释原先的内置shell的.efi二进制可执行文件：
```
#FILE APPLICATION = PCD(gEfiIntelFrameworkModulePkgTokenSpaceGuid.PcdShellFile) {
#    SECTION PE32 = EdkShellBinPkg/FullShell/Ia32/Shell_Full.efi
#  }
```

### 3.完成配置，重新编译edk2
```
edksetup.bat --nt32
build
```

