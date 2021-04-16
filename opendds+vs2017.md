# Win10下编译OpenDDS项目
- 操作系统：`win10`

## 编译工具
- [vs2017](https://msdn.itellyou.cn/)
- [ActivePerl](https://www.activestate.com/products/perl/downloads/)
- [ACE+TAO](https://download.dre.vanderbilt.edu/)，所下版本为ACE+TAO-7.0.1
- windows 8.1 SDK，可通过vs2017 installer安装

## 编译步骤

### 1.下载工具和源码
- 安装`vs2017`和`ActivePerl`
- 下载[OpenDDS](http://download.objectcomputing.com/OpenDDS/)源码，[github](https://github.com/objectcomputing/OpenDDS)
- 解压`ACE+TAO-7.0.1`，并将得到的`ACE_wrappers`目录放在`OpenDDS`根目录下
> 本测试使用的是`OpenDDS-3.16`版本

### 2.配置OpenDDS编译环境
- 打开`vs2017 command line`并进入`OpenDDS`根目录，运行：
```
...\OpenDDS-3.16> configue
```
> `configure.cmd`脚本调用名为`configure`的`Perl`脚本，用于生成`DDS_TAOv2.sln`文件和配置环境  
> `configure`脚本还会创建名为`setenv.cmd`的windows脚本，用于为当前`vs2017 cl`配置编译所需的所有环境变量  
> `setenv.cmd`脚本是为`ACE+TAO`和`OpenDDS`配置环境变量，在编译他前必须配置完成，或通过`windows`系统手动添加

- 在`vs2017 cl`中`OpenDDS`根目录运行：
```
...\OpenDDS-3.16> setenv
```
> `setenv.cmd`将完成的重要环境变量配置有：  
> ACE_ROOT=...\OpenDDS-3.16\ACE_wrappers  
> TAO_ROOT=...\OpenDDS-3.16\ACE_wrappers\TAO  
> DDS_ROOT=...\OpenDDS-3.16

### 3.编译ACE+TAO项目
- `ACE+TAO`项目为`OpenDDS`使用的库

#### 3.1.编译ACE
- 在`vs2017 cl`中运行：
```
...\OpenDDS-3.16\ACE_wrappers> devenv ACE_vs2017.sln
```
- 在`vs2017`中右键项目文件，重新生成解决方案，进行编译
> 本次测试的`OpenDDS`版本为`3.16`，所包含的`vs`解决方案版本只有`vs2017`和`vs2019`  
> 从`vs2017 cl`中打开解决方案的目的为使解决方案可以使用`configure`和`setenv`脚本设置的环境

#### 3.2.编译TAO
- 在`vs2017 cl`中运行：
```
...\OpenDDS-3.16\ACE_wrappers\TAO> devenv TAO_ACE_vs2017.sln
```
- 在`vs2017`中右键项目文件，重新生成解决方案，进行编译

### 4.编译OpenDDS项目
- 在`vs2017 cl`中运行：
```
...\OpenDDS-3.16> devenv DDS_TAOv2.sln
```
- 在`vs2017`中右键项目文件，重新生成解决方案，进行编译
> 编译生成的实例程序在目录`...\OpenDDS-3.16\DevGuideExamples\DCPS`下

#### 4.1.编译OpenDDS中的example
- 在`vs2017 cl`中运行：
```
...\OpenDDS-3.16\examples\DCPS\Messenger_Imr> perl %MPC_ROOT%\mwc.pl -type vs2017
```
> 其中`%MPC_ROOT%`为运行`configure`脚本时为`vs2017 cl`设置的环境变量  
> 绝对路径为`...\OpenDDS-3.16\ACE_wrappers\MPC\mwc.pl`
- 运行完成后，将在`...\OpenDDS-3.16\examples\DCPS\Messenger_Imr`路径下生成`vs2017`解决方案文件

#### 4.2.运行example
- 在`vs2017 cl`中运行：
```
...\OpenDDS-3.16\examples\DCPS\Messenger_Imr> perl run_test.pl
```
- 或者直接打开生成的`sln`文件进行运行调试