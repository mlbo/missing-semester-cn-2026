---
layout: lecture
title: "开发环境与工具"
description: >
  学习 IDE、Vim、语言服务器和 AI 驱动的开发工具。
thumbnail: /static/assets/thumbnails/2026/lec3.png
date: 2026-01-14
ready: true
video:
  aspect: 56.25
  id: QnM1nVzrkx8
---

开发环境是用于开发软件的一组工具。开发环境的核心是文本编辑功能，以及语法高亮、类型检查、代码格式化和自动完成等伴随功能。集成开发环境（IDE）如 [VS Code][vs-code] 将所有这些功能整合到一个应用程序中。基于终端的开发工作流结合了 [tmux](https://github.com/tmux/tmux)（终端复用器）、[Vim](https://www.vim.org/)（文本编辑器）、[Zsh](https://www.zsh.org/)（shell）以及特定语言的命令行工具，如 [Ruff](https://docs.astral.sh/ruff/)（Python 代码检查器和格式化工具）和 [Mypy](https://mypy-lang.org/)（Python 类型检查器）等工具。

IDE 和基于终端的工作流各有优缺点。例如，图形 IDE 可能更容易学习，而且今天的 IDE 通常有更好的开箱即用的 AI 集成，如 AI 自动完成；另一方面，基于终端的工作流更轻量级，而且在你没有 GUI 或无法安装软件的环境中，它们可能是你唯一的选择。我们建议你对两种方式都有基本的了解，并至少精通其中一种。如果你还没有首选的 IDE，我们建议从 [VS Code][vs-code] 开始。

在本讲座中，我们将涵盖：

- [文本编辑与 Vim](#文本编辑与-vim)
- [代码智能与语言服务器](#代码智能与语言服务器)
- [AI 驱动的开发](#ai-驱动的开发)
- [扩展和其他 IDE 功能](#扩展和其他-ide-功能)

[vs-code]: https://code.visualstudio.com/

# 文本编辑与 Vim

在编程时，你大部分时间都在导航代码、阅读代码片段和编辑代码，而不是从头到尾写长段代码或阅读文件。[Vim] 是一个为这种任务分布优化的文本编辑器。

**Vim 的哲学。** Vim 有一个美丽的理念作为基础：它的接口本身就是一种编程语言，专为导航和编辑文本而设计。按键（具有助记名称）是命令，这些命令是可组合的。Vim 避免使用鼠标，因为它太慢了；Vim 甚至避免使用箭头键，因为它需要太多的移动。结果是：一个感觉像脑机接口的编辑器，与你的思维速度相匹配。

**其他软件中的 Vim 支持。** 你不必使用 [Vim] 本身就能从其核心理念中受益。许多涉及任何类型文本编辑的程序都支持"Vim 模式"，要么作为内置功能，要么作为插件。例如，VS Code 有 [VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim) 插件，Zsh 有[内置支持](https://zsh.sourceforge.io/Guide/zshguide04.html)用于 Vim 模拟，甚至 Claude Code 也有[内置支持](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode)用于 Vim 编辑器模式。你使用的任何涉及文本编辑的工具很可能都以某种方式支持 Vim 模式。

## 模态编辑

Vim 是一个模态编辑器：它有不同的操作模式用于不同类别的任务。

- **普通模式**：用于在文件中移动和进行编辑
- **插入模式**：用于插入文本
- **替换模式**：用于替换文本
- **可视模式**（普通、行或块）：用于选择文本块
- **命令行模式**：用于运行命令

按键在不同操作模式下有不同的含义。例如，字母 `x` 在插入模式下只会插入一个字面字符 "x"，但在普通模式下，它会删除光标下的字符，在可视模式下，它会删除选中的内容。

在其默认配置中，Vim 在左下角显示当前模式。初始/默认模式是普通模式。你通常大部分时间会在普通模式和插入模式之间切换。

你可以通过按 `<ESC>`（转义键）从任何模式切换回普通模式。从普通模式，用 `i` 进入插入模式，用 `R` 进入替换模式，用 `v` 进入可视模式，用 `V` 进入可视行模式，用 `<C-v>`（Ctrl-V，有时也写作 `^V`）进入可视块模式，用 `:` 进入命令行模式。

你使用 Vim 时会大量使用 `<ESC>` 键：考虑将 Caps Lock 重新映射为 Escape（[macOS 说明](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)）或创建一个简单的键序列作为 `<ESC>` 的[替代映射](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings)。

## 基础：插入文本

从普通模式，按 `i` 进入插入模式。现在，Vim 的行为就像任何其他文本编辑器，直到你按 `<ESC>` 返回普通模式。这，连同上面解释的基础知识，是你开始使用 Vim 编辑文件所需的一切（虽然如果你把所有时间都花在插入模式下编辑，效率不会特别高）。

## Vim 的接口是一种编程语言

Vim 的接口是一种编程语言。按键（具有助记名称）是命令，这些命令可以组合。这实现了高效的移动和编辑，特别是当命令成为肌肉记忆后，就像你学会键盘布局后打字变得超级高效一样。

### 移动

你应该大部分时间都在普通模式，使用移动命令来导航文件。Vim 中的移动也称为"名词"，因为它们指的是文本块。

- 基本移动：`hjkl`（左、下、上、右）
- 单词：`w`（下一个单词）、`b`（单词开头）、`e`（单词结尾）
- 行：`0`（行首）、`^`（第一个非空白字符）、`$`（行尾）
- 屏幕：`H`（屏幕顶部）、`M`（屏幕中间）、`L`（屏幕底部）
- 滚动：`Ctrl-u`（向上）、`Ctrl-d`（向下）
- 文件：`gg`（文件开头）、`G`（文件结尾）
- 行号：`:{number}<CR>` 或 `{number}G`（第 {number} 行）
    - `<CR>` 指的是回车/回车键
- 杂项：`%`（匹配项，如括号或大括号）
- 查找：`f{character}`、`t{character}`、`F{character}`、`T{character}`
    - 在当前行向前/向后查找/到 {character}
    - `,` / `;` 用于导航匹配项
- 搜索：`/{regex}`，`n` / `N` 用于导航匹配项

### 选择

可视模式：

- 可视模式：`v`
- 可视行模式：`V`
- 可视块模式：`Ctrl-v`

可以使用移动键进行选择。

### 编辑

你过去用鼠标做的一切，现在都用键盘使用与移动命令组合的编辑命令来完成。这就是 Vim 的接口开始看起来像编程语言的地方。Vim 的编辑命令也称为"动词"，因为动词作用于名词。

- `i` 进入插入模式
    - 但对于操作/删除文本，想要使用比退格更强大的东西
- `o` / `O` 在下方/上方插入行
- `d{motion}` 删除 {motion}
    - 例如 `dw` 是删除单词，`d$` 是删除到行尾，`d0` 是删除到行首
- `c{motion}` 更改 {motion}
    - 例如 `cw` 是更改单词
    - 类似于 `d{motion}` 后跟 `i`
- `x` 删除字符（等同于 `dl`）
- `s` 替换字符（等同于 `cl`）
- 可视模式 + 操作
    - 选择文本，`d` 删除它或 `c` 更改它
- `u` 撤销，`<C-r>` 重做
- `y` 复制/"yank"（其他一些命令如 `d` 也会复制）
- `p` 粘贴
- 还有更多要学的：例如，`~` 翻转字符大小写，`J` 连接行

### 计数

你可以将名词和动词与计数组合，这将执行给定操作一定次数。

- `3w` 向前移动 3 个单词
- `5j` 向下移动 5 行
- `7dw` 删除 7 个单词

### 修饰符

你可以使用修饰符来更改名词的含义。一些修饰符是 `i`，意思是"内部"或"里面"，以及 `a`，意思是"周围"。

- `ci(` 更改当前括号对内的内容
- `ci[` 更改当前方括号对内的内容
- `da'` 删除单引号字符串，包括周围的单引号

## 综合运用

这是一个有问题的 [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz) 实现：

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print("fizz", end="")
        if i % 5 == 0:
            print("fizz", end="")
        if i % 3 and i % 5:
            print(i, end="")
        print()

def main():
    fizz_buzz(20)
```

我们使用以下命令序列来修复问题，从普通模式开始：

- Main 从未被调用
    - `G` 跳转到文件末尾
    - `o` 在下方打开一个新行
    - 输入 `if __name__ == "__main__": main()`
        - 如果你的编辑器有 Python 语言支持，它可能会在插入模式中为你做一些自动缩进
    - `<ESC>` 返回普通模式
- 从 0 开始而不是 1
    - `/` 后跟 `range` 和 `<CR>` 搜索 "range"
    - `ww` 向前移动两个单词（你也可以使用 `2w`，但在实践中，对于小计数，重复按键很常见）
    - `i` 切换到插入模式，添加 `1,`
    - `<ESC>` 返回普通模式
    - `e` 跳到下一个单词的末尾
    - `a` 开始追加文本，添加 `+ 1`
    - `<ESC>` 返回普通模式
- 对于 5 的倍数打印 "fizz"
    - `:6<CR>` 跳转到第 6 行
    - `ci"` 更改 `"` 内的内容，更改为 `"buzz"`
    - `<ESC>` 返回普通模式

## 学习 Vim

学习 Vim 的最好方法是学习基础（我们到目前为止涵盖的内容），然后在所有软件中启用 Vim 模式并开始在实践中使用它。避免使用鼠标或箭头键的诱惑；在某些编辑器中，你可以取消绑定箭头键来强迫自己养成良好的习惯。

### 其他资源

- 上一次课程迭代的 [Vim 讲座](/2020/editors/) —— 我们在那里更深入地介绍了 Vim
- `vimtutor` 是随 Vim 安装的教程 —— 如果安装了 Vim，你应该能够从 shell 运行 `vimtutor`
- [Vim Adventures](https://vim-adventures.com/) 是一个学习 Vim 的游戏
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) 有各种 Vim 技巧
- [VimGolf](https://www.vimgolf.com/) 是[代码高尔夫](https://en.wikipedia.org/wiki/Code_golf)，但编程语言是 Vim 的 UI
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/)（书籍）

[Vim]: https://www.vim.org/

# 代码智能与语言服务器

IDE 通常提供需要通过代码语义理解的语言特定支持，这是通过连接到实现[语言服务器协议](https://microsoft.github.io/language-server-protocol/)的语言服务器的 IDE 扩展实现的。例如，[VS Code 的 Python 扩展](https://marketplace.visualstudio.com/items?itemName=ms-python.python)依赖于 [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)，[VS Code 的 Go 扩展](https://marketplace.visualstudio.com/items?itemName=golang.go)依赖于官方的 [gopls](https://go.dev/gopls/)。通过为你使用的语言安装扩展和语言服务器，你可以在 IDE 中启用许多语言特定功能，例如：

- **代码补全。** 更好的自动完成和自动建议，例如能够在输入 `object.` 后看到对象的字段和方法。
- **内联文档。** 在悬停和自动建议时看到文档。
- **跳转到定义。** 从使用位置跳转到定义，例如能够从字段引用 `object.field` 跳转到字段的定义。
- **查找引用。** 上面的反向操作，查找特定项目（如字段或类型）被引用的所有位置。
- **导入帮助。** 组织导入、删除未使用的导入、标记缺失的导入。
- **代码质量。** 这些工具可以独立使用，但此功能通常也由语言服务器提供。代码格式化自动缩进和自动格式化代码，类型检查器和代码检查器在你输入时发现代码中的错误。我们将在[代码质量讲座]({{ site.baseurl }}/2026/code-quality/)中更深入地介绍这类功能。

## 配置语言服务器

对于某些语言，你只需要安装扩展和语言服务器，就可以开始了。对于其他语言，要从语言服务器获得最大收益，你需要告诉 IDE 你的环境。例如，将 VS Code 指向你的 [Python 环境](https://code.visualstudio.com/docs/python/environments)将使语言服务器能够看到你安装的包。环境在我们关于打包和发布代码的[讲座]({{ site.baseurl }}/2026/shipping-code/)中有更深入的介绍。

根据语言的不同，你可能可以为你的语言服务器配置一些设置。例如，使用 VS Code 中的 Python 支持，你可以为不使用 Python 可选类型注解的项目禁用静态类型检查。

# AI 驱动的开发

自 2021 年中 [GitHub Copilot][github-copilot] 使用 OpenAI 的 [Codex 模型](https://openai.com/index/openai-codex/)推出以来，[LLMs](https://en.wikipedia.org/wiki/Large_language_model) 在软件工程中已被广泛采用。目前主要有三种使用形式：自动完成、内联聊天和编码代理。

[github-copilot]: https://github.com/features/copilot/ai-code-editor

## 自动完成

AI 驱动的自动完成与 IDE 中的传统自动完成形式相同，在你输入时在光标位置建议补全。有时，它被用作"开箱即用"的被动功能。除此之外，AI 自动完成通常使用代码注释进行提示。

例如，让我们编写一个脚本来下载这些讲座笔记的内容并提取所有链接。我们可以从以下开始：

```python
import requests

def download_contents(url: str) -> str:
```

模型将自动完成函数体：

```python
    response = requests.get(url)
    return response.text
```

我们可以使用注释进一步引导补全。例如，如果我们开始编写一个函数来提取所有 Markdown 链接，但它的名称不是特别描述性：

```python
def extract(contents: str) -> list[str]:
```

模型将自动完成类似这样的内容：

```python
    lines = contents.splitlines()
    return [line for line in lines if line.strip()]
```

我们可以通过代码注释来引导补全：

```python
def extract(content: str) -> list[str]:
    # extract all Markdown links from the content
```

这次，模型给出了更好的补全：

```python
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)
```

在这里，我们看到了这种 AI 编码工具的一个缺点：它只能在光标处提供补全。在这种情况下，将 `import re` 放在模块级别而不是函数内部会是更好的实践。

上面的例子使用了一个命名不佳的函数来演示如何使用注释来引导代码补全；在实践中，你会想用更具描述性的名称编写函数，如 `extract_links`，你会想编写文档字符串（基于此，模型应该生成类似于上面的补全）。

为了演示目的，我们可以完成脚本：

```python
print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

## 内联聊天

内联聊天让你选择一行或一个块，然后直接提示 AI 模型提出编辑建议。在这种交互模式下，模型可以对现有代码进行更改（这与自动完成不同，后者只完成光标后面的代码）。

继续上面的例子，假设我们决定不使用第三方 `requests` 库。我们可以选择相关的三行代码，调用内联聊天，说一些类似：

```
use built-in libraries instead
```

模型建议：

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')
```

## 编码代理

编码代理在[代理编码]({{ site.baseurl }}/2026/agentic-coding/)讲座中有深入介绍。

## 推荐软件

一些流行的 AI IDE 包括带有 [GitHub Copilot][github-copilot] 扩展的 [VS Code][vs-code] 和 [Cursor](https://cursor.com/)。GitHub Copilot 目前对[学生](https://github.com/education/students)、教师和热门开源项目维护者免费。这是一个快速发展的领域。许多领先的产品具有大致相同的功能。

# 扩展和其他 IDE 功能

IDE 是强大的工具，通过扩展变得更加强大。我们不能在单个讲座中涵盖所有这些功能，但在这里我们提供一些流行扩展的指针。我们鼓励你自己探索这个领域；有很多在线可用的流行 IDE 扩展列表，如用于 Vim 插件的 [Vim Awesome](https://vimawesome.com/) 和[按人气排序的 VS Code 扩展](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Installs)。

- [开发容器](https://containers.dev/)：由流行的 IDE 支持（例如，[VS Code 支持](https://code.visualstudio.com/docs/devcontainers/containers)），开发容器让你使用容器运行开发工具。这对于可移植性或隔离性很有帮助。[打包和发布代码讲座]({{ site.baseurl }}/2026/shipping-code/)更深入地介绍了容器。
- 远程开发：使用 SSH 在远程机器上进行开发（例如，使用 [VS Code 的 Remote SSH 插件](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)）。例如，如果你想在云中的强大 GPU 机器上开发和运行代码，这可能很方便。
- 协作编辑：编辑同一个文件，Google Docs 风格（例如，使用 [VS Code 的 Live Share 插件](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare)）。

# 练习

1. 在所有支持它的软件中启用 Vim 模式，如你的编辑器和你的 shell，并在接下来一个月的所有文本编辑中使用 Vim 模式。每当有什么看起来效率低下时，或者当你认为"一定有更好的方法"时，尝试用 Google 搜索，很可能会找到更好的方法。

1. 完成 [VimGolf](https://www.vimgolf.com/) 的一个挑战。

1. 为你正在进行的项目配置 IDE 扩展和语言服务器。确保所有预期功能（如库依赖的跳转到定义）按预期工作。如果你没有可用于此练习的代码，你可以使用 GitHub 上的一些开源项目（如[这个](https://github.com/spf13/cobra)）。

1. 浏览 IDE 扩展列表并安装一个对你有用的扩展。