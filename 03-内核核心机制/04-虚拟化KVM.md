# KVM / pKVM 虚拟化

## 概述

KVM（Kernel-based Virtual Machine）是 Linux 上的 Type-2 Hypervisor。pKVM（protected KVM）是 ARM 平台上基于硬件的受保护虚拟化扩展，由 Google 在 Android 上推广。

> 高通工作场景：JD 明确要求 KVM/pKVM 开发经验。高通 Snapdragon 平台上的 Android 使用 pKVM 隔离安全敏感负载，高通也在 ARM 服务器领域使用 KVM 虚拟化。

## 基础概念

### ARM 虚拟化异常模型

```
EL0  → 用户空间（VM 的 guest）
EL1  → 内核（VM 的 guest kernel / host kernel）
EL2  → Hypervisor（KVM / pKVM）← 虚拟化层
EL3  → Secure Monitor（ATF / TrustZone）
```

### Stage-2 地址翻译

KVM 使用 ARM 的两级地址翻译（Two-stage translation）：

```
Stage-1 (Guest OS)        Stage-2 (Hypervisor)
VA[guest] → IPA             IPA → PA
                            
Guest 认为自己管理物理内存（IPA）
实际上 Hypervisor 控制真正的物理内存（PA）
```

## KVM — 内核虚拟化

### KVM API — /dev/kvm

KVM 通过字符设备暴露 API，用户态 QEMU/crosvm 通过 ioctl 管理 VM：

```c
// 用户态创建 VM 的流程
int kvm_fd = open("/dev/kvm", O_RDWR);

// 1. 检查 KVM 版本
ioctl(kvm_fd, KVM_GET_API_VERSION, NULL);

// 2. 创建 VM
int vm_fd = ioctl(kvm_fd, KVM_CREATE_VM, 0);

// 3. 创建 vCPU
int vcpu_fd = ioctl(vm_fd, KVM_CREATE_VCPU, 0);

// 4. 设置 guest 内存
struct kvm_userspace_memory_region region = {
    .slot = 0,
    .guest_phys_addr = 0x80000000,  // IPA
    .memory_size = 0x10000000,       // 256MB
    .userspace_addr = (__u64)guest_mem,
};
ioctl(vm_fd, KVM_SET_USER_MEMORY_REGION, &region);

// 5. 加载 guest 内核镜像
// 将 kernel Image、DTB、initrd 复制到 guest 内存

// 6. 设置 vCPU 寄存器（PC、SP、x0-DTB地址）
struct kvm_vcpu_init init;
ioctl(vcpu_fd, KVM_ARM_VCPU_INIT, &init);

// 7. 运行 vCPU
ioctl(vcpu_fd, KVM_RUN, 0);

// 8. 处理 VM exit（ioctl 返回）
struct kvm_run *run = (struct kvm_run *)mmap(vcpu_fd, ...);
switch (run->exit_reason) {
    case KVM_EXIT_MMIO:  // guest 访问外设
        handle_mmio(run);
        break;
    case KVM_EXIT_IRQ:   // 中断
        break;
}
```

### 核心 KVM 数据结构（内核侧）

```c
// virt/kvm/kvm_main.c

struct kvm {
    struct mm_struct *mm;     // 用户空间内存描述符
    struct kvm_vcpu *vcpus[KVM_MAX_VCPUS];
    struct kvm_memslots *memslots;  // guest 物理地址空间
    struct kvm_arch arch;     // 架构特化
};

struct kvm_vcpu {
    struct kvm *kvm;
    int vcpu_id;
    struct kvm_run *run;      // 与用户态共享的 run 结构
    struct kvm_vcpu_arch arch; // 架构特化（含寄存器状态）
    // ...
};
```

### ARM64 KVM 关键文件

```c
// arch/arm64/kvm/
├── main.c           // 模块初始化
├── vm.c             // VM 创建/销毁
├── vcpu.c           // vCPU 初始化
├── run.c            // KVM_RUN 实现
├── guest.c          // 内存设置
├── mmu.c            // Stage-2 MMU 管理
├── handle_exit.c    // VM exit 处理
├── inject_fault.c   // 注入异常到 guest
├── reset.c          // vCPU 状态重置
├── sys_regs.c       // 系统寄存器模拟
├── hyp/             // EL2 hypvisor 代码
│   ├── hyp-entry.S  // EL2 入口
│   ├── switch.c     // world switch
│   └── timer-sr.c   // 定时器保存/恢复
└── vgic/            // GIC 虚拟化
    ├── vgic.c       // VGIC 核心
    ├── vgic-v2.c    // GICv2 虚拟化
    ├── vgic-v3.c    // GICv3 虚拟化
    └── vgic-its.c   // ITS 中断翻译
```

## pKVM — 受保护 KVM

pKVM 是 Google 在 Android 上推广的安全虚拟化方案，将 Android 内核的部分功能移到 EL2：

### 关键区别

```
传统 KVM:               pKVM:
  Host @ EL1              Host @ EL2 (受限)
  Guest @ EL1             Protected VM @ EL1
  VMM @ EL0 (QEMU)        VMM @ EL1 (crosvm)
                           Non-protected VM @ EL1 (传统 Android)
```

### 架构

```
pKVM 核心思想：
  EL2 运行一个最小的 hypervisor
  → 隔离受保护的 VM 内存（即使是 host 也不能访问）
  → 用于运行安全敏感负载（DRM、支付、生物识别）
```

### pKVM 内存所有权模型

```c
// 内存被划分为：
// 1. Host 内存 — Android/Linux 正常使用
// 2. Protected 内存 — 只属于特定 protected VM
// 
// Host 不能直接访问 protected 内存
// 保护通过 stage-2 MMU 强制执行

// pKVM hypervisor 调用接口
// include/linux/kvm_host.h

// 将内存从 host 捐给 pKVM
int pkvm_donate_memory(unsigned long start, unsigned long size);

// 将内存从 pKVM 归还给 host
int pkvm_reclaim_memory(unsigned long start, unsigned long size);
```

### pKVM 代码位置

```c
// arch/arm64/kvm/hyp/nvhe/  (NVHE = Non-VHE, pKVM 模式)
├── hyp-main.c        // pKVM hypervisor 入口
├── mem_protect.c     // 内存保护核心逻辑
├── pkvm.c           // pKVM VM 管理
├── mm.c              // EL2 内存分配器
└── iommu.c           // IOMMU 管理
```

## 实操练习

### 1. 检查系统是否支持 KVM

```bash
# 检查 CPU 是否支持虚拟化
lscpu | grep Virtualization

# 检查 KVM 模块
lsmod | grep kvm
ls -l /dev/kvm

# 检查 ARM 硬件虚拟化支持
cat /proc/cpuinfo | grep -E "Features.*VHE"
# VHE = Virtualization Host Extensions
```

### 2. 使用 kvmtool 创建轻量级 VM

```bash
# kvmtool 是一个轻量级 VMM（比 QEMU 简单很多）
git clone https://github.com/kvmtool/kvmtool
cd kvmtool
make

# 启动 guest kernel
./lkvm run -k /path/to/Image -d /path/to/rootfs.img -m 512
```

### 3. KVM 单元测试框架

```bash
# KVM 开发者测试工具
git clone https://github.com/kvm-unit-tests/kvm-unit-tests
cd kvm-unit-tests
./configure --arch=aarch64
make

# 运行测试
./arm-run arm/selftest.flat
```

## 真实 GitHub 资源

| 仓库 | 内容 |
|------|------|
| [torvalds/linux/virt/kvm/](https://github.com/torvalds/linux/tree/master/virt/kvm) | KVM 平台无关核心代码 |
| [torvalds/linux/arch/arm64/kvm/](https://github.com/torvalds/linux/tree/master/arch/arm64/kvm) | ARM64 KVM 实现 |
| [kvm-unit-tests/kvm-unit-tests](https://github.com/kvm-unit-tests/kvm-unit-tests) | KVM 单元测试框架 |
| [kvmtool/kvmtool](https://github.com/kvmtool/kvmtool) | 轻量级 KVM VMM 工具 |
| [android/kernel-common](https://android.googlesource.com/kernel/common/) | Android 通用内核（含 pKVM） |
| [ARM-software/arm-trusted-firmware](https://github.com/ARM-software/arm-trusted-firmware) | EL3 固件（PSCI 实现） |

## 面试考点

1. **Stage-2 翻译和 Stage-1 的区别？** Stage-1 VA→IPA（guest 控制），Stage-2 IPA→PA（hypervisor 控制）
2. **VM exit 和 World switch 的开销？** 每次 VM exit 导致从 EL1 陷入 EL2，保存/恢复上下文，约数千周期
3. **pKVM 和传统 KVM 的安全模型区别？** pKVM 假设 host 不可信，使用 stage-2 MMU 保护 guest 内存不被 host 访问
4. **VHE 和非 VHE 的区别？** VHE（Virtualization Host Extensions）让 host 内核直接运行在 EL2，减少了 virtualization trap 的开销
5. **什么是 IPA？** Intermediate Physical Address，guest 看到的物理地址，由 hypervisor 通过 stage-2 页表映射到真实物理地址
