---
layout: lecture
title: "代码质量"
description: >
  学习格式化、代码检查、测试、持续集成等内容。
thumbnail: /static/assets/thumbnails/2026/lec9.png
date: 2026-01-23
ready: true
video:
  aspect: 56.25
  id: XBiLUNx84CQ
---

有各种各样的工具和技术可以帮助开发者编写高质量的代码。在本讲座中，我们将涵盖：

- [格式化](#格式化)
- [代码检查](#代码检查)
- [测试](#测试)
- [预提交钩子](#预提交钩子)
- [持续集成](#持续集成)
- [命令运行器](#命令运行器)

作为补充主题，我们还将介绍[正则表达式](#正则表达式)，这是一个跨领域主题，在代码质量（例如，用于运行匹配模式的测试子集）以及 IDE（例如，用于搜索和替换）等其他领域都有应用。

其中许多工具将是特定于语言的（例如，用于 Python 的 [Ruff](https://docs.astral.sh/ruff/) 代码检查器/格式化工具）。在某些情况下，工具支持多种语言（例如，[Prettier](https://prettier.io/) 代码格式化工具）。然而，这些概念几乎是通用的——你可以为任何编程语言找到代码格式化工具、代码检查器、测试库等。

# 格式化

代码自动格式化工具会自动美化表面语法。这样，你可以专注于更深层和更具挑战性的问题，而自动格式化工具处理琐碎细节，如字符串的 `'` 与 `"` 语法的一致性、二元运算符周围的空格（`x + y` 而不是 `x+y`）、`import` 语句的排序顺序以及避免过长的行。代码格式化工具的一个主要好处是它们可以在所有开发同一代码库的开发者之间标准化代码风格。

一些工具如 Prettier [高度可配置](https://prettier.io/docs/configuration)；你应该将配置文件检入项目的[版本控制](/2026/version-control/)。其他工具，如 [Black](https://github.com/psf/black) 和 [gofmt](https://pkg.go.dev/cmd/gofmt) 具有有限或没有可配置性，以减少[无谓争论](https://en.wikipedia.org/wiki/Law_of_triviality)。

你可以设置代码格式化工具的 [IDE 集成](/2026/development-environment/#代码智能与语言服务器)，这样你的代码将在你输入或保存文件时自动格式化。你也可以在项目中添加 [EditorConfig](https://editorconfig.org/) 文件，它向你的 IDE 传达某些项目级别的设置，如每种文件类型的缩进大小。

# 代码检查

代码检查器运行静态分析（在不运行代码的情况下分析你的代码）来发现代码中的反模式和潜在问题。这些工具比自动格式化工具更深入，超越表面语法。分析的深度因工具而异。

代码检查器配备了_规则_列表，以及可以在项目级别配置的预设。一些代码检查规则会产生误报，因此你可以在每个文件或每行基础上禁用它们。

好的代码检查器会有内置帮助或文档，解释每个代码检查规则——规则在寻找什么，为什么它是不好的，以及代码模式的更好替代方案是什么。例如，请参阅 [Ruff](https://docs.astral.sh/ruff/) 中 [SIM102](https://docs.astral.sh/ruff/rules/collapsible-if/) 规则的文档，它捕获 Python 代码中不必要的嵌套 `if` 语句。

一些代码检查器不仅可以标记问题，还可以自动为你修复某些问题。

除了特定于语言的代码检查器外，另一个可能有用的工具是 [semgrep](https://github.com/semgrep/semgrep)，这是一个"语义 grep"工具，在 AST 级别工作（而不是像 grep 那样在字符级别），并支持多种语言。你可以使用 semgrep 轻松为你的项目编写自定义代码检查规则。例如，如果你想防止 Python 中危险的 `subprocess.Popen(..., shell=True)`，你可以用以下命令找到该代码模式：

```bash
semgrep -l python -e "subprocess.Popen(..., shell=True, ...)"
```

# 测试

软件测试是一种标准技术，可以提高你对代码正确性的信心。你编写代码，然后编写代码来演练你编写的代码，如果代码没有按预期工作则引发错误。

你可以为不同粒度级别的代码块编写测试：用于单个函数的_单元测试_、用于模块或服务之间交互的_集成测试_，以及用于端到端场景的_功能测试_。你可以进行_测试驱动开发_，在编写任何实现代码之前先编写测试。当你在代码中发现错误时，你可以编写_回归测试_，这样如果将来功能出现问题，你就会发现。你可以编写_基于属性的测试_，这在 Haskell 的 [QuickCheck](https://hackage.haskell.org/package/QuickCheck) 中开创，并在许多库中实现，如 Python 的 [Hypothesis](https://hypothesis.readthedocs.io/)。哪种测试方法适合取决于你的项目；你可能会采用某种组合。

如果你的程序有数据库或 Web API 等外部依赖，在测试中_模拟_这些依赖可能会有帮助，而不是让代码在测试时与第三方依赖交互。

## 代码覆盖率

代码覆盖率是一个可以衡量测试质量的指标。代码覆盖率查看运行测试时执行了代码的哪些行，因此你可以确保覆盖所有代码路径。代码覆盖率工具可以向你显示逐行覆盖率，以指导你编写测试。诸如 [Codecov](https://app.codecov.io) 之类的服务提供 Web 界面，用于跟踪和查看项目历史中的代码覆盖率。

像任何指标一样，代码覆盖率并不完美；不要过度关注覆盖率，专注于编写高质量的测试。

# 预提交钩子

Git 预提交[钩子](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)，由 [pre-commit](https://pre-commit.com/) 框架简化，在每次 Git 提交之前自动运行用户指定的代码。项目通常使用预提交钩子在每次提交之前自动运行格式化工具和代码检查器，有时还有测试，以确保提交的代码符合项目代码风格并且没有某些问题。

# 持续集成

持续集成（CI）服务如 [GitHub Actions](https://github.com/features/actions) 可以在每次推送代码时（或在每次拉取请求时，或按计划）为你运行脚本。开发者通常使用 CI 服务运行代码质量工具，包括格式化工具、代码检查器和测试。对于编译型语言，你可以确保代码编译；对于静态类型语言，你可以确保它通过类型检查。每次推送新提交时运行 CI 可以捕获引入主版本代码的错误；在拉取请求上运行可以捕获贡献者提交的问题；按计划运行可以捕获外部依赖的问题（例如，开发者意外发布了[语义版本兼容](/2026/shipping-code/#releases--versioning)的破坏性更改）。

由于 CI 脚本与开发者机器分开运行，你可以在那里轻松运行长时间运行的任务。这可以被利用，例如，在不同的操作系统和编程语言版本上运行测试_矩阵_，以确保软件在所有这些环境中正常工作。

通常，在 CI 中运行的脚本不会直接对你的代码进行更改：它将在"仅检查"模式而不是"修复"模式下运行工具，例如，自动格式化工具将在代码不符合格式时引发错误。

仓库通常在其 README 中包含[状态徽章](https://docs.github.com/en/actions/how-tos/monitor-workflows/add-a-status-badge)，显示 CI 状态和其他信息如代码覆盖率。例如，下面是 Missing Semester 的当前构建状态。

[![Build Status](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml) [![Links Status](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml)

> 我们的[链接检查器](https://github.com/missing-semester/missing-semester/blob/master/.github/workflows/links.yml)，使用 [proof-html](https://github.com/anishathalye/proof-html) GitHub Action，经常失败，通常是由于第三方网站的问题。不过，它已经帮助我们捕获并修复了许多损坏的链接（有时是由于错别字，大多数时候是由于网站移动内容而没有添加重定向或网站消失）。

学习 CI 服务、格式化工具、代码检查器和测试库的好方法是通过示例。在 GitHub 上找到高质量的开源项目——在编程语言、领域、规模和范围等方面与你的项目越相似越好——并研究它们的 `pyproject.toml`、`.github/workflows/`、`DEVELOPMENT.md` 和其他相关文件。

## 持续部署

持续部署利用 CI 基础设施来实际_部署_更改。例如，Missing Semester 仓库使用持续部署到 GitHub Pages，这样每当我们 `git push` 更新的讲座笔记时，网站就会自动构建和部署。你可以在 CI 中构建其他类型的[工件](/2026/shipping-code/)，如应用程序的二进制文件或服务的 Docker 镜像。

# 命令运行器

命令运行器如 [just](https://github.com/casey/just) 简化了在项目上下文中运行命令的任务。当你在项目中建立代码质量基础设施时，你不想让你的开发者记住像 `uv run ruff check --fix` 这样的命令。使用命令运行器，这可以变成 `just lint`，你可以有类似的调用如 `just format`、`just typecheck` 等，用于开发者可能想为你的项目运行的所有不同工具。

一些特定于语言的项目或包管理器内置支持此类功能，这意味着你不需要使用像 `just` 这样与语言无关的工具。例如，[npm](https://nodejs.org/en/learn/getting-started/an-introduction-to-the-npm-package-manager)（Node.js）的 `package.json` 中的 `scripts` 部分和 [Hatch](https://hatch.pypa.io/)（Python）的 `pyproject.toml` 中的 `tool.hatch.envs.*.scripts` 部分支持此功能。

# 正则表达式

_正则表达式_，通常缩写为"regex"，是一种用于表示字符串集合的语言。正则表达式模式通常用于各种上下文中的模式匹配，如命令行工具和 IDE。例如，[ag](https://github.com/ggreer/the_silver_searcher) 支持用于代码库范围搜索的正则表达式模式（例如，`ag "import .* as .*"` 将找到 Python 中所有重命名的导入），[go test](https://pkg.go.dev/cmd/go#hdr-Test_packages) 支持 `-run [regexp]` 选项用于选择测试子集。此外，编程语言内置支持或第三方库用于正则表达式匹配，因此你可以使用正则表达式进行模式匹配、验证和解析等功能。

为了帮助建立直觉，下面是一些正则表达式模式的示例。在本讲座中，我们使用 [Python 正则表达式语法](https://docs.python.org/3/library/re.html)。正则表达式有多种变体，它们之间存在细微差异，尤其是在更复杂的功能方面。你可以使用像 [regex101](https://regex101.com/) 这样的在线正则表达式测试器来开发和调试正则表达式。

- `abc` --- 匹配字面量 "abc"。
- `missing|semester` --- 匹配字符串 "missing" 或字符串 "semester"。
- `\d{4}-\d{2}-\d{2}` --- 匹配 YYYY-MM-DD 格式的日期，如 "2026-01-14"。除了确保字符串由四位数字、一个破折号、两位数字、一个破折号和两位数字组成外，这并不验证日期，所以 "2026-01-99" 也匹配这个正则表达式模式。
- `.+@.+` --- 匹配电子邮件地址，包含一些文本、一个 "@" 然后更多文本的字符串。这只是最基本的验证，匹配像 "nonsense@@@email" 这样的字符串。一个没有误报或漏报地匹配电子邮件地址的正则表达式[存在](https://pdw.ex-parrot.com/Mail-RFC822-Address.html)但不切实际。

## 正则表达式语法

你可以在[此文档](https://docs.python.org/3/library/re.html#regular-expression-syntax)（或许多其他在线资源之一）中找到正则表达式语法的综合指南。以下是一些基本构建块：

- `abc` 匹配字面量字符串，当字符没有特殊含义时（在此示例中，"abc"）
- `.` 匹配任意单个字符
- `[abc]` 匹配括号中包含的单个字符（在此示例中，"a"、"b"或 "c"）
- `[^abc]` 匹配除括号中包含的字符以外的单个字符（例如，"d"）
- `[a-f]` 匹配括号中指示范围内包含的单个字符（例如，"c"，但不是 "q"）
- `a|b` 匹配任一模式（例如，"a" 或 "b"）
- `\d` 匹配任意数字字符（例如，"3"）
- `\w` 匹配任意单词字符（例如，"x"）
- `\b` 匹配任意单词_边界_（例如，在字符串 "missing semester" 中，匹配 "m" 之前、"g" 之后、"s" 之前和 "r" 之后）
- `(...)` 匹配模式的组
- `...?` 匹配零个或一个模式，如 `words?` 匹配 "word" 或 "words"
- `...*` 匹配任意数量的模式，如 `.*` 匹配任意数量的任意字符
- `...+` 匹配一个或多个模式，如 `\d+` 匹配任意非零数量的数字
- `...{N}` 匹配恰好 N 个模式，如 `\d{4}` 匹配 4 位数字
- `\.` 匹配字面量 "."
- `\\` 匹配字面量 "\\"
- `^` 匹配行首
- `$` 匹配行尾

## 捕获组和引用

如果你使用正则表达式组 `(...)`，你可以引用匹配的子部分用于提取或搜索替换目的。例如，要从 YYYY-MM-DD 样式日期中仅提取月份，你可以使用以下 Python 代码：

```python
>>> import re
>>> re.match(r"\d{4}-(\d{2})-\d{2}", "2026-01-14").group(1)
'01'
```

在文本编辑器中，你可以在替换模式中使用引用捕获组。语法可能因 IDE 而异。例如，在 VS Code 中，你可以使用像 `$1`、`$2` 等变量，在 Vim 中，你可以使用 `\1`、`\2` 等来引用组。

## 局限性

[正则语言](https://en.wikipedia.org/wiki/Regular_language)强大但有限；存在一些无法用标准正则表达式表示的字符串类（例如，[不可能](https://en.wikipedia.org/wiki/Pumping_lemma_for_regular_languages)编写一个正则表达式来匹配字符串集合 {a^n b^n \| n &ge; 0}，即相同数量的 "a" 后跟相同数量的 "b" 的字符串集合；更实际地说，像 HTML 这样的语言不是正则语言）。在实践中，现代正则表达式引擎支持前瞻和反向引用等功能，将支持扩展到正则语言之外，它们实际上非常有用，但重要的是要知道它们的表达能力仍然有限。对于更复杂的语言，你可能需要使用更强大的解析器类型（例如，参见 [pyparsing](https://github.com/pyparsing/pyparsing)，一个 [PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar) 解析器）。

## 学习正则表达式

我们建议学习基础知识（我们在本讲座中涵盖的内容），然后根据需要查阅正则表达式参考，而不是记住整个语言。

对话式 AI 工具可以有效地帮助你生成正则表达式模式。例如，尝试用以下查询提示你喜欢的 LLM：

```
Write a Python-style regex pattern that matches the requested path from log lines from Nginx. Here is an example log line:

169.254.1.1 - - [09/Jan/2026:21:28:51 +0000] "GET /feed.xml HTTP/2.0" 200 2995 "-" "python-requests/2.32.3"
```

# 练习

1. 为你正在进行的项目配置格式化工具、代码检查器和预提交钩子。如果你有很多错误：自动格式化应该处理格式错误。对于代码检查错误，尝试使用 [AI 代理](/2026/agentic-coding/)修复所有代码检查错误。确保 AI 代理可以运行代码检查器并观察结果，以便它可以在迭代循环中运行以修复所有问题。仔细检查结果以确保 AI 没有破坏你的代码！

1. 学习你所知语言的测试库，并为你正在进行的项目编写单元测试。运行代码覆盖率工具，生成 HTML 格式的覆盖率报告，并观察结果。你能找到被覆盖的行吗？你的代码覆盖率可能很低。尝试手动编写一些测试来提高它。尝试使用 [AI 代理](/2026/agentic-coding/)提高覆盖率；确保编码代理可以运行带覆盖率的测试并生成逐行覆盖率报告，以便它知道在哪里集中精力。AI 生成的测试真的好吗？

1. 为你正在进行的项目设置持续集成，以便在每次推送时运行。让 CI 运行格式化、代码检查和测试。故意破坏你的代码（例如，引入代码检查违规），并确保 CI 捕获它。

1. 尝试编写一个[正则表达式模式](#正则表达式)，并使用 `grep` [命令行工具](/2026/course-shell/)在你的代码中查找 `subprocess.Popen(..., shell=True)` 的出现。现在，尝试"破坏"正则表达式模式。[semgrep](#代码检查)仍然成功匹配使你的 grep 调用出错的危险代码吗？

1. 在你的 IDE 或文本编辑器中练习正则表达式搜索替换，将[这些讲座笔记](https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/code-quality.md)中的 `-` [Markdown 项目符号标记](https://spec.commonmark.org/0.31.2/#bullet-list-marker)替换为 `*` 项目符号标记。注意，只是替换文件中的所有 "-" 字符是不正确的，因为该字符有许多不是项目符号标记的用途。

1. 编写一个正则表达式，从 `{"name": "Alyssa P. Hacker", "college": "MIT"}` 形式的 JSON 结构中捕获名称（例如，在此示例中为 `Alyssa P. Hacker`）。提示：在你的第一次尝试中，你可能最终会编写一个提取 `Alyssa P. Hacker", "college": "MIT` 的正则表达式；阅读 [Python 正则表达式文档](https://docs.python.org/3/library/re.html)中关于贪婪量词的内容来弄清楚如何修复它。
    1. 使正则表达式模式即使在名称中有 `"` 字符的情况下也能工作（双引号可以在 JSON 中用 `\"` 转义）。
    1. 我们**不**建议在实践中使用正则表达式来处理复杂的解析问题。弄清楚如何使用你的编程语言的 JSON 解析器来完成此任务。编写一个命令行程序，在 stdin 上输入上述形式的 JSON 结构，在 stdout 上输出名称。你应该只需要几行代码就能做到。在 Python 中，除了 `import json` 之外，你可以轻松地在一行代码中完成。