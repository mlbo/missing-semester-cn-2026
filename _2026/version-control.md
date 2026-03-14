---
layout: lecture
title: "版本控制与 Git"
description: >
  学习 Git 的数据模型以及如何使用 Git 进行版本控制和协作。
thumbnail: /static/assets/thumbnails/2026/lec5.png
date: 2026-01-16
ready: true
video:
  aspect: 56.25
  id: 9K8lB61dl3Y
---

版本控制系统（VCS）是用于跟踪源代码（或其他文件和文件夹集合）更改的工具。顾名思义，这些工具有助于维护更改历史；此外，它们还促进协作。从逻辑上讲，VCS 跟踪文件夹及其内容在一系列快照中的更改，其中每个快照封装了顶级目录内文件/文件夹的完整状态。VCS 还维护元数据，如谁创建了每个快照、与每个快照关联的消息等。

为什么版本控制有用？即使你独自工作，它也可以让你查看项目的旧快照，保留为什么进行某些更改的日志，并行开发多个分支等等。与他人合作时，它是查看其他人更改内容以及解决并发开发冲突的宝贵工具。

现代 VCS 还可以让你轻松（通常是自动）回答以下问题：

- 谁编写了这个模块？
- 这个特定文件的这一特定行是什么时候编辑的？由谁编辑？为什么要编辑？
- 在最近 1000 次修订中，某个特定的单元测试是什么时候/为什么停止工作的？

虽然存在其他 VCS，但 **Git** 是版本控制的事实标准。这个 [XKCD 漫画](https://xkcd.com/1597/)捕捉了 Git 的名声：

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

因为 Git 的接口是一个泄漏的抽象，自顶向下学习 Git（从其接口/命令行界面开始）可能会导致很多困惑。你可能会记住一些命令并将它们视为魔法咒语，每当出现问题时就遵循上面漫画中的方法。

虽然 Git 的接口确实很难看，但其底层设计和思想是美丽的。难看的接口需要记忆，而美丽的设计可以理解。因此，我们对 Git 进行自底向上的解释，从其数据模型开始，然后介绍命令行界面。一旦理解了数据模型，就可以更好地理解命令是如何操作底层数据模型的。

# Git 的数据模型

Git 的独创性在于其精心设计的数据模型，它支持版本控制的所有良好功能，如维护历史、支持分支和实现协作。

## 快照

Git 将某个顶级目录内的文件和文件夹集合的历史建模为一系列快照。在 Git 术语中，文件称为"blob"，它只是一堆字节。目录称为"tree"，它将名称映射到 blob 或 tree（因此目录可以包含其他目录）。快照是被跟踪的顶级 tree。例如，我们可能有如下的 tree：

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

顶级 tree 包含两个元素，一个 tree "foo"（它本身包含一个元素，一个 blob "bar.txt"），和一个 blob "baz.txt"。

## 建模历史：关联快照

版本控制系统应该如何关联快照？一个简单的模型是线性历史。历史将是按时间顺序排列的快照列表。出于许多原因，Git 不使用这种简单的模型。

在 Git 中，历史是快照的有向无环图（DAG）。这听起来可能像花哨的数学词汇，但不要被吓到。这只是意味着 Git 中的每个快照都引用一组"父提交"，即它之前的快照。它是一组父提交而不是单个父提交（在线性历史中会是这种情况），因为快照可能有多个父提交，例如，由于合并两个并行的开发分支。

Git 将这些快照称为"commit"。可视化提交历史可能看起来像这样：

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

在上面的 ASCII 艺术中，`o` 对应于单个提交（快照）。箭头指向每个提交的父提交（这是一个"先于"关系，而不是"后于"）。在第三个提交之后，历史分叉成两个独立的分支。这可能对应于，例如，两个独立的功能正在并行开发，彼此独立。将来，这些分支可能会被合并，创建一个包含两个功能的新快照，产生一个新的历史，如下所示，新创建的合并提交以粗体显示：

<pre class="highlight">
<code>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</code>
</pre>

Git 中的提交是不可变的。这并不意味着错误无法纠正；只是对提交历史的"编辑"实际上是创建全新的提交，并更新引用（见下文）以指向新的提交。

## 数据模型，以伪代码表示

以伪代码形式查看 Git 的数据模型可能会有帮助：

```
// 文件是一堆字节
type blob = array<byte>

// 目录包含命名的文件和目录
type tree = map<string, tree | blob>

// 提交有父提交、元数据和顶级 tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

这是一个干净、简单的历史模型。

## 对象和内容寻址

"对象"是 blob、tree 或 commit：

```
type object = blob | tree | commit
```

在 Git 的数据存储中，所有对象都通过其 [SHA-1 哈希](https://en.wikipedia.org/wiki/SHA-1)进行内容寻址。

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blob、tree 和 commit 以这种方式统一：它们都是对象。当它们引用其他对象时，它们实际上不会在磁盘表示中包含它们，而是通过其哈希引用它们。

例如，[上面](#快照)示例目录结构的 tree（使用 `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d` 可视化）看起来像这样：

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

tree 本身包含指向其内容的指针，`baz.txt`（一个 blob）和 `foo`（一个 tree）。如果我们使用 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85` 查看 baz.txt 对应的哈希地址的内容，我们得到：

```
git is wonderful
```

## 引用

现在，所有快照都可以通过其 SHA-1 哈希来识别。这很不方便，因为人类不擅长记住 40 个十六进制字符的字符串。

Git 解决这个问题的方法是为 SHA-1 哈希提供人类可读的名称，称为"引用"。引用是指向提交的指针。与不可变的对象不同，引用是可变的（可以更新以指向新提交）。例如，`master` 引用通常指向主分支开发中的最新提交。

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

有了这个，Git 可以使用像 "master" 这样的人类可读名称来引用历史中的特定快照，而不是一长串十六进制字符串。

一个细节是我们经常需要知道我们在历史中"当前位置"的概念，这样当我们创建新快照时，我们知道它是相对于什么（我们如何设置提交的 `parents` 字段）。在 Git 中，"当前位置"是一个称为 "HEAD" 的特殊引用。

## 仓库

最后，我们可以定义什么是 Git 仓库（大致上）：它是数据 `objects` 和 `references`。

在磁盘上，Git 存储的所有内容都是对象和引用：这就是 Git 数据模型的全部。所有 `git` 命令都映射到通过添加对象和添加/更新引用对提交 DAG 的某种操作。

每当你在输入任何命令时，想想该命令对底层图数据结构进行了什么操作。相反，如果你试图对提交 DAG 进行某种更改，例如"丢弃未提交的更改并让 'master' 引用指向提交 `5d83f9e`"，很可能有一个命令可以做到（例如在这种情况下，`git checkout master; git reset --hard 5d83f9e`）。

# 暂存区

这是另一个与数据模型正交的概念，但它是创建提交接口的一部分。

你可能会想象实现上述快照的一种方法是有一个"创建快照"命令，根据工作目录的_当前状态_创建新快照。一些版本控制工具就是这样工作的，但 Git 不是。我们需要干净的快照，而根据当前状态创建快照并不总是理想的。例如，想象这样一个场景：你实现了两个独立的功能，你想创建两个独立的提交，第一个引入第一个功能，下一个引入第二个功能。或者想象这样一个场景：你在代码中到处添加了调试打印语句，同时修复了一个 bug；你想提交 bug 修复，同时丢弃所有打印语句。

Git 通过一种称为"暂存区"的机制来适应这些场景，让你可以指定哪些修改应该包含在下一个快照中。

# Git 命令行界面

为了避免重复信息，我们不会在下面详细解释这些命令。更多信息请参阅强烈推荐的 [Pro Git](https://git-scm.com/book/en/v2)，或观看讲座视频。

## 基础

- `git help <command>`：获取 git 命令的帮助
- `git init`：创建一个新的 git 仓库，数据存储在 `.git` 目录中
- `git status`：告诉你正在发生什么
- `git add <filename>`：将文件添加到暂存区
- `git commit`：创建一个新提交
    - 写[好的提交消息](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)！
    - 写[好的提交消息](https://chris.beams.io/posts/git-commit/)的更多理由！
- `git log`：显示扁平化的历史日志
- `git log --all --graph --decorate`：将历史可视化为 DAG
- `git diff <filename>`：显示你相对于暂存区所做的更改
- `git diff <revision> <filename>`：显示快照之间文件中的差异
- `git checkout <revision>`：更新 HEAD（如果检出分支则更新当前分支）

## 分支和合并

- `git branch`：显示分支
- `git branch <name>`：创建一个分支
- `git switch <name>`：切换到分支
- `git checkout -b <name>`：创建一个分支并切换到它
    - 等同于 `git branch <name>; git switch <name>`
- `git merge <revision>`：合并到当前分支
- `git mergetool`：使用一个漂亮的工具来帮助解决合并冲突
- `git rebase`：将一组补丁变基到新的基础上

## 远程仓库

- `git remote`：列出远程仓库
- `git remote add <name> <url>`：添加一个远程仓库
- `git push <remote> <local branch>:<remote branch>`：将对象发送到远程，并更新远程引用
- `git branch --set-upstream-to=<remote>/<remote branch>`：设置本地分支和远程分支之间的对应关系
- `git fetch`：从远程检索对象/引用
- `git pull`：等同于 `git fetch; git merge`
- `git clone`：从远程下载仓库

## 撤销

- `git commit --amend`：编辑提交的内容/消息
- `git reset <file>`：取消暂存文件
- `git restore`：丢弃更改

# 高级 Git

- `git config`：Git 是[高度可定制的](https://git-scm.com/docs/git-config)
- `git clone --depth=1`：浅克隆，没有完整的版本历史
- `git add -p`：交互式暂存
- `git rebase -i`：交互式变基
- `git blame`：显示谁最后编辑了哪一行
- `git stash`：临时移除工作目录的修改
- `git bisect`：二分查找历史（例如用于回归）
- `git revert`：创建一个新提交来撤销早期提交的效果
- `git worktree`：同时检出多个分支
- `.gitignore`：[指定](https://git-scm.com/docs/gitignore)要忽略的故意未跟踪文件

# 杂项

- **GUI**：有很多 [GUI 客户端](https://git-scm.com/downloads/guis)可用于 Git。我们个人不使用它们，而是使用命令行界面。
- **Shell 集成**：在 shell 提示符中包含 Git 状态非常方便（[zsh](https://github.com/olivierverdier/zsh-git-prompt)、[bash](https://github.com/magicmonty/bash-git-prompt)）。通常包含在 [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) 等框架中。
- **编辑器集成**：与上述类似，具有许多功能的方便集成。[fugitive.vim](https://github.com/tpope/vim-fugitive) 是 Vim 的标准插件。
- **工作流**：我们教了你数据模型，以及一些基本命令；我们没有告诉你在大型项目上应该遵循什么实践（有[很多](https://nvie.com/posts/a-successful-git-branching-model/)不同的[方法](https://www.endoflineblog.com/gitflow-considered-harmful)）。
- **GitHub**：Git 不是 GitHub。GitHub 有一种特定的方式来向其他项目贡献代码，称为[拉取请求](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)。
- **其他 Git 提供商**：GitHub 不是特殊的：有很多 Git 仓库托管服务，如 [GitLab](https://about.gitlab.com/) 和 [BitBucket](https://bitbucket.org/)。

# 资源

- [Pro Git](https://git-scm.com/book/en/v2) 是**强烈推荐阅读的**。阅读第 1-5 章应该能教会你精通使用 Git 所需的大部分知识，现在你已经理解了数据模型。后面的章节有一些有趣的高级内容。
- [Oh Shit, Git!?!](https://ohshitgit.com/) 是一本关于如何从一些常见 Git 错误中恢复的简短指南。
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) 是对 Git 数据模型的简短解释，比这些讲座笔记有更少的伪代码和更多的花哨图表。
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) 是对 Git 实现细节（不仅仅是数据模型）的详细解释，适合好奇的人。
- [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) 是一个浏览器游戏，教你学习 Git。

# 练习

1. 如果你没有任何 Git 经验，可以尝试阅读 [Pro Git](https://git-scm.com/book/en/v2) 的前几章，或者通过 [Learn Git Branching](https://learngitbranching.js.org/) 这样的教程学习。在学习过程中，将 Git 命令与数据模型联系起来。

1. 克隆[课程网站的仓库](https://github.com/missing-semester/missing-semester)。
    1. 通过将其可视化为图来探索版本历史。
    1. 谁是最后修改 `README.md` 的人？（提示：使用带参数的 `git log`）。
    1. 与 `_config.yml` 中 `collections:` 行的最后一次修改相关的提交消息是什么？（提示：使用 `git blame` 和 `git show`）。

1. 学习 Git 时的一个常见错误是提交不应由 Git 管理的大文件或添加敏感信息。尝试向仓库添加一个文件，进行一些提交，然后从_历史_中删除该文件（不仅仅是最新提交）。你可能想查看[这个](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)。

1. 从 GitHub 克隆某个仓库，并修改其中一个现有文件。当你执行 `git stash` 时会发生什么？当你运行 `git log --all --oneline` 时你会看到什么？运行 `git stash pop` 来撤销你用 `git stash` 做的操作。在什么场景下这可能有用？

1. 像许多命令行工具一样，Git 提供了一个配置文件（或点文件），称为 `~/.gitconfig`。在 `~/.gitconfig` 中创建一个别名，这样当你运行 `git graph` 时，你会得到 `git log --all --graph --decorate --oneline` 的输出。你可以直接[编辑](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias) `~/.gitconfig` 文件，也可以使用 `git config` 命令添加别名。关于 git 别名的信息可以在[这里](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)找到。

1. 你可以在运行 `git config --global core.excludesfile ~/.gitignore_global` 后在 `~/.gitignore_global` 中定义全局忽略模式。这设置了 Git 将使用的全局忽略文件的位置，但你仍然需要手动在该路径创建文件。设置你的全局 gitignore 文件以忽略特定于操作系统或编辑器的临时文件，如 `.DS_Store`。

1. Fork[课程网站的仓库](https://github.com/missing-semester/missing-semester)，找到一个错别字或你可以做的其他改进，并在 GitHub 上提交拉取请求（你可能想查看[这个](https://github.com/firstcontributions/first-contributions)）。请只提交有用的 PR（请不要发垃圾邮件！）。如果你找不到可以改进的地方，可以跳过这个练习。

1. 通过模拟协作场景来练习解决合并冲突：
    1. 用 `git init` 创建一个新仓库，并创建一个名为 `recipe.txt` 的文件，包含几行（例如，一个简单的食谱）。
    1. 提交它，然后创建两个分支：`git branch salty` 和 `git branch sweet`。
    1. 在 `salty` 分支中，修改一行（例如，将 "1 cup sugar" 改为 "1 cup salt"）并提交。
    1. 在 `sweet` 分支中，以不同的方式修改同一行（例如，将 "1 cup sugar" 改为 "2 cups sugar"）并提交。
    1. 现在切换到 `master` 并尝试 `git merge salty`，然后 `git merge sweet`。发生了什么？查看 `recipe.txt` 的内容 - `<<<<<<<`、`=======` 和 `>>>>>>>` 标记是什么意思？
    1. 通过编辑文件以保留你想要的内容、删除冲突标记并使用 `git add` 和 `git commit`（或 `git merge --continue`）完成合并来解决冲突。或者，尝试使用 `git mergetool` 通过图形或基于终端的合并工具来解决冲突。
    1. 使用 `git log --graph --oneline` 可视化你刚刚创建的合并历史。