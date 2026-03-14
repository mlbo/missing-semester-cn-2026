---
layout: lecture
title: "打包与发布代码"
description: >
  学习项目打包、环境、版本控制，以及部署库、应用程序和服务。
thumbnail: /static/assets/thumbnails/2026/lec6.png
date: 2026-01-20
ready: true
video:
  aspect: 56.25
  id: KBMiB-8P4Ns
---

让代码按预期工作很难；让同样的代码在与你自己的机器不同的机器上运行往往更难。

发布代码意味着将你编写的代码转换为可用形式，其他人可以在没有你计算机确切设置的情况下运行。发布代码有多种形式，取决于编程语言、系统库和操作系统等众多因素的选择。它还取决于你正在构建什么：软件库、命令行工具和 Web 服务都有不同的要求和部署步骤。无论如何，所有这些场景之间有一个共同模式：我们需要定义交付物是什么——也称为工件——以及它对周围环境做什么假设。

在本讲座中，我们将涵盖：

- [依赖与环境](#依赖与环境)
- [工件与打包](#工件与打包)
- [发布与版本控制](#发布与版本控制)
- [可重现性](#可重现性)
- [虚拟机与容器](#虚拟机与容器)
- [配置](#配置)
- [服务与编排](#服务与编排)
- [发布](#发布)

我们将通过 Python 生态系统中的示例来解释这些概念，因为具体的示例有助于理解。虽然其他编程语言生态系统的工具不同，但概念基本相同。

# 依赖与环境

在现代软件开发中，抽象层无处不在。程序自然地将逻辑卸载给其他库或服务。但是，这引入了你的程序与其运行所需的库之间的依赖关系。例如，在 Python 中，要获取网站内容，我们经常这样做：

```python
import requests

response = requests.get("https://missing.csail.mit.edu")
```

然而 `requests` 库不是 Python 运行时附带的，所以如果我们尝试在没有安装 `requests` 的情况下运行这段代码，Python 会抛出错误：

```console
$ python fetch.py
Traceback (most recent call last):
  File "fetch.py", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
```

要使这个库可用，我们需要先运行 `pip install requests` 来安装它。`pip` 是 Python 编程语言提供的用于安装包的命令行工具。执行 `pip install requests` 会产生以下操作序列：

1. 在 Python 包索引（[PyPI](https://pypi.org/)）中搜索 requests
1. 为我们运行的平台搜索合适的工件
1. 解析依赖——`requests` 库本身依赖其他包，所以安装程序必须找到所有传递依赖的兼容版本并事先安装它们
1. 下载工件，然后解包并将文件复制到我们文件系统中的正确位置

```console
$ pip install requests
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl (64 kB)
Collecting charset-normalizer<4,>=2
  Downloading charset_normalizer-3.4.0-cp311-cp311-manylinux_x86_64.whl (142 kB)
Collecting idna<4,>=2.5
  Downloading idna-3.10-py3-none-any.whl (70 kB)
Collecting urllib3<3,>=1.21.1
  Downloading urllib3-2.2.3-py3-none-any.whl (126 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2024.8.30-py3-none-any.whl (167 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2024.8.30 charset-normalizer-3.4.0 idna-3.10 requests-2.32.3 urllib3-2.2.3
```

这里我们可以看到 `requests` 有自己的依赖，如 `certifi` 或 `charset-normalizer`，它们必须在 `requests` 安装之前安装。一旦安装，Python 运行时就可以在导入时找到这个库。

```console
$ python -c 'import requests; print(requests.__path__)'
['/usr/local/lib/python3.11/dist-packages/requests']

$ pip list | grep requests
requests        2.32.3
```

编程语言有不同的工具、约定和实践来安装和发布库。在一些语言如 Rust 中，工具链是统一的——`cargo` 处理构建、测试、依赖管理和发布。在其他语言如 Python 中，统一发生在规范级别——不是单一工具，而是有标准规范定义打包如何工作，允许每个任务有多种竞争工具（`pip` vs [`uv`](https://docs.astral.sh/uv/)，`setuptools` vs [`hatch`](https://hatch.pypa.io/) vs [`poetry`](https://python-poetry.org/)）。在一些生态系统如 LaTeX 中，TeX Live 或 MacTeX 等发行版附带数千个预安装的包。

引入依赖也会引入依赖冲突。当程序需要同一依赖的不兼容版本时，就会发生冲突。例如，如果 `tensorflow==2.3.0` 需要 `numpy>=1.16.0,<1.19.0`，`pandas==1.2.0` 需要 `numpy>=1.16.5`，那么任何满足 `numpy>=1.16.5,<1.19.0` 的版本都有效。但如果你的项目中另一个包需要 `numpy>=1.19`，你就有了没有有效版本满足所有约束的冲突。

这种情况——多个包需要相互不兼容的共享依赖版本——通常被称为依赖地狱。处理冲突的一种方法是将每个程序的依赖隔离到它们自己的环境中。在 Python 中，我们通过运行以下命令创建虚拟环境：

```console
$ which python
/usr/bin/python
$ pwd
/home/missingsemester
$ python -m venv venv
$ source venv/bin/activate
$ which python
/home/missingsemester/venv/bin/python
$ which pip
/home/missingsemester/venv/bin/pip
$ python -c 'import requests; print(requests.__path__)'
['/home/missingsemester/venv/lib/python3.11/site-packages/requests']

$ pip list
Package Version
------- -------
pip     24.0
```

你可以将环境视为一个完整的独立版本的语言运行时，带有自己的一组已安装包。这个虚拟环境或 venv 将已安装的依赖与全局 Python 安装隔离。为每个项目设置一个虚拟环境是一个好习惯，包含它所需的依赖。

> 虽然许多现代操作系统附带 Python 等编程语言运行时的安装，但修改这些安装是不明智的，因为操作系统可能依赖它们来运行自己的功能。最好使用独立的环境。

在一些语言中，安装协议不是由工具定义的，而是作为规范定义的。在 Python 中，[PEP 517](https://peps.python.org/pep-0517/) 定义了构建系统接口，[PEP 621](https://peps.python.org/pep-0621/) 指定项目元数据如何存储在 `pyproject.toml` 中。这使开发者能够改进 `pip` 并生成更优化的工具如 `uv`。要安装 `uv`，只需执行 `pip install uv`。

使用 `uv` 代替 `pip` 遵循相同的接口，但速度明显更快：

```console
$ uv pip install requests
Resolved 5 packages in 12ms
Prepared 5 packages in 0.45ms
Installed 5 packages in 8ms
 + certifi==2024.8.30
 + charset-normalizer==3.4.0
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

> 我们强烈建议尽可能使用 `uv pip` 代替 `pip`，因为它显著减少了安装时间。

除了依赖隔离，环境还允许你有不同版本的编程语言运行时。

```console
$ uv venv --python 3.12 venv312
Using CPython 3.12.7
Creating virtual environment at: venv312

$ source venv312/bin/activate && python --version
Python 3.12.7

$ uv venv --python 3.11 venv311
Using CPython 3.11.10
Creating virtual environment at: venv311

$ source venv311/bin/activate && python --version
Python 3.11.10
```

这在你需要跨多个 Python 版本测试代码或项目需要特定版本时很有帮助。

> 在一些编程语言中，每个项目自动获得自己的依赖环境，而不是你手动创建，但原则相同。如今大多数语言也有机制在单个系统上管理语言的多个版本，然后为各个项目指定使用哪个版本。

# 工件与打包

在软件开发中，我们区分源代码和工件。开发者编写和阅读源代码，而工件是从该源代码生成的打包的、可分发的输出——准备安装或部署。

工件可以简单到我们运行的一个代码文件，也可以复杂到包含应用程序所有必要组件的整个虚拟机。考虑这个例子，我们在当前目录有一个 Python 文件 `greet.py`：

```console
$ cat greet.py
def greet(name):
    return f"Hello, {name}!"

$ python -c "from greet import greet; print(greet('World'))"
Hello, World!

$ cd /tmp
$ python -c "from greet import greet; print(greet('World'))"
ModuleNotFoundError: No module named 'greet'
```

一旦我们移动到不同的目录，导入就会失败，因为 Python 只在特定位置搜索模块（当前目录、已安装的包和 `PYTHONPATH` 中的路径）。打包通过将代码安装到已知位置来解决这个问题。

在 Python 中，打包库涉及生成 `pip` 或 `uv` 等包安装器可以用来安装相关文件的工件。Python 工件称为 wheel，包含安装包所需的所有必要信息：代码文件、关于包的元数据（名称、版本、依赖）以及文件在环境中放置位置的说明。构建工件需要我们编写项目文件（也常称为清单），详细说明项目的细节、所需依赖、包的版本等信息。在 Python 中，我们为此目的使用 `pyproject.toml`。

> `pyproject.toml` 是现代推荐的方式。虽然早期打包方法如 `requirements.txt` 或 `setup.py` 仍然受支持，但你应该尽可能优先使用 `pyproject.toml`。

这是一个也提供命令行工具的库的最小 `pyproject.toml`：

```toml
[project]
name = "greeting"
version = "0.1.0"
description = "A simple greeting library"
dependencies = ["typer>=0.9"]

[project.scripts]
greet = "greeting:main"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

`typer` 库是一个流行的 Python 包，用于以最少的样板代码创建命令行接口。

以及相应的 `greeting.py`：

```python
import typer


def greet(name: str) -> str:
    return f"Hello, {name}!"


def main(name: str):
    print(greet(name))


if __name__ == "__main__":
    typer.run(main)
```

有了这个文件，我们现在可以构建 wheel：

```console
$ uv build
Building source distribution...
Building wheel from source distribution...
Successfully built dist/greeting-0.1.0.tar.gz
Successfully built dist/greeting-0.1.0-py3-none-any.whl

$ ls dist/
greeting-0.1.0-py3-none-any.whl
greeting-0.1.0.tar.gz
```

`.whl` 文件是 wheel（一个具有特定结构的 zip 归档），`.tar.gz` 是需要在需要从源代码构建的系统上使用的源代码分发包。

你可以检查 wheel 的内容来查看打包了什么：

```console
$ unzip -l dist/greeting-0.1.0-py3-none-any.whl
Archive:  dist/greeting-0.1.0-py3-none-any.whl
  Length      Date    Time    Name
---------  ---------- -----   ----
      150  2024-01-15 10:30   greeting.py
      312  2024-01-15 10:30   greeting-0.1.0.dist-info/METADATA
       92  2024-01-15 10:30   greeting-0.1.0.dist-info/WHEEL
        9  2024-01-15 10:30   greeting-0.1.0.dist-info/top_level.txt
      435  2024-01-15 10:30   greeting-0.1.0.dist-info/RECORD
---------                     -------
      998                     5 files
```

现在如果我们把这个 wheel 给其他人，他们可以通过运行来安装：

```console
$ uv pip install ./greeting-0.1.0-py3-none-any.whl
$ greet Alice
Hello, Alice!
```

这会将我们之前构建的库安装到他们的环境中，包括 `greet` 命令行工具。

这种方法有一些限制。特别是，如果我们的库依赖特定平台的库，例如用于 GPU 加速的 CUDA，那么我们的工件只能在安装了这些特定库的系统上工作，我们可能需要为不同平台（Linux、macOS、Windows）和架构（x86、ARM）构建单独的 wheel。

安装软件时，从源代码安装和安装预构建二进制文件之间有重要区别。从源代码安装意味着下载原始代码并在你的机器上编译——这需要安装编译器和构建工具，对于大型项目可能需要大量时间。

安装预构建二进制文件意味着下载其他人已经编译好的工件——更快更简单，但二进制文件必须匹配你的平台和架构。例如，[ripgrep 的发布页面](https://github.com/BurntSushi/ripgrep/releases)显示了 Linux（x86_64、ARM）、macOS（Intel、Apple Silicon）和 Windows 的预构建二进制文件。

# 发布与版本控制

代码是在连续过程中构建的，但按离散方式发布。在软件开发中，开发环境和生产环境有明确的区分。代码需要在开发环境中证明有效，然后才能发布到生产环境。发布过程涉及许多步骤，包括测试、依赖管理、版本控制、配置、部署和发布。

软件库不是静态的，它们随时间演变，获得修复和新功能。我们通过对应于库在特定时间点状态的离散版本标识符来跟踪这种演变。库行为的更改范围从修复非关键功能的补丁，到扩展其功能的新功能，再到破坏向后兼容性的更改。变更日志记录版本引入的更改——这些是软件开发人员用来传达与新版本相关的更改的文档。

然而，跟踪每个依赖中持续发生的更改是不切实际的，更不用说传递依赖——即我们依赖的依赖。

> 你可以用 `uv tree` 可视化项目的整个依赖树，它以树形格式显示所有包及其传递依赖。

为了简化这个问题，有关于如何版本化软件的约定，其中最流行的是[语义化版本控制](https://semver.org/)或 SemVer。在语义化版本控制下，版本具有 MAJOR.MINOR.PATCH 形式的标识符，其中每个值取整数值。简短版本是升级：

- PATCH（例如 1.2.3 → 1.2.4）应该只包含 bug 修复并完全向后兼容
- MINOR（例如 1.2.3 → 1.3.0）以向后兼容的方式添加新功能
- MAJOR（例如 1.2.3 → 2.0.0）表示可能需要代码修改的破坏性更改

> 这是简化，我们鼓励阅读完整的 SemVer 规范来理解例如为什么从 0.1.3 到 0.2.0 可能导致破坏性更改或 1.0.0-rc.1 是什么意思。

Python 打包原生支持语义化版本控制，所以当我们指定依赖版本时，我们可以使用各种说明符：

在 `pyproject.toml` 中，我们有不同的方式来约束依赖的兼容版本范围：

```toml
[project]
dependencies = [
    "requests==2.32.3",  # 精确版本 - 只有这个特定版本
    "click>=8.0",        # 最低版本 - 8.0 或更新
    "numpy>=1.24,<2.0",  # 范围 - 至少 1.24 但小于 2.0
    "pandas~=2.1.0",     # 兼容版本 - >=2.1.0 且 <2.2.0
]
```

版本说明符在许多包管理器（npm、cargo 等）中存在，具有不同的确切语义。`~=` 操作符是 Python 的"兼容版本"操作符——`~=2.1.0` 意味着"任何与 2.1.0 兼容的版本"，这转化为 `>=2.1.0` 且 `<2.2.0`。这大致相当于 npm 和 cargo 中的插入符号（`^`）操作符，它遵循 SemVer 的兼容性概念。

并非所有软件都使用语义化版本控制。一个常见的替代是基于日历的版本控制（CalVer），其中版本基于发布日期而不是语义含义。例如，Ubuntu 使用类似 `24.04`（2024 年 4 月）和 `24.10`（2024 年 10 月）的版本。CalVer 使查看版本有多旧变得容易，尽管它不传达关于兼容性的任何信息。最后，语义化版本控制并非万无一失，有时维护者会在次要或补丁版本中无意中引入破坏性更改。

# 可重现性

在现代软件开发中，你编写的代码位于大量抽象层之上。这包括你的编程语言运行时、第三方库、操作系统，甚至硬件本身。这些层中的任何差异都可能改变代码的行为或甚至阻止它按预期工作。此外，甚至底层硬件的差异也会影响你发布软件的能力。

固定库是指指定精确版本而不是范围，例如 `requests==2.32.3` 而不是 `requests>=2.0`。

包管理器工作的一部分是考虑依赖——和传递依赖——提供的所有约束，然后生成满足所有约束的有效版本列表。特定版本列表可以保存到文件中以实现可重现性；这些文件称为锁文件。

```console
$ uv lock
Resolved 12 packages in 45ms

$ cat uv.lock | head -20
version = 1
requires-python = ">=3.11"

[[package]]
name = "certifi"
version = "2024.8.30"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/...", hash = "sha256:..." }
wheels = [
    { url = "https://files.pythonhosted.org/...", hash = "sha256:..." },
]
...
```

在处理依赖版本控制和可重现性时，一个关键区别是库和应用程序/服务之间的区别。库旨在被其他代码导入和使用，其他代码可能有自己的依赖，所以指定过于严格的版本约束可能会与用户的其他依赖发生冲突。相比之下，应用程序或服务是软件的最终消费者，通常通过用户界面或 API 公开其功能，而不是通过编程接口。对于库，指定版本范围以最大化与更广泛包生态系统的兼容性是好做法。对于应用程序，固定精确版本确保可重现性——每个运行应用程序的人都使用完全相同的依赖。

对于需要最大可重现性的项目，像 [Nix](https://nixos.org/) 和 [Bazel](https://bazel.build/) 这样的工具提供密封构建——其中每个输入包括编译器、系统库，甚至构建环境本身都被固定和内容寻址。这保证了无论何时何地运行构建，都能获得逐位相同的输出。

> 你甚至可以使用 NixOS 管理整个计算机安装，这样你可以轻松地启动计算机设置的副本，并通过版本控制的配置文件管理它们的完整配置。

软件开发中永无止境的紧张关系是新软件版本有意或无意地引入破坏，而另一方面，旧软件版本随时间推移会因安全漏洞而受损。我们可以通过使用持续集成管道（我们将在代码质量和 CI 讲座中看到更多）来解决这一问题，这些管道针对新软件版本测试我们的应用程序，并设置自动化来检测何时发布我们依赖的新版本，如 [Dependabot](https://github.com/dependabot)。

即使有 CI 测试，升级软件版本时仍会出现问题，通常是由于开发和生产环境之间不可避免的差异。在这种情况下，最好的做法是制定回滚计划，即撤销版本升级并重新部署已知良好的版本。

# 虚拟机与容器

随着你开始依赖更复杂的依赖，你的代码依赖很可能超出包管理器可以处理的范围。一个常见的原因是需要与特定的系统库或硬件驱动程序接口。例如，在科学计算和 AI 中，程序通常需要专门的库和驱动程序来利用 GPU 硬件。许多系统级依赖（GPU 驱动程序、特定的编译器版本、共享库如 OpenSSL）仍需要系统级安装。

传统上，这个更广泛的依赖问题是通过虚拟机（VM）解决的。VM 抽象整个计算机并提供一个完全隔离的环境，带有自己专用的操作系统。一种更现代的方法是容器，它打包应用程序及其依赖、库和文件系统，但共享主机的操作系统内核而不是虚拟化整个计算机。容器比 VM 更轻量级，因为它们共享内核，使它们更快启动和更高效运行。

最流行的容器平台是 [Docker](https://www.docker.com/)。Docker 引入了一种标准化的方式来构建、分发和运行容器。在底层，Docker 使用 containerd 作为其容器运行时——一个其他工具如 Kubernetes 也使用的行业标准。

运行容器很简单。例如，要在容器内运行 Python 解释器，我们使用 `docker run`（`-it` 标志使容器具有交互式终端。退出时，容器停止。）

```console
$ docker run -it python:3.12 python
Python 3.12.7 (main, Nov 5 2024, 02:53:25) [GCC 12.2.0] on linux
>>> print("Hello from inside a container!")
Hello from inside a container!
```

在实践中，你的程序可能依赖整个文件系统。为了克服这个问题，我们可以使用将应用程序的整个文件系统作为工件传输的容器镜像。容器镜像是通过编程方式创建的。使用 docker，我们使用 Dockerfile 语法指定镜像的确切依赖、系统库和配置：

```dockerfile
FROM python:3.12
RUN apt-get update
RUN apt-get install -y gcc
RUN apt-get install -y libpq-dev
RUN pip install numpy
RUN pip install pandas
COPY . /app
WORKDIR /app
RUN pip install .
```

一个重要的区别：Docker 镜像是打包的工件（像模板），而容器是镜像的运行实例。你可以从同一个镜像运行多个容器。镜像以层的形式构建，其中 Dockerfile 中的每条指令（`FROM`、`RUN`、`COPY` 等）创建一个新层。Docker 缓存这些层，所以如果你更改 Dockerfile 中的一行，只有该层和后续层需要重建。

之前的 Dockerfile 有几个问题：它使用完整的 Python 镜像而不是 slim 变体，运行单独的 `RUN` 命令创建不必要的层，版本未固定，并且不清理包管理器缓存，传输不必要的文件。其他常见错误包括以 root 身份不安全地运行容器以及意外将机密嵌入层中。

这是一个改进版本：

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev && \
    rm -rf /var/lib/apt/lists/*
COPY pyproject.toml uv.lock ./
RUN uv pip install --system -r uv.lock
COPY . /app
```

在前面的例子中，我们看到不是从源代码安装 `uv`，而是从 `ghcr.io/astral-sh/uv:latest` 镜像复制预构建的二进制文件。这称为构建器模式。使用这种模式，我们不需要传输编译代码所需的所有工具，只需要运行应用程序所需的最终二进制文件（在这个例子中是 `uv`）。

Docker 有一些重要的限制需要注意。首先，容器镜像通常是特定平台的——为 `linux/amd64` 构建的镜像在没有模拟的情况下不会在 `linux/arm64`（Apple Silicon Mac）上原生运行，这很慢。其次，Docker 容器需要 Linux 内核，所以在 macOS 和 Windows 上，Docker 实际上在底层运行一个轻量级 Linux VM，增加了开销。第三，Docker 的隔离比 VM 弱——容器共享主机内核，这在多租户环境中是一个安全隐患。

> 如今，越来越多的项目也使用 nix 通过 [nix flakes](https://serokell.io/blog/practical-nix-flakes) 管理甚至每个项目的"系统级"库和应用程序。

# 配置

软件本质上是可配置的。在命令行环境讲座中，我们看到程序通过标志、环境变量甚至配置文件（也称为点文件）接收选项。这对于更复杂的应用程序也是如此，在规模化时有管理配置的既定模式。软件配置不应嵌入代码中，而应在运行时提供。几种常见的方式是环境变量和配置文件。

这是一个通过环境变量配置的应用程序示例：

```python
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///local.db")
DEBUG = os.environ.get("DEBUG", "false").lower() == "true"
API_KEY = os.environ["API_KEY"]  # 必需 - 如果未设置将抛出异常
```

应用程序也可以通过配置文件配置（例如，通过 `yaml.load` 加载配置的 Python 程序），`config.yaml`：

```yaml
database:
  url: "postgresql://localhost/myapp"
  pool_size: 5
server:
  host: "0.0.0.0"
  port: 8080
  debug: false
```

思考配置的一个好的经验法则是，同一个代码库应该可以通过仅更改配置部署到不同的环境（开发、预发布、生产），而不是更改代码。

在许多配置选项中，通常有敏感数据如 API 密钥。机密需要小心处理以避免意外暴露，并且绝不能包含在版本控制中。

# 服务与编排

现代应用程序很少孤立存在。一个典型的 Web 应用程序可能需要数据库用于持久存储、缓存用于性能、消息队列用于后台任务，以及各种其他支持服务。现代架构不是将所有东西捆绑到单个单体应用程序中，而是将功能分解为可以独立开发、部署和扩展的独立服务。

例如，如果我们确定我们的应用程序可能受益于使用缓存，我们可以利用现有的经过实战检验的解决方案如 [Redis](https://redis.io/) 或 [Memcached](https://memcached.org/)，而不是自己实现。我们可以通过将其构建为容器的一部分将 Redis 嵌入我们的应用程序依赖中，但这意味着协调 Redis 和我们的应用程序之间的所有依赖，这可能具有挑战性甚至不可行。相反，我们可以做的是将每个应用程序部署在其自己的容器中。这通常称为微服务架构，其中每个组件作为通过网路通信的独立服务运行，通常通过 HTTP API。

[Docker Compose](https://docs.docker.com/compose/) 是一个用于定义和运行多容器应用程序的工具。你可以在单个 YAML 文件中声明所有服务并将它们编排在一起，而不是单独管理容器。现在我们的完整应用程序包含多个容器：

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - REDIS_URL=redis://cache:6379
    depends_on:
      - cache

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

使用 `docker compose up`，两个服务一起启动，Web 应用程序可以使用主机名 `cache` 连接到 Redis（Docker 的内部 DNS 自动解析服务名称）。Docker Compose 让我们声明我们想如何部署一个或多个服务，并处理将它们一起启动、设置它们之间的网络以及管理数据持久化的共享卷的编排。

对于生产部署，你通常希望你的 docker compose 服务在启动时自动启动并在失败时重启。一个常见的方法是使用 systemd 管理 docker compose 部署：

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target
```

这个 systemd 单元文件确保你的应用程序在系统启动时启动（在 Docker 准备好之后），并提供标准控制如 `systemctl start myapp`、`systemctl stop myapp` 和 `systemctl status myapp`。

随着部署要求变得更复杂——需要跨多台机器的可扩展性、服务崩溃时的容错和高可用性保证——组织转向复杂的容器编排平台如 Kubernetes（k8s），它可以跨机器集群管理数千个容器。也就是说，Kubernetes 学习曲线陡峭，运营开销大，所以对于较小的项目通常过度。

这种多容器设置部分可行是因为现代服务通过标准化 API（HTTP REST API）相互通信。例如，每当程序与 OpenAI 或 Anthropic 等 LLM 提供商交互时，底层它正在向它们的服务器发送 HTTP 请求并解析响应：

```console
$ curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2023-06-01" \
    -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 256,
         "messages": [{"role": "user", "content": "Explain containers vs VMs in one sentence."}]}'
```

# 发布

一旦你展示了你的代码有效，你可能会有兴趣分发它供他人下载和安装。分发有多种形式，并且与你操作的编程语言和环境内在相关。

最简单的分发形式是上传工件供人们本地下载和安装。这仍然很常见，你可以在 [Ubuntu 的包存档](http://archive.ubuntu.com/ubuntu/pool/main/)等地方找到它，这本质上是一个 `.deb` 文件的 HTTP 目录列表。

如今，GitHub 已成为发布源代码和工件的事实平台。虽然源代码通常公开可用，但 GitHub Releases 允许维护者将预构建的二进制文件和其他工件附加到标记版本。

包管理器有时支持直接从 GitHub 安装，无论是从源代码还是从预构建的 wheel：

```console
# 从源代码安装（将克隆并构建）
$ pip install git+https://github.com/psf/requests.git

# 从特定标签/分支安装
$ pip install git+https://github.com/psf/requests.git@v2.32.3

# 直接从 GitHub release 安装 wheel
$ pip install https://github.com/user/repo/releases/download/v1.0/package-1.0-py3-none-any.whl
```

事实上，一些语言如 Go 使用去中心化的分发模型——Go 模块直接从其源代码仓库分发，而不是中央包仓库。像 `github.com/gorilla/mux` 这样的模块路径指示代码在哪里，`go get` 直接从那里获取。但是，大多数包管理器如 `pip`、`cargo` 或 `brew` 都有预打包项目的中央索引，以便于分发和安装。如果我们运行：

```console
$ uv pip install requests --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: requests==2.32.5 [compatible] (requests-2.32.5-py3-none-any.whl)
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl.metadata
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl
```

我们看到我们从哪里获取 `requests` wheel。注意文件名中的 `py3-none-any`——这意味着 wheel 适用于任何 Python 3 版本、任何操作系统、任何架构。对于有编译代码的包，wheel 是特定平台的：

```console
$ uv pip install numpy --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: numpy==2.2.1 [compatible] (numpy-2.2.1-cp312-cp312-macosx_14_0_arm64.whl)
```

这里 `cp312-cp312-macosx_14_0_arm64` 表示此 wheel 专门用于 Apple Silicon（ARM64）上 macOS 14+ 的 CPython 3.12。如果你在不同的平台上，`pip` 将下载不同的 wheel 或从源代码构建。

相反，为了让人们能够找到我们创建的包，我们需要将其发布到这些注册表之一。在 Python 中，主要注册表是 [Python 包索引（PyPI）](https://pypi.org)。与安装一样，有多种发布包的方式。`uv publish` 命令提供了一个用于将包上传到 PyPI 的现代接口：

```console
$ uv publish --publish-url https://test.pypi.org/legacy/
Publishing greeting-0.1.0.tar.gz
Publishing greeting-0.1.0-py3-none-any.whl
```

这里我们使用 [TestPyPI](https://test.pypi.org)——一个单独的包注册表，用于测试你的发布工作流而不污染真正的 PyPI。上传后，你可以从 TestPyPI 安装：

```console
$ uv pip install --index-url https://test.pypi.org/simple/ greeting
```

发布软件时的一个关键考虑因素是信任。用户如何验证他们下载的包确实来自你而没有被篡改？包注册表使用校验和验证完整性，一些生态系统支持包签名以提供作者身份的加密证明。

不同的语言有自己的包注册表：Rust 的 [crates.io](https://crates.io)、JavaScript 的 [npm](https://www.npmjs.com)、Ruby 的 [RubyGems](https://rubygems.org) 和容器镜像的 [Docker Hub](https://hub.docker.com)。同时，对于私有或内部包，组织通常部署自己的包仓库（如私有 PyPI 服务器或私有 Docker 注册表）或使用云提供商的托管解决方案。

将 Web 服务部署到互联网涉及额外的基础设施：域名注册、DNS 配置以将你的域指向你的服务器，以及通常像 nginx 这样的反向代理来处理 HTTPS 和路由流量。对于文档或静态站点等简单用例，[GitHub Pages](https://pages.github.com/) 提供直接从仓库的免费托管。

# 练习

1. 用 `printenv` 将你的环境保存到文件，创建一个 venv，激活它，用 `printenv` 保存到另一个文件，然后 `diff before.txt after.txt`。环境中有什么变化？为什么 shell 更喜欢 venv？（提示：查看激活前后的 `$PATH`。）运行 `which deactivate` 并推断 deactivate bash 函数在做什么。

1. 使用 `pyproject.toml` 创建一个 Python 包并在虚拟环境中安装它。创建一个锁文件并检查它。

1. 安装 Docker 并使用 docker compose 在本地构建 Missing Semester 课程网站。

1. 为一个简单的 Python 应用程序编写 Dockerfile。然后编写一个 `docker-compose.yml`，让你的应用程序与 Redis 缓存一起运行。

1. 将 Python 包发布到 TestPyPI（除非值得分享，否则不要发布到真正的 PyPI！）。然后用该包构建 Docker 镜像并推送到 `ghcr.io`。

1. 使用 [GitHub Pages](https://docs.github.com/en/pages/quickstart) 制作一个网站。额外（非）奖励：用自定义域名配置它。