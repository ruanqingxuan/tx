# tx
腾讯实习的一些笔记，主要设计一些网络相关的性能测试工具

![img](https://qnwang.oss-cn-hangzhou.aliyuncs.com/internship/20250424210751823.png)

## 2025.4.24-2025.6.20（未入职阶段）

### 📆 第一阶段学习安排

#### 目标

1. **掌握常用性能分析工具的使用方法**
   - perf（Linux 下的性能分析器）
   - gperftools（Google 的性能工具）
   - valgrind（内存检查、cache分析等）
   - AddressSanitizer（ASan，用于内存错误检测）
2. **能够分析 C++ 应用的性能瓶颈**
   - CPU 占用分析
   - 内存分配与泄露
   - 网络流量监控与分析（如 tcpdump、wireshark、iperf）
3. **理解工具背后的工作原理**
   - perf 基于 Linux 的 perf_event
   - valgrind 使用动态二进制插桩技术
   - gperftools 的 tcmalloc 工作机制
   - ASan 利用编译器插桩技术
4. **整理学习笔记 + 做内部分享准备**

#### 建议的实践顺序

1. 先掌握 `perf` + `valgrind` 组合（Linux 通用工具）
2. 再熟悉 `gperftools` 的使用，理解 `pprof` 结果
3. 编译带 `ASan` 的小程序感受运行时内存错误捕捉
4. 最后可以学习一些高级用法：eBPF、系统级 trace（如 bcc、bpftrace）

每日学习任务表（建议每天 2–3 小时）

| 天数    | 学习主题                        | 学习内容                                                     | 输出内容                                    |
| ------- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------- |
| 第 1 天 | 性能分析基础 + perf 入门        | - 阅读《Linux 性能工具概览》或 perf 官方文档基础部分- 使用 `perf top`、`perf record` + `perf report` 跑 C++ 示例程序 | - perf 基本命令- 输出样例解释- 初步结果图   |
| 第 2 天 | perf 高级使用 + 实战分析        | - 使用 perf 分析 CPU 密集型程序（如排序、多线程）            | - CPU profiling 分析流程- 热点定位笔记      |
| 第 3 天 | valgrind 入门（重点：memcheck） | - 学习 valgrind 原理及子工具（memcheck, cachegrind）- 编写 demo 检测内存泄漏 | - valgrind 输出格式笔记- 常见内存错误总结   |
| 第 4 天 | valgrind 实战 + cache 分析      | - 使用 cachegrind 观察缓存行为- 拓展使用 massif 分析堆内存   | - 工具组合对照表- 使用记录                  |
| 第 5 天 | gperftools + pprof 入门         | - 安装 gperftools，使用 `pprof` 生成火焰图- 配置小程序使用 tcmalloc + CPU Profiler | - pprof 输出解读- 火焰图示例                |
| 第 6 天 | AddressSanitizer（ASan）实战    | - 学习 ASan 编译器参数原理- 制造越界/UAF 问题并检测          | - 错误类型总结- 修复建议整理                |
| 第 7 天 | 总结回顾 + 汇报输出             | - 各工具优劣对比图- 整理速查表、复盘一周学习                 | - 工具对比表- 1 页 PPT 或 markdown 用于分享 |

### 📆 第二阶段学习安排

#### 目标

1. **深入理解开源工具的实现原理**

   - 阅读 `perf`、`valgrind`、`asan`、`gperftools` 的部分源码，了解其核心设计和工作机制。

   - 对比它们在**实现方法**、**性能开销**、**检测能力**等方面的差异。

2. **分析工具的适用场景与局限性**

   - 总结每个工具适合用在什么场景，解决什么问题，以及在哪些场景下不适用。

   - 举出典型案例或对比实验作为支撑材料。

3. **调研商业化闭源工具**

   - 查找并调研当前主流的闭源性能分析工具（如 Intel VTune Profiler、Google Cloud Profiler、Dynatrace、New Relic、AppDynamics 等），

   - 对比它们与开源工具的优势、不足、适用场景。

#### 建议的实践顺序

每日学习任务表（适合每天学习 2-3 小时）

| 日期  | 任务模块                        | 学习任务                                                     | 输出内容                                                     |
| ----- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Day 1 | `perf/`                         | 阅读 `perf` 的核心系统调用路径（`perf_event_open`）与 ring buffer 采样机制 | - 增加 `原理解析`：记录 syscall 过程、内核采样流程  - 绘制采样流程图（可用 Mermaid） |
| Day 2 | `perf/`                         | 阅读 `perf record/report` 具体实现流程，理解其数据如何生成与分析 | - 增加 `使用流程源码分析  - 总结事件类型、采样字段、report 展示机制 |
| Day 3 | `valgrind_asan/`                | 阅读 `valgrind` memcheck 的插桩机制源码（选取 memcheck/mc_main.c 入口等） | - 增加 `valgrind源码阅读`  - 理解 shadow memory 管理方式，记录关键函数 |
| Day 4 | `valgrind_asan/`                | 阅读 ASan 的编译期插桩原理（LLVM Sanitizer 工程）和运行时检测机制 | - 增加 `asan原理与源码分析`  - 总结与 valgrind 的核心实现区别 |
| Day 5 | `gperftools/`                   | 阅读 `gperftools` 中 Heap Profiler 实现，理解其对 malloc/free 的拦截与采样机制 | - 增加 `heap_profiler原理  - 总结采样精度/性能影响机制       |
| Day 6 | `gperftools/`                   | 阅读 CPU Profiler 如何 hook 函数、采样调用栈，理解与 perf 的异同 | - 增加 `cpu_profiler原理  - 完善与 perf 的对比文档           |
| Day 7 | `tools_comparison/`（新建目录） | 调研闭源商业工具（如 VTune、Dynatrace、New Relic）+ 整体对比分析 | - 新增 `tools_comparison/对比分析`：整理开源 vs 商业工具的优缺点表格  - 汇总所有工具适用场景、特长与不足 |
