## 一、介绍

Buttons是一个为Obsidian设计的插件，它使用户可以在菜单栏和快捷键之间轻松添加自定义按钮，以执行特定任务。该插件还允许用户定义按钮的名称、链接、关键字、颜色和位置，以简化操作和提高效率。下面是更详细介绍。下面一句话介绍来自作者：

> **Run commands** and **open links** by clicking on✨Buttons✨

翻译：通过按按钮运行命令或者打开链接。

“按钮”这个东西在生活中的使用场景过多了，基本目的就是希望按钮简化操作和提高效率。

## 二、安装和基本使用

Buttons插件可以通过Obsidian中的内置插件市场进行下载和安装也可以直接到作者GitHub上把文件拉下来。安装完成后，

1.  `Ctrl+P`打开Obsidian命令面板；
2.  输入buttons；
3.  选择`Buttons: Button maker`，即可创建按钮；

![](https://pic4.zhimg.com/80/v2-012706817ec5cba76648d0d2d7f27a73_720w.webp)

Ctrl+P得到Obsidian命令，输入button，查看button相关命令

### 配置

![](https://pic3.zhimg.com/80/v2-d9d1ec3221a6b083339a4d60c527b856_720w.webp)

Button Maker配置页面

基本配置设置好上面四个即可

1.  输入button名字，button上显示的名字；
2.  选择button类型，类型是配置中最重要的部分；
3.  设置button id，方便使用前述的`insert inline button`
4.  设置button颜色，button显示的颜色；

### 配置类型选择

这里配置一个简单的执行`button maker`指令的蓝色按钮。类型配置为`Command`

![](https://pic4.zhimg.com/80/v2-31d23bd2c0785bcb91b6b4cc9507183f_720w.webp)

Button Type的下拉选项一览

具体配置如下：

1.  button名称为`创建Button`
2.  类型为指令，命令为`Button maker`
3.  button id为`btmker`

![](https://pic2.zhimg.com/80/v2-56ae6dbe5c26e0e6d44ea41d140a9695_720w.webp)

配置一个一键执行”创建button“的按钮

创建完成后

![](https://pic4.zhimg.com/80/v2-f56234f1c8d3f861df451b0e7822d34f_720w.webp)

创建完成后，点击按钮即可取代前述”Ctrl+P之后选择Button Maker“

依葫芦画瓢，生成下面三个按钮，左右分屏和上下分屏是在Obsidian中比较好用的两个功能。

![](https://pic2.zhimg.com/80/v2-680f690164d9572e36c26bd2a16edfb1_720w.webp)

依葫芦画瓢生成其他按钮

### 按钮插入

前述配置中，有一个配置选项叫`button id`，id的使用可以只创建一次相关按钮，按照需要insert按钮到自己需要的地方，以实现按钮的复用。例如，我将上述button list文件中创建的按钮插入test文件中，

![](https://pic4.zhimg.com/80/v2-fdb382516619af7dc661d84e9e1b12cf_720w.webp)

Ctrl+P后选择insert button，选择ipconfig

![](https://pic1.zhimg.com/80/v2-eb5459aee19de845af8e1000439187a4_720w.webp)

预览模式下，可以看到插入的ipconfig按钮

## 三、扩展应用

上述基本使用，是以`Obsidian命令`为例，实际上如`Button Type`所示，还可以是链接，模板等。下面的拓展部分可以推荐基于链接和模板的拓展用法。

1.  可以使用按钮**打开常用**电脑**文件夹**，例如`file:///C:\Users`（链接用法）；
2.  打开另一个Obsidian的库（链接用法）；
3.  可以一键运行模板，模板中使用**Templater**插入自建命令，这样就可以实现在Obsidian中**一键**运行**自建脚本**，大大简化了自建脚本的使用（链接用法），这个用法**强烈推荐**（[在我的模板中](https://link.zhihu.com/?target=https%3A//gitee.com/do-it-hank/obsidian_template)，添加了`ipconfig`按钮，即一键调用cmd运行ipconfig命令），Templater插件相关内容见[Obsidian插件之——Templater](https://zhuanlan.zhihu.com/p/611448942)；

![](https://pic4.zhimg.com/80/v2-f46150726e6781d88ef7800ea72febd7_720w.webp)

添加了ipconfig按钮，一键调用cmd运行ipconfig命令

## 四、最后

总之，Button插件使Obsidian用户可以轻松添加自定义按钮到菜单栏和快捷键中，以简化复杂的任务和提高操作效率。它具有广泛的自定义选项，在颜色、链接、位置等方面提供了极大的灵活性。双向链接和响应式按钮的支持可以大大优化使用体验。Button插件还具有轻量级、低占用性的性能，适用于所有类型的笔记本用户。是Obsidian中值得使用和使用的插件之一。