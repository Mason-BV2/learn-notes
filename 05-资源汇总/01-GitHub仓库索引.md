# GitHub 仓库资源索引

> 本文件汇总了 Linux 内核学习和高通平台开发相关的 GitHub 仓库。
> 按难度分为入门、进阶、高阶三个等级。

---

## 一、Linux 内核入门级仓库

### 1. [sysprog21/lkmpg](https://github.com/sysprog21/lkmpg) ⭐ 必看
**The Linux Kernel Module Programming Guide**
- 经典 LKMPG 书籍，已更新至 Linux 5.x/6.x 内核
- 包含大量可运行的内核模块示例
- 适合写完你的第一个 `hello world` 内核模块
- 学习要点：内核模块生命周期、`printk`、`insmod`/`rmmod`、模块参数

### 2. [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat)
**QEMU + GDB 一键式内核实验环境**
- 自动化 QEMU 虚拟机搭建
- GDB 内核级调试，可单步跟踪内核代码
- 海量可运行示例（系统调用、设备驱动、eBPF等）
- 不影响宿主系统，安全实验

### 3. [shizhengLi/linux-kernel-learning](https://github.com/shizhengLi/linux-kernel-learning) ⭐ 中文必看
**Linux Kernel 源代码深度研究项目（中文）**
- 基于 Linux 6.17 内核版本
- 7阶段系统学习计划（16-20周 / 4-5个月）
- 55+ 深度分析文档、150+ 实践代码示例、20+ 动手实验
- **阶段一**：系统架构概述 (1-2周)
- **阶段二**：核心子系统 — 进程/内存/文件系统/驱动 (3-4周)
- **阶段三**：高级子系统 — 网络/系统调用/安全/虚拟化 (2-3周)
- **阶段四**：网络子系统专题 — TCP/IP、eBPF、XDP (2-3周)
- **阶段五**：x86_64 架构 (2-3周)
- **阶段六**：调试与性能分析 (1-2周)
- **阶段七**：安全机制 — LSM、SELinux (1-2周)

### 4. [devSeyf/linux-kernel-roadmap](https://github.com/devSeyf/linux-kernel-roadmap)
**入门级 Linux 内核路线图**
- 从 Linux 基础 → C语言 → OS概念 → 汇编 → 内核修改
- 循序渐进，每一步有资源链接
- 适合零基础跟随

### 5. [alero-awani/linux-kernel-programming](https://github.com/alero-awani/linux-kernel-programming)
**Linux 内核编程实践集合**
- 从编译内核源码开始
- 涵盖：内核模块、系统调用、字符驱动、eBPF程序
- 动手实操导向

### 6. [PanshulVA/Roadmap-for-Development](https://github.com/PanshulVA/Roadmap-for-Development)
**3年硬件导向内核/驱动开发路线图**
- 从 BSc 计算机科学到专业内核/驱动开发
- 按年规划：第一年C → 第二年OS和驱动 → 第三年进阶
- 包含每阶段的学习资源链接

---

## 二、Linux 内核进阶仓库

### 7. [0xAX/linux-insides](https://github.com/0xAX/linux-insides) ⭐ 经典
**Linux 内核启动过程深度分析**
- 从 bootloader 到 init 进程的完整启动流程
- 深入理解：实模式/保护模式切换、分页机制、中断处理
- 英文，但写作清晰，配有图示
- 适合第二~三阶段阅读

### 8. [87170360/linux_kernel_wiki](https://github.com/87170360/linux_kernel_wiki) ⭐ 资料大全
**Linux 内核学习资料维基（中文）**
- 200+ 经典内核文章
- 100+ 内核学术论文
- 50+ 内核开源项目
- 500+ 内核面试题
- 80+ 内核讲解视频
- 五大知识体系：进程管理 / 内存管理 / 文件系统 / 网络协议栈 / 设备驱动
- 多个 fork 版本：[cv-programmer/linux_kernel_wiki](https://github.com/cv-programmer/linux_kernel_wiki)、[CS-Learnings/linuxkerneltravel_linux_kernel_wiki](https://github.com/CS-Learnings/linuxkerneltravel_linux_kernel_wiki)

### 9. [bootlin/training-materials](https://github.com/bootlin/training-materials)
**Bootlin 专业嵌入式 Linux 培训材料（免费）**
- 涵盖：嵌入式Linux系统、内核驱动开发、Yocto项目、设备树
- PDF幻灯片 + 实验练习，结构完整
- Bootlin 是欧洲知名的嵌入式 Linux 培训公司
- 可直接当课程使用

### 10. [torvalds/linux](https://github.com/torvalds/linux)
**Linux 内核官方主线仓库**
- 最权威的内核源码，Linus Torvalds 维护
- 配合 [elixir.bootlin.com](https://elixir.bootlin.com) 在线浏览源码
- 重点目录：
  - `arch/arm64/boot/dts/qcom/` — 高通设备树
  - `drivers/clk/qcom/` — 高通时钟驱动
  - `drivers/soc/qcom/` — 高通 SoC 驱动
  - `mm/` — 内存管理子系统
  - `kernel/` — 进程调度

---

## 三、ARM / 嵌入式 / 高通专项仓库

### 11. [qualcomm-linux/meta-qcom-hwe](https://github.com/qualcomm-linux/meta-qcom-hwe)
**高通官方 Yocto BSP 层**
- 高通 Linux 平台的 Yocto/OE 中间层
- 包含高通硬件的 BSP 支持
- 用于构建高通平台的 Linux 发行版

### 12. CodeLinaro 高通内核仓库
**网址**: [git.codelinaro.org](https://git.codelinaro.org/)
- 高通开源 Linux/Android 内核官方仓库
- `msm-5.4`、`msm-5.15`、`msm-6.1` 等内核分支
- `techpack/` 目录包含高通专有驱动（Camera、Audio、Video等）
- 需要注册账号访问

### 13. [sdm660-mainline/linux](https://github.com/sdm660-mainline/linux)
**高通 SDM660 主线 Linux 内核移植项目**
- 了解如何将高通 SoC 移植到主线内核
- 学习 DTS 设备树编写
- 参与主线内核贡献的参考

### 14. [Linaro/website](https://github.com/Linaro/website)
**Linaro 官网博客资源**
- 包含高通设备启动主线 Linux 的详细教程
- 文章: "Let's boot the mainline Linux kernel on Qualcomm devices"

---

## 四、内核开发辅助工具仓库

### 调试与追踪
| 仓库 | 用途 |
|------|------|
| [iovisor/bcc](https://github.com/iovisor/bcc) | eBPF 工具集（execsnoop、opensnoop等） |
| [brendangregg/perf-tools](https://github.com/brendangregg/perf-tools) | perf 性能分析脚本集合 |
| [torvalds/linux](https://github.com/torvalds/linux) `tools/` | 内核自带工具（perf、objtool、spi等） |

### 源码浏览
| 工具 | 网址 |
|------|------|
| **Elixir Bootlin** | [elixir.bootlin.com/linux/latest/source](https://elixir.bootlin.com/linux/latest/source) |
| **Linux Cross Reference (LXR)** | [lxr.linux.no](https://lxr.linux.no/) |

### 交叉编译工具链
| 仓库 | 用途 |
|------|------|
| [ARM GNU Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain) | 官方 ARM 交叉编译工具链 |
| [Linaro Toolchain](https://releases.linaro.org/components/toolchain/binaries/) | Linaro 预编译工具链 |
