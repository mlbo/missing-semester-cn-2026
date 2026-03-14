---
layout: lecture
title: "课程概览与 shell"
description: >
  了解本课程的动机，并开始学习 shell。
thumbnail: /static/assets/thumbnails/2026/lec1.png
date: 2026-01-12
ready: true
video:
  aspect: 56.25
  id: MSgoeuMqUmU
---

# 我们是谁？

本课程由 [Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/) 和 [Jose](http://josejg.com/) 共同教授。我们都是 MIT 校友，在学生时代创办了这门 MIT IAP 课程。你可以通过 [missing-semester@mit.edu](mailto:missing-semester@mit.edu) 联系我们。

我们不是受薪教授这门课程，也不会以任何方式从中获利。我们将所有[课程材料](https://missing.csail.mit.edu/)和[讲座录像](https://www.youtube.com/@MissingSemester)免费在线发布。如果你想支持我们的工作，最好的方式就是传播这门课程。如果你是在公司、大学或其他组织中向更大规模的人群讲授这些内容，请通过电子邮件向我们发送体验报告/心得，让我们了解情况 :)

# 动机

作为计算机科学家，我们知道计算机最擅长帮助我们完成重复性的工作。但是我们却常常忘记这一点也适用于我们使用计算机的方式，而不仅仅是利用计算机程序去帮我们求解问题。我们拥有触手可及的各种工具，可以帮助我们在处理任何与计算机相关的问题时更加高效地解决更复杂的问题。但是我们中的大多数人实际上只利用了这些工具中的很少一部分，我们常常只是死记硬背一些如咒语般的命令，或是当我们卡住的时候，盲目地从网上复制粘贴一些命令。

本课程意在帮你解决这一问题。

我们希望教会您如何挖掘现有工具的潜力，向您介绍一些新的工具，并希望激发您探索（甚至是开发）更多工具的热情。我们认为这是大多数计算机科学相关课程中缺失的重要一环。

# 课程结构

这门非学分课程包含九个时长一小时的讲座，每个讲座都会关注一个[特定的主题](/2026/)。尽管这些讲座之间基本上是各自独立的，但随着课程的进行，我们会假定您已经掌握了之前的内容。我们在线提供讲座笔记，但课上可能会有一些内容（例如演示）不包含在笔记中。和往年一样，我们会录制讲座并发布到[网上](https://www.youtube.com/@MissingSemester)。

我们试图在短短几个一小时的讲座中涵盖大量内容，因此讲座的信息密度相当大。为了让你能以自己的节奏掌握内容，每个讲座都包含一组练习来帮助你掌握本节课的重点。我们不会专门安排答疑时间，但我们鼓励你在 [OSSU Discord](https://ossu.dev/#community) 的 `#missing-semester-forum` 频道提问，或者发送邮件到 [missing-semester@mit.edu](mailto:missing-semester@mit.edu)。

由于时间有限，我们无法像完整的课程那样深入讲解所有工具。在可能的情况下，我们会尝试为你指出进一步学习某个工具或主题的资源，但如果有什么特别引起你的兴趣，请不要犹豫，联系我们询问更多信息！

最后，如果你对课程有任何反馈，请发送邮件到 [missing-semester@mit.edu](mailto:missing-semester@mit.edu)。

# 主题 1: The Shell

## 什么是 shell？

如今的计算机有着多种多样的交互接口让我们可以进行指令的输入，从炫酷的图形用户界面（GUI）、语音接口、AR/VR，到最近的 LLMs。这些接口可以覆盖 80% 的使用场景，但它们在允许你做的事情上往往有根本性的限制——你不能点击一个不存在的按钮，也不能给出一个还没有被编程的语音指令。为了充分利用计算机提供的工具，我们不得不回到最根本的方式，使用文字接口：Shell。

几乎所有你能接触到的平台都有某种形式的 shell，而且很多平台都有多个 shell 供你选择。虽然它们在细节上可能有所不同，但核心功能大致相同：它们允许你运行程序、给它们输入，并以半结构化的方式检查它们的输出。

要打开一个 shell 提示符（你可以在此输入命令），你首先需要一个终端，它是 shell 的可视化接口。你的设备可能已经预装了一个，或者你可以很容易地安装一个：

- **Linux:**
  按 `Ctrl + Alt + T`（在大多数发行版上有效）。或者在应用程序菜单中搜索 "Terminal"。
- **Windows:**
  按 `Win + R`，输入 `cmd` 或 `powershell`，然后按回车。或者，在开始菜单中搜索 "Terminal" 或 "Command Prompt"。
- **macOS:**
  按 `Cmd + Space` 打开 Spotlight，输入 "Terminal"，然后按回车。或者在应用程序 → 实用工具中找到 Terminal。

在 Linux 和 macOS 上，这通常会打开 Bourne Again SHell，简称 "bash"。这是使用最广泛的 shell 之一，其语法与许多其他 shell 相似。在 Windows 上，你会看到 "batch" 或 "powershell" shell，取决于你运行的命令。这些是 Windows 特有的，不是我们在本课程中要重点关注的，尽管它们有类似的功能。你需要的是 [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) 或 Linux 虚拟机。

还有其他 shell 存在，通常比 bash 有更多人体工程学上的改进（fish 和 zsh 是最常见的）。虽然这些非常流行（所有讲师都使用其中之一），但它们远没有 bash 那么普遍，而且依赖于许多相同的概念，所以我们不会在本讲座中重点讨论这些。

## 为什么要学习 shell？

Shell 不仅（通常）比"点击操作"快得多，它还具有你在任何图形程序中都不容易找到的强大表达能力。正如我们将看到的，shell 让你能够以创造性的方式组合程序，自动化几乎任何任务。

熟悉 shell 对于浏览开源软件世界也非常有用（这些软件通常有需要 shell 的安装说明）、为你的软件项目构建持续集成（如[代码质量讲座](/2026/code-quality/)所述），以及在其他程序失败时调试错误。

## 在 shell 中导航

当你启动终端时，你会看到一个提示符，通常看起来像这样：

```console
missing:~$
```

这是 shell 的主要文本接口。它告诉你，你在机器 `missing` 上，你的"当前工作目录"，或者说你当前所在的位置，是 `~`（"home" 的缩写）。`$` 表示你不是 root 用户（稍后会详细介绍）。在这个提示符下，你可以输入一个命令，然后 shell 会解释它。最基本的命令是执行一个程序：

```console
missing:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
missing:~$
```

这里，我们执行了 `date` 程序，它（也许并不意外地）打印当前日期和时间。然后 shell 再次询问我们要执行的命令。我们也可以执行一个带参数的命令：

```console
missing:~$ echo hello
hello
```

在这种情况下，我们告诉 shell 执行 `echo` 程序，参数是 `hello`。`echo` 程序只是简单地打印出它的参数。shell 通过按空白字符分割命令来解析它，然后运行第一个词指示的程序，将后续每个词作为程序可以访问的参数。如果你想提供一个包含空格或其他特殊字符的参数（例如，一个名为 "My Photos" 的目录），你可以用 `'` 或 `"` 引用参数（`"My Photos"`），或者用 `\` 转义相关字符（`My\ Photos`）。

当你刚开始学习时，也许最重要的命令是 `man`，"manual" 的缩写。`man` 程序可以让你查找系统中任何命令的更多信息。例如，如果你运行 `man date`，它会解释 `date` 是什么，以及你可以传递给它以改变其行为的各种参数。你通常也可以通过向大多数命令传递 `--help` 作为参数来获得简短的帮助信息。

> 考虑安装并使用 [`tldr`](https://tldr.sh/) 作为 `man` 的补充，它会在终端中直接显示常见使用示例。LLMs 通常也非常擅长解释命令的工作原理以及如何调用它们来实现你想要的目标。

在 `man` 之后，最重要的命令是 `cd`，或 "change directory"。这个命令实际上是内置在 shell 中的，不是一个独立的程序（即，`which cd` 会说 "no cd found"）。你传递给它一个路径，该路径就成为你的当前工作目录。你也会在 shell 提示符中看到工作目录：

```console
missing:~$ cd /bin
missing:/bin$ cd /
missing:/$ cd ~
missing:~$
```

> 注意 shell 有自动补全功能，所以你经常可以通过按 `<TAB>` 更快地完成路径输入！

很多命令在没有指定其他内容时会对当前工作目录进行操作。如果你不确定你在哪里，可以运行 `pwd` 或打印 `$PWD` 环境变量（用 `echo $PWD`），两者都会产生当前工作目录。

当前工作目录也很方便，因为它允许我们使用相对路径。到目前为止我们看到的所有路径都是绝对路径——它们以 `/` 开头，给出从文件系统根目录（`/`）导航到某个位置所需的完整目录集合。在实践中，你会更常使用相对路径；之所以称为相对路径，是因为它们相对于当前工作目录。在相对路径中（任何不以 `/` 开头的路径），第一个路径组件会在当前工作目录中查找，后续组件按通常方式遍历。例如：

```console
missing:~$ cd /
missing:/$ cd bin
missing:/bin$
```

每个目录中还有两个"特殊"组件：`.` 和 `..`。`.` 是"当前目录"，`..` 是"父目录"。所以：

```console
missing:~$ cd /
missing:/$ cd bin/../bin/../bin/././../bin/..
missing:/$
```

你通常可以在任何命令参数中交替使用绝对路径和相对路径，只需在使用相对路径时注意你的当前工作目录是什么！

> 考虑安装并使用 [`zoxide`](https://github.com/ajeetdsouza/zoxide) 来加速你的 `cd` 操作——`z` 会记住你经常访问的路径，让你用更少的输入就能访问。

## Shell 中有什么可用？

但是 shell 如何知道如何找到 `date` 或 `echo` 这样的程序呢？当 shell 被要求执行一个命令时，它会查询一个叫做 `$PATH` 的环境变量，该变量列出了 shell 在收到命令时应该搜索程序的目录：

```console
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

当我们运行 `echo` 命令时，shell 看到它应该执行 `echo` 程序，然后在 `$PATH` 中用 `:` 分隔的目录列表中搜索具有该名称的文件。当它找到时，就运行它（假设文件是可执行的；稍后会详细介绍）。我们可以使用 `which` 程序找出给定程序名会执行哪个文件。我们也可以通过给出我们想要执行的文件的路径来完全绕过 `$PATH`。

这也给我们提供了一个线索，让我们知道如何确定我们在 shell 中能够执行的_所有_程序：通过列出 `$PATH` 上所有目录的内容。我们可以通过将给定的目录路径传递给 `ls` 程序来做到这一点，它会列出文件：

```console
missing:~$ ls /bin
```

> 考虑安装并使用 [`eza`](https://eza.rocks/) 获得更人性化的 `ls` 体验。

这会在大多数计算机上打印很多程序，但在这里我们只关注一些最重要的。首先，一些简单的：

- `cat file`，打印 `file` 的内容。
- `sort file`，按排序顺序打印 `file` 的行。
- `uniq file`，从 `file` 中消除连续的重复行。
- `head file` 和 `tail file`，分别打印 `file` 的前几行和最后几行。

> 考虑安装并使用 [`bat`](https://github.com/sharkdp/bat) 代替 `cat`，获得语法高亮和滚动功能。

还有 `grep pattern file`，它在 `file` 中查找匹配 `pattern` 的行。这个命令值得稍微多加关注，因为它既非常有用，又拥有比人们可能预期的更广泛的功能。`pattern` 实际上是一个正则表达式，可以表达非常复杂的模式——我们会在代码质量讲座中[介绍这些](/2026/code-quality/#regular-expressions)。你也可以指定一个目录而不是一个文件（或者不指定则使用 `.`）并传递 `-r` 来递归搜索目录中的所有文件。

> 考虑安装并使用 [`ripgrep`](https://github.com/BurntSushi/ripgrep) 代替 `grep`，获得更快更人性化（但可移植性较差）的替代方案。`ripgrep` 默认也会递归搜索当前工作目录！

还有一些具有稍微复杂接口的非常有用的工具。首先是 `sed`，它是一个程序化的文件编辑器。它有自己的编程语言来进行自动化文件编辑，但最常见的用法是：

```console
missing:~$ sed -i 's/pattern/replacement/g' file
```

这会将 `file` 中所有 `pattern` 的实例替换为 `replacement`。`-i` 表示我们希望替换内联进行（而不是保留 `file` 不修改并打印替换后的内容）。`s/` 是在 sed 编程语言中表达我们要进行替换的方式。`/` 分隔模式和替换。末尾的 `/g` 表示我们要替换每行上的所有匹配项，而不仅仅是第一个。与 `grep` 一样，这里的 `pattern` 是正则表达式，这给了你很大的表达能力。正则表达式替换还允许 `replacement` 引用匹配模式的部分；我们马上会看到一个例子。

接下来是 `find`，它让你找到匹配特定条件的文件（递归地）。例如：

```console
missing:~$ find ~/Downloads -type f -name "*.zip" -mtime +30
```

查找下载目录中超过 30 天的 ZIP 文件。

```console
missing:~$ find ~ -type f -size +100M -exec ls -lh {} \;
```

在你的主目录中查找大于 100M 的文件并列出它们。注意 `-exec` 接受一个以独立的 `;` 结尾的命令（我们需要像转义空格一样转义它），其中 `{}` 会被 `find` 替换为每个匹配的文件路径。

```console
missing:~$ find . -name "*.py" -exec grep -l "TODO" {} \;
```

查找任何包含 TODO 项的 `.py` 文件。

`find` 的语法可能有点令人生畏，但希望这能让你感受到它是多么有用！

> 考虑安装并使用 [`fd`](https://github.com/sharkdp/fd) 代替 `find`，获得更人性化（但可移植性较差）的体验。

接下来是 `awk`，像 `sed` 一样，它有自己的编程语言。`sed` 是为编辑文件而构建的，而 `awk` 是为解析文件而构建的。到目前为止，`awk` 最常见的用法是处理具有常规语法的文件（如 CSV 文件），你想从中提取每条记录（即行）的某些部分：

```console
missing:~$ awk '{print $2}' file
```

打印 `file` 每行的第二个空白分隔列。如果添加 `-F,`，它将打印每行的第二个逗号分隔列。`awk` 可以做更多——过滤行、计算聚合等等——请参阅练习以了解一二。

将这些工具放在一起，我们可以做很酷的事情：

```console
missing:~$ ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```

这从远程服务器获取 SSH 日志（我们会在下一讲中更多地讨论 `ssh`），搜索断开连接的消息，从每条这样的消息中提取用户名，并打印前 10 个用户名，用逗号分隔。全部在一个命令中！我们将剖析每个步骤作为练习。

## Shell 语言（bash）

前面的例子引入了一个新概念：管道（`|`）。它们让你将一个程序的输出与另一个程序的输入串联起来。这之所以有效，是因为大多数命令行程序在没有给出 `file` 参数时会在其"标准输入"（你的按键通常去的地方）上操作。`|` 将 `|` 之前程序的标准输出（通常会打印到终端的内容）作为 `|` 之后程序的标准输入。这允许你组合 shell 程序，这也是使 shell 成为如此高效工作环境的部分原因！

实际上，大多数 shell 实现了一个完整的编程语言（如 bash），就像 Python 或 Ruby 一样。它有变量、条件、循环和函数。当你在 shell 中运行命令时，你实际上是在编写一小段代码，由你的 shell 解释。我们今天不会教你所有的 bash，但有一些部分你会觉得特别有用：

首先是重定向：`>file` 让你将程序的标准输出写入 `file` 而不是终端。这使得事后分析更容易。`>>file` 将追加到 `file` 而不是覆盖它。还有 `<file`，它告诉 shell 从 `file` 而不是从键盘读取作为程序的标准输入。

> 这是提到 `tee` 程序的好时机。`tee` 会将标准输入打印到标准输出（就像 `cat` 一样！），但也会将其写入文件。所以 `verbose cmd | tee verbose.log | grep CRITICAL` 会将完整的详细日志保存到文件，同时保持终端整洁！

接下来是条件：`if command1; then command2; command3; fi` 将执行 `command1`，如果它没有产生错误，将运行 `command2` 和 `command3`。如果你愿意，你也可以有一个 `else` 分支。最常用作 `command1` 的命令是 `test` 命令，通常简写为 `[`，它让你评估诸如"文件是否存在"（`test -f file` / `[ -f file ]`）或"字符串是否等于另一个字符串"（`[ "$var" = "string" ]`）之类的条件。在 bash 中，还有 `[[ ]]`，它是 `test` 的一个"更安全"的内置版本，在引用方面有更少的奇怪行为。

Bash 也有两种形式的循环，`while` 和 `for`。`while command1; do command2; command3; done` 的功能就像等效的 `if` 命令，只是它会一遍又一遍地重新执行整个过程，只要 `command1` 不出错。`for varname in a b c d; do command; done` 执行 `command` 四次，每次 `$varname` 设置为 `a`、`b`、`c` 和 `d` 之一。你不会显式列出项目，你会经常使用"命令替换"，例如：

```bash
for i in $(seq 1 10); do
```

这执行命令 `seq 1 10`（打印从 1 到 10 的数字），然后将整个 `$()` 替换为该命令的输出，给你一个 10 次迭代的 for 循环。在旧代码中，你有时会看到直接的反引号（如 ``for i in `seq 1 10`; do``）代替 `$()`，但你应该强烈首选 `$()` 形式，因为它可以嵌套。

虽然你可以在提示符中直接编写长 shell 脚本，但通常你会想把它们写入 `.sh` 文件。例如，这是一个脚本，它会在循环中运行一个程序直到失败，只打印失败运行的输出，同时在后台给 CPU 施加压力（例如用于复现不稳定的测试）：

```bash
#!/bin/bash
set -euo pipefail

# 在后台启动 CPU 压力测试
stress --cpu 8 &
STRESS_PID=$!

# 设置日志文件
LOGFILE="test_runs_$(date +%s).log"
echo "Logging to $LOGFILE"

# 运行测试直到有一个失败
RUN=1
while cargo test my_test > "$LOGFILE" 2>&1; do
    echo "Run $RUN passed"
    ((RUN++))
done

# 清理并报告
kill $STRESS_PID
echo "Test failed on run $RUN"
echo "Last 20 lines of output:"
tail -n 20 "$LOGFILE"
echo "Full log: $LOGFILE"
```

这里面有一些新东西，我建议你花些时间深入研究，因为它们在编写有用的 shell 调用时非常有用，比如后台作业（`&`）来并发运行程序、更复杂的 [shell 重定向](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)和[算术扩展](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html)。

不过，值得花点时间看看程序的前两行。第一行是"shebang"——你会在其他文件中也看到这个，不仅仅是 shell 脚本。当以魔法咒语 `#!/path` 开头的文件被执行时，shell 会启动 `/path` 处的程序，并将文件内容作为输入传递给它。在 shell 脚本的情况下，这意味着将 shell 脚本的内容传递给 `/bin/bash`，但你也可以用 `/usr/bin/python` 的 shebang 行编写 Python 脚本！

第二行是让 bash 变得"更严格"的一种方式，并缓解编写 shell 脚本时的一些陷阱。`set` 可以接受很多参数，但简单来说：`-e` 使任何命令失败时脚本提前退出；`-u` 使使用未定义变量时脚本崩溃而不是只使用空字符串；`-o pipefail` 使 `|` 序列中的程序失败时整个 shell 脚本也提前退出。

> Shell 编程是一个深奥的话题，就像任何编程语言一样，但要注意：bash 有不寻常多的陷阱，以至于有[多个](https://tldp.org/LDP/abs/html/gotchas.html)网站专门[列出它们](https://mywiki.wooledge.org/BashPitfalls)。我强烈建议在编写它们时大量使用 [shellcheck](https://www.shellcheck.net/)。LLMs 也非常擅长编写和调试 shell 脚本，以及在它们变得过于笨重时将它们翻译成"真正的"编程语言（如 Python）（超过 100 行）。

# 下一步

现在你知道如何在 shell 中完成基本任务了。你应该能够导航查找感兴趣的文件并使用大多数程序的基本功能。在下一讲中，我们将讨论如何使用 shell 和许多方便的命令行程序来执行和自动化更复杂的任务。

# 练习

本课程的所有课程都附带一系列练习。有些给你一个具体的任务要做，而另一些是开放式的，比如"尝试使用 X 和 Y 程序"。我们强烈鼓励你尝试一下。

我们没有为练习编写解答。如果你在某个特定问题上卡住了，请随时在 [Discord](https://ossu.dev/#community) 的 `#missing-semester-forum` 发帖，或者给我们发送一封电子邮件，描述你迄今为止尝试的内容，我们会尽力帮助你。这些练习也可能作为与 LLM 对话的初始提示，你可以在其中交互式地深入探讨该主题。这些练习的真正价值在于发现答案的旅程，而不是答案本身。我们鼓励你在完成练习时沿着切线探索并问"为什么"，而不是仅仅寻找解决问题的最短路径。

1. 对于本课程，你需要使用 Unix shell，如 Bash 或 ZSH。如果你使用的是 Linux 或 macOS，你不需要做任何特殊的事情。如果你使用的是 Windows，你需要确保你没有运行 cmd.exe 或 PowerShell；你可以使用 [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) 或 Linux 虚拟机来使用 Unix 风格的命令行工具。要确保你正在运行适当的 shell，你可以尝试命令 `echo $SHELL`。如果它显示类似 `/bin/bash` 或 `/usr/bin/zsh` 的内容，那意味着你正在运行正确的程序。

1. `ls` 的 `-l` 标志做什么？运行 `ls -l /` 并检查输出。每行的前 10 个字符是什么意思？（提示：`man ls`）

1. 在命令 `find ~/Downloads -type f -name "*.zip" -mtime +30` 中，`*.zip` 是一个"glob"。什么是 glob？创建一个包含一些文件的测试目录，并尝试使用 `ls *.txt`、`ls file?.txt` 和 `ls {a,b,c}.txt` 等模式。参见 Bash 手册中的[模式匹配](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html)。

1. `'单引号'`、`"双引号"` 和 `$'ANSI 引号'` 之间有什么区别？编写一个命令，打印包含字面 `$`、`!` 和换行符的字符串。参见[引用](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)。

1. Shell 有三个标准流：stdin（0）、stdout（1）和 stderr（2）。运行 `ls /nonexistent /tmp` 并将 stdout 重定向到一个文件，stderr 重定向到另一个文件。你会如何将两者重定向到同一个文件？参见[重定向](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)。

1. `$?` 保存最后一个命令的退出状态（0 = 成功）。`&&` 仅在前一个成功时运行下一个命令；`||` 仅在前一个失败时运行它。编写一个单行命令，仅在 `/tmp/mydir` 不存在时创建它。参见[退出状态](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)。

1. 为什么 `cd` 必须内置在 shell 本身而不是一个独立程序？（提示：考虑子进程在父进程中能做什么和不能做什么。）

1. 编写一个脚本，接受一个文件名作为参数（`$1`）并使用 `test -f` 或 `[ -f ... ]` 检查文件是否存在。它应该根据文件是否存在打印不同的消息。参见 [Bash 条件表达式](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)。

1. 将上一次练习的脚本保存到一个文件（例如 `check.sh`）。尝试用 `./check.sh somefile` 运行它。发生了什么？现在运行 `chmod +x check.sh` 再试一次。为什么这个步骤是必要的？（提示：在 `chmod` 前后查看 `ls -l check.sh`。）

1. 如果你在脚本的 `set` 标志中添加 `-x` 会发生什么？用一个简单的脚本尝试并观察输出。参见 [Set 内置命令](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)。

1. 编写一个命令，将文件复制到带有今天日期的备份文件中（例如 `notes.txt` → `notes_2026-01-12.txt`）。（提示：`$(date +%Y-%m-%d)`）。参见[命令替换](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)。

1. 修改讲座中的不稳定测试脚本，接受测试命令作为参数而不是硬编码 `cargo test my_test`。（提示：`$1` 或 `$@`）。参见[特殊参数](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)。

1. 使用管道查找你的主目录中最常见的 5 个文件扩展名。（提示：结合 `find`、`grep` 或 `sed` 或 `awk`、`sort`、`uniq -c` 和 `head`。）

1. `xargs` 将 stdin 的行转换为命令参数。一起使用 `find` 和 `xargs`（不是 `find -exec`）查找目录中的所有 `.sh` 文件并用 `wc -l` 计算每行的行数。加分项：让它处理带有空格的文件名。（提示：`-print0` 和 `-0`）。参见 `man xargs`。

1. 使用 `curl` 获取课程网站的 HTML（`https://missing.csail.mit.edu/`）并将其管道到 `grep` 以计算列出了多少讲座。（提示：查找每个讲座出现一次的模式；使用 `curl -s` 静默进度输出。）

1. [`jq`](https://jqlang.github.io/jq/) 是处理 JSON 数据的强大工具。用 `curl` 获取 `https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json` 的示例数据，并使用 `jq` 提取版本号大于 6 的人的姓名。（提示：先管道到 `jq .` 查看结构；然后尝试 `jq '.[] | select(...) | .name'`）

1. `awk` 可以根据列值过滤行并操作输出。例如，`awk '$3 ~ /pattern/ {$4=""; print}'` 只打印第三列匹配 `pattern` 的行，同时省略第四列。编写一个 `awk` 命令，只打印第二列大于 100 的行，并交换第一列和第三列。用 `printf 'a 50 x\nb 150 y\nc 200 z\n'` 测试。

1. 剖析讲座中的 SSH 日志管道：每一步做什么？然后构建类似的东西，从 `~/.bash_history`（或 `~/.zsh_history`）中查找你最常用的 shell 命令。