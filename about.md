---
layout: lecture
title: "开设此课程的动机"
---

在传统的计算机科学课程中，你可能会修很多教授高级主题的课程，从操作系统到编程语言再到机器学习。但在许多学校，有一个至关重要的主题却很少被专门讲授，而是留给学生们自己去探索：计算生态系统素养。

这些年，我们在麻省理工学院参与了许多课程的教学，一次又一次地发现很多学生对于可用工具的了解有限。计算机设计的初衷是自动化手动任务，然而学生们却常常手动执行重复性任务，或者无法充分利用版本控制和文本编辑器等强大工具。最好的情况下，这导致效率低下和浪费时间；最坏的情况下，这会导致数据丢失或无法完成某些任务等问题。

这些主题不是大学课程的一部分：学生们从未被教导如何使用这些工具，或者至少不知道如何高效地使用，因此在本应简单的任务上浪费时间和精力。标准的计算机科学课程缺少了关于计算生态系统的关键主题，而这些主题可以让学生的生活轻松很多。

# The missing semester of your CS education

为了帮助解决这个问题，我们开设了一个课程，涵盖我们认为对成为高效计算机科学家和程序员至关重要的所有主题。这个课程实用且具有很强的实践性，提供了各种能够立即广泛应用解决问题的趁手工具指导。该课程的最新版本对材料进行了大幅修订，正在 MIT 2026 年 1 月的”独立活动期”开设——这是一个为期一个月的学期，主要开设学生主导的短期课程。虽然讲座本身仅面向 MIT 社区，但我们将向公众提供所有讲座材料和讲座录像。

如果这听起来适合你，以下是一些具体的课程示例：

## 命令行与 shell 工具

如何使用别名、脚本和构建系统来自动化执行通用重复的任务。不再总是从文档中拷贝粘贴
命令。不要再“逐个执行这 15 个命令”，不要再“你忘了执行这个命令”、“你忘了传那个
参数”，类似的对话不要再有了。

例如，快速搜索历史记录可以节省大量时间。在下面这个示例中，我们展示了如何通过`convert`命令
在历史记录中跳转的一些技巧。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ site.baseurl }}/static/media/demos/history.mp4" type="video/mp4">
</video>

## 版本控制

如何**正确地**使用版本控制，利用它避免尴尬的情况发生。与他人协作，并且能够快速定位
有问题的提交
不再大量注释代码。不再为解决 bug 而找遍所有代码。不再“我去，刚才是删了有用的代码？！”。
我们将教你如何通过拉取请求来为他人的项目贡献代码。

下面这个示例中，我们使用`git bisect`来定位哪个提交破坏了单元测试，并且通过`git revert`来进行修复。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ site.baseurl }}/static/media/demos/git.mp4" type="video/mp4">
</video>

## 文本编辑

不论是本地还是远程，如何通过命令行高效地编辑文件，并且充分利用编辑器特性。不再来回复制文件。不再重复编辑文件。

Vim 的宏是它最好的特性之一，在下面这个示例中，我们使用嵌套的 Vim 宏快速地将 html 表格转换成了 csv 格式。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ site.baseurl }}/static/media/demos/vim.mp4" type="video/mp4">
</video>

## 远程服务器

使用 SSH 密钥连接远程机器进行工作时如何保持连接，并且让终端能够复用。不再为了仅执行个别命令
总是打开许多命令行终端。不再每次连接都总输入密码。不再因为网络断开或必须重启笔记本时
就丢失全部上下文。

以下示例，我们使用`tmux`来保持远程服务器的会话存在，并使用`mosh`来支持网络漫游和断开连接。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ site.baseurl }}/static/media/demos/ssh.mp4" type="video/mp4">
</video>

## 查找文件

如何快速查找你需要的文件。不再挨个点击项目中的文件，直到找到你所需的代码。

以下示例，我们通过`fd`快速查找文件，通过`rg`找代码片段。我们也用到了`fasd`快速`cd`并`vim`最近/常用的文件/文件夹。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ site.baseurl }}/static/media/demos/find.mp4" type="video/mp4">
</video>

## 数据处理

如何通过命令行直接轻松快速地修改、查看、解析、绘制和计算数据和文件。不再从日志文件拷贝
粘贴。不再手动统计数据。不再用电子表格画图。

## 代码质量与持续集成

如何使用自动格式化、代码检查、测试和代码覆盖率工具来提高代码质量。不再有丑陋的代码。不再有回归问题。不再有"在我电脑上能运行"但在其他人电脑上崩溃的代码。

## 代码之外

如何编写优秀的文档、与开源维护者清晰沟通、提交可操作的问题，以及贡献能被合并的拉取请求。不再有困惑的用户无法上手你的软件。不再被维护者忽视。

# 结论

这 9 节课将包括但不限于以上内容，同时每堂课都提供了能帮助你熟悉这些工具的练手小测验。如果不能
等到 2026 年 1 月，你也可以看下[往年课程]({{ site.baseurl }}/2020/)，其中涵盖了许多相同的主题。

我们希望在 1 月见到你，无论是线上还是线下！

Happy hacking,<br>
[Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/) 和 [Jose](https://josejg.com/)
