# 从零到高通 Linux 内核工程师 — 学习路线

> 目标岗位：Qualcomm Software Engineer（Linux 内核方向）
> 岗位要求：BS+3年 / MS+1年 经验，Linux 内核/驱动开发
> 起点：有计算机基础
> 🌎 英语：高通是外企，英语贯穿28周全程训练

---

## ⚡ 28 周固定课程（推荐）

[📅 点击此处查看完整课表 → 00-课程大纲.md](00-课程大纲.md)

**每周固定内容 + 固定项目 + 固定验收标准。** 这不是"参考路线"，是**固定课表**。

| 文档 | 说明 |
|------|------|
| [📅 00-课程大纲.md](00-课程大纲.md) | 28 周完整课表（每日计划 + 每周项目） |
| [⚡ 00-QEMU环境搭建.md](00-QEMU环境搭建.md) | 第1周必须搭建的实验基地 |
| [📋 00-项目清单.md](00-项目清单.md) | 全部 24 个项目及验收标准 |

---

## 学习路线总览

```
阶段一：内核模块驱动（第1-4周）
  环境搭建 → C语言特训 → Git工作流 → 内核编译系统
    ↓
阶段二：内核模块驱动（第5-8周）
  内核模块 → 字符设备 → /proc接口 → 调试工具
    ↓
阶段三：内核核心机制（第9-14周）
  进程调度 → 中断定时器 → 内存管理 → DMA/ION → KVM
    ↓
阶段四：平台驱动实战（第15-20周）
  DeviceTree → I2C/SPI/UART → Android内核 → Bring-up
    ↓
阶段五：进阶与求职（第23-28周）
  ARM架构 → 性能优化 → Python自动化 → Upstream → 综合项目 → 面试冲刺
```

---

## 目录结构

| 目录 | 内容 | 对应 JD 要求 |
|------|------|-------------|
| [01-计算机基础](./01-计算机基础/) | 计算机组成、操作系统、C语言、数据结构 | C/C++ programming |
| [02-内核模块驱动](./02-内核模块驱动/) | 内核模块、字符设备、编译系统 | Linux kernel/driver development, Makefile |
| [03-内核核心机制](./03-内核核心机制/) | 调度、中断、内存管理、KVM虚拟化 | virtualization, scheduler, interrupt, timer |
| [04-平台驱动实战](./04-平台驱动实战/) | DeviceTree、外设驱动、Android内核、Bring-up | peripheral drivers, platform bring-up, Android baseline |
| [05-资源汇总](./05-资源汇总/) | GitHub仓库、书籍、开发板、路线图 | — |
| [06-进阶与工具](./06-进阶与工具/) | ARM架构、ARM服务器、Upstream、Git/Repo、Trace32、面试、AI Agent、**英语** | ARM assembly, upstream, Git/repo, Trace32, Python, **English** |

---

## 各模块详细介绍

### [02-内核模块驱动](./02-内核模块驱动/)
| 文件 | 内容 | 难度 |
|------|------|------|
| [01-Linux内核模块编程.md](./02-内核模块驱动/01-Linux内核模块编程.md) | 模块结构、加载/卸载、printk、模块参数 | ⭐ 入门 |
| [02-字符设备驱动.md](./02-内核模块驱动/02-字符设备驱动.md) | file_operations、cdev、udev、ioctl | ⭐⭐ 基础 |
| [03-内核编译系统.md](./02-内核模块驱动/03-内核编译系统.md) | Kconfig、Makefile、defconfig | ⭐⭐ 基础 |

### [03-内核核心机制](./03-内核核心机制/)
| 文件 | 内容 | 难度 |
|------|------|------|
| [01-进程调度.md](./03-内核核心机制/01-进程调度.md) | CFS、EAS、调度类、cpufreq、cgroup | ⭐⭐⭐ 进阶 |
| [02-中断与定时器.md](./03-内核核心机制/02-中断与定时器.md) | IRQ、threaded IRQ、softirq、timer、hrtimer | ⭐⭐⭐ 进阶 |
| [03-内存管理.md](./03-内核核心机制/03-内存管理.md) | 页表、SLAB、DMA、ION、CMA | ⭐⭐⭐ 进阶 |
| [04-虚拟化KVM.md](./03-内核核心机制/04-虚拟化KVM.md) | KVM/pKVM、hypervisor、stage-2 MMU | ⭐⭐⭐⭐ 高阶 |
| [05-并发与同步.md](./03-内核核心机制/05-并发与同步.md) | spinlock、mutex、RCU、lockdep | ⭐⭐⭐ 进阶 |
| [06-文件系统.md](./03-内核核心机制/06-文件系统.md) | VFS、debugfs、sysfs、proc | ⭐⭐⭐ 进阶 |
| [07-内存调试.md](./03-内核核心机制/07-内存调试.md) | KASAN、kmemleak、slub debug | ⭐⭐⭐ 进阶 |

### [04-平台驱动实战](./04-平台驱动实战/)
| 文件 | 内容 | 难度 |
|------|------|------|
| [01-DeviceTree.md](./04-平台驱动实战/01-DeviceTree.md) | DTS 语法、bindings、pinctrl | ⭐⭐ 基础 |
| [02-外设驱动.md](./04-平台驱动实战/02-外设驱动.md) | I2C/SPI/UART/MMC/PCIe 驱动 | ⭐⭐⭐ 进阶 |
| [03-Android内核.md](./04-平台驱动实战/03-Android内核.md) | AOSP 内核、GKI、HAL、HIDL、Treble | ⭐⭐⭐ 进阶 |
| [04-PlatformBringUp.md](./04-平台驱动实战/04-PlatformBringUp.md) | UART log、bootloader、kdump、ramdump | ⭐⭐⭐⭐ 高阶 |

### [06-进阶与工具](./06-进阶与工具/)
| 文件 | 内容 | 难度 |
|------|------|------|
| [01-ARM架构.md](./06-进阶与工具/01-ARM架构.md) | ARMv8-A 异常级别、MMU、GIC、缓存 | ⭐⭐⭐⭐ 高阶 |
| [02-ARM服务器.md](./06-进阶与工具/02-ARM服务器.md) | SBSA/SBBR、ACPI、UEFI | ⭐⭐⭐⭐ 高阶 |
| [03-Upstream提交.md](./06-进阶与工具/03-Upstream提交.md) | 补丁格式、review 流程、maintainer | ⭐⭐⭐ 进阶 |
| [04-GitRepo工作流.md](./06-进阶与工具/04-GitRepo工作流.md) | Git 高级、repo 工具、基线升级 | ⭐⭐ 基础 |
| [05-Trace32调试.md](./06-进阶与工具/05-Trace32调试.md) | Lauterbach 脚本、调试分析 | ⭐⭐⭐⭐ 高阶 |
| [06-面试准备.md](./06-进阶与工具/06-面试准备.md) | C/C++、内核、ARM 面试题 | ⭐⭐⭐ 进阶 |
| [07-AIAgent开发.md](./06-进阶与工具/07-AIAgent开发.md) | Claude Code、AI 辅助内核开发 | ⭐⭐⭐ 进阶 |
| [08-英语学习.md](./06-进阶与工具/08-英语学习.md) | 技术英语、外企场景、面试英语 | ⭐⭐ 基础 |
| [09-性能优化.md](./06-进阶与工具/09-性能优化.md) | perf、tracepoint、ftrace | ⭐⭐⭐⭐ 高阶 |
| [10-Python自动化.md](./06-进阶与工具/10-Python自动化.md) | 构建自动化、日志分析、DTS处理 | ⭐⭐⭐ 进阶 |

---

## 资源快速入口

| 资源类型 | 文件 |
|----------|------|
| GitHub 仓库索引 | [05-资源汇总/01-GitHub仓库索引.md](./05-资源汇总/01-GitHub仓库索引.md) |
| 推荐书籍 | [05-资源汇总/02-推荐书籍.md](./05-资源汇总/02-推荐书籍.md) |
| 开发板与工具 | [05-资源汇总/03-开发板与工具.md](./05-资源汇总/03-开发板与工具.md) |
| 学习路线图 | [05-资源汇总/04-学习路线图.md](./05-资源汇总/04-学习路线图.md) |
| 高通平台专项资源 | [05-资源汇总/05-高通平台专项资源.md](./05-资源汇总/05-高通平台专项资源.md) |
| 零基础计算机基础 | [05-资源汇总/06-零基础计算机基础知识.md](./05-资源汇总/06-零基础计算机基础知识.md) |
| C语言学习资源 | [05-资源汇总/07-C语言零基础学习资源.md](./05-资源汇总/07-C语言零基础学习资源.md) |
| 数据结构与算法 | [05-资源汇总/08-数据结构与算法资源.md](./05-资源汇总/08-数据结构与算法资源.md) |
| CS自学路线 | [05-资源汇总/09-CS自学路线资源.md](./05-资源汇总/09-CS自学路线资源.md) |

---

## 关键 GitHub 仓库速览

| 仓库 | 用途 | 难度 |
|------|------|------|
| [sysprog21/lkmpg](https://github.com/sysprog21/lkmpg) | 内核模块编程指南（6.x内核） | 入门 |
| [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat) | QEMU 实验环境 + 100+ 示例 | 入门 |
| [0xAX/linux-insides](https://github.com/0xAX/linux-insides) | Linux 内核深度分析 | 进阶 |
| [bootlin/training-materials](https://github.com/bootlin/training-materials) | 专业嵌入式 Linux 培训材料 | 入门-进阶 |
| [torvalds/linux](https://github.com/torvalds/linux) | Linux 内核源码 | 全阶段 |
| [qualcomm-linux/meta-qcom-hwe](https://github.com/qualcomm-linux/meta-qcom-hwe) | 高通 Yocto BSP 层 | 高阶 |
| [kvm-unit-tests/kvm-unit-tests](https://github.com/kvm-unit-tests/kvm-unit-tests) | KVM 单元测试 | 高阶 |

---

## 推荐开发板

| 阶段 | 推荐 | 价格参考 |
|------|------|----------|
| 入门 | Raspberry Pi 4/5 | ¥300-500 |
| 进阶 | DragonBoard 410c (APQ8016E) | ¥500-800 |
| 实战 | DragonBoard 845c (SDM845) | ¥1000-2000 |
| 专业 | Qualcomm RB5 (QRB5165) | ¥3000+ |

---

## 学习建议

1. **每个模块都有实操练习**，确保每学完一个知识点就动手
2. **用 QEMU 实验**，大部分练习不需要真实硬件
3. **读源码 > 看教程**，内核源码是最好的老师
4. **做好笔记**，这个 APK 就是你的知识库
5. **坚持**，内核工程师没有速成路径，但针对 JD 定向学习可以大幅缩短时间
