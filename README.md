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

#### ✅ 第 1 天：性能分析基础 + perf 入门

- 阅读：《Linux 性能工具概览》 or perf 官方文档基础部分
- 实操：用 `perf top` / `perf record + report` 跑一个 C++ 示例程序
- 笔记：perf 基本命令 + 输出样例解释 + 整理初步结果图

#### ✅ 第 2 天：perf 高级使用 + 实战分析

- 实操：用 perf 分析一个 CPU 密集型程序（如排序、多线程测试）
- 笔记：整理 CPU profiling 分析流程、热点定位过程

#### ✅ 第 3 天：valgrind 入门（重点：memcheck）

- 学习：valgrind 的原理与主要子工具（memcheck, cachegrind）
- 实操：写几个小 demo 程序进行内存泄漏分析
- 笔记：valgrind 输出格式 + 常见内存错误总结

#### ✅ 第 4 天：valgrind 实战 + cache 分析

- 实操：使用 cachegrind 观察 cache miss
- 拓展：了解 massif 工具进行 heap 使用分析
- 笔记：valgrind 工具组合对照表 + 用法记录

#### ✅ 第 5 天：gperftools + pprof 入门

- 学习：gperftools 安装 + 使用 `pprof` 生成火焰图
- 实操：配置一个小程序使用 tcmalloc + CPU profiler
- 笔记：pprof 输出解读 + 火焰图解读示例

#### ✅ 第 6 天：AddressSanitizer（ASan）实战

- 学习：ASan 编译参数原理
- 实操：故意制造内存越界 + UAF 问题并用 ASan 检出
- 笔记：错误类型分类 + 对应修复建议整理

#### ✅ 第 7 天：总结回顾 + 整理一份汇报/文档

- 回顾各工具对比图
- 输出一份《C++ 性能分析工具速查表》
- 准备 1 页 PPT 或一篇 markdown 用于分享

