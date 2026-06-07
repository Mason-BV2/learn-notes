# DeviceTree — 设备树

## 概述

DeviceTree 是 ARM 架构上描述硬件信息的标准方式。内核通过 DeviceTree 在启动时发现硬件配置，无需硬编码地址和资源。

> 高通工作场景：你每天都会和 dts 文件打交道 — 添加新外设节点、配置 pinctrl、修改 memory 分区、调试 DT 覆盖层。

## DTS 语法基础

### 基础结构

```dts
// arch/arm64/boot/dts/qcom/sdm845.dtsi (SoC 级定义)
/dts-v1/;

#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/clock/qcom,gcc-sdm845.h>

/ {
    model = "Qualcomm Technologies, Inc. SDM845";
    compatible = "qcom,sdm845";
    
    #address-cells = <2>;   // 64-bit 地址
    #size-cells = <2>;      // 64-bit 大小
    
    // 中断控制器
    interrupt-parent = <&gic>;
    
    // CPU 核心
    cpus {
        #address-cells = <2>;
        #size-cells = <0>;
        
        CPU0: cpu@0 {
            device_type = "cpu";
            compatible = "qcom,kryo385";
            reg = <0x0 0x0>;
            enable-method = "psci";
            clocks = <&cpufreq_hw 0>;
            operating-points-v2 = <&cpu0_opp_table>;
            capacity-dmips-mhz = <1024>;
            dynamic-power-coefficient = <290>;
            next-level-cache = <&L2_0>;
        };
    };
    
    // GIC 中断控制器
    gic: interrupt-controller@17a00000 {
        compatible = "arm,gic-v3";
        reg = <0x0 0x17a00000 0x0 0x10000>,  // GICD
              <0x0 0x17a60000 0x0 0x100000>;  // GICR
        interrupt-controller;
        #interrupt-cells = <4>;
        #redistributor-regions = <1>;
    };
    
    // 定时器
    timer {
        compatible = "arm,armv8-timer";
        interrupts = <GIC_PPI 13 IRQ_TYPE_LEVEL_LOW>,  // EL1物理
                     <GIC_PPI 14 IRQ_TYPE_LEVEL_LOW>,  // EL1虚拟
                     <GIC_PPI 11 IRQ_TYPE_LEVEL_LOW>,  // 非安全
                     <GIC_PPI 10 IRQ_TYPE_LEVEL_LOW>;  // 安全
    };
};
```

### Board 级 dts

```dts
// arch/arm64/boot/dts/qcom/sdm845-db845c.dts (板上具体设备)
/dts-v1/;

#include "sdm845.dtsi"

/ {
    model = "Thundercomm DB845c";
    compatible = "thundercomm,db845c", "qcom,sdm845";
    
    // 板上内存
    memory@80000000 {
        device_type = "memory";
        reg = <0x0 0x80000000 0x0 0x80000000>;  // 2GB
    };
    
    // I2C 总线上的外设
    &i2c1 {
        status = "okay";
        
        /* 板上 I2C 外设 */
        touchscreen@20 {
            compatible = "goodix,gt911";
            reg = <0x20>;
            interrupt-parent = <&tlmm>;
            interrupts = <31 IRQ_TYPE_EDGE_RISING>;
            reset-gpios = <&tlmm 32 GPIO_ACTIVE_LOW>;
        };
        
        // 音频编解码器
        wcd9340: codec@1d {
            compatible = "slim,wcd9340";
            reg = <0x1d>;
        };
    };
    
    &uart2 {
        status = "okay";
        /* 蓝牙 UART */
        bluetooth {
            compatible = "qcom,wcn3990-bt";
        };
    };
};
```

### DTS 常用属性

```dts
// 寄存器地址
reg = <base_addr size>;
// 示例：reg = <0x0 0x17100000 0x0 0x1000>;

// 中断
interrupts = <中断类型 IRQ号 触发方式>;
// GIC_SPI = 0, GIC_PPI = 1
// 示例：interrupts = <GIC_SPI 123 IRQ_TYPE_LEVEL_HIGH>;

// 时钟
clocks = <&clock_controller 时钟ID>;
clock-names = "name";

// GPIO
gpios = <&tlmm 引脚号 GPIO_ACTIVE_HIGH/LOW>;

// 电源
vdd-supply = <&pm8998_l17>;
vdd-io-supply = <&pm8998_l6>;

// 引脚 mux 控制（pinctrl）
pinctrl-names = "default", "sleep";
pinctrl-0 = <&i2c1_default>;
pinctrl-1 = <&i2c1_sleep>;
```

## Pinctrl — 引脚控制

Pinctrl 是高通驱动中的一个关键概念，用于配置 GPIO 的功能复用：

```dts
// arch/arm64/boot/dts/qcom/sdm845-pinctrl.dtsi
&tlmm {
    /* I2C1 引脚配置 */
    i2c1_default: i2c1-default {
        pins = "gpio2", "gpio3";  // I2C SDA, SCL
        function = "i2c1";         // 功能复用为 I2C
        drive-strength = <2>;      // 驱动强度 (2/4/6/8 mA)
        bias-disable;              // 不使能上下拉
    };
    
    i2c1_sleep: i2c1-sleep {
        pins = "gpio2", "gpio3";
        function = "gpio";         // 休眠时切回 GPIO
        bias-pull-up;              // 上拉
    };
    
    /* UART2 引脚配置（蓝牙） */
    uart2_default: uart2-default {
        pins = "gpio4", "gpio5", "gpio6", "gpio7";
        function = "uart2";
        drive-strength = <2>;
        bias-disable;
    };
};
```

## Driver 中匹配 DTS

```c
// 驱动中的 of_match_table
#include <linux/of.h>
#include <linux/of_device.h>

static const struct of_device_id qcom_i2c_dt_match[] = {
    { .compatible = "qcom,i2c-geni", .data = &geni_i2c_v1 },
    { .compatible = "qcom,i2c-geni-v2", .data = &geni_i2c_v2 },
    { }
};
MODULE_DEVICE_TABLE(of, qcom_i2c_dt_match);

static struct platform_driver qcom_i2c_driver = {
    .probe  = qcom_i2c_probe,
    .remove = qcom_i2c_remove,
    .driver = {
        .name = "qcom_i2c",
        .of_match_table = qcom_i2c_dt_match,
        .pm = &qcom_i2c_pm_ops,
    },
};

// probe 函数中读取 DT 属性
static int qcom_i2c_probe(struct platform_device *pdev)
{
    struct device *dev = &pdev->dev;
    struct resource *res;
    
    // 读取寄存器地址
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    base = devm_ioremap_resource(dev, res);
    
    // 读取中断
    irq = platform_get_irq(pdev, 0);
    
    // 读取 DT 属性
    u32 freq;
    of_property_read_u32(dev->of_node, "clock-frequency", &freq);
    
    // 获取 GPIO
    gpiod = devm_gpiod_get(dev, "reset", GPIOD_OUT_LOW);
    
    // 获取时钟
    clk = devm_clk_get(dev, "se");
}
```

## DT Overlay

DT Overlay 允许在运行时动态修改设备树，Android 广泛使用：

```dts
// dtbo — device tree blob overlay
/dts-v1/;
/plugin/;

&i2c1 {
    /* 动态添加 sensor */
    sensor@48 {
        compatible = "st,lsm6dso";
        reg = <0x48>;
        interrupt-parent = <&tlmm>;
        interrupts = <117 IRQ_TYPE_EDGE_RISING>;
    };
};
```

```bash
# 编译 overlay
dtc -@ -I dts -O dtb -o sensor-overlay.dtbo sensor-overlay.dts

# 应用 overlay
mkdir /sys/kernel/config/device-tree/overlays/sensor
cat sensor-overlay.dtbo > /sys/kernel/config/device-tree/overlays/sensor/dtbo
```

## 实操练习

### 1. 在 QEMU 上添加虚拟设备节点

```bash
# 使用 qemu-system-aarch64 模拟树莓派
qemu-system-aarch64 -machine virt -kernel Image -dtb myboard.dtb

# 查看当前设备树
dtc -I dtb -O dts -o /tmp/dump.dts /sys/firmware/fdt
```

### 2. 编写自己的 platform 驱动

```c
// minimal_platform.c — 一个由 DT 节点触发的 platform 驱动
#include <linux/module.h>
#include <linux/platform_device.h>

static int my_probe(struct platform_device *pdev)
{
    pr_info("my_device: probed!\n");
    
    struct resource *res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (res)
        pr_info("my_device: reg = 0x%llx, size = 0x%llx\n",
                (u64)res->start, (u64)(res->end - res->start + 1));
    
    return 0;
}

static int my_remove(struct platform_device *pdev)
{
    pr_info("my_device: removed\n");
    return 0;
}

static const struct of_device_id my_dt_ids[] = {
    { .compatible = "my,virtual-device" },
    { }
};
MODULE_DEVICE_TABLE(of, my_dt_ids);

static struct platform_driver my_driver = {
    .probe  = my_probe,
    .remove = my_remove,
    .driver = {
        .name = "my_device",
        .of_match_table = my_dt_ids,
    },
};
module_platform_driver(my_driver);

MODULE_LICENSE("GPL");
```

对应的 dts 节点：
```dts
/ {
    my-device {
        compatible = "my,virtual-device";
        reg = <0x0 0x1c00000 0x0 0x1000>;
    };
};
```

## 真实 GitHub 资源

| 仓库 | 内容 |
|------|------|
| [devicetree-org/devicetree-specification](https://github.com/devicetree-org/devicetree-specification) | DeviceTree 规范 |
| [devicetree-org/dt-schema](https://github.com/devicetree-org/dt-schema) | DT schema 验证工具 |
| [torvalds/linux/Documentation/devicetree/bindings/](https://github.com/torvalds/linux/tree/master/Documentation/devicetree/bindings) | 内核官方 DT bindings |
| [torvalds/linux/arch/arm64/boot/dts/qcom/](https://github.com/torvalds/linux/tree/master/arch/arm64/boot/dts/qcom) | 高通平台 dts 文件 |
| [bootlin/training-materials](https://github.com/bootlin/training-materials) | Bootlin 设备树培训 |

## 面试考点

1. **compatible 属性的匹配规则？** 从最具体到最不具体（如 `"thundercomm,db845c", "qcom,sdm845"`）
2. **dtsi 和 dts 的区别？** .dtsi 是 SoC 级 include 文件，.dts 是 board 级文件
3. **pinctrl 的作用？** 配置引脚复用功能（GPIO/I2C/SPI/UART）、驱动强度、上下拉
4. **status = "okay" 和 "disabled" 的作用？** .dtsi 中默认 disabled，board dts 中开启
5. **如何调试 DT 问题？** 检查 /sys/firmware/devicetree/ 下的内容，用 dtc 反编译 dtb
