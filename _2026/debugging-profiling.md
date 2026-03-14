---
layout: lecture
title: "调试与分析"
description: >
  学习如何使用日志和调试器调试程序，以及如何分析代码性能。
thumbnail: /static/assets/thumbnails/2026/lec4.png
date: 2026-01-15
ready: true
panopto: "https://mit.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=a72c48e3-5eb2-46fa-aa03-b3b700e1ca8d"
video:
  aspect: 56.25
  id: 8VYT9TcUmKs
---

编程的一条黄金法则：代码不会做你期望它做的事，而是做你告诉它做的事。弥合这一差距有时可能是一项相当困难的壮举。在本讲座中，我们将介绍处理有缺陷和资源消耗代码的有用技术：调试和分析。

# 调试

## Printf 调试与日志

> "最有效的调试工具仍然是仔细的思考，加上明智放置的打印语句" —— Brian Kernighan，《Unix for Beginners》。

调试程序的第一种方法是在检测到问题的地方周围添加打印语句，并不断迭代，直到你提取到足够的信息来理解是什么导致了问题。

第二种方法是在程序中使用日志，而不是临时的打印语句。日志本质上是"更用心的打印"，通常通过日志框架完成，它内置支持以下功能：

- 将日志（或日志的子集）定向到其他输出位置的能力；
- 设置严重级别（如 INFO、DEBUG、WARN、ERROR 等），并允许你根据这些级别过滤输出；以及
- 支持与日志条目相关的数据的结构化日志记录，之后可以更容易地提取。

你通常还会在编程时主动放入日志语句，这样你调试所需的数据可能已经存在！确实，一旦你使用打印语句发现并修复了一个问题，通常值得在删除它们之前将这些打印转换为适当的日志语句。这样，如果将来出现类似的 bug，你已经有所需的诊断信息，而无需修改代码。

> **第三方日志**：许多程序支持 `-v` 或 `--verbose` 标志以在运行时打印更多信息。这对于发现给定命令为什么失败很有用。有些甚至允许重复该标志以获取更多详细信息。在调试服务（数据库、Web 服务器等）问题时，检查它们的日志——在 Linux 上通常在 `/var/log/`。使用 `journalctl -u <service>` 查看 systemd 服务的日志。对于第三方库，检查它们是否通过环境变量或配置支持调试日志。

## 调试器

当你知道要打印什么并且可以轻松修改和重新运行代码时，打印调试效果很好。当你不确定需要什么信息，当 bug 只在难以重现的条件下出现，或者修改和重新启动程序代价很高（长启动时间、复杂的状态重建等）时，调试器就变得有价值。

调试器是让你与程序执行交互的程序，允许你：

- 当执行到某一行时暂停执行。
- 一次单步执行一条指令。
- 在崩溃后检查变量的值。
- 当给定条件满足时有条件地暂停执行。
- 以及更多高级功能。

大多数编程语言都支持（或附带）某种形式的调试器。最通用的是**通用调试器**，如 [`gdb`](https://www.gnu.org/software/gdb/)（GNU 调试器）和 [`lldb`](https://lldb.llvm.org/)（LLVM 调试器），它们可以调试任何本地二进制文件。许多语言还有与运行时更紧密集成的**语言特定调试器**（如 Python 的 pdb 或 Java 的 jdb）。

`gdb` 是 C、C++、Rust 和其他编译语言的事实标准调试器。它让你可以探测几乎任何进程并获取其当前机器状态：寄存器、栈、程序计数器等。

一些有用的 GDB 命令：

- `run` - 启动程序
- `b {function}` 或 `b {file}:{line}` - 设置断点
- `c` - 继续执行
- `step` / `next` / `finish` - 单步进入 / 单步跳过 / 单步退出
- `p {variable}` - 打印变量的值
- `bt` - 显示回溯（调用栈）
- `watch {expression}` - 当值改变时中断

> 考虑使用 GDB 的 TUI 模式（`gdb -tui` 或在 GDB 中按 `Ctrl-x a`），获得分屏视图，同时显示源代码和命令提示符。

### 录制回放调试

一些最令人沮丧的 bug 是海森堡 bug：当你试图观察它们时，它们似乎会消失或改变行为的 bug。竞争条件、时序相关的 bug 以及只在某些系统条件下出现的问题属于这一类。传统调试在这里通常无能为力，因为再次运行程序会产生不同的行为（例如，打印语句可能会充分减慢代码速度，使竞争不再发生）。

**录制回放调试**通过录制程序的执行并允许你根据需要多次确定性地回放它来解决这个问题。更棒的是，你可以反向遍历执行，准确找到问题出在哪里。

[rr](https://rr-project.org/) 是一个强大的 Linux 工具，它录制程序执行并允许具有完整调试功能的确定性回放。它与 GDB 配合使用，所以你已经知道接口。

基本用法：

```bash
# 录制程序执行
rr record ./my_program

# 回放录制（打开 GDB）
rr replay
```

魔法发生在回放期间。因为执行是确定性的，你可以使用**反向调试**命令：

- `reverse-continue` (`rc`) - 反向运行直到命中断点
- `reverse-step` (`rs`) - 反向单步执行一行
- `reverse-next` (`rn`) - 反向单步执行，跳过函数调用
- `reverse-finish` - 反向运行直到进入当前函数

这对于调试非常强大。假设你有一个崩溃——你可以：

1. 运行到崩溃
2. 检查损坏的状态
3. 在损坏的变量上设置观察点
4. `reverse-continue` 找到它被损坏的确切位置

**何时使用 rr：**
- 间歇性失败的测试
- 竞争条件和线程 bug
- 难以重现的崩溃
- 任何你希望可以"回到过去"的 bug

> 注意：rr 只在 Linux 上工作，需要硬件性能计数器。它在不暴露这些计数器的 VM 中不工作，如在大多数 AWS EC2 实例上，并且不支持 GPU 访问。对于 macOS，请查看 [Warpspeed](https://warpspeed.dev/)。

> **rr 和并发**：因为 rr 确定性地录制执行，它序列化线程调度。这意味着如果某些竞争条件依赖于特定时序，它们可能不会在 rr 下出现。rr 对于调试竞争仍然有用——一旦你捕获到失败的运行，你可以可靠地回放它——但你可能需要多次录制尝试来捕获间歇性 bug。对于不涉及并发的 bug，rr 大放异彩：你总是可以重现确切的执行并使用反向调试来追踪损坏。

## 系统调用追踪

有时你需要了解程序如何与操作系统交互。程序通过[系统调用](https://en.wikipedia.org/wiki/System_call)请求内核的服务——打开文件、分配内存、创建进程等。追踪这些调用可以揭示程序为什么挂起、它试图访问什么文件，或者它在哪里花费时间等待。

### strace（Linux）和 dtruss（macOS）

[`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) 让你观察程序进行的每一个系统调用：

```bash
# 追踪所有系统调用
strace ./my_program

# 只追踪文件相关调用
strace -e trace=file ./my_program

# 跟踪子进程（对于启动其他程序的程序很重要）
strace -f ./my_program

# 追踪正在运行的进程
strace -p <PID>

# 显示计时信息
strace -T ./my_program
```

> 在 macOS 和 BSD 上，使用 [`dtruss`](https://www.manpagez.com/man/1/dtruss/)（它封装了 `dtrace`）获得类似功能：

> 要更深入了解 `strace`，请查看 Julia Evans 出色的 [strace zine](https://jvns.ca/strace-zine-unfolded.pdf)。

### bpftrace 和 eBPF

[eBPF](https://ebpf.io/)（扩展伯克利包过滤器）是一种强大的 Linux 技术，允许在内核中运行沙箱程序。[`bpftrace`](https://github.com/iovisor/bpftrace) 提供了用于编写 eBPF 程序的高级语法。这些是在内核中运行的任意程序，因此具有巨大的表达能力（尽管也有有点笨拙的类似 awk 的语法）。它们最常见的用例是调查正在调用什么系统调用，包括聚合（如计数或延迟统计）或内省（甚至过滤）系统调用参数。

```bash
# 全系统追踪文件打开（立即打印）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# 按名称计数系统调用（在 Ctrl-C 时打印摘要）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }'
```

但是，你也可以使用像 [`bcc`](https://github.com/iovisor/bcc) 这样的工具链直接用 C 编写 eBPF 程序，它还附带[许多方便的工具](https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html)，如 `biosnoop` 用于打印磁盘操作的延迟分布，或 `opensnoop` 用于打印所有打开的文件。

`strace` 有用是因为它容易"直接上手运行"，而 `bpftrace` 是当你需要更低开销、想追踪内核函数、需要做任何类型的聚合等时应该使用的工具。注意 `bpftrace` 必须以 `root` 运行，而且它通常监控整个内核，不仅仅是特定进程。要针对特定程序，你可以按命令名或 PID 过滤：

```bash
# 按命令名过滤（在 Ctrl-C 时打印摘要）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /comm == "bash"/ { @[probe] = count(); }'

# 使用 -c 从启动追踪特定命令（cpid = 子进程 PID）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == cpid/ { @[probe] = count(); }' -c 'ls -la'
```

`-c` 标志运行指定的命令并将 `cpid` 设置为其 PID，这对于从程序启动那一刻开始追踪很有用。当被追踪的命令退出时，bpftrace 打印聚合结果。

### 网络调试

对于网络问题，[`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) 和 [Wireshark](https://www.wireshark.org/) 让你捕获和分析网络数据包：

```bash
# 捕获端口 80 上的数据包
sudo tcpdump -i any port 80

# 捕获并保存到文件供 Wireshark 分析
sudo tcpdump -i any -w capture.pcap
```

对于 HTTPS 流量，加密使 tcpdump 不太有用。像 [mitmproxy](https://mitmproxy.org/) 这样的工具可以充当拦截代理来检查加密流量。浏览器开发者工具（Network 标签页）通常是调试来自 Web 应用程序的 HTTPS 请求的最简单方式——它们显示解密的请求/响应数据、标头和计时。

## 内存调试

内存 bug——缓冲区溢出、释放后使用、内存泄漏——是最危险和最难调试的 bug 之一。它们通常不会立即崩溃，而是以稍后导致问题的方式损坏内存。

### 消毒器

发现内存 bug 的一种方法是使用消毒器，这是编译器功能，用于检测代码以在运行时检测错误。例如，广泛使用的 AddressSanitizer（ASan）检测：

- 缓冲区溢出（栈、堆和全局）
- 释放后使用
- 返回后使用
- 内存泄漏

```bash
# 使用 AddressSanitizer 编译
gcc -fsanitize=address -g program.c -o program
./program
```

有各种有用的消毒器：

- **ThreadSanitizer（TSan）**：检测多线程代码中的数据竞争（`-fsanitize=thread`）
- **MemorySanitizer（MSan）**：检测未初始化内存的读取（`-fsanitize=memory`）
- **UndefinedBehaviorSanitizer（UBSan）**：检测未定义行为，如整数溢出（`-fsanitize=undefined`）

消毒器需要重新编译，但足够快，可以在 CI 管道和常规开发中使用。

### Valgrind：当你无法重新编译时

[Valgrind](https://valgrind.org/) 在类似于虚拟机的环境中运行你的程序来检测内存错误。它比消毒器慢，但不需要重新编译：

```bash
valgrind --leak-check=full ./my_program
```

在以下情况下使用 Valgrind：

- 你没有源代码
- 你无法重新编译（第三方库）
- 你需要消毒器不可用的特定工具

Valgrind 实际上是一个非常强大的受控执行环境，当我们进入分析时，我们会看到更多！

## AI 用于调试

大型语言模型已成为令人惊讶的有用调试助手。它们擅长某些补充传统工具的调试任务。

**LLMs 擅长的方面：**

- **解释晦涩的错误消息**：编译器错误，尤其是来自 C++ 模板或 Rust 借用检查器的错误，可能非常晦涩。LLMs 可以将它们翻译成通俗易懂的语言并建议修复。

- **跨越语言和抽象边界**：如果你正在调试一个跨多种语言的问题（比如一个 C 库中的 bug 通过 Python 绑定表现出来），LLMs 可以帮助导航不同的层。它们特别擅长理解 FFI 边界、构建系统问题和跨语言调试。

- **将症状与根本原因关联**："我的程序工作正常但使用的内存比预期多 10 倍"是 LLMs 可以帮助调查的那种模糊症状，建议可能的原因和要查找的内容。

- **分析崩溃转储和栈跟踪**：粘贴栈跟踪并询问可能导致了什么。

> **关于调试符号的说明**：为了获得有意义的栈跟踪和调试，请确保你的二进制文件（和任何链接的库）使用调试符号（`-g` 标志）编译。调试信息通常以 DWARF 格式存储。此外，使用帧指针编译（`-fno-omit-frame-pointer`）使栈跟踪更可靠，特别是对于分析工具。没有这些，栈跟踪可能只显示内存地址或不完整。这对本地编译的程序（C++、Rust）比 Python 或 Java 更重要。

**要记住的局限性：**
- LLMs 可能会产生听起来合理但错误的解释
- 它们可能建议掩盖 bug 而不是修复 bug 的修复
- 始终使用实际调试工具验证建议
- 它们作为补充而不是替代理解你的代码效果最好

> 这与开发环境讲座中介绍的[通用 AI 编码能力]({{ site.baseurl }}/2026/development-environment/#ai-驱动的开发)不同。这里我们专门讨论将 LLMs 用作调试辅助。

# 分析

即使你的代码在功能上按预期运行，如果它占用了你所有的 CPU 或内存，那可能还不够好。算法课程通常教授大 O 表示法，但不教你如何找到程序中的热点。由于[过早优化是万恶之源](https://wiki.c2.com/?PrematureOptimization)，你应该了解分析器和监控工具。它们将帮助你了解程序的哪些部分占用了大部分时间和/或资源，这样你就可以专注于优化这些部分。

## 计时

测量性能的最简单方法是计时。在许多场景中，打印代码两点之间花费的时间可能就足够了。

但是，墙上时钟时间可能会产生误导，因为你的计算机可能同时运行其他进程或等待事件发生。`time` 命令区分了 Real、User 和 Sys 时间：

- **Real** - 从开始到结束的墙上时钟时间，包括等待时间
- **User** - CPU 运行用户代码花费的时间
- **Sys** - CPU 运行内核代码花费的时间

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real	0m0.272s
user	0m0.079s
sys	    0m0.028s
```

这里请求花费了近 300 毫秒（实际时间），但只有 107ms 的 CPU 时间（用户 + 系统）。其余时间是在等待网络。

## 资源监控

有时分析程序性能的第一步是了解其实际资源消耗。当资源受限时，程序通常运行缓慢。

- **常规监控**：[`htop`](https://htop.dev/) 是 `top` 的改进版本，显示当前运行进程的各种统计信息。有用的快捷键：`<F6>` 排序进程，`t` 显示树形层次结构，`h` 切换线程。还有 [`btop`](https://github.com/aristocratos/btop) 监控更多东西。

- **I/O 操作**：[`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) 显示实时 I/O 使用信息。

- **内存使用**：[`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) 显示总的空闲和已用内存。

- **打开的文件**：[`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) 列出进程打开的文件信息。用于检查哪个进程打开了特定文件。

- **网络连接**：[`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) 让你监控网络连接。常见用例是找出什么进程正在使用给定端口：`ss -tlnp | grep :8080`。

- **网络使用**：[`nethogs`](https://github.com/raboof/nethogs) 和 [`iftop`](https://pdw.ex-parrot.com/iftop/) 是很好的交互式 CLI 工具，用于监控每个进程的网络使用情况。

## 可视化性能数据

人类在图表中发现模式比在数字表格中快得多。在分析性能时，绘制数据通常会揭示在原始数字中不可见的趋势、峰值和异常。

**使数据可绘制**：在添加打印或日志语句进行调试时，考虑格式化输出以便以后可以轻松绘图。CSV 格式的简单时间戳和值（`1705012345,42.5`）比散文句子更容易绘图。JSON 结构化日志也可以用最少的努力解析和绘图。换句话说，以整洁的方式[记录你的数据](https://vita.had.co.nz/papers/tidy-data.pdf)。

**使用 gnuplot 快速绘图**：对于简单的命令行绘图，[`gnuplot`](http://www.gnuplot.info/) 可以直接从数据文件生成图表：

```bash
# 绘制带有时间戳、值的简单 CSV
gnuplot -e "set datafile separator ','; plot 'latency.csv' using 1:2 with lines"
```

**使用 matplotlib 和 ggplot2 进行迭代探索**：对于更深入的分析，Python 的 [`matplotlib`](https://matplotlib.org/) 和 R 的 [`ggplot2`](https://ggplot2.tidyverse.org/) 支持迭代探索。与一次性绘图不同，这些工具让你快速切片和转换数据来调查假设。ggplot2 的分面图特别强大——你可以按类别将单个数据集分割成多个子图（例如，按端点或一天中的时间分面请求延迟），以揭示否则会被隐藏的模式。

**示例用例：**
- 绘制随时间变化的请求延迟揭示周期性减速（垃圾收集、cron 作业、流量模式），原始百分位数会掩盖这些
- 可视化增长数据结构的插入时间可以暴露算法复杂性问题——向量插入的图表将在后备数组大小翻倍时显示特征性峰值
- 按不同维度（请求类型、用户群组、服务器）分面指标通常揭示"全系统"问题实际上只限于某个类别

## CPU 分析器

大多数时候人们提到分析器时，他们指的是 CPU 分析器。主要有两种类型：

- **追踪分析器**记录你的程序进行的每个函数调用
- **采样分析器**定期探测你的程序（通常每毫秒）并记录程序的栈

采样分析器开销较低，通常更适合生产使用。

### perf：采样分析器

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) 是标准的 Linux 分析器。它可以分析任何程序而无需重新编译：

`perf stat` 让你快速了解时间花在哪里：

```bash
$ perf stat ./slow_program

 Performance counter stats for './slow_program':

         3,210.45 msec task-clock                #    0.998 CPUs utilized
               12      context-switches          #    3.738 /sec
                0      cpu-migrations            #    0.000 /sec
              156      page-faults               #   48.587 /sec
   12,345,678,901      cycles                    #    3.845 GHz
    9,876,543,210      instructions              #    0.80  insn per cycle
    1,234,567,890      branches                  #  384.532 M/sec
       12,345,678      branch-misses             #    1.00% of all branches
```

真实世界程序的分析器输出将包含大量信息。人类是视觉生物，非常不擅长阅读大量数字。[火焰图](https://www.brendangregg.com/flamegraphs.html)是一种使分析数据更容易理解的可视化。

火焰图沿 Y 轴显示函数调用的层次结构，沿 X 轴显示与时间成比例。它们是交互式的——你可以点击放大程序的特定部分。

[![FlameGraph](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

要从 `perf` 数据生成火焰图：

```bash
# 录制分析
perf record -g ./my_program

# 生成火焰图（需要 flamegraph 脚本）
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

> 考虑使用 [Speedscope](https://www.speedscope.app/) 获得交互式基于 Web 的火焰图查看器，或使用 [Perfetto](https://perfetto.dev/) 进行全面的系统级分析。

### Valgrind 的 Callgrind：追踪分析器

[`callgrind`](https://valgrind.org/docs/manual/cl-manual.html) 是一个分析工具，记录程序的调用历史和指令计数。与采样分析器不同，它提供精确的调用计数，并可以显示调用者和被调用者之间的关系：

```bash
# 使用 callgrind 运行
valgrind --tool=callgrind ./my_program

# 使用 callgrind_annotate（文本）或 kcachegrind（GUI）分析
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid>
```

Callgrind 比采样分析器慢，但提供精确的调用计数，如果需要，可以选择模拟缓存行为（使用 `--cache-sim=yes`）。

> 如果你使用特定的语言，可能有更专业的分析器。例如，Python 有 [`cProfile`](https://docs.python.org/3/library/profile.html) 和 [`py-spy`](https://github.com/benfred/py-spy)，Go 有 [`go tool pprof`](https://pkg.go.dev/cmd/pprof)，Rust 有 [`cargo-flamegraph`](https://github.com/flamegraph-rs/flamegraph)（实际上适用于任何编译程序！）。

## 内存分析器

内存分析器帮助你了解程序如何随时间使用内存并发现内存泄漏。

### Valgrind 的 Massif

[`massif`](https://valgrind.org/docs/manual/ms-manual.html) 分析堆内存使用：

```bash
valgrind --tool=massif ./my_program
ms_print massif.out.<pid>
```

这显示随时间变化的堆使用，帮助识别内存泄漏和过度分配。

> 对于 Python，[`memory-profiler`](https://pypi.org/project/memory-profiler/) 提供逐行内存使用信息。

## 基准测试

当你需要比较不同实现或工具的性能时，[`hyperfine`](https://github.com/sharkdp/hyperfine) 非常适合基准测试命令行程序：

```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

> 对于 Web 开发，浏览器开发者工具包含出色的分析器。请参阅 [Firefox Profiler](https://profiler.firefox.com/docs/) 和 [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/rendering-tools) 文档。

# 练习

## 调试

1. **调试排序算法**：以下伪代码实现了归并排序，但包含一个 bug。用你选择的语言实现它，然后使用调试器（gdb、lldb、pdb 或你 IDE 的调试器）找到并修复 bug。

   ```
   function merge_sort(arr):
       if length(arr) <= 1:
           return arr
       mid = length(arr) / 2
       left = merge_sort(arr[0..mid])
       right = merge_sort(arr[mid..end])
       return merge(left, right)

   function merge(left, right):
       result = []
       i = 0, j = 0
       while i < length(left) AND j < length(right):
           if left[i] <= right[j]:
               append result, left[i]
               i = i + 1
           else:
               append result, right[i]
               j = j + 1
       append remaining elements from left and right
       return result
   ```

   测试用例：`merge_sort([3, 1, 4, 1, 5, 9, 2, 6])` 应返回 `[1, 1, 2, 3, 4, 5, 6, 9]`。使用断点并单步执行 merge 函数，找到错误元素被选中的位置。

1. 安装 [`rr`](https://rr-project.org/) 并使用反向调试找到损坏 bug。将此程序保存为 `corruption.c`：

   ```c
   #include <stdio.h>

   typedef struct {
       int id;
       int scores[3];
   } Student;

   Student students[2];

   void init() {
       students[0].id = 1001;
       students[0].scores[0] = 85;
       students[0].scores[1] = 92;
       students[0].scores[2] = 78;

       students[1].id = 1002;
       students[1].scores[0] = 90;
       students[1].scores[1] = 88;
       students[1].scores[2] = 95;
   }

   void curve_scores(int student_idx, int curve) {
       for (int i = 0; i < 4; i++) {
           students[student_idx].scores[i] += curve;
       }
   }

   int main() {
       init();
       printf("=== Initial state ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       curve_scores(0, 5);

       printf("\n=== After curving ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       if (students[1].id != 1002) {
           printf("\nERROR: Student 1's ID was corrupted! Expected 1002, got %d\n",
                  students[1].id);
           return 1;
       }
       return 0;
   }
   ```

   用 `gcc -g corruption.c -o corruption` 编译并运行它。学生 1 的 ID 被损坏，但损坏发生在一个只接触学生 0 的函数中。使用 `rr record ./corruption` 和 `rr replay` 找到罪魁祸首。在 `students[1].id` 上设置观察点，在损坏后使用 `reverse-continue` 找到确切是哪一行代码覆盖了它。

1. 使用 AddressSanitizer 调试内存错误。将此保存为 `uaf.c`：

   ```c
   #include <stdlib.h>
   #include <string.h>
   #include <stdio.h>

   int main() {
       char *greeting = malloc(32);
       strcpy(greeting, "Hello, world!");
       printf("%s\n", greeting);

       free(greeting);

       greeting[0] = 'J';
       printf("%s\n", greeting);

       return 0;
   }
   ```

   首先不使用消毒器编译运行：`gcc uaf.c -o uaf && ./uaf`。它可能看起来工作正常。现在使用 AddressSanitizer 编译：`gcc -fsanitize=address -g uaf.c -o uaf && ./uaf`。阅读错误报告。ASan 发现了什么 bug？修复它识别的问题。

1. 使用 `strace`（Linux）或 `dtruss`（macOS）追踪像 `ls -l` 这样的命令进行的系统调用。它在进行什么系统调用？尝试追踪更复杂的程序，看看它打开什么文件。

1. 使用 LLM 帮助调试晦涩的错误消息。尝试复制编译器错误（尤其是来自 C++ 模板或 Rust 的错误）并请求解释和修复。尝试将 `strace` 或地址消毒器的一些输出放入其中。

## 分析

1. 使用 `perf stat` 获取你选择的程序的基本性能统计信息。不同的计数器是什么意思？

1. 使用 `perf record` 进行分析。将此保存为 `slow.c`：

   ```c
   #include <math.h>
   #include <stdio.h>

   double slow_computation(int n) {
       double result = 0;
       for (int i = 0; i < n; i++) {
           for (int j = 0; j < 1000; j++) {
               result += sin(i * j) * cos(i + j);
           }
       }
       return result;
   }

   int main() {
       double r = 0;
       for (int i = 0; i < 100; i++) {
           r += slow_computation(1000);
       }
       printf("Result: %f\n", r);
       return 0;
   }
   ```

   使用调试符号编译：`gcc -g -O2 slow.c -o slow -lm`。运行 `perf record -g ./slow`，然后 `perf report` 查看时间花在哪里。尝试使用 flamegraph 脚本生成火焰图。

1. 使用 `hyperfine` 对同一任务的两个不同实现进行基准测试（例如 `find` vs `fd`、`grep` vs `ripgrep`，或你自己代码的两个版本）。

1. 使用 `htop` 在运行资源密集型程序时监控系统。尝试使用 `taskset` 限制进程可以使用的 CPU：`taskset --cpu-list 0,2 stress -c 3`。为什么 `stress` 不使用三个 CPU？

1. 一个常见问题是你想监听的端口已被另一个进程占用。学习如何发现该进程：首先执行 `python -m http.server 4444` 在端口 4444 上启动一个最小的 Web 服务器。在另一个终端运行 `ss -tlnp | grep 4444` 找到进程。用 `kill <PID>` 终止它。