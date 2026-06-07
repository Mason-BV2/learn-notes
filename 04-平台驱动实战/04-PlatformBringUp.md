# Platform Bring-Up

## 概述

Platform Bring-up 是将一个新的 SoC 或新硬件平台从裸机启动到系统可运行的全过程。这是高通系统软件工程师的核心工作之一。

> 高通工作场景：JD 中明确要求 "Bring up new platforms for Qualcomm Android/Linux smartphone/compute/IoT products"

## Bring-up 流程概览

```
HW Reset
  ↓
ROM Code (PBL)              → 初始加载、DDR 训练
  ↓
SBL / XBL                   → 安全启动、加载下一阶段
  ↓
ABL / UEFI                  → 启动加载器、选择启动分区
  ↓
boot.img (Linux Kernel)      → 内核启动
  ↓
init (PID 1)                 → Android/Linux 用户空间
  ↓
Android Framework / systemd  → 系统服务
```

## 高通启动流程详解

### 1. PBL (Primary Boot Loader)

PBL 存储在 SoC 内部的 ROM 中，不可修改：

```
PBL 做的事情：
  1. 读取 boot 引脚配置（USB/emmc/UART 哪种启动方式）
  2. 初始化 SRAM
  3. 从 boot 设备加载 SBL 到 SRAM
  4. 验证 SBL 签名（安全启动）
  5. 跳转到 SBL

如果启动失败 → 进入 EDL (Emergency Download) Mode
  → 可以通过 QDLoader 工具刷机
```

### 2. SBL/XBL (Secondary Boot Loader / eXtensible Boot Loader)

```bash
# SBL 初始化 DDR 内存
# 配置 TZ (TrustZone)
# 加载 GPT 分区表
# 加载并验证 ABL/bootloader

# 查看分区表
adb shell cat /proc/partitions
adb shell ls -l /dev/block/bootdevice/by-name/
# 输出示例：
# lrwxrwxrwx ... abl_a -> /dev/block/sda20
# lrwxrwxrwx ... xbl_a -> /dev/block/sda3
# lrwxrwxrwx ... boot_a -> /dev/block/sda27
```

### 3. ABL/UEFI (Android Bootloader / UEFI)

```
ABL 负责：
  1. 显示启动画面 (boot splash)
  2. 读取 boot/recovery 分区
  3. 解析 boot.img header
  4. 加载设备树 (DTB)
  5. 将 DTB 地址传给内核 (x0 寄存器)
  6. 跳转到内核入口
```

## Early UART — Bring-up 的第一步

当你有一个新板子时，Early UART 是第一个需要工作的接口：

```dts
// DTS 中 UART 配置
&blsp1_uart2 {
    status = "okay";
    pinctrl-0 = <&uart2_default>;
    pinctrl-names = "default";
};

// 内核早期启动日志输出
// earlycon 允许在串口驱动初始化前输出日志
// 内核 cmdline 添加：
// earlycon=qcom_geni,0xa84000 console=ttyMSM0,115200
```

```bash
# 使用 minicom 连接 UART（115200 8N1）
minicom -D /dev/ttyUSB0 -b 115200

# 高通平台期望的 UART 输出：
#
# [0.000000] Booting Linux on physical CPU 0x0000000000 [0x51af8014]
# [0.000000] Linux version 5.10.120-android-...
# [0.000000] Machine model: Qualcomm Technologies, Inc. SM8250
# [0.000000] earlycon: qcom_geni at MMIO 0x00a84000
# ...
```

## 内核早期初始化流程

```c
// start_kernel() 在 init/main.c 中的初始化顺序
start_kernel:
    setup_arch()        // 架构初始化（含 DT 解析）
    mm_init()           // 内存管理初始化
    sched_init()        // 调度器初始化
    init_IRQ()          // 中断初始化
    time_init()         // 定时器初始化
    console_init()      // 控制台初始化
    late_time_init()    // 时钟源初始化
    calibrate_delay()   // 延迟校准
    arch_call_rest_init() // 创建 init 进程
```

```c
// 启动过程中常见的调试点
// 1. 内核崩溃 -> 检查 log
// 2. 挂死 -> 看是在哪个初始化函数中
// 3. Panic -> 检查 panic 信息和调用栈

// 调试技巧：在 early boot 中添加 printk
pr_info("Booting: checking UART at 0x%lx\n", uart_base);
```

## Debug 工具

### kdump — 内核崩溃转储

```bash
# 配置内核支持 kdump
CONFIG_CRASH_DUMP=y
CONFIG_PROC_VMCORE=y

# 预留 crashkernel 内存
# 内核 cmdline: crashkernel=256M

# 触发 crash
echo c > /proc/sysrq-trigger

# kdump 后分析 vmcore
crash vmlinux /var/crash/vmcore
```

### Ramdump — 高通调试

Qualcomm 平台使用 ramdump 保存崩溃时的内存镜像：

```bash
# 高通的 RAM dump 流程
# 当 watchdog 触发或 kernel panic：
# 1. SoC 进入紧急模式
# 2. 内存内容被保留
# 3. 可以通过 USB 连接 PC 导出

# PC 端使用 QPST 或 QDLoader 工具导出 ramdump
# 然后使用高通的 RAMDUMP 解析器分析
# 或者使用 crash 工具分析
```

### Trace32 — 硬件调试器

Trace32 是 ARM 平台上常用的 JTAG/SWD 调试工具，由 Lauterbach 提供：

```bash
# Trace32 可以：
# 1. 连接 JTAG/SWD 调试接口
# 2. 设置硬件断点
# 3. 单步执行 CPU 指令
# 4. 查看 CPU 寄存器
# 5. 查看内存内容
# 6. 在 SoC 启动早期调试（当 UART 还没工作时）

# Trace32 脚本示例 (.cmm)
# ; 初始化调试会话
# SYSTEM.CPU ARMv8
# SYStem.MemAccess D64
# SYStem.Option POSIX
# 
# ; 连接目标板
# SYStem.CONNECT JTAG
# 
# ; 加载符号表
# Data.LOAD.ELF vmlinux /NoCODE
# 
# ; 查看内核变量
# PRINT %hex jiffies
# 
# ; 设置断点
# Break.Set start_kernel
# Go
```

## Bring-up 检查清单

```
□ Early UART 输出正常
□ Bootloader 阶段正常通过
□ 内核解压成功
□ DTB 正确加载
□ 内存大小正确
□ 时钟和 PLL 初始化正常
□ 中断控制器 (GIC) 工作
□ 定时器工作
□ 存储设备 (eMMC/UFS) 识别
□ 根文件系统挂载
□ init 进程启动
□ ADB/SSH 连接
□ 网络/WiFi 工作
□ 显示输出
```

## 实际操作

### 1. 分析内核启动日志

```bash
# 查看完整启动日志
dmesg

# 查看启动时间分布
dmesg | grep -E "sched_clock|clocksource"
initcall_debug  # 内核 cmdline 加这个参数查看每个 initcall 耗时
```

### 2. 模拟 SoC bring-up（使用 QEMU）

```bash
# QEMU 模拟 ARM64 virt 平台的 Bring-up
qemu-system-aarch64 \
    -machine virt \
    -cpu cortex-a72 \
    -smp 4 \
    -m 2G \
    -kernel Image \
    -append "console=ttyAMA0,115200 earlycon" \
    -serial stdio \
    -drive file=rootfs.img,format=raw,id=hd0 \
    -device virtio-blk-device,drive=hd0 \
    -netdev user,id=net0 \
    -device virtio-net-device,netdev=net0
```

## 真实 GitHub 资源

| 仓库 | 内容 |
|------|------|
| [ARM-software/arm-trusted-firmware](https://github.com/ARM-software/arm-trusted-firmware) | ARM 可信固件（EL3 启动代码） |
| [u-boot/u-boot](https://github.com/u-boot/u-boot) | U-Boot 启动加载器 |
| [coreboot/coreboot](https://github.com/coreboot/coreboot) | coreboot 开源固件 |
| [qemu/qemu](https://github.com/qemu/qemu) | QEMU 模拟器（测试 bring-up） |
| [buildroot/buildroot](https://github.com/buildroot/buildroot) | Buildroot 嵌入式 Linux 构建系统 |
| [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat) | QEMU + 内核实验环境 |

## 面试考点

1. **Early UART 的作用？** 在串口驱动初始化前输出日志方便调试，是 bring-up 的必备技能
2. **bootloader 的主要职责？** 初始化硬件、加载内核和设备树、传递给内核启动参数
3. **kdump 的工作原理？** 预留一块内存运行 crash kernel，主内核崩溃时 crash kernel 接管并保存 vmcore
4. **高通 EDL 模式是什么？** Emergency Download 模式，当 normal boot 失败时的应急下载模式，通过 USB 刷机
5. **为什么 bootloader 需要 DDR 训练？** DDR 参数因板级设计不同而异，必须在运行时校准时序
