# gperftools（Google 提供的开源性能优化工具集）

## 一、工具概览

### 什么是 gperftools？

- gperftools 是 Google 提供的开源性能优化工具集，主要用于性能分析和内存管理。
- 包括 `tcmalloc`（高效的内存分配器）和 `cpu profiler`（CPU 性能分析器）。

### 主要功能

- **tcmalloc**：优化的内存分配器，替代标准的 `malloc`。
- **cpu profiler**：通过性能采样记录 **CPU 使用情况**，生成火焰图（flame graph）。
- **heap profiler**：分析程序的堆内存使用，检测内存泄漏和过度使用。

## 二、安装与配置

### **安装方式**

- 使用 `apt`（Ubuntu）：

  ```bash
  sudo apt-get install google-perftools libgoogle-perftools-dev
  ```

- 使用 `brew`（macOS）：

  ```bash
  brew install gperftools
  ```

- 编译源码：

  ```bash
  git clone https://github.com/gperftools/gperftools.git
  cd gperftools
  ./configure
  make
  sudo make install
  ```

### **配置环境**

- 环境变量设置：

  ```bash
  export LD_PRELOAD=/usr/local/lib/libtcmalloc.so
  ```

## 三、常用工具与命令

### 1. tcmalloc（内存分配器）

- 作用：提高内存分配与释放效率，减少内存碎片。
- 使用：
  - 在应用中通过设置 `LD_PRELOAD` 来加载 `libtcmalloc`。
  - 用 `gperftools` 编译的程序会自动使用 `tcmalloc`，无须修改代码。

### 2. CPU Profiler（CPU 性能分析）

#### 启用

- 编译时使用 `-lprofiler` 链接 profiler 库。

  ```bash
  # LD_PRELOAD 设置加载libprofiler.so
  # CPUPROFILE 设置采样文件
  # CPUPROFILE_FREQUENCY 设置采样频率，默认100
  # 强行结束不能获得采样文件 CPUPROFILESIGNAL设置采样开关信号，关闭采样只需要killall -12 perftest
  LD_PRELOAD=/usr/local/lib/libprofiler.so CPUPROFILE=data.prof CPUPROFILE_FREQUENCY=555 CPUPROFILESIGNAL=12 ./perftest --benchmark_min_time=10000x 
  ```

- 在代码中启动/停止 profiling：

  ```c++
  // 精确控制代码采样的区域
  #include <gperftools/profiler.h>
  ProfilerStart("data.prof");
  // 你的代码
  ProfilerStop();
  ```

#### 分析

- 使用 `pprof` 或`google-pprof`工具解析生成的 `data.prof` 文件：

  ```bash
  pprof --text ./your_program data.prof
  ```

  ![image-20250429163444432](https://qnwang.oss-cn-hangzhou.aliyuncs.com/internship/20250429163444540.png)

  ```bash
  # 每一列含义分析，以红框为例
  # 函数中的样本数	函数中样本百分比	当前打印的样本百分比	函数以及其调用函数中的样本数量	函数以及其调用子函数中的样本百分比	函数名
  ```

- 还可以加入`--stackes`选项打印调用栈

  ```bash
  pprof --text --stacks ./your_program data.prof
  ```

  ![image-20250429164022498](https://qnwang.oss-cn-hangzhou.aliyuncs.com/internship/20250429164022560.png)

- 发现这样不够直观，可以用`--svg`生成调用图更加直观

  ```bash
  pprof --svg ./your_program ./data.prof > svgout.svg
  ```

  ![image-20250429164315976](https://qnwang.oss-cn-hangzhou.aliyuncs.com/internship/20250429164316052.png)

  ```bash
  # 结果分析
  # 第一行类名
  # 第二行函数名
  # 本函数执行执行的时间（直接执行的时间）
  # 本函数调用函数执行的时间（调用其他函数的时间）
  ```

- 可以生成火焰图：

  ```bash
  pprof --flame ./your_program data.prof
  ```

### 3. Heap Profiler（堆内存分析）

- 启用：

  - 编译时使用 `-lprofiler`。

  - 运行时启用 heap profiler：

    ```cpp
    #include <gperftools/heap-profiler.h>
    HeapProfilerStart("heap_profile");
    ```

- 分析：

  - 查看生成的 `heap_profile` 文件，定位内存泄漏、内存过度分配等问题。

### 4. Leak Detector（内存泄漏检测）

- 自动检测：当启用 `tcmalloc` 时，`gperftools` 自动跟踪内存分配，报告泄漏。

- 例子：

  ```bash
  export HEAPPROFILE=heap_profile
  ./your_program
  ```

### 5. 常用命令总结

```bash
# 第一个是用源码编译的，第二个是用apt下载的
pprof --help
google-pprof --help
```

| **Options**               | **功能 (NAME)**                          |
| ------------------------- | ---------------------------------------- |
| `--cum`                   | 按累计数据排序                           |
| `--base=<base>`           | 在显示前从配置文件中减去 `<base>`        |
| **Reporting Granularity** | **报告粒度**                             |
| `--addresses`             | 按地址级别报告                           |
| `--lines`                 | 按源代码行级别报告                       |
| `--functions`             | 按函数级别报告（默认）                   |
| `--files`                 | 按源代码文件级别报告                     |
| **Output Type**           | **输出类型**                             |
| `--text`                  | 生成文本报告（默认）                     |
| `--gv`                    | 生成 Postscript 并显示                   |
| `--list=<regexp>`         | 生成匹配常规表达式的源代码列表           |
| `--disasm=<regexp>`       | 生成匹配常规表达式的指令反汇编代码       |
| `--dot`                   | 生成 DOT 格式的文件并输出到标准输出      |
| `--ps`                    | 生成 Postscript 格式文件输出到标准输出   |
| `--pdf`                   | 生成 PDF 格式文件输出到标准输出          |
| `--gif`                   | 生成 GIF 格式文件输出到标准输出          |
| **Heap-Profile Options**  | **堆分析选项**                           |
| `--inuse_space`           | 显示当前使用的内存（以MB为单位，默认）   |
| `--inuse_objects`         | 显示当前使用的对象数量                   |
| `--alloc_space`           | 显示已分配的内存（以MB为单位）           |
| `--alloc_objects`         | 显示已分配的对象数量                     |
| `--show_bytes`            | 显示内存使用量（以字节为单位）           |
| `--drop_negative`         | 忽略负值差异                             |
| **Call-graph Options**    | **调用图选项**                           |
| `--nodecount=<n>`         | 显示最多 `<n>` 个节点（默认80）          |
| `--nodefraction=<f>`      | 隐藏占总数小于 `<f>` 的节点（默认0.005） |
| `--edgefraction=<f>`      | 隐藏占总数小于 `<f>` 的边（默认0.001）   |
| `--focus=<regexp>`        | 只聚焦匹配 `<regexp>` 的节点             |
| `--ignore=<regexp>`       | 忽略匹配 `<regexp>` 的节点               |
| `--scale=<n>`             | 设置 GV 渲染的缩放比例（默认0）          |

## 四、工具集成与调优

### **集成与调试**

- 配合 **perf** 或 **Valgrind** 使用，进行内存、CPU 性能多角度分析。
- 与 **gdb** 配合，调试性能问题和内存泄漏。

### **性能优化**

- 优化内存使用，减少内存分配开销。
- 通过 `CPU profiler` 定位 CPU 瓶颈函数，优化代码热点。
- 使用 `Heap Profiler` 分析程序内存消耗，减少不必要的内存分配。

## 五、实战案例与分析

### **案例 1：使用 tcmalloc 优化内存**

- 比较 `malloc` 和 `tcmalloc` 在高频次分配下的性能差异。
- 分析内存碎片、内存占用的变化。

### **案例 2：使用 CPU Profiler 分析热点**

- 编写一个包含复杂运算的程序，用 `CPU Profiler` 查找 CPU 使用的瓶颈函数。
- 使用火焰图进行可视化分析，优化计算热点。

### **案例 3：内存泄漏定位与修复**

- 故意写一个内存泄漏示例，使用 `Heap Profiler` 进行检测，定位泄漏源并修复。

## 六、常见问题与坑

- **性能开销**：启用 `gperftools` 后会带来一定的性能开销，尤其是在高负载时。要注意选择合适的 profiling 级别。
- **与其他库的兼容性**：某些情况下，`tcmalloc` 与第三方库的内存分配可能会发生冲突，需要根据具体情况选择关闭。
- **内存泄漏的误报**：`Heap Profiler` 有时会报告不准确的内存泄漏，需要结合代码逐步排查。

## 七、总结与心得

- **优点**：
  - 高效的内存管理，适合高性能应用。
  - 支持详细的性能分析，帮助定位 CPU 和内存瓶颈。
- **缺点**：
  - 使用时有性能开销，建议仅在开发和调试阶段启用。
  - 配置较为复杂，需要一定的学习曲线。
- **实用性**：
  - 在高性能要求的 C++ 项目中使用 `gperftools`，可以大幅提升内存管理效率和 CPU 性能。