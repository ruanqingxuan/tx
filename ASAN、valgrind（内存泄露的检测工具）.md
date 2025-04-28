# ASAN、valgrind（内存泄露的检测工具）

## 一、工具概览

### ASan

#### 什么是 AddressSanitizer？

[AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)（又名 ASan）是一款适用于 C/C++ 的内存错误检测器。

#### 主要检测什么问题？

ASan可以检测出程序中不合理的内存使用行为，主要包括以下行为：

| **错误行为**              | **简介**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| global buffer overflow    | 全局内存访问越界                                             |
| heap buffer overflow      | 堆内存访问越界[github.com](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleHeapOutOfBounds) |
| initialization order bugs | 全局变量初始化顺序异常全局变量初始化间存在依赖，导致实际运行时因初始化顺序问题导致的初始值异常 |
| memory leaks              | 内存泄漏在程序正常退出时输出报告                             |
| stack buffer overflow     | 栈内存访问越界                                               |
| use after free            | 访问已经释放的内存，在释放内存后仍然尝试访问此内存[AddressSanitizerExampleUseAfterFree · google/sanitizers Wiki](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleUseAfterFree) |
| use after return          | 访问生命周期结束的对象在函数退出后尝试访问函数内声明的局部变量 |
| user after scope          | 访问生命周期结束的对象在"{}"包起来的代码块外访问代码块内声明的局部变量 |

### Valgrind

#### 什么是 Valgrind（重点是 Memcheck）？

Valgrind 是一个用于构建动态分析工具的插桩框架。Valgrind 中的一些工具可以自动检测许多内存管理和线程错误，并详细分析您的程序。还可以使用 Valgrind 构建新的工具。

Valgrind 发行版目前包含七个生产级工具：一个内存错误检测器、两个线程错误检测器、一个缓存和分支预测分析器、一个调用图生成缓存和分支预测分析器，以及两个不同的堆分析器。其中最受欢迎的工具是 Memcheck。它可以检测 C 和 C++ 程序中常见的许多内存相关错误，这些错误可能导致程序崩溃和不可预测的行为。

#### 主要检测什么问题？

检测内存泄露，数组越界等一些程序中常见的错误。特别的，它的 memcheck 工具能够检测内存泄漏、未初始化内存的使用和非法的内存访问。

## 二、原理机制

### ASan工作机制

- 在编译时，ASan会替换malloc/free接口
- 在程序申请内存时，ASan会额外分配一部分内存来标识该内存的状态
- 在程序使用内存时，ASan会额外进行判断，确认该内存是否可以被访问，并在访问异常时输出错误信息
- 详细的工作原理官方文档：https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm

### Valgrind工作机制

- 动态二进制翻译（Dynamic Binary Translation）
- 在程序运行时监控所有内存操作

## 三、安装与使用方式

### ASan 使用

#### 安装Asan

在 Ubuntu 上，**ASan (AddressSanitizer)** **不需要单独安装**，因为它是 **GCC** 或 **Clang** 编译器自带的功能模块。你只需要确保你的 **GCC** 或 **Clang** 版本支持 ASan（一般 Ubuntu 18.04+ 默认都支持）。

#### 使用Asan

1. 使用要求：编译时加上 `-fsanitize=address -g`（`-fsanitize=address`：启用 AddressSanitizer，`-g`：生成调试符号，方便错误定位到具体源码行号）

   ```bash
   # eg
   g++ -fsanitize=address -g your_code.cpp -o your_program
   # -fno-omit-frame-pointer 让栈追溯信息更加友好
   ```

2. 编译后直接运行，出现非法内存访问、溢出、use-after-free 等时，ASan 会自动终止程序并输出详细错误栈信息。

   ```bash
   ./your_program
   ```

3. 进阶配置：可以通过环境变量控制 ASan 行为：

   - 显示更多详细信息：

     ```bash
     export ASAN_OPTIONS=verbosity=1
     ```

   - 打印出泄漏检测（需要加 `-fsanitize=leak`）：

     ```bash
     export ASAN_OPTIONS=detect_leaks=1
     ```

     ```bash
     # eg：
     export ASAN_OPTIONS=detect_leaks=1:abort_on_error=1
     ./your_program
     ```

     | **flag**                      | **含义**                                                     |
     | ----------------------------- | ------------------------------------------------------------ |
     | halt_on_error                 | 默认为1，ASan检测到错误后会中止程序设为0后，ASan检测到错误后不会中止程序 |
     | log_path                      | 设置错误信息的输出路径                                       |
     | detect_stack_use_after_return | 是否检测use after return错误默认为0，即关闭该检测功能        |
     | help=1                        | 输出所有支持的参数                                           |

- 编译加上 `-fsanitize=address -g`

  

更多ASAN_OPTIONS可以参考：https://github.com/google/sanitizers/wiki/AddressSanitizerFlags

### Valgrind 使用

#### 安装 Valgrind

```sh
sudo apt install valgrind 
```

![image-20250426221902277](https://qnwang.oss-cn-hangzhou.aliyuncs.com/internship/20250426221902305.png)

#### 使用valgrind

1. 使用要求：编译程序时使用调试信息选项 -g（使用编译程序以-g包含调试信息，以便 Memcheck 的错误消息包含准确的行号。-O0如果您可以忍受速度变慢，使用也是一个好主意。 -O1错误消息中的行号可能不准确，但通常情况下，在编译的代码上运行 Memcheck-O1效果很好，而且与直接运行相比，速度提升-O0非常显著。 -O2不建议使用及以上版本，因为 Memcheck 偶尔会报告实际上并不存在未初始化值的错误）。
2. 使用方法：使用 valgrind 运行程序：valgrind --leak-check=full ./your_program
3. 常用选项：
   - --leak-check=full：启用详细的内存泄漏检测。
   - --track-origins=yes：在内存未初始化时，报告其来源。
   - --log-file=filename：将 Valgrind 的输出写入文件 filename

## 四、检测能力比较

| 比较项         | ASan                            | Valgrind                          |
| -------------- | ------------------------------- | --------------------------------- |
| 检测内存越界   | ✅ 高效准确                      | ✅ 高效准确                        |
| 检测内存泄漏   | ✅（可选配 LeakSanitizer）       | ✅（默认内置）                     |
| 检测未初始化读 | ❌（需要另加 MSan）              | ✅                                 |
| 检测堆栈溢出   | ✅（有限支持）                   | ✅（通过 Memcheck 支持）           |
| 检测速度       | 🔥 极快（通常 2x-3x 原程序速度） | 🐢 较慢（通常 10x-50x 原程序速度） |
| 支持大程序     | ✅ 支持大内存程序                | ❌ 容易内存爆掉                    |
| 平台支持       | Linux、macOS、Windows (部分)    | Linux、macOS（Windows 支持差）    |

## 五、输出示例与解读

### ASan 错误输出示例

#### 验证代码

```c++
int main(int argc, char **argv) {
  int *array = new int[100];
  delete [] array;
  return array[argc];  // BOOM
}
// RUN: clang -O -g -fsanitize=address %t && ./a.out
```

#### 验证结果

```c++
=================================================================
==6254== ERROR: AddressSanitizer: heap-use-after-free on address 0x603e0001fc64 at pc 0x417f6a bp 0x7fff626b3250 sp 0x7fff626b3248
READ of size 4 at 0x603e0001fc64 thread T0
    #0 0x417f69 in main example_UseAfterFree.cc:5
    #1 0x7fae62b5076c (/lib/x86_64-linux-gnu/libc.so.6+0x2176c)
    #2 0x417e54 (a.out+0x417e54)
0x603e0001fc64 is located 4 bytes inside of 400-byte region [0x603e0001fc60,0x603e0001fdf0)
freed by thread T0 here:
    #0 0x40d4d2 in operator delete[](void*) /home/kcc/llvm/projects/compiler-rt/lib/asan/asan_new_delete.cc:61
    #1 0x417f2e in main example_UseAfterFree.cc:4
previously allocated by thread T0 here:
    #0 0x40d312 in operator new[](unsigned long) /home/kcc/llvm/projects/compiler-rt/lib/asan/asan_new_delete.cc:46
    #1 0x417f1e in main example_UseAfterFree.cc:3
Shadow bytes around the buggy address:
  0x1c07c0003f30: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x1c07c0003f40: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x1c07c0003f50: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x1c07c0003f60: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x1c07c0003f70: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
=>0x1c07c0003f80: fa fa fa fa fa fa fa fa fa fa fa fa[fd]fd fd fd
  0x1c07c0003f90: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x1c07c0003fa0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x1c07c0003fb0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fa fa
  0x1c07c0003fc0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x1c07c0003fd0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:     fa
  Heap righ redzone:     fb
  Freed Heap region:     fd
  Stack left redzone:    f1
  Stack mid redzone:     f2
  Stack right redzone:   f3
  Stack partial redzone: f4
  Stack after return:    f5
  Stack use after scope: f8
  Global redzone:        f9
  Global init order:     f6
  Poisoned by user:      f7
  ASan internal:         fe
==6254== ABORTING
```

#### 结果分析

```sh
1. 错误类型摘要
    # ASan 检测到错误类型是 heap-use-after-free，表示访问了已经释放的堆内存。
    ==6254== ERROR: AddressSanitizer: heap-use-after-free on address 0x603e0001fc64

2. 出错位置
    # 错误发生在 example_UseAfterFree.cc 文件的第5行 main 函数中。
    # 出错时正在读取（READ）4字节的数据。
    READ of size 4 at 0x603e0001fc64 thread T0
      #0 0x417f69 in main example_UseAfterFree.cc:5
      #1 0x7fae62b5076c (/lib/x86_64-linux-gnu/libc.so.6+0x2176c)
      #2 0x417e54 (a.out+0x417e54)

3. 内存分配与释放信息
    # 这块内存是在 example_UseAfterFree.cc 文件第3行通过 new[] 分配的。
    previously allocated by thread T0 here:
      #0 0x40d312 in operator new[](unsigned long)
      #1 0x417f1e in main example_UseAfterFree.cc:3

    # 这块内存已经在第4行通过 delete[] 被释放。
    freed by thread T0 here:
      #0 0x40d4d2 in operator delete[](void*)
      #1 0x417f2e in main example_UseAfterFree.cc:4

4. Shadow Memory 状态
    # 出错地址周围的 Shadow Memory 信息如下：
    Shadow bytes around the buggy address:
      0x1c07c0003f80: fa fa fa fa fa fa fa fa fa fa fa fa [fd]fd fd fd
    # 其中 fd 表示 "Freed Heap Region"，即已经释放的堆内存。
    # 说明正在访问一块已经释放的堆区域，因此发生了 heap-use-after-free 错误。

5. Shadow Byte Legend
    # Shadow Byte 各个标记含义：
    - 00: 正常可访问内存
    - fd: 已释放的堆内存（Freed Heap region）
    - fa: 堆红区（Heap redzone，用于保护越界）
```

更多例子在：https://github.com/google/sanitizers/wiki/addresssanitizer的introduction部分

### Valgrind 输出示例

#### 验证代码

```c++
#include <iostream>
void func(){
    int* p = new int(10); // 只申请不回收，leak
}
int main(){

    func();
    return 0;
}
// sh命令
g++ test.cpp -g
valgrind --leak-check=full ./a.out
```

#### 验证结果

```sh
[root:~/stutest]# valgrind --leak-check=full ./a.out 
==1536437== Memcheck, a memory error detector
==1536437== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==1536437== Using Valgrind-3.18.1 and LibVEX; rerun with -h for copyright info
==1536437== Command: ./a.out
==1536437== 
==1536437== 
==1536437== HEAP SUMMARY: # 堆摘要
==1536437==     in use at exit: 4 bytes in 1 blocks
==1536437==   total heap usage: 2 allocs, 1 frees, 72,708 bytes allocated
==1536437== 
==1536437== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1
==1536437==    at 0x48657B8: operator new(unsigned long) (in /usr/libexec/valgrind/vgpreload_memcheck-arm64-linux.so)
==1536437==    by 0x1088E3: func() (test.cpp:4)
==1536437==    by 0x108907: main (test.cpp:9)
==1536437== 
==1536437== LEAK SUMMARY: # 泄露摘要
==1536437==    definitely lost: 4 bytes in 1 blocks
==1536437==    indirectly lost: 0 bytes in 0 blocks
==1536437==      possibly lost: 0 bytes in 0 blocks
==1536437==    still reachable: 0 bytes in 0 blocks
==1536437==         suppressed: 0 bytes in 0 blocks
==1536437== 
==1536437== For lists of detected and suppressed errors, rerun with: -s
==1536437== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

#### 结果分析

```sh
1. 堆摘要：
	# 在程序结束时，有 4 字节的内存仍然在使用，这意味着有一块 4 字节的内存没有被释放。
    in use at exit: 4 bytes in 1 blocks
    # 程序总共进行了 2 次内存分配操作（allocs），但是只进行了 1 次内存释放操作（frees），总共分配了 72,708 字节的内存。
    total heap usage: 2 allocs, 1 frees, 72,708 bytes allocated
2. 泄漏摘要
	# Valgrind 确认有 4 字节内存泄漏，这次泄漏的详细信息如下：
	4 bytes in 1 blocks are definitely lost in loss record 1 of 1
	# 内存是在调用 operator new 时分配的，这通常表示你使用了 new 操作符分配了内存。
	at 0x486578B: operator new(unsigned long) (in /usr/libexec/valgrind/vgpreload_memcheck-arm64-linux.so)
	# func() 函数中进行了内存分配，这是导致内存泄漏的地方。
	by 0x1088E3: func() (test.cpp:4)
	# main 函数调用了 func()，导致了内存泄漏。
	by 0x108907: main (test.cpp:9)
3. 错误摘要
	# 总共检测到 1 个内存泄漏错误，且没有任何错误被抑制。
    ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

### 常见内存泄漏的场景

```c++
#include <iostream>
#include <cstring>

void memoryLeak1() {
    // 简单的内存泄漏，没有释放分配的内存
    int* leakyArray = new int[100];
    // 忘记释放 leakyArray
}

void memoryLeak2() {
    // 动态分配的内存覆盖了原先分配的内存，导致原内存泄漏
    char* leakyString = new char[25];
    strcpy(leakyString, "Initial allocation");
    
    // 重新分配，原内存未释放
    leakyString = new char[50];
    strcpy(leakyString, "Reallocation causes leak");

    delete[] leakyString; // 释放第二次分配的内存
}

void memoryLeak3() {
    // 部分内存泄漏，未释放结构体中的某些成员
    struct Node {
        int* value;
        Node* next;
    };

    Node* node = new Node;
    node->value = new int(10);
    node->next = nullptr;

    delete node; // 只释放了 node，没有释放 node->value
}

void memoryLeak4() {
    // 使用未初始化的指针
    int* uninitializedPtr;
    *uninitializedPtr = 42; // 未定义行为
}

int main() {
    memoryLeak1();
    memoryLeak2();
    memoryLeak3();
    memoryLeak4();

    std::cout << "Done testing memory leaks!" << std::endl;
    return 0;
}
```

## 六、常见问题与坑

### **ASan 常见问题**

- 与某些库冲突（如 glibc 的 hook）
- 需要匹配 libc 版本
- asan版本程序在Linux环境下运行时会额外申请20TB的虚拟内存
  - 需要确保/proc/sys/vm/overcommit_memory的值不为2
  - 这也可以作为检验ASan是否工作的标志
- asan工具不是万能的，必须要跑到有问题的代码才可以暴露出来

### **Valgrind 常见问题**

- 运行特别慢
- 不支持 AVX-512、SIMD 优化程序

## 七、实战应用场景建议

### 什么时候优先用 ASan？

- 开发阶段，快速找到内存错误
- 代码量很大的时候用Asan

### 什么时候优先用 Valgrind？

- 找难以发现的泄漏和未初始化读问题，做深入分析

## 八、总结

- 两者优缺点汇总
- 推荐搭配使用的方法（如开发中用 ASan，发布前用 Valgrind 细扫）

学习资料

- https://valgrind.org/
- https://hardcore.feishu.cn/docx/doxcnXfsINxeICDFXePBelN3p5f

