---
layout: lecture
title: "代理编码"
description: >
  学习如何有效地使用 AI 编码代理进行软件开发任务。
thumbnail: /static/assets/thumbnails/2026/lec7.png
date: 2026-01-21
ready: true
video:
  aspect: 56.25
  id: sTdz6PZoAnw
---

编码代理是具有访问工具能力的对话式 AI 模型，这些工具包括读/写文件、网络搜索和调用 shell 命令。它们存在于 IDE 中或独立的命令行或 GUI 工具中。编码代理是高度自主和强大的工具，支持各种用例。

本讲座建立在[开发环境与工具]({{ site.baseurl }}/2026/development-environment/)讲座中 AI 驱动的开发材料基础上。作为快速演示，让我们继续[AI 驱动的开发]({{ site.baseurl }}/2026/development-environment/#ai-驱动的开发)部分的示例：

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')

def extract(content: str) -> list[str]:
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)

print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

我们可以尝试用以下任务提示编码代理：

```
Turn this into a proper command-line program, with argparse for argument parsing. Add type annotations, and make sure the program passes type checking.
```

代理将读取文件以理解它，然后进行一些编辑，最后调用类型检查器以确保类型注解正确。如果它犯了一个导致类型检查失败的错误，它可能会迭代，尽管这是一个简单的任务，所以这不太可能发生。因为编码代理可以访问可能有危害的工具，默认情况下，代理框架会提示用户确认工具调用。

> 如果编码代理犯了错误——例如，如果你直接在 `$PATH` 上有 `mypy` 二进制文件，但代理尝试调用 `python -m mypy`——你可以给它文本反馈来帮助它纠正方向。

编码代理支持多轮交互，因此你可以通过与代理的来回对话迭代工作。如果代理走错了方向，你甚至可以打断它。一个有用的心智模型可能是实习生的经理：实习生会做具体的工作，但需要指导，偶尔会做错事并需要被纠正。

> 对于更说明性的演示，尝试作为后续要求代理运行结果脚本。观察输出，并尝试要求它进行更改（例如，要求它只包含绝对 URL）。

# AI 模型和代理如何工作

完全解释现代[大型语言模型 (LLMs)](https://en.wikipedia.org/wiki/Large_language_model)和代理框架等基础设施的内部工作原理超出了本课程的范围。然而，对一些关键思想有高层次的理解有助于有效地_使用_这一前沿技术并理解其局限性。

LLMs 可以被视为对给定提示字符串（输入）的完成字符串（输出）的概率分布进行建模。LLM 推理（当你例如向对话式聊天应用提供查询时发生的事情）从这个概率分布中_采样_。LLM 有一个固定的_上下文窗口_，即输入和输出字符串的最大长度。

{% comment %}
> In mathematical notation, the LLM models the probability distribution $\pi_\theta$ of completions $y$ conditioned on prompts $x$, and we sample from this distribution: $\hat{y} \sim \pi_\theta(\cdot \mid x)$.
{% endcomment %}

AI 工具如对话式聊天和编码代理建立在这个原语之上。对于多轮交互，聊天应用和代理使用轮次标记，每次有新的用户提示时提供整个对话历史作为提示字符串，每个用户提示调用一次 LLM 推理。对于工具调用代理，框架将某些 LLM 输出解释为调用工具的请求，框架将工具调用的结果作为提示字符串的一部分反馈给模型（因此每次有工具调用/响应时 LLM 推理都会再次运行）。工具调用代理的核心概念可以[在 200 行代码中实现](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)。

## 隐私

大多数 AI 编码工具在其标准配置中会向云端发送大量数据。有时框架在本地运行，而 LLM 推理在云端运行，其他时候更多的软件在云端运行（例如，服务提供商可能有效地获得你整个仓库的副本以及你与 AI 工具的所有交互）。

有开源 AI 编码工具和开源 LLM，它们相当不错（虽然不如专有模型），但目前，对于大多数用户来说，由于硬件限制，在本地运行最前沿的开源 LLM 将是不可行的。

# 用例

编码代理可以帮助完成各种任务。一些例子：

- **实现新功能。** 如上例所示，你可以要求编码代理实现一个功能。给出好的规范目前更像是一门艺术而不是科学；你希望给代理的输入足够描述性，以便代理做你想让它做的事（至少朝着正确的方向，以便你可以迭代），但又不要过于描述性以至于你自己做了太多工作。测试驱动开发可能特别有效：编写测试（或使用编码代理帮助你编写测试），审计它们以确保它们捕获你想要的内容，然后要求编码代理实现功能。模型在不断改进，所以你必须保持对模型能力的直觉更新。
    > 我们使用 Claude Code [实现](https://github.com/missing-semester/missing-semester/pull/345)了这些 Tufte 风格的边注。
{%- comment %}
No need to demo this, since the intro of a lecture was a small demo of adding a new feature.
{% endcomment %}
- **修复错误。** 如果你的编译器、代码检查器、类型检查器或测试有错误，你可以要求你的代理纠正它们，例如使用像 "fix the issues with mypy" 这样的提示。当你能让编码模型进入反馈循环时特别有效，所以尝试设置让模型可以直接运行失败的检查，这将让它自主迭代。如果这不切实际，你可以手动给模型反馈。
    > 在 missing-semester 仓库的提交 [f552b55](https://github.com/missing-semester/missing-semester/commit/f552b5523462b22b8893a8404d2110c4e59613dd)，我们提示 Claude Code "Review the agentic coding lecture for typos and grammatical issues"，然后要求它修复它发现的问题，这些问题在 [f1e1c41](https://github.com/missing-semester/missing-semester/commit/f1e1c417adba6b4149f7eef91ff5624de40dc637) 中提交。
{%- comment %}
Demo a coding agent fixing the bug in https://github.com/anishathalye/dotbot/commit/cef40c902ef0f52f484153413142b5154bbc5e99.

Write the failing tests to demo the bug, and then ask the agent to fix. Prepped in branch demo-bugfix.

Can run the failing test with:

    hatch test tests/test_cli.py::test_issue_357

Can prompt coding agent with:

    There is a bug I wrote a failing test for, you can repro it with `hatch test tests/test_cli.py::test_issue_357`. Fix the bug.

Get it to commit the changes.
{% endcomment %}
- **重构。** 你可以使用编码代理以各种方式重构代码，从简单的任务如重命名方法（这种重构也由[代码智能]({{ site.baseurl }}/2026/development-environment/#代码智能与语言服务器)支持）到更复杂的任务如将功能拆分到单独的模块。
    > 我们使用 Claude Code [拆分](https://github.com/missing-semester/missing-semester/pull/344)代理编码为其自己的讲座。
{%- comment %}
Show usage in Missing Semester, point out that the agent did make some mistakes.
{% endcomment %}
- **代码审查。** 你可以要求编码代理审查代码。你可以给它们基本的指导，比如 "review my latest changes that are not yet committed"。如果你想审查拉取请求，你的编码代理支持网络获取，或者你安装了像 [GitHub CLI](https://cli.github.com/) 这样的命令行工具，你甚至可能能够要求编码代理 "Review the pull request {link}"，它会从那里处理。
{%- comment %}
In Porcupine repo, prompt agent with:

    Review this PR: https://github.com/anishathalye/porcupine/pull/39
{% endcomment %}
- **代码理解。** 你可以向编码代理询问关于代码库的问题，这对于入门特别有帮助。
{%- comment %}
Some prompts to try in the missing-semester repo:

    How do I run this site locally?

    How are the social preview cards implemented?
{% endcomment %}
- **作为 shell。** 你可以要求编码代理使用特定工具来完成任务，这样你可以使用自然语言调用 shell 命令，例如 "use the find command to find all files older than 30 days" 或 "use mogrify to resize all the jpgs to 50% of their original size"。
{%- comment %}
In Dotbot repo, prompt agent with:

    Use the ag command to find all Python renaming imports
{% endcomment %}
- **Vibe coding。** 代理足够强大，你可以在不自己编写一行代码的情况下实现一些应用程序。
    > [这是一个例子](https://github.com/cleanlab/office-presence-dashboard)，其中一个讲师 vibe-coded 的真实项目。
{%- comment %}
In missing-semester repo, prompt agent with:

    Make this site look retro.
{% endcomment %}

# 高级代理

在这里，我们简要概述编码代理的一些更高级的使用模式和功能。

- **可重用提示。** 创建可重用提示或模板。例如，你可以编写一个详细的提示来以特定方式进行代码审查，并将其保存为可重用提示。
    > 代理工具发展很快。在某些工具中，可重用提示作为独立功能已被弃用。例如，在 Codex 和 Claude Code 中，它们被[技能](https://code.claude.com/docs/en/skills)[取代](https://developers.openai.com/codex/custom-prompts)。
- **并行代理。** 编码代理可能很慢：你可以提示代理，它可能在一个问题上工作几十分钟。你可以同时运行多个代理副本，要么处理同一任务（LLM 是随机的，所以多次运行相同的事情并采用最佳解决方案可能有帮助），要么处理不同任务（例如，同时实现两个不重叠的功能）。为了防止不同代理的更改相互干扰，你可以使用 [git worktrees](https://git-scm.com/docs/git-worktree)，我们在[版本控制]({{ site.baseurl }}/2026/version-control/)讲座中介绍过。
- **MCPs。** MCP，代表_模型上下文协议_，是一个开放协议，你可以用它将编码代理与工具连接。例如，这个 [Notion MCP 服务器](https://github.com/makenotion/notion-mcp-server)可以让你的代理读/写 Notion 文档，实现用例如"阅读 {Notion 文档} 中链接的规范，在 Notion 中起草实现计划作为新页面，然后实现原型"。要发现 MCP，你可以使用像 [Pulse](https://www.pulsemcp.com/servers) 和 [Glama](https://glama.ai/mcp/servers) 这样的目录。
- **上下文管理。** 正如我们[上面](#how-ai-models-and-agents-work)指出的，编码代理底层的 LLM 有一个有限的_上下文窗口_。有效使用编码代理需要善用上下文。你想确保代理有它需要的信息，但避免不必要的上下文以避免溢出上下文窗口或降低模型性能（随着上下文大小增长往往会发生这种情况，即使它没有溢出上下文窗口）。代理框架自动提供上下文，并在某种程度上管理上下文，但很多控制权留给用户。
    - **清除上下文窗口。** 最基本的控制，编码代理支持清除上下文窗口（开始新对话），对于不相关的查询你应该这样做。
    - **回退对话。** 一些编码代理支持撤销对话历史中的步骤。在"撤销"更有意义的情况下，不是给出后续消息引导代理朝不同方向，这更有效地管理上下文。
{%- comment %}
Make up a quick demo.
{% endcomment %}
    - **压缩。** 为了实现无限长度的对话，编码代理支持上下文_压缩_：如果对话历史变得太长，它们会自动调用 LLM 来总结对话的前缀，并用总结替换对话历史。一些代理给用户控制权，在需要时调用压缩。
{%- comment %}
Show `/compact` in Claude Code, show full summary.
{% endcomment %}
    - **llms.txt。** `/llms.txt` 文件是一个提议的[标准](https://llmstxt.org/)位置，用于供 LLM 在推理时使用的文档。产品（例如，[cursor.com/llms.txt](https://cursor.com/llms.txt)）、软件库（例如，[ai.pydantic.dev/llms.txt](https://ai.pydantic.dev/llms.txt)）和 API（例如，[apify.com/llms.txt](https://apify.com/llms.txt)）可能有 `llms.txt` 文件，这对于开发很方便。这样的文档每个 token 信息密度更高，因此比让你的编码代理获取和阅读 HTML 页面更节省上下文。外部文档在编码代理没有关于你尝试使用的依赖项的内置知识时很有用（例如，因为它是在 LLM 知识截止日期后发布的）。
{%- comment %}
Side-by-side comparison in an empty repo (on Desktop or some other self-contained place, with `git init` run in it):

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers.

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers. See https://semlib.anish.io/llms.txt. Follow links to Markdown versions of any pages linked in llms.txt files.

Not sure why the agent doesn't do this by default. You'd probably put that last sentence in a CLAUDE.md file.
{% endcomment %}
    - **AGENTS.md。** 大多数编码代理支持 [AGENTS.md](https://agents.md/) 或类似的（例如，Claude Code 查找 `CLAUDE.md`）作为编码代理的 README。当代理启动时，它会将 `AGENTS.md` 的全部内容预填充到上下文中。你可以使用它来为代理提供跨会话通用的建议（例如，指示它在进行代码更改后始终运行类型检查器，解释如何运行单元测试，或提供代理可以浏览的第三方文档链接）。一些编码代理可以自动生成此文件（例如，Claude Code 中的 `/init` 命令）。参见[这里](https://github.com/pydantic/pydantic-ai/blob/main/CLAUDE.md)获取 `AGENTS.md` 的真实示例。
{%- comment %}
Dotbot example, CLAUDE.md that includes @DEVELOPMENT.md and says to always run the type checker and code formatter after making any changes to Python code.

Example prompt, off of master:

    Remove the "--version" command-line flag.

This is something that'll be fast, for demonstration purposes.
{% endcomment %}
    - **技能。** `AGENTS.md` 中的内容始终被完整加载到代理的上下文窗口中。_技能_增加了一层间接性以避免上下文膨胀：你可以为代理提供技能列表以及描述，代理可以根据需要"打开"技能（将其加载到上下文窗口中）。
    - **子代理。** 一些编码代理让你定义子代理，这些是用于特定任务工作流的代理。顶级编码代理可以调用子代理来完成特定任务，这使得顶级代理和子代理都能更有效地管理上下文。顶级代理的上下文不会被子代理看到的所有内容膨胀，子代理可以只获得其任务所需的上下文。作为一个例子，一些编码代理将网络研究实现为子代理：顶级代理将向子代理提出查询，子代理将运行网络搜索，获取单个网页，分析它们，并向顶级代理提供查询的答案。这样，顶级代理的上下文不会被所有检索到的网页的完整内容膨胀，子代理的上下文中没有顶级代理对话历史的其余部分。

对于许多需要编写提示的高级功能（例如，技能或子代理），你可以使用 LLM 来开始。一些编码代理甚至有内置支持来做到这一点。例如，Claude Code 可以从简短提示生成子代理（调用 `/agents` 并创建新代理）。尝试用此提示创建子代理：

```
A Python code checking agent that uses `mypy` and `ruff` to type-check, lint, and format *check* any files that have been modified from the last git commit.
```

然后，你可以使用顶级代理通过像 "use the code checker subagent" 这样的消息显式调用子代理。你可能还能让顶级代理在适当的时候自动调用子代理，例如，在修改任何 Python 文件后。

# 需要注意什么

AI 工具可能会犯错。它们建立在 LLM 之上，而 LLM 只是概率性的下一个 token 预测模型。它们不像人类那样"智能"。审查 AI 输出的正确性和安全漏洞。有时验证代码可能比你自己编写代码更难；对于关键代码，考虑手动编写。AI 可能会陷入死胡同并试图对你进行煤气灯效应；注意调试螺旋。不要把 AI 当作拐杖，要警惕过度依赖或理解肤浅。仍然有一大类编程任务是 AI 仍然无法完成的。计算思维仍然有价值。

# 推荐软件

许多 IDE / AI 编码扩展包括编码代理（参见[开发环境讲座]({{ site.baseurl }}/2026/development-environment/)中的推荐）。其他流行的编码代理包括 Anthropic 的 [Claude Code](https://www.claude.com/product/claude-code)、OpenAI 的 [Codex](https://openai.com/codex/)，以及开源代理如 [opencode](https://github.com/anomalyco/opencode)。

# 练习

1. 通过完成相同的编程任务四次，比较手动编码、使用 AI 自动完成、内联聊天和代理的体验。最佳候选是你已经进行的项目中的中小型功能。如果你在寻找其他想法，可以考虑在 GitHub 上的开源项目中完成"good first issue"风格的任务，或 [Advent of Code](https://adventofcode.com/) 或 [LeetCode](https://leetcode.com/) 问题。

1. 使用 AI 编码代理导航不熟悉的代码库。这最好在你实际关心的项目中调试或添加新功能的背景下进行。如果你没有想到的，尝试使用 AI 代理理解 [opencode](https://github.com/anomalyco/opencode) 代理中与安全相关的功能是如何工作的。

1. Vibe code 一个从头开始的小应用。不要手动编写一行代码。

1. 对于你选择的编码代理，创建并测试 `AGENTS.md`（或你选择的代理的类似文件，如 `CLAUDE.md`）、一个技能（例如，[Claude Code 中的技能](https://code.claude.com/docs/en/skills)或 [Codex 中的技能](https://developers.openai.com/codex/skills/)），以及一个子代理（例如，[Claude Code 中的子代理](https://code.claude.com/docs/en/sub-agents)）。思考何时想使用其中一个而不是另一个。注意，你选择的编码代理可能不支持某些功能；你可以跳过它们，或尝试支持的其他编码代理。

1. 使用编码代理完成与[代码质量讲座]({{ site.baseurl }}/2026/code-quality/)中的 Markdown 项目符号正则表达式练习相同的目标。它是否通过直接文件编辑完成任务？代理直接编辑文件来完成此类任务的缺点和局限性是什么？弄清楚如何提示代理使其不通过直接文件编辑完成任务。提示：要求代理使用[第一讲]({{ site.baseurl }}/2026/course-shell/)中提到的命令行工具之一。

1. 大多数编码代理支持一种"yolo mode"形式（例如，在 Claude Code 中，`--dangerously-skip-permissions`）。直接使用此模式是不安全的，但在虚拟机或容器等隔离环境中运行编码代理然后启用自主操作可能是可接受的。在你的机器上运行此设置。[Claude Code devcontainers](https://code.claude.com/docs/en/devcontainer) 或 [Docker Sandboxes / Claude Code](https://docs.docker.com/ai/sandboxes/agents/claude-code/) 等文档可能会有帮助。有不止一种方法可以设置这个。