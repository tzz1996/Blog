# windwos下编译udk2018
- windwos下通过Visual Studio编译udk，udk2018中关于windows下vs编译器的配置程序如下：
> 文件位置：BaseTools\get_vsvars.bat

```
if defined VS140COMNTOOLS  call :read_vsvars  "%VS140COMNTOOLS%"
if defined VS120COMNTOOLS  call :read_vsvars  "%VS120COMNTOOLS%"
if defined VS110COMNTOOLS  call :read_vsvars  "%VS110COMNTOOLS%"
if defined VS100COMNTOOLS  call :read_vsvars  "%VS100COMNTOOLS%"
if defined VS90COMNTOOLS   call :read_vsvars  "%VS90COMNTOOLS%"
if defined VS80COMNTOOLS   call :read_vsvars  "%VS80COMNTOOLS%"
if defined VS71COMNTOOLS   call :read_vsvars  "%VS71COMNTOOLS%"
```

- 其中VS140COMNTOOLS等为安装vs后自动添加的系统环境变量，此处使用的是vs2015，对应的变量是VS140COMNTOOLS（VS80COMNTOOLS对应vs2008）
- 如之前的老版本udk可能不支持较新版本的vs，具体的要求需要到edksetup.bat中或其他地方查看支持的vs环境变量信息。

## 编译步骤

### 1.下载udk2018和edk2-BaseTools-win32
- basetools为udk的编译相关的工具，但在windows平台下的部分没有集成在udk项目中，所以需要单独下载。（[udk2018](https://github.com/tianocore/edk2/tree/UDK2018)，[edk2-BaseTools-win32](https://github.com/tianocore/edk2-BaseTools-win32)）
- 或者使用svn工具从地址`https://svn.code.sf.net/p/edk2-toolbinaries/code`拉取。
- 下载后的edk2-BaseTools-win32中的内容需要放到`BaseTools\Bin\Win32`中。

### 2.更改target.txt配置文件
- edk中`Conf/target.txt`，`Conf/build_rule.txt`，`Conf/tools_def.txt`三个配置文件有相应的生成模板，edksetup.bat环境配置程序会根据vs版本等信息自动生成这三个文件，其中只需要修改`Conf/target.txt`来指定目标编译器，此处修改如下：`TOOL_CHAIN_TAG = VS2015x86`


### 3.编译
- cmd中：

```
edksetup.bat --nt32
build
```

### 4.可能出现的问题
- 报错：

```
Traceback (most recent call last):
  File "C:\Python27\lib\site-packages\cx_Freeze\initscripts\Console.py", line 27, in <module>
  File "GenFds\GenFds.py", line 24, in <module>
ValueError: Attempted relative import in non-package

build...
 : error 7000: Failed to execute command
        GenFds -f c:\edk2-vudk2018\Nt32Pkg\Nt32Pkg.fdf --conf=c:\edk2-vudk2018\conf -o c:\edk2-vudk2018\Build\NT32IA32\DEBUG_VS2015x86 -t VS2015x86 -b DEBUG -p c:\edk2-vudk2018\Nt32Pkg\Nt32Pkg.dsc -a IA32  -D "EFI_SOURCE=c:\\edk2-vudk2018\\edkcompatibilitypkg"  -D "EDK_SOURCE=c:\\edk2-vudk2018\\edkcompatibilitypkg"  -D "TOOL_CHAIN_TAG=VS2015x86"  -D "TOOLCHAIN=VS2015x86"  -D "TARGET=DEBUG"  -D "FAMILY=MSFT"  -D "WORKSPACE=c:\\edk2-vudk2018"  -D "EDK_TOOLS_PATH=c:\\edk2-vudk2018\\basetools"  -D "ARCH=IA32"  -D "ECP_SOURCE=c:\\edk2-vudk2018\\edkcompatibilitypkg" [C:\edk2-vUDK2018]
```

- 解决方法：将路径`BaseTools\Bin\Win32`下`GenFds.exe`更名为`GenFds.LABZ`，并在`edksetup.bat`中正常运行可执行的代码段中加入：

```
path BaseTools\BinWrappers\WindowsLike;
set PYTHON_HOME=c:\python27;
```
