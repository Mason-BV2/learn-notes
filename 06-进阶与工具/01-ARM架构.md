# ARM 架构 — ARMv8-A

## 概述

ARMv8-A 是 64 位 ARM 架构，支持 AArch64（64位）和 AArch32（32位兼容）两种执行状态。

> 高通工作场景：所有 Snapdragon SoC 都基于 ARM 架构。你需要理解异常级别、页表、缓存一致性、GIC 中断等 ARM 核心技术。

## 异常级别（Exception Levels）

```
EL0  → 用户空间 (Applications)
EL1  → 内核 (OS Kernel, Linux / Android)
EL2  → 虚拟机监视器 (Hypervisor, KVM)
EL3  → 安全监视器 (Secure Monitor, ATF / TrustZone)
```

### 异常级别切换

```asm
// 从 EL1 切换到 EL2 (通过 HVC 指令)
hvc #0              // Hypervisor Call

// 从 EL1 切换到 EL3 (通过 SMC 指令)
smc #0              // Secure Monitor Call

// 从 EL0 切换到 EL1 (通过 SVC 指令)
svc #0              // Supervisor Call (系统调用)

// 异常返回
eret                // 返回到较低的异常级别
```

### 寄存器文件

```asm
// AArch64 通用寄存器
// X0-X30: 64-bit 通用寄存器
// W0-W30: Xn 的低 32 位
// SP: 栈指针
// PC: 程序计数器
// X30: 链接寄存器 (LR)

// 特殊寄存器
// SPSR_ELx: 保存的程序状态寄存器
// ELR_ELx: 异常链接寄存器（返回地址）
// ESR_ELx: 异常综合寄存器（原因）
// FAR_ELx: 错误地址寄存器
```

## MMU 和页表

### ARMv8-A 页表配置

```
页大小选项：
  4KB → 3-4 级页表（最小粒度）
  16KB → 2-3 级页表
  64KB → 2 级页表（TLB 覆盖范围最大）

虚拟地址范围：
  AA64 48-bit 地址空间 (256TB)
  AA64 52-bit 地址空间 (4PB) [ARMv8.2 可选]
```

```asm
// 系统寄存器配置
// TTBR0_EL1: 用户空间页表基址
// TTBR1_EL1: 内核空间页表基址
// TCR_EL1:   页表控制寄存器（页大小、缓存策略等）
// MAIR_EL1:  内存属性寄存器（缓存策略定义）

// 页表描述符格式
// [63]  → 描述符类型 (1=页表, 0=块/页)
// [62]  → NS (非安全)
// [61]  → AP[1] (访问权限)
// [60]  → AP[2]
// [59:51] → 软件位
// [50:48] → 连续性指示器
// [47:16] → 输出地址 [4KB 页]
// [15:12] → nG/AF/SH/AP/XN
// [11:2]  → 内存属性 (Attrs)
// [1:0]   → 描述符类型
```

## GICv3 — 中断控制器

```asm
// GICv3 系统寄存器接口（比 GICv2 的内存映射接口更快）
// ICC_SRE_ELx:  系统寄存器使能
// ICC_PMR_EL1:  优先级掩码
// ICC_IAR0_EL1: 中断确认寄存器
// ICC_EOIR0_EL1:中断结束寄存器
// ICC_IGRPEN1_EL1: 中断组使能

// 中断优先级配置
// 优先级值 = 0-255，0 = 最高优先级
// PMR 配置允许接收的最低优先级
```

## 缓存架构

```
ARMv8 缓存层次：

CPU Core 0         CPU Core 1
  L1 I-cache         L1 I-cache
  L1 D-cache         L1 D-cache
      \                 /
       L2 Cache (每核心私有或共享)
              |
       L3 Cache (SoC 级别共享)
              |
          内存控制器
```

### 缓存维护指令

```asm
// 数据缓存维护（MMIO 和 DMA 时需要）
DC IVAC, X0     // 使能并废弃虚拟地址缓存行
DC CVAC, X0     // 清理虚拟地址缓存行（写回）
DC CVAU, X0     // 清理到点统一点（PoU）
DC CVAP, X0     // 清理到持久点（PoP）
DC CIVAC, X0    // 清理并使能虚拟地址缓存行 ← DMA 最常用

// 指令缓存维护
IC IVAU, X0     // 使能指令缓存行
IC IALLU        // 全部使能
```

## 内存屏障

```asm
// ARMv8 内存屏障指令
DMB ISH          // 数据内存屏障（内部可共享域）
DSB ISH          // 数据同步屏障（等待所有内存访问完成）
ISB              // 指令同步屏障（刷新流水线）

// 场景：驱动中写 MMIO 寄存器后
writel_relaxed(val, reg);  // 非屏障写
writel(val, reg);          // 带 writel 隐式 DMB

// DMA 操作时
dma_wmb();       // DMA 写内存屏障
dma_rmb();       // DMA 读内存屏障
dma_mb();        // DMA 全屏障
```

## ARM 汇编基础（内核驱动常用）

```asm
// 读写系统寄存器
.macro read_sysreg, reg, name
    mrs \reg, \name
.endm

.macro write_sysreg, val, name
    msr \name, \val
.endm

// 示例：读取当前异常级别
    mrs x0, CurrentEL
    and x0, x0, #0x0c
    lsr x0, x0, #2
    // x0 = 0 (EL0), 1 (EL1), 2 (EL2), 3 (EL3)

// 原子操作
    ldrex w0, [x1]      // 加载独占
    strex w2, w3, [x1]  // 存储独占
    // CAS 指令 [ARMv8.1+]
    cas w0, w1, [x2]    // 比较并交换

// 内核启动入口 (head.S)
    .section ".text.head"
    .globl _start
_start:
    // x0 = DTB 物理地址 (由 bootloader 传入)
    // x1 = 0
    // x2 = 0
    // x3 = 0
    
    mov x19, x0         // 保存 DTB 地址
    mrs x0, CurrentEL
    cmp x0, #(2 << 2)   // EL2?
    b.eq el2_setup
    
el1_setup:
    // 配置 EL1 阶段
    // ...

el2_setup:
    // 配置 EL2 阶段（VHE/非VHE）
    // ...
    // 切换到 EL1
    eret
```

## 实操练习

### 1. 查看 CPU 特性

```bash
# 查看 CPU 架构
cat /proc/cpuinfo | grep "CPU architecture"

# 查看 ARM 特性
cat /proc/cpuinfo | grep Features

# 查看异常级别（内核 ftrace）
cat /sys/kernel/debug/sched_features
```

### 2. 使用 ARM 汇编编写原子操作

```c
// Linux 内核中 ARM64 原子操作示例
// arch/arm64/include/asm/atomic.h

static inline void atomic_add(int i, atomic_t *v)
{
    unsigned long tmp;
    int result;
    
    asm volatile ("// atomic_add\n"
    "1: ldxr    %w0, %2\n"
    "   add     %w0, %w0, %w3\n"
    "   stxr    %w1, %w0, %2\n"
    "   cbnz    %w1, 1b"
    : "=&r" (result), "=&r" (tmp), "+Q" (v->counter)
    : "Ir" (i));
}
```

## 真实 GitHub 资源

| 仓库 | 内容 |
|------|------|
| [ARM-software/arm-trusted-firmware](https://github.com/ARM-software/arm-trusted-firmware) | ARM 可信固件（参考实现） |
| [ARM-software/sbsa-tcs](https://github.com/ARM-software/sbsa-tcs) | ARM SBSA 架构测试 |
| [torvalds/linux/arch/arm64/](https://github.com/torvalds/linux/tree/master/arch/arm64) | ARM64 内核源码 |
| [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat) | 含 ARM 汇编示例 |
| [ARM-software/abi-aa](https://github.com/ARM-software/abi-aa) | ARM ABI 标准 |

## 面试考点

1. **ARM 异常级别的作用？** EL0 用户、EL1 内核、EL2 虚拟机、EL3 安全，隔离不同特权级
2. **ARM 缓存维护什么时候需要？** DMA 传输前后、修改代码后（指令缓存）、MMIO 操作
3. **DMB 和 DSB 的区别？** DMB 保证内存访问顺序，DSB 保证所有内存访问完成后才继续执行
4. **SMC 和 HVC 调用区别？** SMC 进入 EL3（安全），HVC 进入 EL2（虚拟化）
5. **virt_to_phys 在 ARM64 上是如何实现的？** 通过直接映射偏移（PAGE_OFFSET）计算，无需查页表
