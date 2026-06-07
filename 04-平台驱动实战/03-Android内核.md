# Android 内核

## 概述

Android 基于 Linux 内核，但增加了大量移动设备特有功能。Qualcomm 是 Android 芯片最大供应商，高通 Android 内核开发是岗位核心工作之一。

> 高通工作场景：你需要维护高通 Android 内核版本、升级 Android baseline、处理 GKI（通用内核镜像）兼容性、调试 HAL 层问题。

## Android 内核架构

```
用户空间 (Java/Kotlin)
  └── Android Framework
        └── HAL (Hardware Abstraction Layer)
              ├── HIDL (HAL Interface Definition Language) [旧]
              └── AIDL (Android Interface Definition Language) [新]
                    │
                    ├── Stub HAL        → 内核驱动 (旧方式)
                    └── passthrough HAL  → 内核驱动
                          ↓
                    Linux 内核驱动
                          ↓
                    硬件 (Qualcomm SoC)
```

## AOSP 内核分支

Android 有多个内核分支，高通有自己的分支：

```bash
# Android 通用内核
# https://android.googlesource.com/kernel/common/
android-4.14-stable
android-4.19-stable
android-5.4-stable
android-5.10-stable
android-5.15-stable
android-common          # 最新的 GKI 内核

# 高通 Android 内核（Code Aurora -> 已迁移到 qualcomm 仓库）
# https://github.com/qualcomm-linux/android-kernel
```

## Android 特有的内核机制

### 1. Binder — IPC 机制

Binder 是 Android 进程间通信核心，实现为内核驱动：

```c
// drivers/android/binder.c
// Binder 通过 ioctl 通信

struct binder_write_read {
    struct binder_write_read bwr;
    // write_size: 写入缓冲区大小
    // read_size:  读取缓冲区大小
    // write_buffer/write_consumed
    // read_buffer/read_consumed
};

// 核心概念
// 1. Binder 节点 → Service 的表示
// 2. 事务 (Transaction) → 进程间的调用
// 3. 引用 (Ref) → 跨进程的对象引用
// 4. 线程池 → 每个进程有 Binder 线程池
```

### 2. Ashmem — 匿名共享内存

```c
// drivers/android/ashmem.c
// 已被 ION 和 dma-buf 取代，但仍存在于旧内核中

int fd = ashmem_create_region("my_region", size);
// → 返回 fd，可通过 Binder 传递给其他进程
```

### 3. ION — 内存分配器

见 [03-内存管理.md](../03-内核核心机制/03-内存管理.md) 的 ION 章节。

## GKI — 通用内核镜像

Android 12+ 引入 GKI（Generic Kernel Image），将内核分为 GKI 内核和供应商模块：

```
GKI 架构：
  GKI 内核 (boot.img)
    └── 通用内核 (android-common)
          ├── 核心内核代码 (调度、内存、文件系统等)
          └── 内核模块接口 (KMI)
                └── 供应商模块 (vendor_boot.img)
                      ├── 高通 SoC 驱动
                      ├── 板级 dts
                      └── 外设驱动
```

### KMI — 内核模块接口

```c
// KMI 是 GKI 的核心理念
// 导出的符号和数据结构必须保持稳定
// 供应商只通过模块加载驱动，不能修改内核核心代码

// KMI 符号保护
// 如果供应商需要导出新的符号给模块使用：
EXPORT_SYMBOL_GPL(qcom_smmu_cfg);  // 不能随意新增

// 必须通过 Google 审核或使用 ABI 符号列表
// out/android-5.15/dist/abi_symbol_list
```

### GKI 构建和刷写

```bash
# 构建 GKI 内核
git clone https://android.googlesource.com/kernel/common
cd common
git checkout android-5.15-2022-12

# 编译
BUILD_CONFIG=common/build.config.gki.aarch64 build/build.sh
# 输出: out/android-5.15/dist/Image

# 构建高通供应商模块
# 需要高通提供的 vendor_boot 配置
BUILD_CONFIG=private/msm-google/build.config.gki build/build.sh
```

## Treble — 项目 Treble 架构

Project Treble 将 HAL 和内核分离，使 OEM 可以升级 Android 版本而无需更改芯片厂商代码：

```
Android 8.0+
  ┌─────────────────────┐
  │ Android Framework    │
  │ (System UI, Apps)    │ ← 可以升级
  ├─────────────────────┤
  │ HAL Interface        │ ← 稳定的 HIDL/AIDL 接口
  ├─────────────────────┤
  │ Vendor Implementation│ ← 高通提供，保持不变
  │ (HAL + kernel)       │
  └─────────────────────┘
```

## Android 内核调试

### 1. logcat 和内核日志

```bash
# 内核日志
adb shell dmesg
adb shell dmesg | grep qcom

# Android 日志
adb logcat
adb logcat -b kernel  # 内核日志
adb logcat -b main    # 应用日志
adb logcat -b system  # 系统日志
```

### 2. Ramdump 和 Watchdog

```bash
# 高通的看门狗超时处理
# 当系统 Hang 住时，Watchdog 触发重启并保存 ramdump

# 触发 kernel panic 调试
echo c > /proc/sysrq-trigger

# 查看上次重启原因
adb shell dmesg | grep "Kernel panic"
```

### 3. 内核调试常用节点

```bash
# CPU 信息
adb shell cat /sys/devices/system/cpu/possible

# cpufreq
adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq

# 内存
adb shell cat /proc/meminfo

# IOMMU
adb shell cat /sys/kernel/debug/iommu/arm-smmu/addresses

# 挂载信息
adb shell cat /proc/mounts
```

## 真实 GitHub 资源

| 仓库 | 内容 |
|------|------|
| [android/kernel-common](https://android.googlesource.com/kernel/common/) | Android 通用内核（GKI 基础） |
| [android/platform/system/core](https://android.googlesource.com/platform/system/core/) | Android 核心系统（init、adb、logd） |
| [android/platform/hardware/interfaces](https://android.googlesource.com/platform/hardware/interfaces/) | HAL 接口定义 |
| [qualcomm-linux/android-kernel](https://github.com/qualcomm-linux/android-kernel) | 高通 Android 内核（含多个分支） |
| [AOSP source](https://source.android.com/docs/core/architecture/kernel) | Android 内核架构文档 |
| [Android Generic Kernel Image](https://source.android.com/docs/core/architecture/kernel/generic-kernel-image) | GKI 官方文档 |

## 面试考点

1. **GKI 的核心目标？** 将内核核心和供应商驱动分离，使 Google 可以独立更新内核，OEM/SOC 只维护内核模块
2. **Binder 相比传统 IPC 的优势？** 只需一次拷贝（而非两次）、基于 mmap、支持对象引用计数
3. **HIDL 和 AIDL HAL 的区别？** HIDL 是老的方式（Android 8-10），AIDL HAL 是新的统一方式（Android 11+）
4. **Treble 解决了什么问题？** 将 Android 框架和厂商实现分离，缩短 Android 版本升级周期
5. **KMI 稳定性的意义？** 允许供应商模块在不同内核版本上工作，不需要每次内核更新都重新编译
