# ARM 服务器

## 概述

ARM 服务器是 JD 中的加分项。高通在 ARM 服务器市场也有布局（如 Qualcomm Centriq 系列，以及 Cloud AI 100 推理加速器）。ARM 服务器与传统 x86 服务器的主要区别在于固件接口（ACPI/UEFI 替代 DeviceTree）和架构一致性标准（SBSA/SBBR）。

> 高通工作场景：JD 要求 "Experience with ARM server development is a plus"。高通的服务器芯片（如 Cloud AI 100）使用 ARM 架构。

## SBSA — Server Base System Architecture

SBSA 定义了服务器级 ARM 系统必须满足的硬件要求：

```
SBSA Level 要求：
  Level 0: 必须
    - ARMv8-A 架构
    - GICv3 中断控制器
    - Generic Timer
    - 至少 4GB 内存
  
  Level 1: 企业级
    - SMMU (IOMMU)
    - PCIe 根复杂
  
  Level 2: 高性能
    - PCIe ATS (Address Translation Services)
    - PCIe PRI (Page Request Interface)
  
  Level 3: 虚拟化
    - IOMMU 支持 PCIe 直通
    - GICv3 ITS
    - SMMUv3
  
  Level 4: 完整
    - CCID 支持
    - RAS 扩展
```

## SBBR — Server Base Boot Requirements

SBBR 定义了服务器 ARM 系统的启动要求：

```
SBBR 要求：
  1. UEFI 固件（替代 DeviceTree/bootloader）
  2. ACPI 表（替代 DeviceTree）
  3. SMBIOS 系统信息
  4. 遵循 UEFI Shell 规范
  5. 支持 UEFI Secure Boot
```

## ACPI — Advanced Configuration and Power Interface

在 ARM 服务器上，ACPI 替代 DeviceTree 作为硬件描述方式：

```c
// ACPI 表结构
// DSDT  → 差异化系统描述表（主要硬件信息）
// SSDT  → 辅助系统描述表
// MADT  → 多 APIC 描述表（中断控制器信息）
// GTDT  → 通用定时器描述表
// IORT  → IO 远程化表（SMMU/PCIe 拓扑）
// PPTT  → 处理器属性拓扑表（缓存/CPU 拓扑）
// DBG2  → 调试端口表 2
// SPCR  → 串行端口控制台重定向表
```

```bash
# 查看 ACPI 表
# 在 ARM 服务器上：
ls /sys/firmware/acpi/tables/
cat /sys/firmware/acpi/tables/DSDT | xxd | head -20

# 反编译 ACPI
apt install acpica-tools
iasl -d /sys/firmware/acpi/tables/DSDT
```

### 驱动代码中使用 ACPI

```c
static const struct acpi_device_id qcom_i2c_acpi_match[] = {
    { "QCOM0010", (kernel_ulong_t)&geni_i2c_v1 },
    { "QCOM0011", (kernel_ulong_t)&geni_i2c_v2 },
    { }
};
MODULE_DEVICE_TABLE(acpi, qcom_i2c_acpi_match);

static struct platform_driver qcom_i2c_driver = {
    .driver = {
        .acpi_match_table = qcom_i2c_acpi_match,  // ACPI 匹配
        .of_match_table = qcom_i2c_dt_match,       // DT 匹配（兼容）
    },
};
```

## UEFI — Unified Extensible Firmware Interface

UEFI 是 x86/ARM64 服务器的标准固件接口：

```bash
# UEFI 启动流程
# 1. SEC (安全阶段) → CPU/SEC 初始化
# 2. PEI (EFI 前期初始化) → 内存初始化
# 3. DXE (驱动执行环境) → 大部分设备初始化
# 4. BDS (启动设备选择) → 选择启动项
# 5. TSL (瞬时系统加载) → 启动内核
# 6. RT (运行时) → OS 运行

# UEFI Shell 命令
Shell> map -r           # 列出设备映射
Shell> fs0:             # 切换到第一个 FAT 分区
Shell> bcfg boot dump   # 查看启动项
Shell> bcfg boot add 0 fs0:\EFI\linux\Image.efi "Linux"
```

```bash
# 编译 UEFI 启动的内核
# 内核配置
CONFIG_EFI=y
CONFIG_EFI_STUB=y       # 内核作为 UEFI 应用程序
CONFIG_ACPI=y

# 编译为 EFI 镜像
make Image.gz
# 或者直接作为 EFI 应用：
make Image
```

## ARM 服务器实操

### 1. 使用 QEMU 模拟 ARM 服务器

```bash
# QEMU ARM SBSA 参考平台
qemu-system-aarch64 \
    -machine sbsa-ref \       # SBSA 参考平台
    -cpu cortex-a72 \
    -smp 4 \
    -m 4G \
    -bios SBSA_FLASH.fd \     # UEFI 固件
    -kernel Image \
    -append "console=ttyAMA0 earlycon=pl011,0x9000000" \
    -serial stdio \
    -drive file=disk.img,format=raw,if=none,id=hd0 \
    -device virtio-blk-pci,drive=hd0 \
    -netdev user,id=net0 \
    -device virtio-net-pci,netdev=net0
```

### 2. EDK2 — UEFI 固件

```bash
# 编译 ARM 服务器 UEFI 固件
git clone https://github.com/tianocore/edk2
git clone https://github.com/tianocore/edk2-platforms
cd edk2

# 编译 SBSA 参考平台固件
source edksetup.sh
build -a AARCH64 -t GCC5 -p Platform/ARM/SbsaQemu/SbsaQemu.dsc
# 输出: Build/Platform-ARM-SbsaQemu/DEBUG_GCC5/FV/SBSA_FLASH.fd
```

## 真实 GitHub 资源

| 仓库 | 内容 |
|------|------|
| [ARM-software/sbsa-tcs](https://github.com/ARM-software/sbsa-tcs) | SBSA 架构合规测试工具 |
| [tianocore/edk2](https://github.com/tianocore/edk2) | 开源 UEFI 固件实现 |
| [tianocore/edk2-platforms](https://github.com/tianocore/edk2-platforms) | 各平台 UEFI 实现 |
| [acpica/acpica](https://github.com/acpica/acpica) | ACPI 工具（iasl 编译器） |
| [qemu/qemu](https://github.com/qemu/qemu) | QEMU（含 sbsa-ref 平台） |
| [neoverse-reference-design](https://github.com/ARM-software/rd-infra) | ARM Neoverse 参考设计 |

## 面试考点

1. **SBSA 和 SBBR 的关系？** SBSA 定义硬件架构标准，SBBR 定义固件和启动标准
2. **ARM 服务器上为什么用 ACPI 替代 DeviceTree？** ACPI 适合管理复杂的 PCIe 拓扑、热插拔和电源管理
3. **ACPI 和 DT 驱动有什么区别？** DT 通过 compatible 匹配，ACPI 通过 _HID 匹配；ACPI 驱动不需要设备节点
4. **UEFI 相比传统 bootloader 的优势？** 标准化的接口、驱动模型、安全启动、运行时服务
5. **IORT 表的作用？** IO 远程化表，描述 IOMMU/SMMU 拓扑，OS 根据 IORT 配置 DMA 映射
