---
layout: lecture
title: "命令行环境"
description: >
  学习命令行程序的工作原理，包括输入/输出流、环境变量，以及通过 SSH 连接远程机器。
thumbnail: /static/assets/thumbnails/2026/lec2.png
date: 2026-01-13
ready: true
video:
  aspect: 56.25
  id: ccBGsPedE9Q
---

正如我们在上一讲中介绍的，大多数 shell 不仅仅是启动其他程序的启动器，实际上它们提供了一个完整的编程语言，包含常见的模式和抽象。但是，与大多数编程语言不同，shell 脚本的一切都围绕着运行程序并让它们简单高效地相互通信而设计。

特别是，shell 脚本与约定紧密绑定。要让命令行接口（CLI）程序在更广泛的 shell 环境中良好运行，它需要遵循一些常见的模式。我们现在将介绍理解命令行程序工作原理所需的许多概念，以及如何使用和配置它们的通用约定。

# 命令行接口

在大多数编程语言中，编写一个函数看起来像这样：

```
def add(x: int, y: int) -> int:
    return x + y
```

这里我们可以清楚地看到程序的输入和输出。相比之下，shell 脚本看起来可能很不同。

```shell
#!/usr/bin/env bash

if [[ -f $1 ]]; then
    echo "Target file already exists"
    exit 1
else
    if $DEBUG; then
        grep 'error' - | tee $1
    else
        grep 'error' - > $1
    fi
    exit 0
fi
```

要正确理解像这样的脚本，我们首先需要介绍一些当 shell 程序相互通信或与 shell 环境通信时经常出现的概念：

- 参数
- 流
- 环境变量
- 返回码
- 信号

## 参数

Shell 程序在执行时接收一个参数列表。在 shell 中参数是纯字符串，程序如何解释它们取决于程序本身。例如，当我们执行 `ls -l folder/` 时，我们正在执行程序 `/bin/ls`，参数是 `['-l', 'folder/']`。

在 shell 脚本中，我们通过特殊的 shell 语法访问这些参数。要访问第一个参数，我们访问变量 `$1`，第二个参数 `$2`，以此类推直到 `$9`。要以列表形式访问所有参数，我们使用 `$@`，要获取参数数量使用 `$#`。此外，我们也可以用 `$0` 访问程序的名称。

对于大多数程序，参数将由标志和常规字符串的混合组成。标志可以通过前面的单破折号（`-`）或双破折号（`--`）来识别。标志通常是可选的，它们的作用是修改程序的行为。例如，`ls -l` 改变了 `ls` 格式化其输出的方式。

你会看到带有长名称的双破折号标志，如 `--all`，以及单破折号标志，如 `-a`，后面通常跟一个字母。同一个选项可能以两种格式指定，`ls -a` 和 `ls --all` 是等价的。单破折号标志经常被分组，所以 `ls -l -a` 和 `ls -la` 也是等价的。标志的顺序通常也不重要，`ls -la` 和 `ls -al` 产生相同的结果。有些标志相当普遍，随着你对 shell 环境越来越熟悉，你会直观地使用它们，例如（`--help`、`--verbose`、`--version`）。

> 标志是 shell 约定的一个很好的例子。shell 语言并不要求我们的程序以这种特定方式使用 `-` 或 `--`。没有什么能阻止我们编写语法为 `myprogram +myoption myfile` 的程序，但这会导致混淆，因为期望是我们使用破折号。
>
> 在实践中，大多数编程语言提供 CLI 标志解析库（例如 Python 的 `argparse` 来解析带破折号语法的参数）。

CLI 程序中的另一个常见约定是程序接受可变数量的相同类型参数。当以这种方式给定参数时，命令对每个参数执行相同的操作。

```shell
mkdir src
mkdir docs
# 等同于
mkdir src docs
```

这种语法糖起初可能看起来不必要，但当与通配符结合时变得非常强大。

通配符是 shell 在调用程序之前会展开的特殊模式。

假设我们想非递归地删除当前文件夹中的所有 .py 文件。根据我们在上一讲学到的，我们可以通过运行以下命令来实现：

```shell
for file in $(ls | grep -P '\.py$'); do
    rm "$file"
done
```

但我们可以只用 `rm *.py` 来替换它！

当我们在终端中输入 `rm *.py` 时，shell 不会用参数 `['*.py']` 调用 `/bin/rm` 程序。相反，shell 会在当前文件夹中搜索匹配模式 `*.py` 的文件，其中 `*` 可以匹配零个或多个任意类型的字符。所以如果我们的文件夹有 `main.py` 和 `utils.py`，那么 `rm` 程序将接收参数 `['main.py', 'utils.py']`。

你会发现最常见的通配符是通配符 `*`（零个或多个任何字符）、`?`（恰好一个任何字符）和花括号。花括号 `{}` 将逗号分隔的模式列表展开为多个参数。

在实践中，通配符最好通过示例来理解。

```shell
touch folder/{a,b,c}.py
# 将展开为
touch folder/a.py folder/b.py folder/c.py

convert image.{png,jpg}
# 将展开为
convert image.png image.jpg

cp /path/to/project/{setup,build,deploy}.sh /newpath
# 将展开为
cp /path/to/project/setup.sh /path/to/project/build.sh /path/to/project/deploy.sh /newpath

# 通配符技术也可以组合
mv *{.py,.sh} folder
# 将移动所有 *.py 和 *.sh 文件
```

> 一些 shell（如 zsh）支持更高级的通配符形式，如 `**`，它将展开以包括递归路径。所以 `rm **/*.py` 将递归删除所有 .py 文件。

## 流

每当我们执行一个程序管道，如：

```shell
cat myfile | grep -P '\d+' | uniq -c
```

我们看到 `grep` 程序正在与 `cat` 和 `uniq` 程序通信。

这里一个重要的观察是，所有三个程序都在同时执行。也就是说，shell 不是先调用 cat，然后 grep，然后 uniq。相反，所有三个程序同时被生成，shell 正在将 cat 的输出连接到 grep 的输入，grep 的输出连接到 uniq 的输入。当使用管道操作符 `|` 时，shell 操作从一个程序流向链中下一个程序的数据流。

我们可以演示这种并发性，管道中的所有命令立即启动：

```console
$ (sleep 15 && cat numbers.txt) | grep -P '^\d$' | sort | uniq  &
[1] 12345
$ ps | grep -P '(sleep|cat|grep|sort|uniq)'
  32930 pts/1    00:00:00 sleep
  32931 pts/1    00:00:00 grep
  32932 pts/1    00:00:00 sort
  32933 pts/1    00:00:00 uniq
  32948 pts/1    00:00:00 grep
```

我们可以看到除了 `cat` 之外的所有进程都在运行。shell 在它们任何一个完成之前就生成了所有进程并连接它们的流。`cat` 只会在 sleep 完成后启动，`cat` 的输出将被发送到 grep，以此类推。

每个程序都有一个输入流，标记为 stdin（标准输入）。当使用管道时，stdin 自动连接。在脚本中，许多程序接受 `-` 作为文件名，意思是"从 stdin 读取"：

```shell
# 当数据来自管道时，这些是等价的
echo "hello" | grep "hello"
echo "hello" | grep "hello" -
```

同样，每个程序有两个输出流：stdout 和 stderr。标准输出是最常遇到的，它用于将程序的输出管道到管道中的下一个命令。标准错误是一个替代流，旨在让程序报告警告和其他类型的问题，而该输出不会被链中的下一个命令解析。

```console
$ ls /nonexistent
ls: cannot access '/nonexistent': No such file or directory
$ ls /nonexistent | grep "pattern"
ls: cannot access '/nonexistent': No such file or directory
# 错误消息仍然出现，因为 stderr 没有被管道传输
$ ls /nonexistent 2>/dev/null
# 没有输出 - stderr 被重定向到 /dev/null
```

Shell 提供了重定向这些流的语法。以下是一些说明性示例：

```shell
# 将 stdout 重定向到文件（覆盖）
echo "hello" > output.txt

# 将 stdout 重定向到文件（追加）
echo "world" >> output.txt

# 将 stderr 重定向到文件
ls foobar 2> errors.txt

# 将 stdout 和 stderr 都重定向到同一个文件
ls foobar &> all_output.txt

# 从文件重定向 stdin
grep "pattern" < input.txt

# 通过重定向到 /dev/null 丢弃输出
cmd > /dev/null 2>&1
```

另一个体现 Unix 哲学的强大工具是 [`fzf`](https://github.com/junegunn/fzf)，一个模糊查找器。它从 stdin 读取行并提供交互式界面来过滤和选择：

```console
$ ls | fzf
$ cat ~/.bash_history | fzf
```

`fzf` 可以与许多 shell 操作集成。我们将在讨论 shell 定制时看到它的更多用途。

## 环境变量

在 bash 中赋值变量我们使用语法 `foo=bar`，然后用 `$foo` 语法访问变量的值。注意 `foo = bar` 是无效语法，因为 shell 会将其解析为调用程序 `foo`，参数是 `['=', 'bar']`。在 shell 脚本中，空格字符的作用是执行参数分割。这种行为可能令人困惑且难以习惯，所以请记住这一点。

Shell 变量没有类型，它们都是字符串。注意在 shell 中编写字符串表达式时，单引号和双引号不可互换。用 `'` 分隔的字符串是字面字符串，不会展开变量、执行命令替换或处理转义序列，而 `"` 分隔的字符串会。

```shell
foo=bar
echo "$foo"
# 打印 bar
echo '$foo'
# 打印 $foo
```

要将命令的输出捕获到变量中，我们使用命令替换。当我们执行：
```shell
files=$(ls)
echo "$files" | grep README
echo "$files" | grep ".py"
```
ls 的输出（具体来说是 stdout）被放入变量 `$files`，我们稍后可以访问它。`$files` 变量的内容确实包括 ls 输出中的换行符，这就是像 `grep` 这样的程序知道如何独立处理每个项目的方式。

一个不太为人知的类似功能是进程替换，`<( CMD )` 将执行 `CMD` 并将输出放在一个临时文件中，然后用该文件名替换 `<()`。当命令期望通过文件而不是 STDIN 传递值时，这很有用。例如，`diff <(ls src) <(ls docs)` 将显示目录 `src` 和 `docs` 中文件的差异。

每当 shell 程序调用另一个程序时，它都会传递一组变量，这些变量通常被称为环境变量。在 shell 中，我们可以通过运行 `printenv` 找到当前的环境变量。要显式传递环境变量，我们可以在命令前添加变量赋值。

> 环境变量按约定以全大写形式编写（例如 `HOME`、`PATH`、`DEBUG`）。这是约定，不是技术要求，但遵循它有助于区分环境变量和通常小写的本地 shell 变量。

```shell
TZ=Asia/Tokyo date  # 打印东京的当前时间
echo $TZ  # 这将为空，因为 TZ 只为子命令设置
```

或者，我们可以使用 `export` 内置函数，它将修改我们当前的环境，因此所有子进程都将继承该变量：

```shell
export DEBUG=1
# 从此时起的所有程序在其环境中都会有 DEBUG=1
bash -c 'echo $DEBUG'
# 打印 1
```

要删除变量，使用 `unset` 内置命令，例如 `unset DEBUG`。

> 环境变量是另一个 shell 约定。它们可用于隐式而不是显式地修改许多程序的行为。例如，shell 设置 `$HOME` 环境变量为当前用户的主文件夹路径。然后程序可以访问此变量获取此信息，而不需要显式的 `--home /home/alice`。另一个常见的例子是 `$TZ`，许多程序使用它根据指定的时区格式化日期和时间。

## 返回码

正如我们之前看到的，shell 程序的主要输出通过 stdout/stderr 流和文件系统副作用传达。

默认情况下，shell 脚本将返回退出码零。约定是零表示一切顺利，而非零表示遇到了一些问题。要返回非零退出码，我们必须使用 `exit NUM` shell 内置命令。我们可以通过访问特殊变量 `$?` 来获取最后运行命令的返回码。

Shell 有布尔操作符 `&&` 和 `||` 分别执行 AND 和 OR 操作。与常规编程语言中遇到的那些不同，shell 中的这些操作符对程序的返回码进行操作。两者都是[短路求值](https://en.wikipedia.org/wiki/Short-circuit_evaluation)操作符。这意味着它们可以用于根据前一个命令的成功或失败有条件地运行命令，其中成功是基于返回码是否为零来判断的。一些示例：

```shell
# echo 只会在 grep 成功（找到匹配）时运行
grep -q "pattern" file.txt && echo "Pattern found"

# echo 只会在 grep 失败（没有匹配）时运行
grep -q "pattern" file.txt || echo "Pattern not found"

# true 是一个总是成功的 shell 程序
true && echo "This will always print"

# false 是一个总是失败的 shell 程序
false || echo "This will always print"
```

同样的原则适用于 `if` 和 `while` 语句，它们都使用返回码来做决定：

```shell
# if 使用条件命令的返回码（0 = true，非零 = false）
if grep -q "pattern" file.txt; then
    echo "Found"
fi

# while 循环只要命令返回 0 就继续
while read line; do
    echo "$line"
done < file.txt
```

## 信号

在某些情况下，你需要中断正在执行的程序，例如如果命令花费太长时间才能完成。中断程序的最简单方法是按 `Ctrl-C`，命令可能会停止。但这实际上是如何工作的，为什么有时会无法停止进程？

```console
$ sleep 100
^C
$
```

> 注意，这里 `^C` 是在终端中输入时 `Ctrl-C` 的显示方式。

在底层，发生的事情如下：

1. 我们按下了 `Ctrl-C`
2. Shell 识别了这个特殊字符组合
3. Shell 进程向 `sleep` 进程发送了 SIGINT 信号
4. 信号中断了 `sleep` 进程的执行

信号是一种特殊的通信机制。当进程收到信号时，它会停止执行，处理信号，并可能根据信号传递的信息改变执行流程。因此，信号是软件中断。

在我们的情况下，当输入 `Ctrl-C` 时，这会提示 shell 向进程传递 `SIGINT` 信号。这是一个捕获 `SIGINT` 并忽略它、不再停止的最小 Python 程序示例。要杀死这个程序，我们现在可以使用 `SIGQUIT` 信号，通过输入 `Ctrl-\`。

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

如果我们向这个程序发送两次 `SIGINT`，然后发送 `SIGQUIT` 会发生什么。注意 `^` 是在终端中输入时 `Ctrl` 的显示方式。

```console
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

虽然 `SIGINT` 和 `SIGQUIT` 通常与终端相关请求相关联，但请求进程优雅退出的更通用信号是 `SIGTERM` 信号。要发送此信号，我们可以使用 [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) 命令，语法为 `kill -TERM <PID>`。

信号除了杀死进程外还可以做其他事情。例如，`SIGSTOP` 暂停进程。在终端中，输入 `Ctrl-Z` 将提示 shell 发送 `SIGTSTP` 信号，是 Terminal Stop 的缩写（即终端版本的 `SIGSTOP`）。

然后我们可以使用 [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) 或 [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html) 分别在前景或后台继续暂停的作业。

[`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) 命令列出与当前终端会话关联的未完成作业。你可以使用它们的 pid 引用这些作业（你可以使用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 查找）。更直观地，你也可以使用百分号后跟其作业号（由 `jobs` 显示）来引用进程。要引用最后一个后台作业，你可以使用 `$!` 特殊参数。

还有一点要知道的是，命令中的 `&` 后缀将在后台运行命令，让你得到提示符，尽管它仍会使用 shell 的 STDOUT，这可能会很烦人（在这种情况下使用 shell 重定向）。等效地，要后台化一个已经在运行的程序，你可以执行 `Ctrl-Z` 后跟 `bg`。

注意后台进程仍然是你终端的子进程，如果你关闭终端它们会死亡（这将发送另一个信号 `SIGHUP`）。要防止这种情况发生，你可以用 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) 运行程序（一个忽略 `SIGHUP` 的包装器），或者如果进程已经启动则使用 `disown`。或者，你可以使用终端复用器，我们将在下一节中看到。

下面是一个示例会话，展示其中一些概念。

```
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ kill -SIGHUP %2   # nohup 保护免受 SIGHUP

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000
```

一个特殊的信号是 `SIGKILL`，因为它不能被进程捕获，它会立即终止进程。但是，它可能有不良副作用，如留下孤儿子进程。

你可以在[这里](https://en.wikipedia.org/wiki/Signal_(IPC))或输入 [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) 或 `kill -l` 了解更多关于这些和其他信号的信息。

在 shell 脚本中，你可以使用 `trap` 内置命令在收到信号时执行命令。这对于清理操作很有用：

```shell
#!/usr/bin/env bash
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT  # 在脚本退出时运行清理
trap cleanup SIGINT SIGTERM  # 也在 Ctrl-C 或 kill 时运行
```

# 远程机器

程序员在日常工作中使用远程服务器已经变得越来越普遍。这里最常见的工具是 SSH（Secure Shell），它将帮助我们连接到远程服务器并提供熟悉的 shell 接口。我们用如下命令连接到服务器：

```bash
ssh alice@server.mit.edu
```

这里我们尝试以用户 `alice` 身份 ssh 到服务器 `server.mit.edu`。

`ssh` 一个经常被忽视的功能是能够非交互式地运行命令。`ssh` 正确处理发送命令的 stdin 和接收命令的 stdout，所以我们可以将其与其他命令结合使用：

```shell
# 这里 ls 在远程运行，wc 在本地运行
ssh alice@server ls | wc -l

# 这里 ls 和 wc 都在服务器上运行
ssh alice@server 'ls | wc -l'
```

> 尝试安装 [Mosh](https://mosh.org/) 作为 SSH 的替代品，它可以处理断开连接、进入/退出睡眠、切换网络和处理高延迟链接。

要让 `ssh` 让我们在远程服务器上运行命令，我们需要证明我们有权限这样做。我们可以通过密码或 ssh 密钥来做到这一点。基于密钥的身份验证利用公钥密码学向服务器证明客户端拥有秘密私钥而不泄露密钥。基于密钥的身份验证既更方便又更安全，所以你应该优先使用它。注意私钥（通常是 `~/.ssh/id_rsa`，最近是 `~/.ssh/id_ed25519`）实际上就是你的密码，所以要像对待密码一样对待它，永远不要分享其内容。

要生成一对密钥，你可以运行 [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html)：
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

如果你曾经使用 SSH 密钥配置推送到 GitHub，那么你可能已经完成了[这里](https://help.github.com/articles/connecting-to-github-with-ssh/)列出的步骤，并且已经有一个有效的密钥对。要检查你是否有密码短语并验证它，你可以运行 `ssh-keygen -y -f /path/to/key`。

在服务器端，`ssh` 会查看 `.ssh/authorized_keys` 来确定应该让哪些客户端进入。要复制公钥，你可以使用：

```bash
cat .ssh/id_ed25519.pub | ssh alice@remote 'cat >> ~/.ssh/authorized_keys'

# 或者更简单（如果 ssh-copy-id 可用）

ssh-copy-id -i .ssh/id_ed25519 alice@remote
```

除了运行命令，ssh 建立的连接还可以用于安全地从服务器传输文件或传输文件到服务器。[`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) 是最传统的工具，语法是 `scp path/to/local_file remote_host:path/to/remote_file`。[`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) 通过检测本地和远程中的相同文件并防止再次复制它们来改进 `scp`。它还提供对符号链接、权限更精细的控制，并有额外功能如 `--partial` 标志，可以从之前中断的复制恢复。`rsync` 的语法与 `scp` 类似。

SSH 客户端配置位于 `~/.ssh/config`，它让我们声明主机并为它们设置默认设置。这个配置文件不仅被 `ssh` 读取，还被其他程序如 `scp`、`rsync`、`mosh` 等读取。

```bash
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# 配置也可以使用通配符
Host *.mit.edu
    User alice
```

# 终端复用器

使用命令行界面时，你经常想同时运行多件事情。例如，你可能想并排运行编辑器和程序。虽然这可以通过打开新的终端窗口来实现，但使用终端复用器是一个更通用的解决方案。

像 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) 这样的终端复用器允许你使用窗格和标签页复用终端窗口，这样你可以高效地与多个 shell 会话交互。此外，终端复用器让你分离当前终端会话并在稍后的某个时间点重新连接。因此，终端复用器在处理远程机器时非常方便，因为它避免了使用 `nohup` 和类似技巧的需要。

如今最流行的终端复用器是 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html)。`tmux` 高度可配置，通过使用关联的键绑定，你可以创建多个标签页和窗格并快速在它们之间导航。

`tmux` 期望你知道它的键绑定，它们都有 `<C-b> x` 的形式，意思是 (1) 按 `Ctrl+b`，(2) 释放 `Ctrl+b`，然后 (3) 按 `x`。`tmux` 有以下对象层次结构：
- **会话** - 会话是一个独立的工作区，包含一个或多个窗口
    + `tmux` 启动一个新会话。
    + `tmux new -s NAME` 以该名称启动它。
    + `tmux ls` 列出当前会话
    + 在 `tmux` 中输入 `<C-b> d` 分离当前会话
    + `tmux a` 附加最后一个会话。你可以使用 `-t` 标志指定哪个

- **窗口** - 等同于编辑器或浏览器中的标签页，它们是同一会话的可视化独立部分
    + `<C-b> c` 创建一个新窗口。要关闭它，你可以直接终止 shell，执行 `<C-d>`
    + `<C-b> N` 跳转到第 N 个窗口。注意它们是编号的
    + `<C-b> p` 跳转到上一个窗口
    + `<C-b> n` 跳转到下一个窗口
    + `<C-b> ,` 重命名当前窗口
    + `<C-b> w` 列出当前窗口

- **窗格** - 像 vim 分屏，窗格让你在同一可视显示中有多个 shell。
    + `<C-b> "` 水平分割当前窗格
    + `<C-b> %` 垂直分割当前窗格
    + `<C-b> <方向>` 移动到指定方向的窗格。方向这里指箭头键。
    + `<C-b> z` 切换当前窗格的缩放
    + `<C-b> [` 开始滚动回看。然后你可以按 `<space>` 开始选择，按 `<enter>` 复制该选择。
    + `<C-b> <space>` 循环窗格排列。

> 要了解更多关于 tmux 的信息，考虑阅读[这个](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)快速教程和[这个](https://linuxcommand.org/lc3_adv_termmux.php)更详细的解释。

有了 tmux 和 SSH 在你的工具箱中，你会想让你的环境在任何机器上都感觉像家一样。这就是 shell 定制发挥作用的地方。

# 定制 Shell

各种各样的命令行程序使用称为点文件的纯文本文件配置（因为文件名以 `.` 开头，例如 `~/.vimrc`，这样它们在目录列表 `ls` 中默认隐藏）。

> 点文件是另一个 shell 约定。前面的点是为了在列出时"隐藏"它们（是的，另一个约定）。

Shell 是使用此类文件配置的程序示例之一。启动时，你的 shell 会读取许多文件来加载其配置。根据 shell 以及你是否正在启动登录和/或交互式会话，整个过程可能相当复杂。[这里](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)是关于该主题的绝佳资源。

对于 `bash`，编辑你的 `.bashrc` 或 `.bash_profile` 在大多数系统中都可以工作。其他一些可以通过点文件配置的工具示例是：

- `bash` - `~/.bashrc`、`~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` 和 `~/.vim` 文件夹
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

一个常见的配置更改是为 shell 添加新位置来查找程序。你在安装软件时会遇到这种模式：

```shell
export PATH="$PATH:path/to/append"
```

这里，我们告诉 shell 将 `$PATH` 变量的值设置为其当前值加上一个新路径，并让所有子进程继承这个新的 PATH 值。这将允许子进程找到位于 `path/to/append` 下的程序。

定制你的 shell 通常意味着安装新的命令行工具。包管理器使这变得容易。它们处理下载、安装和更新软件。不同的操作系统有不同的包管理器：macOS 使用 [Homebrew](https://brew.sh/)，Ubuntu/Debian 使用 `apt`，Fedora 使用 `dnf`，Arch 使用 `pacman`。我们将在发布代码讲座中更深入地介绍包管理器。

以下是如何在 macOS 上使用 Homebrew 安装两个有用的工具：

```shell
# ripgrep: 更快的 grep，有更好的默认值
brew install ripgrep

# fd: 更快、用户友好的 find
brew install fd
```

安装后，你可以使用 `rg` 代替 `grep`，使用 `fd` 代替 `find`。

> **关于 `curl | bash` 的警告**：你经常会看到像 `curl -fsSL https://example.com/install.sh | bash` 这样的安装说明。这种模式下载脚本并立即执行它，这很方便但有风险；你正在运行你没有检查的代码。更安全的方法是先下载、审查、然后执行：
> ```shell
> curl -fsSL https://example.com/install.sh -o install.sh
> less install.sh  # 审查脚本
> bash install.sh
> ```
> 一些安装程序使用稍微安全的变体：`/bin/bash -c "$(curl -fsSL https://url)"` 这至少确保 bash 解释脚本而不是你当前的 shell。

当你尝试运行一个未安装的命令时，你的 shell 会显示 `command not found`。网站 [command-not-found.com](https://command-not-found.com) 是一个有用的资源，你可以用它搜索任何命令，找出如何在不同包管理器和发行版中安装它。

另一个有用的工具是 [`tldr`](https://tldr.sh/)，它提供简化的、以示例为重点的 man 页面。你可以快速查看常见的使用模式，而不是阅读冗长的文档：

```console
$ tldr fd
  An alternative to find.
  Aims to be faster and easier to use than find.

  Recursively find files matching a pattern in the current directory:
      fd "pattern"

  Find files that begin with "foo":
      fd "^foo"

  Find files with a specific extension:
      fd --extension txt
```

有时你不需要一个全新的程序，而只是需要一个带有特定标志的现有命令的快捷方式。这就是别名发挥作用的地方。

我们也可以使用 `alias` shell 内置命令创建自己的命令别名。shell 别名是另一个命令的简写形式，你的 shell 会在评估表达式之前自动替换它。例如，bash 中的别名有以下结构：

```bash
alias alias_name="command_to_alias arg1 arg2"
```

> 注意等号 `=` 周围没有空格，因为 [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) 是一个接受单个参数的 shell 命令。

别名有很多方便的功能：

```bash
# 为常见标志创建简写
alias ll="ls -lh"

# 为常见命令节省大量输入
alias gs="git status"
alias gc="git commit"

# 防止你打错
alias sl=ls

# 用更好的默认值覆盖现有命令
alias mv="mv -i"           # -i 在覆盖前提示
alias mkdir="mkdir -p"     # -p 根据需要创建父目录
alias df="df -h"           # -h 打印人类可读格式

# 别名可以组合
alias la="ls -A"
alias lla="la -l"

# 要忽略别名，在前面加上 \
\ls
# 或者用 unalias 完全禁用别名
unalias la

# 要获取别名定义，只需用 alias 调用它
alias ll
# 将打印 ll='ls -lh'
```

别名有限制：它们不能在命令中间接受参数。对于更复杂的行为，你应该使用 shell 函数。

大多数 shell 支持 `Ctrl-R` 进行反向历史搜索。输入 `Ctrl-R` 并开始输入来搜索以前的命令。早些时候我们介绍了 `fzf` 作为模糊查找器；配置了 fzf 的 shell 集成后，`Ctrl-R` 变成对你整个历史的交互式模糊搜索，远比默认功能强大。

你应该如何组织你的点文件？它们应该在它们自己的文件夹中，在版本控制下，并使用脚本符号链接到正确位置。这有以下好处：

- **易于安装**：如果你登录到一台新机器，应用你的定制只需要一分钟。
- **可移植性**：你的工具在任何地方都以相同方式工作。
- **同步**：你可以在任何地方更新你的点文件并保持它们全部同步。
- **变更追踪**：你可能会在整个编程职业生涯中维护你的点文件，版本历史对于长期项目很有用。

你应该在你的点文件中放什么？你可以通过阅读在线文档或 [man 页面](https://en.wikipedia.org/wiki/Man_page)了解你的工具设置。另一个好方法是在网上搜索关于特定程序的博客文章，作者会告诉你他们喜欢的定制。学习定制的另一种方法是查看其他人的点文件：你可以在 GitHub 上找到大量的[点文件仓库](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)——请参阅最受欢迎的一个[这里](https://github.com/mathiasbynens/dotfiles)（我们建议你不要盲目复制配置）。[这里](https://dotfiles.github.io/)是关于该主题的另一个好资源。

所有课程讲师都有他们的点文件在 GitHub 上公开可访问：[Anish](https://github.com/anishathalye/dotfiles)、[Jon](https://github.com/jonhoo/configs)、[Jose](https://github.com/jjgo/dotfiles)。

**框架和插件**也可以改进你的 shell。一些流行的通用框架是 [prezto](https://github.com/sorin-ionescu/prezto) 或 [oh-my-zsh](https://ohmyz.sh/)，以及专注于特定功能的较小插件：

- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - 在输入时为有效/无效命令着色
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) - 在输入时从历史建议命令
- [zsh-completions](https://github.com/zsh-users/zsh-completions) - 额外的补全定义
- [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) - 类似 fish 的历史搜索
- [powerlevel10k](https://github.com/romkatv/powerlevel10k) - 快速、可定制的提示符主题

像 [fish](https://fishshell.com/) 这样的 shell 默认包含许多这些功能。

> 你不需要像 oh-my-zsh 这样的大型框架来获得这些功能。安装单个插件通常更快，给你更多控制。大型框架可能会显著减慢 shell 启动时间，所以考虑只安装你实际使用的东西。

# Shell 中的 AI

有很多方法可以在 shell 中集成 AI 工具。以下是不同集成级别的几个示例：

**命令生成**：像 [`simonw/llm`](https://github.com/simonw/llm) 这样的工具可以帮助从自然语言描述生成 shell 命令：

```console
$ llm cmd "find all python files modified in the last week"
find . -name "*.py" -mtime -7
```

**管道集成**：LLMs 可以集成到 shell 管道中来处理和转换数据。当你需要从不一致的格式中提取信息时，它们特别有用，因为正则表达式会很痛苦：

```console
$ cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor
$ INSTRUCTIONS="Extract just the username from each line, one per line, nothing else"
$ llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor
```

注意我们如何使用 `"$INSTRUCTIONS"`（带引号），因为变量包含空格，以及 `< users.txt` 将文件内容重定向到 stdin。

**AI shells**：像 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 这样的工具充当元 shell，接受英语命令并将其翻译为 shell 操作、文件编辑和更复杂的多步骤任务。

# 终端模拟器

除了定制你的 shell，值得花一些时间弄清楚你选择的终端模拟器及其设置。终端模拟器是一个 GUI 程序，提供你的 shell 运行的基于文本的界面。有很多终端模拟器。

由于你可能会在终端中花费数百到数千小时，研究其设置是值得的。你可能想要在终端中修改的一些方面包括：

- 字体选择
- 配色方案
- 键盘快捷键
- 标签页/窗格支持
- 滚动回看配置
- 性能（一些较新的终端如 [Alacritty](https://github.com/alacritty/alacritty) 或 [Ghostty](https://ghostty.org/) 提供 GPU 加速）。

# 练习

## 参数和通配符

1. 你可能会看到像 `cmd --flag -- --notaflag` 这样的命令。`--` 是一个特殊参数，告诉程序停止解析标志。`--` 之后的所有内容都被视为位置参数。为什么这可能有用？尝试运行 `touch -- -myfile`，然后在没有 `--` 的情况下删除它。

1. 阅读 [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) 并编写一个 `ls` 命令，按以下方式列出文件：
    - 包括所有文件，包括隐藏文件
    - 大小以人类可读格式列出（例如 454M 而不是 454279954）
    - 文件按最近时间排序
    - 输出带颜色

    示例输出如下所示：

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

1. 进程替换 `<(command)` 让你使用命令的输出就像它是一个文件一样。使用带有进程替换的 `diff` 比较 `printenv` 和 `export` 的输出。为什么它们不同？（提示：尝试 `diff <(printenv | sort) <(export | sort)`）。

## 环境变量

1. 编写 bash 函数 `marco` 和 `polo`，执行以下操作：每当你执行 `marco` 时，当前工作目录应该以某种方式保存，然后当你执行 `polo` 时，无论你在哪个目录，`polo` 都应该 `cd` 回到你执行 `marco` 的目录。为了便于调试，你可以将代码写在文件 `marco.sh` 中，并通过执行 `source marco.sh`（重新）加载定义到你的 shell。

## 返回码

1. 假设你有一个很少失败的命令。为了调试它，你需要捕获它的输出，但获得一次失败运行可能很耗时。编写一个 bash 脚本，运行以下脚本直到它失败，并将其标准输出和错误流捕获到文件中，最后打印所有内容。如果你还能报告脚本失败需要多少次运行，那就有加分。

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi

    echo "Everything went according to plan"
    ```

## 信号和作业控制

1. 在终端中启动一个 `sleep 10000` 作业，用 `Ctrl-Z` 后台化它，然后用 `bg` 继续它的执行。现在使用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 找到它的 pid，并用 [`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html) 杀死它，而无需输入 pid 本身。（提示：使用 `-af` 标志）。

1. 假设你不希望在另一个进程完成之前启动一个进程。你会怎么做？在这个练习中，我们的限制进程将总是 `sleep 60 &`。一种实现方法是使用 [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) 命令。尝试启动 sleep 命令并让 `ls` 等待直到后台进程完成。

    但是，如果我们在不同的 bash 会话中启动，这个策略将失败，因为 `wait` 只对子进程有效。我们在笔记中没有讨论的一个功能是 `kill` 命令的退出状态在成功时为零，否则为非零。`kill -0` 不发送信号，但如果进程不存在会给出非零退出状态。编写一个名为 `pidwait` 的 bash 函数，接受一个 pid 并等待直到给定进程完成。你应该使用 `sleep` 来避免不必要地浪费 CPU。

## 文件和权限

1. （高级）编写一个命令或脚本来递归查找目录中最近修改的文件。更一般地，你能按最近时间列出所有文件吗？

## 终端复用器

1. 按照这个 `tmux` [教程](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)，然后按照[这些步骤](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)学习如何做一些基本的定制。

## 别名和点文件

1. 创建一个别名 `dc`，当你输错时解析为 `cd`。

1. 运行 `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` 获取你最常用的 10 个命令，并考虑为它们编写更短的别名。注意：这对 Bash 有效；如果你使用 ZSH，使用 `history 1` 而不是 `history`。

1. 为你的点文件创建一个文件夹并设置版本控制。

1. 为至少一个程序添加配置，例如你的 shell，带有一些定制（开始时，可以像通过设置 `$PS1` 定制你的 shell 提示符这样简单）。

1. 设置一种在新机器上快速安装你的点文件的方法（无需手动操作）。这可以简单到为每个文件调用 `ln -s` 的 shell 脚本，或者你可以使用[专门的工具](https://dotfiles.github.io/utilities/)。

1. 在新的虚拟机上测试你的安装脚本。

1. 将你当前所有的工具配置迁移到你的点文件仓库。

1. 在 GitHub 上发布你的点文件。

## 远程机器（SSH）

安装一个 Linux 虚拟机（或使用已有的）来做这些练习。如果你不熟悉虚拟机，请查看[这个](https://hibbard.eu/install-ubuntu-virtual-box/)安装教程。

1. 进入 `~/.ssh/` 并检查你是否有一对 SSH 密钥。如果没有，用 `ssh-keygen -a 100 -t ed25519` 生成它们。建议你使用密码并使用 `ssh-agent`，更多信息在[这里](https://www.ssh.com/ssh/agent)。

1. 编辑 `.ssh/config` 使其有以下条目：

    ```bash
    Host vm
        User username_goes_here
        HostName ip_goes_here
        IdentityFile ~/.ssh/id_ed25519
        LocalForward 9999 localhost:8888
    ```

1. 使用 `ssh-copy-id vm` 将你的 ssh 密钥复制到服务器。

1. 通过执行 `python -m http.server 8888` 在你的 VM 中启动一个 Web 服务器。通过在机器上导航到 `http://localhost:9999` 访问 VM Web 服务器。

1. 通过执行 `sudo vim /etc/ssh/sshd_config` 编辑你的 SSH 服务器配置，并通过编辑 `PasswordAuthentication` 的值禁用密码认证。通过编辑 `PermitRootLogin` 的值禁用 root 登录。用 `sudo service sshd restart` 重启 `ssh` 服务。尝试再次 ssh。

1. （挑战）在 VM 中安装 [`mosh`](https://mosh.org/) 并建立连接。然后断开服务器/VM 的网络适配器。mosh 能正确恢复吗？

1. （挑战）研究 `ssh` 中的 `-N` 和 `-f` 标志做什么，并找出一个命令来实现后台端口转发。