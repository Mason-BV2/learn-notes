# Linux 内核工程师系统学习计划（高通方向）

## 目标

- **本计划目标**：从0基础出发，系统掌握 Linux 内核工程师核心技能，达到高通 Senior Engineer 岗位要求
- **最终交付能力**：能独立开发 ARM SoC 上的 Linux 内核驱动，理解内存管理/虚拟化，熟悉 Android 平台内核层，具备 upstream patch 提交能力
- **为什么重要**：高通 Linux 内核工程师是嵌入式/移动/汽车芯片领域的核心岗位，技术壁垒高、薪资竞争力强、职业天花板高

## 当前起点

- **已有基础**：有高通内核工作经验（patch 验证、驱动调试），但系统知识体系不完整，自评"0经验"
- **已有工具**：本地有高通内核代码、编译环境、刷机设备
- **当前卡点**：知识碎片化，缺乏从底层原理到上层应用的完整体系
- **每周可投入时间**：建议 10-15 小时（工作日 1-2h/天 + 周末集中）

---

## 主线能力点（6大模块）

| 模块 | 核心能力 | 优先级 |
|------|---------|--------|
| M1 | C语言精通 + Linux 系统编程基础 | ★★★★★ |
| M2 | Linux 内核架构 + 驱动开发 | ★★★★★ |
| M3 | ARM 体系结构 + 嵌入式底层 | ★★★★☆ |
| M4 | 内存管理 + 进程调度 + 同步机制 | ★★★★☆ |
| M5 | Android 内核层（GKI/HAL/AOSP） | ★★★☆☆ |
| M6 | 虚拟化/Hypervisor + Rust + 开源社区 | ★★★☆☆ |

---

## 学习路线总览（18个月）

```
Phase 1（月1-2）：C语言 + Linux 基础夯实
Phase 2（月3-5）：Linux 内核架构 + 驱动开发入门
Phase 3（月6-8）：ARM 体系结构 + 嵌入式驱动实战
Phase 4（月9-11）：内核子系统深入（内存/调度/同步）
Phase 5（月12-14）：Android 内核层 + 高通平台专项
Phase 6（月15-18）：虚拟化/Hypervisor + Rust + 开源贡献
```

---

## Phase 1：C语言精通 + Linux 系统编程基础（月1-2）

### 目标
能用 C 写出健壮的系统级代码，理解 Linux 用户态编程模型，为内核学习打地基。

### 权威资料

**必读书籍**：
- 《C程序设计语言》（K&R，第2版）— C语言之父所著，简洁经典
- 《C和指针》（Kenneth Reek）— 指针专题深入讲解，适合配合K&R补充
- 《UNIX环境高级编程》（APUE，Stevens）— 进程/文件/信号/IPC 圣经
- 《Linux/UNIX系统编程手册》（Michael Kerrisk）— Linux man-page 维护者所著，比APUE更新

**推荐书籍（备选）**：
- 《C专家编程》（Peter van der Linden）— C语言陷阱与最佳实践，跨过"入门→熟练"的坎
- 《C陷阱与缺陷》（Andrew Koenig）— 常见C语言错误，写内核前必读
- 《鸟哥的Linux私房菜：基础学习篇》— Linux 入门最佳中文书
- 《Unix/Linux编程实践教程》（Bruce Molay）— 通过项目学系统编程

**在线资源**：
- Linux man pages（`man 2 xxx`）— 系统调用第一手资料
- [Linux kernel 链表实现 list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h) — 内核经典数据结构

### Week 1-2：C语言核心
- **学什么**：指针、内存布局（栈/堆/BSS/data/text）、结构体、位操作、函数指针、宏
- **练习**：实现链表、哈希表、环形缓冲区（纯C，无STL）
- **小产出**：写一个通用链表库（`list.h` 风格，参考 Linux kernel 的 `include/linux/list.h`）
- **验收标准**：能解释指针与数组的区别、能手写内存对齐计算、能用 `valgrind` 检查内存泄漏

### Week 3-4：Linux 系统调用 + 进程/文件
- **学什么**：`fork/exec/wait`、文件描述符、`mmap`、信号、管道、`epoll`
- **练习**：写一个简单的 shell（支持管道和重定向）
- **小产出**：实现 `myshell`，支持 `ls | grep xxx > file.txt`
- **验收标准**：能解释 `fork` 后的地址空间复制、能用 `strace` 追踪系统调用

### Week 5-6：多线程 + 同步原语
- **学什么**：`pthread`、mutex、条件变量、信号量、原子操作
- **练习**：实现生产者-消费者、读写锁
- **小产出**：写一个线程池库
- **验收标准**：能解释死锁的4个必要条件、能用 `helgrind` 检测竞态

### Week 7-8：调试工具链
- **学什么**：`gdb`（断点/watchpoint/backtrace）、`valgrind`、`perf`、`objdump`、`readelf`
- **练习**：用 `gdb` 调试一个有 bug 的多线程程序
- **小产出**：整理一份《内核工程师常用调试命令速查表》
- **验收标准**：能用 `gdb` 在没有源码的情况下分析 core dump

### Phase 1 推荐资源汇总

**GitHub 仓库**：
- [sysprog21/lkmpg](https://github.com/sysprog21/lkmpg) ⭐ — 内核模块编程指南，Phase 1 了解即可，Phase 2 重点用
- [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat) — QEMU + GDB 内核实验环境
- [shizhengLi/linux-kernel-learning](https://github.com/shizhengLi/linux-kernel-learning) ⭐ — 中文，7阶段内核学习，55+分析文档
- [mattpocock/skills](https://github.com/mattpocock/skills) ⭐124K — Agent Skill 工程化参考，可用于将自己的方法论固化为 AI 可执行的工作流
- [study/ai-landscape-2026.md](./ai-landscape-2026.md) — 2026年 AI 辅助内核开发现状全景（面试/学习方向参考）

**开发板（按需购买）**：
- Raspberry Pi 4/5（¥300-500）— 学习 Linux 操作、命令行、简单驱动

**开发环境**：
- Ubuntu 22.04/24.04 LTS（WSL2 或虚拟机即可）
- VS Code + C/C++ 扩展
- `gcc`、`make`、`gdb`、`valgrind`

### Phase 1 里程碑
> ✅ 能独立用 C 实现数据结构、能用 gdb/strace/valgrind 调试程序、理解 Linux 进程/文件/内存模型

---

## Phase 2：Linux 内核架构 + 驱动开发入门（月3-5）

### 目标
理解内核整体架构，能写出可编译、可加载的字符设备驱动。

### 权威资料

**必读书籍**：
- 《Linux Device Drivers》第3版（LDD3，O'Reilly）⭐ — 驱动开发圣经，免费在线 https://lwn.net/Kernel/LDD3/
- 《Linux Kernel Development》第3版（Robert Love）⭐ — 内核入门首选，200页覆盖核心子系统
- 《Linux设备驱动开发详解》（宋宝华）— 中文，配合 LDD3 学习

**推荐书籍（备选）**：
- 《深入理解Linux内核》（ULK，Bovet & Cesati）— 深入每个子系统的实现
- 《Linux内核源代码情景分析》（毛德操）— 中文，逐行分析内核源码
- 《精通Linux设备驱动程序开发》（Sreekrishnan）— 从实践角度讲解

**在线资源**：
- kernel.org 官方文档：`Documentation/` 目录
- [Bootlin 内核培训材料](https://bootlin.com/training/kernel/) ⭐ — 免费 PDF，可直接当课程使用
- [Elixir Bootlin](https://elixir.bootlin.com/linux/latest/source) — 在线内核源码交叉引用
- [Linux 内核模块编程指南（在线版）](https://sysprog21.github.io/lkmpg/) — 更新至 6.x 内核

### Week 9-10：内核架构概览
- **学什么**：内核空间 vs 用户空间、系统调用机制、内核模块（`module_init/exit`）、`printk`、`/proc` 文件系统
- **练习**：写 Hello World 内核模块，在真实设备上加载
- **小产出**：在高通设备上加载自己写的模块，`dmesg` 看到输出
- **验收标准**：能解释为什么内核不能用 `printf`、能解释模块加载的完整流程

### Week 11-12：字符设备驱动
- **学什么**：`cdev`、`file_operations`（open/read/write/ioctl/release）、设备号（major/minor）、`udev`
- **练习**：写一个虚拟字符设备（内存缓冲区），支持 read/write/ioctl
- **小产出**：`/dev/mydev` 可以被用户态程序读写
- **验收标准**：能解释 `copy_to_user/copy_from_user` 为什么必须用、能解释设备号分配机制

### Week 13-14：中断 + 定时器
- **学什么**：中断处理（`request_irq`）、上半部/下半部（tasklet/workqueue/softirq）、内核定时器（`hrtimer`）
- **练习**：写一个基于中断的驱动（可用虚拟中断模拟）
- **小产出**：实现一个周期性打印时间戳的内核定时器模块
- **验收标准**：能解释为什么中断上下文不能睡眠、能区分 tasklet 和 workqueue 的使用场景

### Week 15-16：平台驱动 + 设备树
- **学什么**：`platform_driver`、`platform_device`、设备树（DTS/DTB）、`of_match_table`、`probe/remove`
- **练习**：写一个 platform driver，从设备树读取配置
- **小产出**：在高通设备树中添加一个虚拟节点，驱动成功 probe
- **验收标准**：能解释设备树如何替代 board file、能手写一个最小 DTS 节点

### Week 17-20：I2C/SPI/GPIO 驱动
- **学什么**：I2C 子系统（`i2c_driver`/`i2c_client`）、SPI 子系统、GPIO 子系统（`gpiod_*`）、Regmap
- **练习**：写一个 I2C 传感器驱动（可用 I2C 模拟器）
- **小产出**：驱动能正确读取 I2C 设备寄存器，数据通过 sysfs 暴露
- **验收标准**：能解释 I2C 总线仲裁机制、能用 `i2cdetect` 和 `i2cdump` 调试

### Phase 2 推荐资源汇总

**GitHub 仓库**：
- [sysprog21/lkmpg](https://github.com/sysprog21/lkmpg) ⭐ — 内核模块编程指南，大量可运行示例
- [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat) ⭐ — QEMU + GDB 一键式内核调试
- [shizhengLi/linux-kernel-learning](https://github.com/shizhengLi/linux-kernel-learning) ⭐ — 中文内核源码分析，55+ 深度文档
- [0xAX/linux-insides](https://github.com/0xAX/linux-insides) — 内核启动到 init 的完整流程
- [87170360/linux_kernel_wiki](https://github.com/87170360/linux_kernel_wiki) — 200+文章、100+论文、500+面试题
- [bootlin/training-materials](https://github.com/bootlin/training-materials) — Bootlin 专业培训 PDF + 实验

**开发板**：
- DragonBoard 410c（¥500-800）— 高通入门开发板，主线 Linux 支持
- DragonBoard 820c（¥800-1200）— 性能更强

**实验环境搭建**：
参考 `00-QEMU环境搭建.md`：BusyBox + rootfs + QEMU 启动 ARM64 内核

### Phase 2 里程碑
> ✅ 能独立写字符设备/平台驱动、理解中断机制、能读写设备树、能在真实设备上验证驱动

---

## Phase 3：ARM 体系结构 + 嵌入式驱动实战（月6-8）

### 目标
深入理解 ARM64 架构，掌握 MMU/Cache/异常处理，能调试底层硬件问题。

### 权威资料

**必读书籍**：
- ARM Architecture Reference Manual（ARMv8-A）— ARM 官方手册（免费注册下载）
- 《ARM System Developer's Guide》（Sloss）— ARM 体系结构经典
- 《ARM64架构与编程》（杜春雷）— ARM64指令集+异常处理+GIC，中文

**推荐书籍**：
- 《嵌入式Linux系统开发教程》（华清远见）— 从头搭建嵌入式 Linux
- 《精通Yocto项目》（Richard Purdie）— Yocto 构建系统，高通 BSP 基于此

**在线资源**：
- [Bootlin ARM 培训材料](https://bootlin.com/training/arm/) — 免费的 ARM 培训 PDF
- [ARM 官方文档中心](https://developer.arm.com/documentation/) — 所有 ARM 手册
- [Linaro 资源](https://www.linaro.org/) — ARM 生态开源协作

### Week 21-24：ARM64 核心架构
- **学什么**：寄存器（通用/系统寄存器）、异常级别（EL0-EL3）、异常向量表、AArch64 指令集基础
- **练习**：读懂 Linux 内核的 `arch/arm64/kernel/entry.S`
- **小产出**：写一篇《ARM64 异常处理流程》笔记，画出从异常触发到内核处理的完整流程图
- **验收标准**：能解释 EL0/EL1/EL2/EL3 的区别、能解释 `ESR_EL1` 寄存器的作用

### Week 25-28：MMU + Cache + TLB
- **学什么**：虚拟地址/物理地址转换、页表结构（4级页表）、TLB 刷新、Cache 一致性（MESI）、`__flush_dcache_area`
- **练习**：在内核中手动操作页表（参考 `arch/arm64/mm/`）
- **小产出**：写一篇《ARM64 MMU 地址翻译详解》，包含页表 walk 的每一步
- **验收标准**：能手算一个虚拟地址的页表 walk 过程、能解释 Cache 别名问题

### Week 29-32：DMA + IOMMU
- **学什么**：DMA 一致性内存、`dma_alloc_coherent`、`dma_map_single`、IOMMU 原理、SMMU（ARM 的 IOMMU）
- **练习**：写一个使用 DMA 的驱动（可用 virt 平台模拟）
- **小产出**：实现一个 DMA 传输的字符设备驱动
- **验收标准**：能解释为什么 DMA 需要物理地址、能解释 IOMMU 的安全作用

### Phase 3 推荐资源汇总

**GitHub 仓库**：
- [sdm660-mainline/linux](https://github.com/sdm660-mainline/linux) — 高通 SDM660 主线移植，学习 DTS 编写
- [Linaro/website](https://github.com/Linaro/website) — 高通设备启动主线 Linux 教程
- [torvalds/linux](https://github.com/torvalds/linux) `arch/arm64/` — ARM64 架构代码

**开发板**：
- DragonBoard 845c（¥1000-2000）— 主流高通开发平台，Linaro 支持
- Qualcomm RB3（¥1500-2500）— 机器人/嵌入式平台

**调试工具**：
- `gdb` + `kgdb` — 内核调试
- `objdump` / `readelf` — 二进制分析
- `sparse` — 静态分析

### Phase 3 里程碑
> ✅ 能读懂 ARM64 汇编、理解 MMU/Cache/DMA 机制、能调试硬件相关的内核 panic

---

## Phase 4：内核子系统深入（月9-11）

### 目标
深入掌握内存管理、进程调度、内核同步三大核心子系统。

### 权威资料

**必读书籍**：
- 《Understanding the Linux Kernel》第3版（Bovet & Cesati）— 内核子系统圣经
- 《Linux Kernel Development》第3版（Robert Love）— 再读一遍，这次深入理解
- 《深入Linux内核架构》（Wolfgang Mauerer）— 从 2.6 看架构设计

**推荐书籍**：
- 《奔跑吧Linux内核》（张天飞）— 中文，内核实验+分析
- 《Linux内核观测技术BPF》（David Calavera）— eBPF 可观测性

**在线资源**：
- kernel.org 源码：`mm/`、`kernel/sched/`、`kernel/locking/`
- [ftrace 官方文档](https://www.kernel.org/doc/html/latest/trace/ftrace.html) — 内核函数追踪
- [perf wiki](https://perf.wiki.kernel.org/) — 性能分析

### Week 33-36：内存管理子系统
- **学什么**：物理内存管理（Buddy System、Slab/Slub 分配器）、虚拟内存（VMA、`mmap`、缺页异常）、内存回收（LRU、kswapd）、OOM Killer
- **练习**：用 `/proc/meminfo`、`/proc/slabinfo`、`vmstat` 分析内存状态
- **小产出**：写一个内核模块，分配/释放不同大小的内存，观察 Buddy/Slab 行为
- **验收标准**：能解释 `kmalloc` vs `vmalloc` 的区别、能分析一个 OOM 的 dmesg log

### Week 37-40：进程调度
- **学什么**：CFS 调度器（红黑树、vruntime）、实时调度（FIFO/RR）、调度类、负载均衡、`cgroup` CPU 子系统
- **练习**：用 `perf sched` 分析调度延迟
- **小产出**：写一篇《CFS 调度器工作原理》，包含 `pick_next_task` 的调用链
- **验收标准**：能解释为什么 CFS 用红黑树、能用 `chrt` 设置实时优先级并观察效果

### Week 41-44：内核同步机制
- **学什么**：自旋锁（spinlock）、互斥锁（mutex）、RCU（Read-Copy-Update）、原子操作、内存屏障（memory barrier）、per-CPU 变量
- **练习**：分析一个内核死锁场景（用 `lockdep`）
- **小产出**：写一个使用 RCU 的内核模块，对比 spinlock 和 RCU 的性能差异
- **验收标准**：能解释 RCU 的核心思想、能解释为什么 `spin_lock_irqsave` 需要保存 flags

### Phase 4 推荐资源汇总

**GitHub 仓库**：
- [brendangregg/perf-tools](https://github.com/brendangregg/perf-tools) — perf 性能分析脚本集
- [iovisor/bcc](https://github.com/iovisor/bcc) — eBPF 工具集（execsnoop/opensnoop 等）
- [torvalds/linux](https://github.com/torvalds/linux) `tools/` — 内核自带工具（perf、objtool）

**源码浏览工具**：
- [Elixir Bootlin](https://elixir.bootlin.com/linux/latest/source) — 在线源码交叉引用
- `ctags` / `cscope` — 本地源码跳转

**调试工具**：
- `lockdep` — 死锁检测
- `kasan` / `ubsan` — 内存错误检测
- `crash` — 内核 dump 分析
- `ftrace` / `trace-cmd` — 函数追踪

### Phase 4 里程碑
> ✅ 能分析内存/调度/锁相关的内核 bug、能用 lockdep/perf/ftrace 定位问题

---

## Phase 5：Android 内核层 + 高通平台专项（月12-14）

### 目标
掌握 Android 内核特有机制，熟悉高通 Snapdragon 平台驱动生态。

### 权威资料

**官方文档**：
- [Android 内核官方文档](https://source.android.com/docs/core/architecture/kernel) — AOSP 内核架构
- [CodeLinaro](https://git.codelinaro.org/) — 高通开源 Linux/Android 内核官方 Git
- [GKI 规范文档](https://source.android.com/docs/core/architecture/kernel/generic-kernel-image) — Generic Kernel Image
- [linux-arm-msm 邮件列表](https://lore.kernel.org/linux-arm-msm/) — 高通内核开发讨论

**高通平台参考**：
- `arch/arm64/boot/dts/qcom/` — 高通设备树
- `drivers/clk/qcom/` — 高通时钟驱动
- `drivers/soc/qcom/` — 高通 SoC 驱动
- `drivers/iommu/arm/arm-smmu/` — SMMU 驱动（高通平台核心）

**推荐书籍**：
- 《精通Yocto项目》— 高通 BSP 基于 Yocto 构建

### Week 45-48：Android 内核特有机制
- **学什么**：Binder IPC 驱动、Ashmem/ION 内存、Wakelocks、Low Memory Killer（LMK）、Android Logger
- **练习**：阅读 `drivers/android/binder.c` 核心路径
- **小产出**：写一篇《Binder 驱动工作原理》，画出一次 IPC 调用的完整内核路径
- **验收标准**：能解释 Binder 为什么比 Socket 快、能解释 ION 的内存共享机制

### Week 49-52：GKI + HAL + VNDK
- **学什么**：GKI 架构（稳定 ABI、KMI）、HAL 接口（HIDL/AIDL）、VNDK、`vendor_hook`（eBPF/tracepoint）
- **练习**：在 GKI 约束下修改一个驱动，确保不破坏 KMI
- **小产出**：整理《GKI 约束下驱动开发注意事项》checklist
- **验收标准**：能解释为什么 GKI 要求稳定 KMI、能用 `abi_gki_aarch64` 检查 ABI 变化

### Week 53-56：高通平台专项
- **学什么**：高通 PMIC（电源管理）、Regulator 框架、Clock 框架、Thermal 管理、QCOM RPM/RPMH
- **练习**：在高通设备上添加一个 regulator consumer 驱动
- **小产出**：写一篇《高通 PMIC 驱动架构》笔记
- **验收标准**：能解释 `clk_prepare_enable` 的完整调用链、能用 `debugfs` 查看 clock 树

### Phase 5 推荐资源汇总

**高通平台资源**：
- [CodeLinaro 高通内核](https://git.codelinaro.org/) — `msm-5.4`、`msm-5.15`、`msm-6.1` 分支
- [qualcomm-linux/meta-qcom-hwe](https://github.com/qualcomm-linux/meta-qcom-hwe) — 高通官方 Yocto BSP 层
- [Linaro Wiki](https://wiki.linaro.org/) — 高通设备主线化
- [Qualcomm Developer Network](https://developer.qualcomm.com/) — SDK、工具下载
- [sdm660-mainline/linux](https://github.com/sdm660-mainline/linux) — 高通 SoC 主线移植参考

**高通内核源码关键目录**：
| 目录 | 内容 |
|------|------|
| `arch/arm64/boot/dts/qcom/` | 高通 SoC 设备树 |
| `drivers/clk/qcom/` | 时钟控制器（GCC） |
| `drivers/pinctrl/qcom/` | GPIO 引脚复用（TLMM） |
| `drivers/soc/qcom/` | SoC 级驱动（RPM/SMEM/SMD） |
| `drivers/iommu/arm/arm-smmu/` | SMMU 驱动 |
| `drivers/gpu/drm/msm/` | Adreno GPU 驱动 |
| `drivers/media/platform/qcom/` | Camera 驱动 |

**高通平台专用工具**：
| 工具 | 用途 |
|------|------|
| QXDM | Qualcomm Modem/RF 调试（Windows） |
| QPA (Qualcomm Performance Analyzer) | CPU/GPU/Memory 性能监控 |
| JTAG (Trace32) | 硬件级调试 |
| ADB / Fastboot | Android 设备调试 |

**开发板**：
- DragonBoard 845c（¥1000-2000）— 主流高通平台
- Qualcomm RB5（¥3000+）— AI/ML 推理，Hexagon DSP
- Rubik Pi 3（¥2000+）— QCS6490 IoT 专用

### Phase 5 里程碑
> ✅ 能在高通 Android 设备上独立开发/调试驱动、理解 GKI 约束、熟悉高通平台子系统

---

## Phase 6：虚拟化/Hypervisor + Rust + 开源贡献（月15-18）

### 目标
掌握虚拟化技术，学会 Rust 内核编程，完成第一个 upstream patch 提交。

### 权威资料

**虚拟化**：
- 《Virtual Machines》（Smith & Nair）— 虚拟化基础理论
- [KVM 官方文档](https://www.linux-kvm.org/) — Linux KVM
- ARM 虚拟化扩展：VHE（EL2）、NVHE、Stage-2 页表

**Rust**：
- [Rust 官方 Book](https://doc.rust-lang.org/book/) — Rust 入门
- [Rust for Linux](https://rust-for-linux.com/) — 内核 Rust 编程
- 《Rust 程序设计》— 深入理解所有权/借用/生命周期

**开源社区**：
- [Linux 内核邮件列表](https://lore.kernel.org/) — 所有 patch 讨论
- [checkpatch.pl](https://github.com/torvalds/linux/blob/master/scripts/checkpatch.pl) — 内核代码风格检查
- [kernel.org 补丁提交指南](https://www.kernel.org/doc/html/latest/process/submitting-patches.html)

### Week 57-60：虚拟化基础 + KVM
- **学什么**：虚拟化类型（Type1/Type2）、ARM 虚拟化扩展（VHE/NVHE）、KVM 架构、VFIO 设备直通
- **练习**：在 ARM 平台上运行 KVM 虚拟机，分析 VM exit 路径
- **小产出**：写一篇《ARM KVM 工作原理》，包含 VM entry/exit 的完整流程
- **验收标准**：能解释 Stage-2 页表的作用、能解释 VFIO 如何实现设备直通

### Week 61-64：Rust 内核编程
- **学什么**：Rust 基础（所有权/借用/生命周期）、Rust for Linux API（`kernel::` crate）、用 Rust 写内核模块
- **练习**：用 Rust 重写一个简单的字符设备驱动
- **小产出**：一个可加载的 Rust 内核模块，功能等价于 Phase 2 的 C 驱动
- **验收标准**：能解释 Rust 如何在内核中避免 use-after-free、能解释 `Pin<T>` 在内核中的用途

### Week 65-72：开源社区 + Upstream 贡献
- **学什么**：Linux 内核 patch 提交流程（`git format-patch`、`git send-email`）、`checkpatch.pl`、代码审查规范、Maintainer 体系
- **练习**：向 Linux 内核提交一个真实的 bug fix 或文档改进
- **小产出**：至少一个被 maintainer 回复的 patch（哪怕是 NACK 也算）
- **验收标准**：能独立完成从发现问题→写 patch→发邮件→回复 review 的完整流程

### Phase 6 推荐资源汇总

**虚拟化**：
- [KVM 官方文档](https://www.linux-kvm.org/)
- [ARM 虚拟化扩展文档](https://developer.arm.com/architectures/system-architectures/armv8-a-virtualization)
- QEMU + KVM 实验环境

**Rust**：
- [Rust 官方 Book](https://doc.rust-lang.org/book/)
- [Rust for Linux 官网](https://rust-for-linux.com/)
- [Rust for Linux GitHub](https://github.com/Rust-for-Linux/linux)

**开源社区**：
- [lore.kernel.org](https://lore.kernel.org/) — 内核邮件列表归档
- [Linux Kernel 补丁提交指南](https://www.kernel.org/doc/html/latest/process/submitting-patches.html)
- [linux-arm-msm 邮件列表](https://lore.kernel.org/linux-arm-msm/) — 高通相关讨论

**GitHub 仓库**：
- [torvalds/linux](https://github.com/torvalds/linux) — Linux 主线源码
- [rust-for-linux/linux](https://github.com/Rust-for-Linux/linux) — Rust for Linux 分支

**开发板**：
- Qualcomm RB5（¥3000+）— 适合虚拟化实验
- 已有高通设备继续使用

### Phase 6 里程碑
> ✅ 理解 ARM 虚拟化机制、能用 Rust 写内核模块、有 upstream 社区参与经历

---

## 核心工具链（贯穿全程）

| 工具 | 用途 | 何时学 |
|------|------|--------|
| `gdb` + `kgdb` | 内核调试 | Phase 1-2 |
| `ftrace` / `trace-cmd` | 函数追踪 | Phase 2 |
| `perf` | 性能分析 | Phase 2-4 |
| `lockdep` | 死锁检测 | Phase 4 |
| `kasan` / `ubsan` | 内存错误检测 | Phase 2 |
| `crash` | 内核 dump 分析 | Phase 2 |
| `checkpatch.pl` | 代码风格检查 | Phase 2 |
| `sparse` | 静态分析 | Phase 3 |
| `eBPF` / `bpftrace` | 动态追踪 | Phase 5 |

---

## 开发板选购指南（贯穿全程）

| 阶段 | 推荐开发板 | 参考价格 |
|------|-----------|----------|
| Phase 1-2 | Raspberry Pi 4/5 | ¥300-500 |
| Phase 2-3 | DragonBoard 410c/820c | ¥500-1200 |
| Phase 3-5 | DragonBoard 845c / RB3 | ¥1000-2500 |
| Phase 5-6 | Qualcomm RB5 / Rubik Pi 3 | ¥2000-3000+ |

> 策略：每个阶段按需购买，一开始不需要买最贵的。Raspberry Pi 足够学完 Phase 1-2。

---

## 推荐书单（完整版）

### 必读（按阅读顺序）
1. 《C程序设计语言》K&R 第2版 — Phase 1
2. 《C和指针》Kenneth Reek — Phase 1
3. 《UNIX环境高级编程》Stevens — Phase 1
4. 《Linux Device Drivers》第3版（LDD3）— Phase 2
5. 《Linux Kernel Development》Robert Love 第3版 — Phase 2
6. 《Understanding the Linux Kernel》Bovet & Cesati — Phase 4
7. ARM Architecture Reference Manual（ARMv8-A）— Phase 3

### 参考书（按需查阅）
8. 《C专家编程》— C 陷阱和最佳实践
9. 《C陷阱与缺陷》— 常见 C 错误
10. 《Linux/UNIX系统编程手册》— 比 APUE 更新更全
11. 《深入Linux内核架构》— 架构视角
12. 《ARM64架构与编程》杜春雷 — ARM64 中文参考
13. 《ARM System Developer's Guide》— ARM 体系结构经典
14. 《Linux内核观测技术BPF》— eBPF 可观测性
15. 《Virtual Machines》— 虚拟化理论

---

## 里程碑总览

| 时间 | 里程碑 | 验收方式 |
|------|--------|---------|
| 月2末 | C + Linux 系统编程扎实 | 能写线程池、能用 gdb 调试 core dump |
| 月5末 | 能独立写并在设备上验证驱动 | 字符设备/平台驱动在高通设备上跑通 |
| 月8末 | 理解 ARM64 底层 | 能解释 MMU/Cache/DMA，能读汇编 |
| 月11末 | 内核子系统深度掌握 | 能分析 OOM/死锁/调度延迟问题 |
| 月14末 | 高通 Android 平台专项 | 能在 GKI 约束下开发高通驱动 |
| 月18末 | 达到 Senior 岗位要求 | 有 upstream patch、能用 Rust 写驱动 |

---

## 风险提示

| 风险 | 避免方法 |
|------|---------|
| 看书不动手，知识不落地 | 每个 Week 必须有可运行的小产出 |
| 贪多嚼不烂，跳步学习 | 严格按 Phase 顺序，前置不过关不推进 |
| 只学理论不调试 | 每个模块至少用真实设备验证一次 |
| 孤立学习，不看真实代码 | 每个概念都要对应到内核源码的具体位置 |
| 长期无反馈，失去动力 | 每月做一次掌握度检查，记录进展 |

---

## 下一步行动（本周）

1. **今天**：确认每周可投入时间，设定固定学习时段
2. **本周**：开始 Phase 1 Week 1，复习 C 指针和内存布局
3. **本周产出**：用 C 实现一个 Linux kernel 风格的链表（参考 `include/linux/list.h`）
4. **工具准备**：确认 `gcc`、`gdb`、`valgrind`、`make` 环境可用

---

*计划创建时间：2026-06-10*
*最后更新：2026-06-10 — 整合全部学习资源到各 Phase*
