# 工具

准备好一个大于 4G 的 U 盘
下载 Centos7 操作系统的 DVD 安装包
下载安装 UltraISO 软件

# 一、CentOS 简介

CentOS（Community Enterprise Operating System，社区企业操作系统）是一个基于 Red Hat Linux 提供的可自由使用源代码的企业级 Linux 发行版本，以高效、稳定著称。它使用与 Red Hat 相同的源代码编译而成，而且是开源免费的，因此有些要求高度稳定性的服务器以 CentOS 替代商业版的 Red Hat Enterprise Linux 使用，是很多中小服务器站点的首选。
CentOS 拥有 Red Hat 的所有功能，它们的不同之处在于 CentOS 并不包含封闭源代码软件，即 Red Hat 提供的额外的商业服务。

# 二、CentOS 镜像下载步骤

CentOS 官网: [https://www.centos.org/](https://www.centos.org/)

## 2.1 CentOS linux 和 CentOS stream 区别

在 CentOS 官网你可以看到有两大块，一个是 CentOS Linux DVD ISO 和 CentOS Stream DVD ISO

**1）CentOS Stream**
Centos Stream 是一个滚动发布的 Linux 发行版，它介于 Fedora Linux 的上游开发和 RHEL 的下游开发之间而存在。你可以把 CentOS Streams 当成是用来体验最新红帽系 Linux 特性的一个版本，尝鲜使用

**2）CentOS Linux**
CentOS Linux 就是普通使用的 CentOS 的系统了，如果追求稳定性，和正式使用，日常使用，还是强力推荐使用这个的
![[Obsidian/附件/用U盘制作CentOS系统启动盘.png]]

## 2.2 下载界面

1）选择 CentOS Linux 7 - X86_64 位系统：

> Intel CPU 是 X86
> ARM CPU 是 ARM64
> ![[Obsidian/附件/用U盘制作CentOS系统启动盘-1.png]]

2）选择下载连接开始下载
![[Obsidian/附件/用U盘制作CentOS系统启动盘-2.png]]

3）选择 CentOS 软件版本开始下载
CentOS 7.X 各版本说明
CentOS 7 提供的 ISO 镜像文件：

- DVD ISO：标准安装版，推荐使用
- Everything ISO ：对完整版安装盘的软件进行补充，集成所有软
- Minimal ISO：精简版，自带的软件最少
  ![[Obsidian/附件/用U盘制作CentOS系统启动盘-3.png]]

# 三、下载安装 UltraISO 软件

UltraISO 下载连接:
[https://pan.baidu.com/s/1KSBjqJoqtPXSnO7A7PN0yg](https://pan.baidu.com/s/1KSBjqJoqtPXSnO7A7PN0yg)
提取码：h1cg

![[Obsidian/附件/用U盘制作CentOS系统启动盘-4.png]]

# 四、CentOS 启动盘制作

1）选择文件—打开
![[Obsidian/附件/用U盘制作CentOS系统启动盘-5.png]]

2）找到你下载的 Centos7 的 DVD 文件是 ISO 格式的
加载完成之后可以看到 UltraISO 中的 Centos7 文件
![[Obsidian/附件/用U盘制作CentOS系统启动盘-6.png]]

3）选择启动—写入硬盘镜像
![[Obsidian/附件/用U盘制作CentOS系统启动盘-7.png]]

4）选择你的 U 盘
选择写入方式为 USB-ZIP

> [!done|noicon]+ **写入方式介绍** > **USB-HDD：**
> 硬盘仿真模式，DOS 启动后显示 C:盘；兼容性很高，对于一些只支持 USB-ZIP 模式的电脑则无法启动。
> **USB-ZIP：**
> 大容量软盘仿真模式，DOS 启动后显示 A:盘；在一些比较老的计算机上是唯一的可选模式。
> **USB-HDD+：**
> 增强的 USB-HDD 模式，DOS 启动后显示 C:盘，兼容性极高。缺点：对仅支持 USB-ZIP 的电脑无法启动。
> **USB-ZIP+：**
> 增强的 USB-ZIP 模式，支持 USB-HDD/USB-ZIP 双模式启动，从而达到很高的兼容性。缺点：有些支持 USB-HDD 的电脑会将此模式的 U 盘认为是 USB-ZIP 来启动，从而导致 4GB 以上大容量 U 盘的兼容性有所降低。
> **USB-HDD+ v2：**
> USB-HDD 的增强模式，兼容性高于 USB-HDD+模式，但对仅支持 USB-ZIP 的电脑无法启动。在 DOS 下启动后 U 盘盘符仍然显示为 C:盘
> **USB-ZIP+ v2：**
> USB-ZIP 的增强模式，兼容性高于 USB-ZIP+模式。
> **RAW：**
> 中文解释是“原材料”或“未经处理的东西”，RAW/ISO 中 ISO 指的是未处理过的镜像文件，RAW 指的是无字幕的原版文件

![[Obsidian/附件/用U盘制作CentOS系统启动盘-8.png]]

确认无误后点击“写入”按钮，弹出以下提示对话框，选择“是”，开始将系统写入 U 盘，如下图所示；
![[Obsidian/附件/用U盘制作CentOS系统启动盘-9.png]]

5）开始写入
![[Obsidian/附件/用U盘制作CentOS系统启动盘-10.png]]

6）写入完成
![[Obsidian/附件/用U盘制作CentOS系统启动盘-11.png]]

至此，CentOS 系统启动盘制作完成
